---
layout:     post
title:      "Linux 下 Reactor高并发服务器"
subtitle:   " \"learning……\""
date:       2025-03-01 15:14:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - linux
    - 网络编程
    - Reactor服务器
---

> 人是生而自由的，但却无往不在枷锁之中。


# Reactor模型  
Reactor模型（Reactor Pattern）是一种用于处理高并发事件驱动应用的设计模式，常用于服务器架构（如Web服务器、网络服务器）中。它允许单个或少量的线程处理大量并发请求，提高系统的吞吐量和资源利用率。  

# 最简单的非阻塞epoll程序  
命令如下：  
````
./tcpepoll 192.168.157.128 5085
./client 192.168.157.128 5085
netstat -na|grep 5085        //查询TCP链接
````

### 客户端  
````
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<errno.h>
#include<string.h>
#include<netinet/in.h>
#include<sys/socket.h>
#include<arpa/inet.h>
#include<time.h>


/*
 * 用于演示socket通信的客户端
 */
int main(int argc,char* argv[])
{
    if(argc!=3)
    {
        printf("Using \"ifconfig\" or \"ip addr\" to find ip");
        printf("example: ./client 192.168.76.128 5005\n\n");
        return -1;
    }

    //step1:create socket for client
    int sockfd;
    struct sockaddr_in servaddr;
    char buf[1024];

    if((sockfd=socket(AF_INET,SOCK_STREAM,0))<0)
    {
        printf("socket() failed.\n");return -1;
    }
    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family=AF_INET;
    servaddr.sin_port=htons(atoi(argv[2]));
    servaddr.sin_addr.s_addr=inet_addr(argv[1]);


    if(connect(sockfd,(struct sockaddr*)&servaddr,sizeof(servaddr))!=0)//向服务端发起请求连接
    {
        printf("connect(%s:%s)failed.\n",argv[1],argv[2]);
        close(sockfd);
        return -1;
    }
    printf("connect ok.\n");

    //step3:与服务端通信，发送一个请求报文后等待回复，然后发送下一个请求报文
    
    for(int i=0;i<100000;++i)//循环n次，与服务端进行n次通信。
    {
        memset(buf,0,sizeof(buf));
        printf("please input:");
        scanf("%s",buf);
        //向服务端发送请求报文
        if((send(sockfd,buf,strlen(buf),0))<=0)
        {
            printf("write()failed.\n");
            close(sockfd);
            return -1;
        }
        memset(buf,0,sizeof(buf));
        //接受服务端的回应报文，如果服务端没有发送回应报文，recv()函数将阻塞等待
        if((recv(sockfd,buf,sizeof(buf),0))<=0)
        {
            printf("read()failed.\n");
            close(sockfd);
            return -1;
        }
        printf("recv:%s\n",buf);
    }
    //step4:关闭socket，释放资源
    close(sockfd);
    return 0;
}
````

### 服务端  
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
#include<sys/types.h>
#include<arpa/inet.h>
#include<sys/fcntl.h>
#include<sys/epoll.h>
#include<netinet/tcp.h>// TCP_NODELAY需要包含这个头文件
//TCP_NODELAY用于禁用Nagle算法


//设置非阻塞的IO
void setnonblocking(int fd)
{
    fcntl(fd,F_SETFL,fcntl(fd,F_GETFL)|O_NONBLOCK);
}

int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }


    //创建服务端用于监听的端口
    int listenfd=socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);
    if(listenfd<0)
    {
        perror("socket() failed");
        return -1;
    }

    //设置listenfd的属性
    int opt=1;
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEADDR,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,TCP_NODELAY,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEPORT,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_KEEPALIVE,&opt,static_cast<socklen_t>(sizeof opt));

    setnonblocking(listenfd);//把服务端的listenfd设置为非阻塞的

    struct sockaddr_in servaddr;//服务端地址结构体
    servaddr.sin_family=AF_INET;//ipv4网络套接字
    servaddr.sin_addr.s_addr = inet_addr(argv[1]);//服务端用于监听的ip地址
    servaddr.sin_port=htons(atoi(argv[2]));//服务端用于监听的端口


    if(bind(listenfd,(struct sockaddr*)&servaddr,sizeof(servaddr))<0)
    {
        perror("bind() failed");
        close(listenfd);
        return -1;
    }
    
    if(listen(listenfd,128)!=0)//在高并发服务其中，第二个参数要大一些
    {
        perror("listen() failed");
        close(listenfd);
        return -1;
    }
    int epollfd = epoll_create(1);//创建epoll句柄（红黑树）

    //为服务端的listenfd准备读事件

    //为服务端的listensock准备可读事件。
    struct epoll_event ev;     //申明事件的数据结构
    ev.data.fd=listenfd;//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
    ev.events=EPOLLIN;  //打算让epoll监视listensock的读事件,采用水平触发

    epoll_ctl(epollfd,EPOLL_CTL_ADD,listenfd,&ev); //把需要监视的socket加入epollfd中。
    
    struct epoll_event evs[10];//存放epoll返回的事件。
    
    
    while(1)//事件循环
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
            if(evs[i].data.fd==listenfd)
            {
                struct sockaddr_in clientaddr;
                socklen_t len=sizeof(clientaddr);
                int clientfd=accept(listenfd,(struct sockaddr*)&clientaddr,&len);
                setnonblocking(clientfd);//客户端连接的fd必须设置为非阻塞的
                
                printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientfd,inet_ntoa(clientaddr.sin_addr),ntohs(clientaddr.sin_port));

                //为新客户准备可读事件，并添加到epoll中
                ev.data.fd=clientfd;
                ev.events=EPOLLIN|EPOLLET;//边缘触发
                epoll_ctl(epollfd,EPOLL_CTL_ADD,clientfd,&ev);
            }
            else
            {
                //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
                //////////////////////////////
                if(evs[i].events&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
                {
                    //如果客户端已经断开
                    printf("client(eventfd=%d)disconnected.\n",evs[i].data.fd);
                    close(evs[i].data.fd);//关闭客户端的fd
                }
                else if(evs[i].events&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
                {
                    char buffer[1024];//存放从客户端读取的数据
                    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
                    {
                        bzero(&buffer,sizeof(buffer));
                        ssize_t nread = read(evs[i].data.fd,buffer,sizeof(buffer));
                        if(nread>0)//成功读取到了数据
                        {
                            //把接收到的数据原封不动的返回回去
                            printf("recv(eventfd=%d);%s\n",evs[i].data.fd,buffer);
                            send(evs[i].data.fd,buffer,strlen(buffer),0);
                        }
                        else if(nread==-1&&errno==EINTR)//读取数据的时候信号中断，继续读取
                        {
                            continue;
                        }
                        else if(nread==-1&&((errno==EAGAIN)||(errno==EWOULDBLOCK)))//全部的数据已读取完必
                        {
                            break;
                        }
                        else if(nread==0)//客户端连接已经断开
                        {
                            printf("client(eventfd=%d) disconnected.\n",evs[i].data.fd);
                            close(evs[i].data.fd);
                            break;
                        }
                    }
                }
                else if(evs[i].events&EPOLLOUT)//有数据要写，暂时没代码，以后在说
                {

                }
                else //其他事件，都视为错误,或者对方关闭了链接。
                {
                    printf("client(eventfd=%d)error.\n",evs[i].data.fd);
                    close(evs[i].data.fd);
                }
                /////////////////////////////
                
            }
        }
    }


    return 0;
}
````

# 最简单的非阻塞epoll程序优化  
### 服务端程序  
创建socket时加一个SOCK_NONBLOCK就能非阻塞。删除了非阻塞的函数。稍微调整了一下事件循环逻辑。  
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
#include<sys/types.h>
#include<arpa/inet.h>
#include<sys/fcntl.h>
#include<sys/epoll.h>
#include<netinet/tcp.h>// TCP_NODELAY需要包含这个头文件
//TCP_NODELAY用于禁用Nagle算法

int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }


    //创建服务端用于监听的端口
    int listenfd=socket(AF_INET,SOCK_STREAM|SOCK_NONBLOCK,IPPROTO_TCP);//加一个SOCK_NONBLOCK就是非阻塞IO
    if(listenfd<0)
    {
        perror("socket() failed");
        return -1;
    }

    //设置listenfd的属性
    int opt=1;
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEADDR,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,TCP_NODELAY,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEPORT,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_KEEPALIVE,&opt,static_cast<socklen_t>(sizeof opt));

    //setnonblocking(listenfd);//把服务端的listenfd设置为非阻塞的

    struct sockaddr_in servaddr;//服务端地址结构体
    servaddr.sin_family=AF_INET;//ipv4网络套接字
    servaddr.sin_addr.s_addr = inet_addr(argv[1]);//服务端用于监听的ip地址
    servaddr.sin_port=htons(atoi(argv[2]));//服务端用于监听的端口


    if(bind(listenfd,(struct sockaddr*)&servaddr,sizeof(servaddr))<0)
    {
        perror("bind() failed");
        close(listenfd);
        return -1;
    }
    
    if(listen(listenfd,128)!=0)//在高并发服务其中，第二个参数要大一些
    {
        perror("listen() failed");
        close(listenfd);
        return -1;
    }
    int epollfd = epoll_create(1);//创建epoll句柄（红黑树）

    //为服务端的listenfd准备读事件

    //为服务端的listensock准备可读事件。
    struct epoll_event ev;     //申明事件的数据结构
    ev.data.fd=listenfd;//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
    ev.events=EPOLLIN;  //打算让epoll监视listensock的读事件,采用水平触发

    epoll_ctl(epollfd,EPOLL_CTL_ADD,listenfd,&ev); //把需要监视的socket加入epollfd中。
    
    struct epoll_event evs[10];//存放epoll返回的事件。
    
    
    while(1)//事件循环
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
            //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
            //////////////////////////////
            if(evs[i].events&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
            {
                //如果客户端已经断开
                printf("client(eventfd=%d)disconnected.\n",evs[i].data.fd);
                close(evs[i].data.fd);//关闭客户端的fd
            }
            else if(evs[i].events&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
            {
                //如果发生事件的是listensock，表示有新的客户端连上来
                if(evs[i].data.fd==listenfd)
                {
                    struct sockaddr_in clientaddr;
                    socklen_t len=sizeof(clientaddr);
                    int clientfd=accept4(listenfd,(struct sockaddr*)&clientaddr,&len,SOCK_NONBLOCK);
                    //setnonblocking(clientfd);//客户端连接的fd必须设置为非阻塞的
                    
                    printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientfd,inet_ntoa(clientaddr.sin_addr),ntohs(clientaddr.sin_port));

                    //为新客户准备可读事件，并添加到epoll中
                    ev.data.fd=clientfd;
                    ev.events=EPOLLIN|EPOLLET;//边缘触发
                    epoll_ctl(epollfd,EPOLL_CTL_ADD,clientfd,&ev);
                }
                else
                {
                    char buffer[1024];//存放从客户端读取的数据
                    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
                    {
                        bzero(&buffer,sizeof(buffer));
                        ssize_t nread = read(evs[i].data.fd,buffer,sizeof(buffer));
                        if(nread>0)//成功读取到了数据
                        {
                            //把接收到的数据原封不动的返回回去
                            printf("recv(eventfd=%d);%s\n",evs[i].data.fd,buffer);
                            send(evs[i].data.fd,buffer,strlen(buffer),0);
                        }
                        else if(nread==-1&&errno==EINTR)//读取数据的时候信号中断，继续读取
                        {
                            continue;
                        }
                        else if(nread==-1&&((errno==EAGAIN)||(errno==EWOULDBLOCK)))//全部的数据已读取完必
                        {
                            break;
                        }
                        else if(nread==0)//客户端连接已经断开
                        {
                            printf("client(eventfd=%d) disconnected.\n",evs[i].data.fd);
                            close(evs[i].data.fd);
                            break;
                        }
                    }
                }
                
            }
            else if(evs[i].events&EPOLLOUT)//有数据要写，暂时没代码，以后在说
            {

            }
            else //其他事件，都视为错误,或者对方关闭了链接。
            {
                printf("client(eventfd=%d)error.\n",evs[i].data.fd);
                close(evs[i].data.fd);
            }
            /////////////////////////////
                
        }
    }
    return 0;
}
````


# 封装InetAddress类  
### 头文件  
````
#pragma once

#include<arpa/inet.h>
#include<netinet/in.h>
#include<string>

//socket的地址协议簇
class InetAddress
{
private:
    sockaddr_in addr_;      //表示地址协议的结构体
public:
    InetAddress();
    InetAddress(const std::string &ip,uint16_t port);//如果是监听的fd，用这个构造函数
    InetAddress(const sockaddr_in addr);//如果是客户端连上来的fd，用这个构造函数
    
    ~InetAddress();

    const char*ip() const;  //返回字符串表示的IP地址，例如192.168.216.128
    uint16_t port() const;  //返回整数表示的端口，例如80、8080
    const sockaddr*addr() const;//返回addr_成员的地址，转换成了sockaddr类型
    void setaddr(sockaddr_in clientaddr); //设置addr_成员的值
};
````

### 源文件  
````
#include "InetAddress.h"

/*
class InetAddress
{
private:
    sockaddr_in addr_;      //表示地址协议的结构体
public:
    InetAddress();
    InetAddress(const std::string &ip,uint16_t port);//如果是监听的fd，用这个构造函数
    InetAddress(const sockaddr_in addr):addr_(addr){}//如果是客户端连上来的fd，用这个构造函数
    ~InetAddress();

    const char*ip() const;  //返回字符串表示的IP地址，例如192.168.216.128
    uint16_t port() const;  //返回整数表示的端口，例如80、8080
    const sockaddr*addr() const;//返回addr_成员的地址，转换成了sockaddr类型
    void setaddr(sockaddr_in clientaddr); //设置addr_成员的值
};
*/
InetAddress::InetAddress()
{
    
}
InetAddress::InetAddress(const std::string &ip,uint16_t port)//如果是监听的fd，用这个构造函数
{
    addr_.sin_family=AF_INET;//ipv4网络套接字
    addr_.sin_addr.s_addr = inet_addr(ip.c_str());//服务端用于监听的ip地址
    addr_.sin_port=htons(port);//服务端用于监听的端口
}
InetAddress::InetAddress(const sockaddr_in addr):addr_(addr)//如果是客户端连上来的fd，用这个构造函数
{

}
InetAddress::~InetAddress()
{

}

const char* InetAddress::ip() const  //返回字符串表示的IP地址，例如192.168.216.128
{
    return inet_ntoa(addr_.sin_addr);
    
}
uint16_t InetAddress::port() const  //返回整数表示的端口，例如80、8080
{
    return ntohs(addr_.sin_port);
}
const sockaddr* InetAddress::addr() const//返回addr_成员的地址，转换成了sockaddr类型
{
    return (sockaddr*)&addr_;
}
void InetAddress::setaddr(sockaddr_in clientaddr) //设置addr_成员的值
{
    addr_=clientaddr;
}


````
### Makefile
````
all:client tcpepoll

client:client.cpp
	g++ -g client.cpp -o client

tcpepoll:tcpepoll.cpp InetAddress.cpp
	g++ -g tcpepoll.cpp InetAddress.cpp -o tcpepoll

clean:
	rm -f client tcpepoll
````

### 修改服务端  
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
#include<sys/types.h>
#include<arpa/inet.h>
#include<sys/fcntl.h>
#include<sys/epoll.h>
#include<netinet/tcp.h>// TCP_NODELAY需要包含这个头文件
//TCP_NODELAY用于禁用Nagle算法
#include"InetAddress.h"


int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }


    //创建服务端用于监听的端口
    int listenfd=socket(AF_INET,SOCK_STREAM|SOCK_NONBLOCK,IPPROTO_TCP);//加一个SOCK_NONBLOCK就是非阻塞IO
    if(listenfd<0)
    {
        perror("socket() failed");
        return -1;
    }

    //设置listenfd的属性
    int opt=1;
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEADDR,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,TCP_NODELAY,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEPORT,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_KEEPALIVE,&opt,static_cast<socklen_t>(sizeof opt));

    //setnonblocking(listenfd);//把服务端的listenfd设置为非阻塞的

    //sockaddr_in servaddr;//服务端地址结构体
    //servaddr.sin_family=AF_INET;//ipv4网络套接字
    //servaddr.sin_addr.s_addr = inet_addr(argv[1]);//服务端用于监听的ip地址
    //servaddr.sin_port=htons(atoi(argv[2]));//服务端用于监听的端口
    InetAddress servaddr(argv[1],atoi(argv[2]));

    if(bind(listenfd,servaddr.addr(),sizeof(sockaddr))<0)
    {
        perror("bind() failed");
        close(listenfd);
        return -1;
    }
    
    if(listen(listenfd,128)!=0)//在高并发服务其中，第二个参数要大一些
    {
        perror("listen() failed");
        close(listenfd);
        return -1;
    }
    int epollfd = epoll_create(1);//创建epoll句柄（红黑树）

    //为服务端的listenfd准备读事件

    //为服务端的listensock准备可读事件。
    epoll_event ev;     //申明事件的数据结构
    ev.data.fd=listenfd;//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
    ev.events=EPOLLIN;  //打算让epoll监视listensock的读事件,采用水平触发

    epoll_ctl(epollfd,EPOLL_CTL_ADD,listenfd,&ev); //把需要监视的socket加入epollfd中。
    
    epoll_event evs[10];//存放epoll返回的事件。
    
    
    while(1)//事件循环
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
            //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
            //////////////////////////////
            if(evs[i].events&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
            {
                //如果客户端已经断开
                printf("client(eventfd=%d)disconnected.\n",evs[i].data.fd);
                close(evs[i].data.fd);//关闭客户端的fd
            }
            else if(evs[i].events&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
            {
                //如果发生事件的是listensock，表示有新的客户端连上来
                if(evs[i].data.fd==listenfd)
                {
                    sockaddr_in peeraddr;
                    socklen_t len=sizeof(peeraddr);
                    int clientfd=accept4(listenfd,(sockaddr*)&peeraddr,&len,SOCK_NONBLOCK);
                    
                    InetAddress clientaddr(peeraddr);

                    printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientfd,clientaddr.ip(),clientaddr.port());

                    //为新客户准备可读事件，并添加到epoll中
                    ev.data.fd=clientfd;
                    ev.events=EPOLLIN|EPOLLET;//边缘触发
                    epoll_ctl(epollfd,EPOLL_CTL_ADD,clientfd,&ev);
                }
                else
                {
                    char buffer[1024];//存放从客户端读取的数据
                    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
                    {
                        bzero(&buffer,sizeof(buffer));
                        ssize_t nread = read(evs[i].data.fd,buffer,sizeof(buffer));
                        if(nread>0)//成功读取到了数据
                        {
                            //把接收到的数据原封不动的返回回去
                            printf("recv(eventfd=%d);%s\n",evs[i].data.fd,buffer);
                            send(evs[i].data.fd,buffer,strlen(buffer),0);
                        }
                        else if(nread==-1&&errno==EINTR)//读取数据的时候信号中断，继续读取
                        {
                            continue;
                        }
                        else if(nread==-1&&((errno==EAGAIN)||(errno==EWOULDBLOCK)))//全部的数据已读取完必
                        {
                            break;
                        }
                        else if(nread==0)//客户端连接已经断开
                        {
                            printf("client(eventfd=%d) disconnected.\n",evs[i].data.fd);
                            close(evs[i].data.fd);
                            break;
                        }
                    }
                }
                
            }
            else if(evs[i].events&EPOLLOUT)//有数据要写，暂时没代码，以后在说
            {

            }
            else //其他事件，都视为错误,或者对方关闭了链接。
            {
                printf("client(eventfd=%d)error.\n",evs[i].data.fd);
                close(evs[i].data.fd);
            }
            /////////////////////////////
                
        }
    }


    return 0;
}
````

# 封装socket类  
### 头文件  
````
#pragma once
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/tcp.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include "InetAddress.h"

//创建一个非阻塞的socket
int createnonblocking();

//socekt 类
class Socket
{
private:
    const int fd_;  //Socket持有的fd，在构造函数中传进来
public:
    Socket(int fd); //构造函数，传入一个已经准备好的fd
    ~Socket();      //在析构函数中，关闭fd_

    int fd() const; //返回fd成员
    void setreuseaddr(bool on);  //设置SO_REUSEADDR选项，true打开，false关闭
    void setreuseport(bool on);  //设置SO_REUSEPORT选项
    void settcpnodelay(bool on); //设置TCP_NODELAY选项
    void setkeepalive(bool on);  //设置SO_KEEPALIVE选项
    void bind(const InetAddress& servaddr); //服务端的socket将调用此函数
    void listen(int nn=128);                //服务端的socket将调用此函数
    int accept(InetAddress& clientaddr);   //服务端的socket将调用此函数
};
````


### 源文件  
````
#include "Socket.h"
//创建一个非阻塞的socket
int createnonblocking()
{
    //创建服务端用于监听的端口
    int listenfd=socket(AF_INET,SOCK_STREAM|SOCK_NONBLOCK,IPPROTO_TCP);//加一个SOCK_NONBLOCK就是非阻塞IO
    if(listenfd<0)
    {
        printf("%s:%s:%d listen socket create error:%d\n",__FILE__,__FUNCTION__,__LINE__,errno);
        exit(-1);
    }
    return listenfd;
}

//构造函数，传入一个已经准备好的fd
Socket::Socket(int fd):fd_(fd)
{

}

//在析构函数中，关闭fd_
Socket::~Socket()      
{
    ::close(fd_);
}


//返回fd成员
int Socket::fd() const
{
    return fd_;
} 


//设置SO_REUSEADDR选项，true打开，false关闭
void Socket::setreuseaddr(bool on)
{
    int optval = on?1:0;
    ::setsockopt(fd_,SOL_SOCKET,SO_REUSEADDR,&optval,sizeof(optval));
}

void Socket::setreuseport(bool on)  //设置SO_REUSEPORT选项
{
    int optval = on?1:0;
    ::setsockopt(fd_,SOL_SOCKET,SO_REUSEPORT,&optval,sizeof(optval));
}

void Socket::settcpnodelay(bool on) //设置TCP_NODELAY选项
{
    int optval = on?1:0;
    ::setsockopt(fd_,IPPROTO_TCP,TCP_NODELAY,&optval,sizeof(optval));
}



void Socket::setkeepalive(bool on)  //设置SO_KEEPALIVE选项
{
    int optval = on?1:0;
    ::setsockopt(fd_,SOL_SOCKET,SO_KEEPALIVE,&optval,sizeof(optval));    
}

void Socket::bind(const InetAddress& servaddr) //服务端的socket将调用此函数
{
    if(::bind(fd_,servaddr.addr(),sizeof(sockaddr))<0)
    {
        perror("bind() failed");
        close(fd_);
        exit(-1);
    }
}


void Socket::listen(int nn)                //服务端的socket将调用此函数
{
    if(::listen(fd_,nn)!=0)//在高并发服务其中，第二个参数要大一些
    {
        perror("listen() failed");
        close(fd_);
        exit(-1);
    }
}

int Socket::accept(InetAddress& clientaddr)   //服务端的socket将调用此函数
{
    sockaddr_in peeraddr;
    socklen_t len=sizeof(peeraddr);
    int clientfd=accept4(fd_,(sockaddr*)&peeraddr,&len,SOCK_NONBLOCK);
    
    clientaddr.setaddr(peeraddr);

    return clientfd;

}

````

### Makefile  
````
all:client tcpepoll

client:client.cpp
	g++ -g client.cpp -o client

tcpepoll:tcpepoll.cpp InetAddress.cpp Socket.cpp
	g++ -g tcpepoll.cpp InetAddress.cpp Socket.cpp -o tcpepoll

clean:
	rm -f client tcpepoll
````

### 修改服务端  
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
#include<sys/types.h>
#include<arpa/inet.h>
#include<sys/fcntl.h>
#include<sys/epoll.h>
#include<netinet/tcp.h>// TCP_NODELAY需要包含这个头文件
//TCP_NODELAY用于禁用Nagle算法
#include"InetAddress.h"
#include"Socket.h"

int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }

    /*
    //创建服务端用于监听的端口
    int listenfd=socket(AF_INET,SOCK_STREAM|SOCK_NONBLOCK,IPPROTO_TCP);//加一个SOCK_NONBLOCK就是非阻塞IO
    if(listenfd<0)
    {
        perror("socket() failed");
        return -1;
    }

    //设置listenfd的属性
    int opt=1;
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEADDR,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,TCP_NODELAY,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_REUSEPORT,&opt,static_cast<socklen_t>(sizeof opt));
    setsockopt(listenfd,SOL_SOCKET,SO_KEEPALIVE,&opt,static_cast<socklen_t>(sizeof opt));

    InetAddress servaddr(argv[1],atoi(argv[2]));

    if(bind(listenfd,servaddr.addr(),sizeof(sockaddr))<0)
    {
        perror("bind() failed");
        close(listenfd);
        return -1;
    }
    
    if(listen(listenfd,128)!=0)//在高并发服务其中，第二个参数要大一些
    {
        perror("listen() failed");
        close(listenfd);
        return -1;
    }
    */
    Socket servsock(createnonblocking());
    InetAddress servaddr(argv[1],atoi(argv[2]));
    servsock.setreuseaddr(true);
    servsock.settcpnodelay(true);
    servsock.setreuseport(true);
    servsock.setkeepalive(true);
    servsock.bind(servaddr);
    servsock.listen();


    int epollfd = epoll_create(1);//创建epoll句柄（红黑树）

    //为服务端的listenfd准备读事件

    //为服务端的listensock准备可读事件。
    epoll_event ev;     //申明事件的数据结构
    ev.data.fd=servsock.fd();//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
    ev.events=EPOLLIN;  //打算让epoll监视listensock的读事件,采用水平触发

    epoll_ctl(epollfd,EPOLL_CTL_ADD,servsock.fd(),&ev); //把需要监视的socket加入epollfd中。
    
    epoll_event evs[10];//存放epoll返回的事件。
    
    
    while(1)//事件循环
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
            //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
            //////////////////////////////
            if(evs[i].events&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
            {
                //如果客户端已经断开
                printf("client(eventfd=%d)disconnected.\n",evs[i].data.fd);
                close(evs[i].data.fd);//关闭客户端的fd
            }
            else if(evs[i].events&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
            {
                //如果发生事件的是listensock，表示有新的客户端连上来
                if(evs[i].data.fd==servsock.fd())
                {
                    //sockaddr_in peeraddr;
                    //socklen_t len=sizeof(peeraddr);
                    //int clientfd=accept4(listenfd,(sockaddr*)&peeraddr,&len,SOCK_NONBLOCK);
                    
                    InetAddress clientaddr;
                    Socket* clientsock=new Socket(servsock.accept(clientaddr));

                    printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientsock->fd(),clientaddr.ip(),clientaddr.port());

                    //为新客户准备可读事件，并添加到epoll中
                    ev.data.fd=clientsock->fd();
                    ev.events=EPOLLIN|EPOLLET;//边缘触发
                    epoll_ctl(epollfd,EPOLL_CTL_ADD,clientsock->fd(),&ev);
                }
                else
                {
                    char buffer[1024];//存放从客户端读取的数据
                    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
                    {
                        bzero(&buffer,sizeof(buffer));
                        ssize_t nread = read(evs[i].data.fd,buffer,sizeof(buffer));
                        if(nread>0)//成功读取到了数据
                        {
                            //把接收到的数据原封不动的返回回去
                            printf("recv(eventfd=%d);%s\n",evs[i].data.fd,buffer);
                            send(evs[i].data.fd,buffer,strlen(buffer),0);
                        }
                        else if(nread==-1&&errno==EINTR)//读取数据的时候信号中断，继续读取
                        {
                            continue;
                        }
                        else if(nread==-1&&((errno==EAGAIN)||(errno==EWOULDBLOCK)))//全部的数据已读取完必
                        {
                            break;
                        }
                        else if(nread==0)//客户端连接已经断开
                        {
                            printf("client(eventfd=%d) disconnected.\n",evs[i].data.fd);
                            close(evs[i].data.fd);
                            break;
                        }
                    }
                }
                
            }
            else if(evs[i].events&EPOLLOUT)//有数据要写，暂时没代码，以后在说
            {

            }
            else //其他事件，都视为错误,或者对方关闭了链接。
            {
                printf("client(eventfd=%d)error.\n",evs[i].data.fd);
                close(evs[i].data.fd);
            }
            /////////////////////////////
                
        }
    }
    return 0;
}
````

# 封装epoll类  
### 





