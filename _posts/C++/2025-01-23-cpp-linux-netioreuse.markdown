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

### select模型demo（一般只关注读事件）  
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

### select模型的写事件  
* 如果TCP的发送缓冲区没满，那么socket连接是可写的。  
* 一般来说，发送缓冲区不容易被填满。  
* 如果发送的数据量太大，或者网络带宽不够，发送缓冲区有填满的可能。  


带写事件的服务端demo，但在实际开发中，一般不关注写事件，只关注读事件。  
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
        fd_set tmpfds1=readfds;//复制一份bitmap，用于监视写事件
        //select()等待事件发生
        int infds = select(maxfd+1,&tmpfds,&tmpfds1,NULL,&timeout);
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

        //判断写事件
        for(int eventfd=0;eventfd<=maxfd;eventfd++)
        {
            if(FD_ISSET(eventfd,&tmpfds1)==0)
            {
                continue;//如果eventfd在bitmap中的标志为0,表示它没有事件，continue
            }
            printf("eventfd=%d可以写。\n",eventfd);
        }
        //如果infds>0，表示有读事件发生，infds存放了已发生事件的个数
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

### select模型-水平触发  
* select函数监视的socket如果发生了事件，select()会返回（通知应用程序处理事件）。
* 如果事件没有被处理，再次调用select会立即通知。

两个应用场景：  
* 如果有listenfd有连接第一次未处理，第二次调select依旧会处理这个连接。  
* 如果读数据一次没读完，第二次select会继续读取数据。  

### select模型总结  
性能：120000个请求/秒  

存在的问题：
* 采用轮询的方式扫描bitmap，性能会随着socket数量增多而下降。  
* 每次调用select()，需要拷贝bitmap。  
* bitmap的大小（单个进/线程打开的socket数量）由FD_SETSIZE宏设置，默认是1024个，可以修改，但是效率会降低。bitmap不是高效的数据结构，只能遍历。  



# poll模型  
pollfd 是用于 I/O 多路复用 的结构体，主要用于在 Linux 或类 Unix 系统上处理多个文件描述符的事件检测。它是 poll 系统调用的一部分，用于代替 select 系统调用来处理多个文件描述符的读写事件。  
poll结构体如下：  
````
struct pollfd {
    int fd;         // 文件描述符
    short events;   // 监视的事件
    short revents;  // 实际发生的事件
};
````

### poll模型demo  
poll模型的客户端代码和select一样，都是client.cpp。  
poll模型的写事件与select也是一样的，整体代码逻辑相似。  
poll模型也是水平触发。  

服务端：  
````
/*
 *此程序用于演示poll模型实现网络通信服务端
 */
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>
#include<poll.h>
#include<sys/socket.h>
#include<arpa/inet.h>
#include<sys/fcntl.h>

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
        printf("usage: ./tcppoll port\n");
        return -1;
    }
    //初始化服务端用于监听的socket
    int listensock = initserver(atoi(argv[1]));
    printf("listensock=%d\n",listensock);

    if(listensock<0)
    {
        printf("initserver() failed.\n");
        return -1;
    }
    //printf("111111");
    //创建poll结构体数组
    pollfd fds[1024];   //fds存放需要监视的socket

    //初始化数据，把全部的socket置为-1,如果数组中的socket值为-1,那么poll将忽略它
    for(int i=0;i<1024;i++)
    {
        fds[i].fd=-1;
    }
    //让poll监视listensock读事件
    fds[listensock].fd=listensock;
    fds[listensock].events=POLLIN;  //POLLIN表示读事件，POLLOUT表示写事件
    //fds[listensock].events=POLLIN|POLLOUT;

    int maxfd = listensock;//fds数组中需要监视socket的实际大小

    while(true)
    {
        //调用poll()等待事件的发生（监视哪些socket发生了事件）
        int infds=poll(fds,maxfd+1,10000);//超时时间为10秒
        //如果infds<0,表示调用poll()失败
        if(infds<0)
        {
            perror("poll() failed\n");
            break;
        }
        //如果infds==0,表示调用poll()超时
        if(infds==0)
        {
            printf("poll() timeout.\n");
            continue;
        }
        //如果infds>0,表示有事件发生，infds存放了已发生事件的个数
        for(int eventfd=0;eventfd<=maxfd;eventfd++)
        {
            if(fds[eventfd].fd<0) continue;

            if((fds[eventfd].revents&POLLIN)==0) continue;
            //如果发生事件的是listensock,表示已连接队列中有已经准备好的socket(有新的客户连接上来了)
            if(eventfd==listensock)
            {
                struct sockaddr_in client;
                socklen_t len = sizeof(client);
                int clientsock = accept(listensock,(struct sockaddr*)&client,&len);
                if(clientsock<0)
                {
                    perror("accept() failed");
                    continue;
                }
                printf("accept client(sock=%d)ok.\n",clientsock);
                //修改fds数组中clientsock位置的元素
                fds[clientsock].fd=clientsock;
                fds[clientsock].events=POLLIN;
                if(maxfd<clientsock) maxfd=clientsock;//更新maxfd的值
            }
            else
            {
                //如果客户端连接的sock有事件，表示有报文或者连接断开
                char buffer[1024];
                memset(buffer,0,sizeof(buffer));
                if(recv(eventfd,buffer,sizeof(buffer),0)<=0)
                {
                    //如果客户端已经断开
                    printf("client(eventfd=%d)disconnected.\n",eventfd);
                    close(eventfd);
                    fds[eventfd].fd=-1;
                    if(eventfd==maxfd)
                    {
                        for(int i=maxfd;i>0;--i)
                        {
                            if(fds[i].fd!=-1)
                            {
                                maxfd=i;
                                break;
                            }
                        }
                    }
                }
                else
                {
                    //如果客户端有报文发过来
                    printf("recv(eventfd=%d):%s\n",eventfd,buffer);
                    send(eventfd,buffer,strlen(buffer),0);
                }
            }
        }

    }
    return 0;
}
````

### poll模型总结  
存在的问题：
* 在程序中，poll模型数据类型是结构体数组，进入内核后转化成了链表。  
* 每次调用select需要拷贝两次bitmap，poll只要拷贝一次结构体数组。  
* poll监视的连接数没有1024的限制，但是也是遍历方法，监视的socket越多，效率越低。  

# epoll模型  
epoll 是 Linux 操作系统提供的一种高效的 I/O 多路复用机制，通常用于处理大量并发连接的场景，例如高并发的 Web 服务器。相比于传统的 select 和 poll，epoll 在处理大量文件描述符时具有更好的性能。  
epoll 采用了事件驱动机制，能高效地处理大量并发的 I/O 操作。它的核心思想是通过一个事件通知机制来避免每次检查所有的文件描述符，而是只对发生事件的文件描述符进行处理。它采用了 水平触发（Level Triggered） 和 边缘触发（Edge Triggered） 两种模式。  

核心组成：
* epoll_create：创建一个 epoll 实例，返回一个 epoll 文件描述符。  
* epoll_ctl：用来控制 epoll 的行为，主要是向 epoll 实例中添加、删除或修改感兴趣的文件描述符。  
* epoll_wait：阻塞或非阻塞的等待事件发生，返回已准备就绪的文件描述符。  

### epoll模型demo  
客户端用client.cpp，与select和poll一样。  
服务端tcpepoll.cpp：  
````
/*
 *此程序用于演示epoll模型实现网络通信服务端
 */
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>
#include<errno.h>
#include<sys/socket.h>
#include<arpa/inet.h>
#include<sys/fcntl.h>
#include<sys/epoll.h>

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
        printf("usage: ./tcpepoll port\n");
        return -1;
    }
    //初始化服务端用于监听的socket
    int listensock = initserver(atoi(argv[1]));
    printf("listensock=%d\n",listensock);

    if(listensock<0)
    {
        printf("initserver() failed.\n");
        return -1;
    }
    //创建epoll句柄  
    int epollfd=epoll_create(1);

    //为服务端的listensock准备可读事件。
    epoll_event ev;     //申明事件的数据结构
    ev.data.fd=listensock;//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
    ev.events=EPOLLIN;  //打算让epoll监视listensock的读事件

    epoll_ctl(epollfd,EPOLL_CTL_ADD,listensock,&ev); //把需要监视的socket加入epollfd中。
    
    epoll_event evs[10];//存放epoll返回的事件。
    
    while(true)
    {
        //等待监视的socket有事件发生
        int infds = epoll_wait(epollfd,evs,10,-1);

        //返回失败
        if(infds<0)
        {
            perror("epoll() failed\n");break;
        }
        //超时
        if(infds==0)
        {
            printf("epoll() timeout.\n");
            continue;
        }
        //如果infds>0,表示有事件发生的socket的数量
        for(int i=0;i<infds;i++)
        {
            //如果发生事件的是listensock，表示有新的客户端连上来
            if(evs[i].data.fd==listensock)
            {
                struct sockaddr_in client;
                socklen_t len=sizeof(client);
                int clientsock=accept(listensock,(struct sockaddr*)&client,&len);
                printf("accept client(sock=%d)ok.\n",clientsock);

                //为新客户准备可读事件，并添加到epoll中
                ev.data.fd=clientsock;
                ev.events=EPOLLIN;
                epoll_ctl(epollfd,EPOLL_CTL_ADD,clientsock,&ev);
            }
            else
            {
                //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
                char buffer[1024];//存放从客户端读取的数据
                memset(buffer,0,sizeof(buffer));
                if(recv(evs[i].data.fd,buffer,sizeof(buffer),0)<=0)
                {
                    //如果客户端已经断开
                    printf("client(eventfd=%d)disconnected.\n",evs[i].data.fd);
                    close(evs[i].data.fd);
                }
                else
                {
                    //如果客户端有报文发过来
                    printf("recv(eventfd=%d):%s\n",evs[i].data.fd,buffer);
                    send(evs[i].data.fd,buffer,strlen(buffer),0);
                }

            }
        }
    }

    return 0;
}
````

# sock阻塞&非阻塞I/O  
* 阻塞：在进/线程中，发起一个调用时，再调用返回之前进/线程会被阻塞等待，等待中的进/线程让出CPU的使用权。  
* 非阻塞：在进/线程中，发起一个调用时会立即返回。
* 会阻塞的四个函数：connect(),accept(),send(),recv()

在传统的网络通信服务端中（每连接进/线程），会阻塞I/O。在I/O复用模型中，事件循环不能被阻塞在任何一个环节，所以应该采用非阻塞I/O。但是在网络通信中有上述四个函数会阻塞。**应对方法如下**：在 Linux 中，我们可以通过 fcntl() 系统调用来将套接字（socket）设置为非阻塞模式。具体来说，fcntl() 可以用于获取和修改文件描述符的状态标志。通过设置 O_NONBLOCK 标志，我们可以将套接字设置为非阻塞模式。  

1. 非阻塞I/O-connect()：
    * 对非阻塞的IO调用connect()函数，不管是否能连接成功，connect()都会立即返回失败，errno== EINPROGRESS。
    * 对非阻塞的IO调用connect()函数后，如果socket的状态是可写的，证明连接是成功的，否则是失败的。
2. 非阻塞I/O-accept()：
    * 对非阻塞的IO调用accept()，如果已连接队列中没有socket，函数立即返回失败，errno==EAGAIN
3. 非阻塞I/O-recv()：
    * 对非阻塞的IO调用recv()，如果没数据可读(接收缓冲区为空)，函数立即返回失败，errno==EAGAIN
4. 非阻塞I/O-send()：
    * 对非阻塞的IO调用send()，如果socket不可写（发送缓冲区已满)，函数立即返回失败，errno==EAGAIN

# 水平触发和边沿触发  
**水平触发：**  
* 读事件:如果epoll_wait触发了读事件，表示有数据可读，如果程序没有把数据读完，再次调用epoll_wait的时候，将立即再次触发读事件。  
* 写事件:如果发送缓冲区没有满，表示可以写入数据，只要缓冲区没有被写满，再次调用epoll_wait的时候，将立即再次触发写事件。  

**边缘触发：**
* 读事件: epoll_wait触发读事件后，不管程序有没有处理读事件，epoll_wait都不会再触发读事件，只有当新的数据到达时，才再次触发读事件。
* 写事件: epoll_wait触发写事件之后，如果发送缓冲区仍可以写（发送缓冲区没有满)，epoll_wait不会再次触发写事件，只有当发送缓冲区由满变成不满时，才再次触发写事件。





























