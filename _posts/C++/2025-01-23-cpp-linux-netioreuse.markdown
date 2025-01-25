---
layout:     post
title:      "Linux 网络编程 网络IO复用技术"
subtitle:   " \"learning……\""
date:       2025-01-23 18:14:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - linux
    - socket
    - 网络编程
    - 网络IO复用技术
    - TCP IP/select/poll/epoll
---

> 从别后，忆相逢，几回魂梦与君同......

# TCP 三次握手和四次挥手 
TCP是面向连接的可靠的协议，建立TCP连接需要三次对话（三次握手），拆除TCP连接需要四次对话（四次挥手），确保通信双方能够正常发送和接收数据。  

### 三次握手  
服务端调用listen()函数后就进入监听（等待）状态，这时候，客户端就可以调用connect()函数发起TCP连接请求了，客户端connect()会引发三次握手，
完成后会建立一个双向的连接通道。  

![image](https://github.com/user-attachments/assets/944ee81a-0df7-4a56-9680-76fc2dfd2250)

````
客户端                         服务端
   | SYN    ------------------>   |
   |        <----------------- SYN + ACK
   | ACK    ------------------>   |
````

1. 第一次握手（客户端 → 服务端）：
    * 客户端向服务端发送一个带有 SYN 标志的数据包，表示请求建立连接。
    * 此时客户端进入 SYN_SENT 状态。
2. 第二次握手（服务端 → 客户端）：
    * 服务端收到客户端的 SYN 请求后，向客户端发送一个 SYN + ACK 数据包：SYN 表示服务端同意建立连接。ACK 表示确认收到客户端的 SYN 数据包。
    * 此时服务端进入 SYN_RECV 状态。
3. 第三次握手（客户端 → 服务端）：
    * 客户端收到服务端的 SYN + ACK 后，向服务端发送一个 ACK 数据包，表示确认。
    * 客户端进入 ESTABLISHED 状态，服务端也随之进入 ESTABLISHED 状态，连接建立成功。

一些细节：
* 客户端的socket也有端口，一般为系统随机分配，不必关注。一般socket通信中的地址包括IP和端口号。  
* 服务端的bind()函数，普通用户只能使用1024以上端口，root可以使用任意端口。  
* listen()函数的第二个参数+1为已连接队列(ESTABLISHED状态)的大小。在高并发服务程序中应该调大一些。即等待被accept()函数受理的连接数。  
* SYN_RECV也被称为半连接。  
* CLOSED状态是假想状态，实际不存在。  

### 四次挥手  
TCP 四次挥手用于断开连接，确保双方都能安全地关闭通信。断开连接的时候，客户端和服务端需要相互总共发送四个包以确认连接的断开。在socket编程中，这一过程由客户端或者服务端任意一方执行close()函数触发。  

![image](https://github.com/user-attachments/assets/e3aea550-0e17-40d6-af00-b4db689bf67b)

````
客户端                         服务端
   | FIN    ------------------>   |
   |        <----------------- ACK
   |        <----------------- FIN
   | ACK    ------------------>   |
````

1. 第一次挥手（客户端 → 服务端）：
    * 客户端发送一个带有 FIN 标志的数据包，表示想要关闭连接。
    * 此时客户端进入 FIN_WAIT_1 状态。
2. 第二次挥手（服务端 → 客户端）：
    * 服务端收到 FIN 后，发送一个 ACK 数据包，表示确认收到关闭请求。
    * 此时服务端进入 CLOSE_WAIT 状态，客户端进入 FIN_WAIT_2 状态。
3. 第三次挥手（服务端 → 客户端）：
    * 服务端发送一个 FIN 数据包，表示服务端也想关闭连接。
    * 此时服务端进入 LAST_ACK 状态。
4. 第四次挥手（客户端 → 服务端）：
    * 客户端收到 FIN 后，发送一个 ACK 数据包，表示确认关闭连接。
    * 客户端进入 TIME_WAIT 状态，等待一段时间（通常是 2 倍的最大报文段生存时间）以确保服务端收到 ACK 后再关闭。
    * 服务端收到 ACK 后直接进入 CLOSED 状态，连接断开。

细节：
* 主动断开的端在四次挥手之后，socket进入TIME_WAIT状态，持续2MSL（30s/1min/2min），超过这个时间的报文会被丢弃。因为主动关闭时对端可能还在向自己发消息。
* 如果是客户端主动断开，那么TIME_WAIT对socket基本无影响和危害。
* 如果是服务端主动断开，那会有两方面的危害：一是socket没有立即释放，二是端口号只能在2MSL后才能重新启用。**遇到过这种情况：5005端口提示地址已经被使用**。

# TCP缓存  
系统为每个socket创建了发送缓冲区和接收缓冲区，应用程序调用send()/write()函数发送数据的时候，内核把数据从应用进程拷贝到socket的发送缓冲区中;应用程序调用recv()/read()函数接收数据，内核会把数据从socket接收缓冲区拷贝到应用进程中。  
发送数据即把数据放入发送缓冲区。  
接收数据即从接收缓冲区中取数据。  

![image](https://github.com/user-attachments/assets/6625cf78-f2ad-4ded-8421-c82c9f253d1a)

细节：
* send()函数会阻塞吗：会的，如果自己的发送缓冲区或者对端的接收缓冲区都满了，会阻塞。  
* 向socket中写入数据后，如果关闭了socket，对端还能收到数据吗：会的，因为关闭socket的事件在接收缓冲区里排在发送的数据之后。  

### TCP发送端Nagle算法
一个时间段内只能有一个未被确认的TCP分组在传输中。如果当前未确认（没有收到接收端的ACK回复）的数据包（Unacknowledged Packet）存在，新的数据将被暂存在缓冲区中，而不是立即发送。只有在以下情况下才能发送新的数据：  
* 缓冲区中的数据足够填满一个数据包（MTU大小）。
* 已经收到之前数据包的确认（ACK）。

对实时性要求很高的系统一般会禁用Nagle算法，譬如证券交易、联机游戏。  

### TCP接收端ACK延迟机制
延迟确认（Delayed ACK）：  
* TCP的一种机制，推迟发送ACK，以便在接收到更多数据时合并ACK和回复的数据包。
* Nagle算法和延迟确认的结合可能导致糟糕的延迟现象（“延迟病”），尤其是在交互应用中。


# I/O多路复用  
为每个客户端提供一个进/线程，消耗的资源是很多的，1核2G的虚拟机大概可以创建一百多个进程。  
I/O复用可以用一个进/线程处理多个TCP连接，减少系统开销。有三种模型：select（1024）、poll（数千）、epoll（百万）。  

网络通信读事件：  
* 已连接队列中有已经准备好的socket（有新的客户端连上来）  
* 接收缓存中有数据可以读（对端发送的报文已到达）  
* TCP连接已断开（对端调用close()函数关闭了连接）  

网络通信写事件：  
* 发送缓冲区没有满，可以写入数据（可以向对端发送报文）  

# select模型  
select函数可以监视多个socket，把需要监视的socket传给它，如果被监视的socket没有事件发生，就会阻塞等待；如果有事件发生就会返回。  
select() 是一种 I/O 多路复用技术，允许一个进程同时监视多个文件描述符（sockets、文件、管道等），并在这些文件描述符上的 I/O 操作变得可用时（如可读、可写或有异常）做出响应。select() 常用于高效的网络编程中，特别是在处理大量并发连接时。  

### select() 的工作原理  
select() 系统调用通过检查一个或多个文件描述符的状态，决定哪些文件描述符可以执行 I/O 操作。进程在调用 select() 时，可以指定多个文件描述符集合，这些集合包括：  
* 读集合 (readfds)：监视哪些文件描述符可以读取（即数据已经准备好可以读取）。
* 写集合 (writefds)：监视哪些文件描述符可以写入（即可以向文件描述符写入数据）。
* 异常集合 (exceptfds)：监视哪些文件描述符发生了异常（例如连接断开、错误发生等）。

select() 会阻塞直到以下情况之一发生：  
* 有一个文件描述符准备好读取。
* 有一个文件描述符准备好写入。
* 发生了一个异常。
* 超过指定的超时限制。

### select() 函数原型  
````
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
````

* nfds: select() 监听的文件描述符的最大值 + 1。通常使用 FD_SETSIZE 来指定。
* readfds: 一个 fd_set 类型的集合，表示需要监视是否可以读取的文件描述符。
* writefds: 一个 fd_set 类型的集合，表示需要监视是否可以写入的文件描述符。
* exceptfds: 一个 fd_set 类型的集合，表示需要监视是否有异常的文件描述符。
* timeout: 一个 struct timeval 类型的结构体，用来指定阻塞时间（超时）。如果为 NULL，select() 会阻塞直到文件描述符准备好。如果为 0，select() 会立即返回。


### select() 的步骤  
1. 初始化文件描述符集合： 使用 FD_ZERO() 清空集合，使用 FD_SET() 将文件描述符添加到集合中，或者使用 FD_CLR() 从集合中删除文件描述符。
2. 调用 select()： 通过 select() 来监视多个文件描述符，直到一个或多个文件描述符的 I/O 状态改变（如可读、可写、异常）。
3. 检查文件描述符集合： 调用 select() 后，返回的集合会被更新。你可以检查哪些文件描述符就绪，然后执行相应的操作。
4. 处理就绪的文件描述符： 一旦 select() 返回，你可以通过遍历文件描述符集合来处理哪些文件描述符已经准备好进行读写操作。

### select模型demo  
client.cpp  
````
#include<iostream>
#include<cstdio>
#include<cstring>
#include<cstdlib>
#include<unistd.h>
#include<netdb.h>
#include<sys/types.h>
#include<sys/socket.h>
#include<arpa/inet.h>
using namespace std;

/*
 * 用于演示socket通信的客户端
 */
int main(int argc,char* argv[])
{
    if(argc!=3)
    {
        cout<<"Using \"ifconfig\" or \"ip addr\" to find ip"<<endl;
        cout<<"Using ./demo1 192.168.76.128 5005\n\n";
        return -1;
    }

    //step1:create socket for client
    int sockfd = socket(AF_INET,SOCK_STREAM,0);
    if(sockfd==-1)
    {
        perror("socket");
        return -1;
    }
    //step2:send a request to the server
    struct hostent* h;  //存放服务端IP的结构体
    if((h=gethostbyname(argv[1]))==0)//用于存放服务端IP和端口的结构体
    {
        cout<<"gethostbyname failed.\n"<<endl;
        close(sockfd);
        return -1;
    }
    struct sockaddr_in servaddr;//存放服务端IP和端口的结构体
    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    memcpy(&servaddr.sin_addr,h->h_addr,h->h_length);//指定服务端的IP地址
    servaddr.sin_port=htons(atoi(argv[2]));//指定服务端的通信端口
    if(connect(sockfd,(struct sockaddr*)&servaddr,sizeof(servaddr))!=0)//向服务端发起请求连接
    {
        perror("connect");
        close(sockfd);
        return -1;
    }
    printf("connect ok.\n");
    //step3:与服务端通信，发送一个请求报文后等待回复，然后发送下一个请求报文
    char buffer[1024];
    for(int i=0;i<3;++i)//循环三次，与服务端进行三次通信。
    {
        memset(buffer,0,sizeof(buffer));
        printf("please input:");
        scanf("%s",buffer);
        //向服务端发送请求报文
        if((send(sockfd,buffer,strlen(buffer),0))<=0)
        {
            perror("send");
            close(sockfd);
            return -1;
        }
        memset(buffer,0,sizeof(buffer));
        //接受服务端的回应报文，如果服务端没有发送回应报文，recv()函数将阻塞等待
        if((recv(sockfd,buffer,sizeof(buffer),0))<=0)
        {
            perror("recv");
            close(sockfd);
            return -1;
        }
        cout<<"接收:"<<buffer<<endl;
    }
    //step4:关闭socket，释放资源
    close(sockfd);
    return 0;
}
````

tcpselect.cpp  
````
/*
 * select模型实现网络通信服务端
 */
#include<stdio.h>
#include<iostream>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>
#include<sys/socket.h>
#include<arpa/inet.h>
#include<sys/fcntl.h>
using namespace std;

//初始化服务端的监听端口
int initserver(int port)
{
    int sock=socket(AF_INET,SOCK_STREAM,0);
    if(sock<0)
    {
        perror("socket() failed");
        return -1;
    }
    int opt=1;
    unsigned int len = sizeof(opt);
    setsockopt(sock,SOL_SOCKET,SO_REUSEADDR,&opt,len);

    struct sockaddr_in servaddr;
    servaddr.sin_family=AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port=htons(port);

    if(bind(sock,(struct sockaddr*)&servaddr,sizeof(servaddr))<0)
    {
        perror("bind() failed");
        close(sock);
        return -1;
    }
    if(listen(sock,5)!=0)
    {
        perror("listen() failed");
        close(sock);
        return -1;
    }

    return sock;
}   

int main(int argc,char* argv[])
{
    if(argc!=2)
    {
        cout<<"usage:./tcpselect port\n";
        return -1;
    }
    //初始化服务端用于监听的socket
    int listensock = initserver(atoi(argv[1]));
    printf("listensock=%d\n",listensock);

    if(listensock<0)
    {
        cout<<"initserver() failed.\n";
        return -1;
    }

    //读事件
    //写事件

    fd_set readfds;                 //需要监听读事件的socket的合集，大小为16字节（1024位）的bitmap
    FD_ZERO(&readfds);              //初始化readfds，把bitmap的每一位都置为0.
    FD_SET(listensock,&readfds);    //把服务端用于监听的socket加入readfds.

    int maxfd=listensock;           //readfds中socket最大的值

    while(true)
    {
        struct timeval timeout;
        timeout.tv_sec=10; //秒
        timeout.tv_usec=0; //微秒
        
        fd_set tmpfds=readfds;//在select()函数中，会修改bitmap，所以要把readfds复制一份给tmpfds，在把tmpfds传给select()
        //select()等待事件发生
        int infds = select(maxfd+1,&tmpfds,NULL,NULL,&timeout);
        //如果infds<0,表示调用select()失败
        if(infds<0)
        {
            perror("select() failed");
            break;
        }
        //如果infds==0,表示select()超时
        if(infds==0)
        {
            printf("select() timeout.\n");
            continue;
        }
        //如果infds>0，表示有事件发生，infds存放了已发生事件的个数
        for(int eventfd=0;eventfd<=maxfd;eventfd++)
        {
            if(FD_ISSET(eventfd,&tmpfds)==0)
            {
                continue;//如果eventfd在bitmap中的标志为0,表示它没有事件，continue
            }
            //如果发生事件的是listensock，表示已连接队列中有已经准备好的socket（有新的客户连上来了）
            if(eventfd==listensock)
            {
                struct sockaddr client;
                socklen_t len =sizeof(client);
                int clientsock=accept(listensock,(struct sockaddr*)&client,&len);
                if(clientsock<0)
                {
                    perror("accept() failed");
                    continue;
                }
                printf("accept client(sock=%d)ok.\n",clientsock);
                FD_SET(clientsock,&readfds);
                if(maxfd<clientsock) maxfd=clientsock;
            }
            else
            {//客户端连接的socket有事件，表示接收缓存中有数据可以读（对端发送的报文已到达），或者所有的客户端已经断开连接。
                char buffer[1024];
                memset(buffer,0,sizeof(buffer));
                if(recv(eventfd,buffer,sizeof(buffer),0)<=0)
                {
                    //如果客户端已经断开
                    printf("client(eventfd=%d)disconnected.\n",eventfd);
                    close(eventfd);
                    FD_CLR(eventfd,&readfds);
                    if(eventfd==maxfd)
                    {
                        for(int i=maxfd;i>0;--i)
                        {
                            if(FD_ISSET(i,&readfds))
                            {
                                maxfd=i;
                                break;
                            }
                        }
                    }
                }
                else
                {
                    //如果客户端有报文发过来。
                    printf("recv(eventfd=%d):%s\n",eventfd,buffer);
                    //把接收到的报文内容原封不动的发回去
                    send(eventfd,buffer,strlen(buffer),0);
                }
            }
        }
    }
    return 0;


}

````





