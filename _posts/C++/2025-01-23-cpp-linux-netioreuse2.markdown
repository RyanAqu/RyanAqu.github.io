---
layout:     post
title:      "Linux下poll/epoll模型"
subtitle:   " \"learning……\""
date:       2025-01-23 20:14:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - linux
    - 网络编程
    - 网络I/O复用模型
    - poll/epoll
---

> 酒入愁肠，吨吨吨吨吨......


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

