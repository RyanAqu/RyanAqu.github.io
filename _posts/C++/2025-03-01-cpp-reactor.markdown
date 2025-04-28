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
Inet类用于创建ip地址和port端口，比如在main函数里面利用main输入参数ip和port直接构造用于监听的InetAddress，这个地址和端口接下来会被bind绑定到特定的listenfd。  

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
传入fd，socket类来修饰非阻塞的网络套接字，修饰需要的fd文件描述符，并为之设置一些socket属性，网络通信是通过fd实现的。比如通过createnonblocking函数创建一个非阻塞的套接字，然后传入socket类进行修饰。    
除此之外，socket类还封装了一些对fd的基础网络操作，比如listen、accept、bind等。  
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
epoll类把epollfd的句柄式创建变成了创建类的对象，对epollfd进行管理，对epoll的create和wait系统调用等进行了封装，每个epoll监控一棵红黑树，监控一个事件循环loop，并且更新epoll监控的channel，每个channel有自己的fd以及对应的感兴趣的事件类型；  
epoll类还暴露了对外的添加事件的接口，用于channel类进行添加；  
epoll类还要监控事件循环，用指针把对应事件传递给对应的channel，并且把返回事件的处理交给事件循环loop类进行运作，返还给对应channel进行回调处理；  
### 头文件  
````
#pragma once
#include<stdio.h>
#include<stdlib.h>
#include<errno.h>
#include<strings.h>
#include<string.h>
#include<sys/epoll.h>
#include<vector>
#include<unistd.h>

//Epoll类
class Epoll
{
private:
    static const int MaxEvents=100;//epoll_wait()返回事件数组的大小
    int epollfd_ = -1;//epoll句柄，在构造函数中创建
    epoll_event events_[MaxEvents];//存放epoll返回的事件。
public:
    Epoll();        //构造函数创建epollfd_
    ~Epoll();       //析构函数关闭epollfd_
    void addfd(int fd,uint32_t op); //把fd和他需要监视的事件添加到红黑树上
    std::vector<epoll_event> loop(int timeout=-1);//运行epoll_wait()，等待事件的发生，已发生的事件用vector容器返回
};
````

### 源文件  
````
#include"Epoll.h"

Epoll::Epoll()        //构造函数创建epollfd_
{
    if((epollfd_ = epoll_create(1))==-1) //创建epoll句柄（红黑树）
    {
        printf("epoll_create() failed(%d).\n",errno);
        exit(-1);
    }

}
Epoll::~Epoll()      //析构函数关闭epollfd_
{
    close(epollfd_);
}
void Epoll::addfd(int fd,uint32_t op) //把fd和他需要监视的事件添加到红黑树上
{
    epoll_event ev;     //申明事件的数据结构
    ev.data.fd=fd;//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
    ev.events=op;  //
    if(epoll_ctl(epollfd_,EPOLL_CTL_ADD,fd,&ev)==-1) //把需要监视的socket加入epollfd中。
    {
        printf("epoll_ctl() failed(%d).\n",errno);
        exit(-1);
    }
}
std::vector<epoll_event> Epoll::loop(int timeout)//运行epoll_wait()，等待事件的发生，已发生的事件用vector容器返回
{
    std::vector<epoll_event> evs;//存放返回的epoll_wait()事件
    bzero(events_,sizeof(events_));
    
    //等待监视的socket有事件发生
    int infds = epoll_wait(epollfd_,events_,MaxEvents,timeout);

    
    //返回失败
    if(infds<0)
    {
        perror("epoll() failed\n");exit(-1);
    }
    //超时
    if(infds==0)
    {
        printf("epoll() timeout.\n");
        return evs;
    }
    //如果infds>0,表示有事件发生的socket的数量
    for(int i=0;i<infds;i++)
    {
        evs.push_back(events_[i]);
    }
    return evs;
}
````

### Makefile  
````
all:client tcpepoll

client:client.cpp
	g++ -g client.cpp -o client

tcpepoll:tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp
	g++ -g tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp -o tcpepoll

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
#include"InetAddress.h"//TCP_NODELAY用于禁用Nagle算法
#include"Socket.h"
#include"Epoll.h"

int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }

    Socket servsock(createnonblocking());
    InetAddress servaddr(argv[1],atoi(argv[2]));
    servsock.setreuseaddr(true);
    servsock.settcpnodelay(true);
    servsock.setreuseport(true);
    servsock.setkeepalive(true);
    servsock.bind(servaddr);
    servsock.listen();

    Epoll ep;
    ep.addfd(servsock.fd(),EPOLLIN);//水平触发，监听listenfd的读事件
    std::vector<epoll_event>evs;//存放epoll_wait()返回事件。

    while(1)//事件循环
    {
        evs=ep.loop();  //等待监视的fd有事件发生

        //如果infds>0,表示有事件发生的socket的数量
        for(auto& ev:evs)
        {
            //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
            //////////////////////////////
            if(ev.events&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
            {
                //如果客户端已经断开
                printf("client(eventfd=%d)disconnected.\n",ev.data.fd);
                close(ev.data.fd);//关闭客户端的fd
            }
            else if(ev.events&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
            {
                //如果发生事件的是listensock，表示有新的客户端连上来
                if(ev.data.fd==servsock.fd())
                {
                           
                    InetAddress clientaddr;
                    Socket* clientsock=new Socket(servsock.accept(clientaddr));

                    printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientsock->fd(),clientaddr.ip(),clientaddr.port());

                    //为新客户准备可读事件，并添加到epoll中
                    ep.addfd(clientsock->fd(),EPOLLIN|EPOLLET);   //客户端采用边缘触发
                    
                }
                else
                {
                    char buffer[1024];//存放从客户端读取的数据
                    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
                    {
                        bzero(&buffer,sizeof(buffer));
                        ssize_t nread = read(ev.data.fd,buffer,sizeof(buffer));
                        if(nread>0)//成功读取到了数据
                        {
                            //把接收到的数据原封不动的返回回去
                            printf("recv(eventfd=%d);%s\n",ev.data.fd,buffer);
                            send(ev.data.fd,buffer,strlen(buffer),0);
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
                            printf("client(eventfd=%d) disconnected.\n",ev.data.fd);
                            close(ev.data.fd);
                            break;
                        }
                    }
                }
                
            }
            else if(ev.events&EPOLLOUT)//有数据要写，暂时没代码，以后在说
            {

            }
            else //其他事件，都视为错误,或者对方关闭了链接。
            {
                printf("client(eventfd=%d)error.\n",ev.data.fd);
                close(ev.data.fd);
            }
            /////////////////////////////
                
        }
    }
    return 0;
}
````

# 封装Channel类  
channel类封装了单个fd对应的关注的事件和类型，每个channel创建时需要知道自己对应的fd和epoll红黑树；   
channel类主要封装了回调函数来处理各个事件，比如有连接断开，有数据发送过来，有数据需要发送；  
channel类还要设置事件对应的回调函数；  
channel类设置完以后要调用updatechannel使得红黑树关注新事件；  

### epoll结构体  
epoll 是 Linux 中用于高效 I/O 事件通知的机制，特别适合处理大量文件描述符（如网络套接字）。它的核心是三个系统调用：epoll_create、epoll_ctl 和 epoll_wait，以及一个关键的数据结构 struct epoll_event。  
````
#include <sys/epoll.h>

struct epoll_event {
    uint32_t     events;  // 需要监听的事件类型
    epoll_data_t data;    // 用户数据，通常用于存储文件描述符或指针
};

typedef union epoll_data {
    void* ptr;  // 可以存储任意指针
    int fd;     // 通常用于存储文件描述符
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;
````

之前用的都是fd，在reactor模型中要用void* ptr。

### 用ptr的服务端demo  
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

class Channel
{
private:
    int fd_;
    bool islisten_=false;//true表示监听的fd，false表示客户端连上来的fd
    //.....
public:
    Channel(int fd,bool islisten=false):fd_(fd),islisten_(islisten)
    {

    }
    int fd()
    {
        return fd_;
    }
    bool islisten()
    {
        return islisten_;
    }
    //.....
};

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

    Channel* servchannel=new Channel(listenfd,true);
    //为服务端的listenfd准备读事件

    //为服务端的listensock准备可读事件。
    struct epoll_event ev;     //申明事件的数据结构
    ev.data.ptr=servchannel;//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
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
            Channel* ch=(Channel*)evs[i].data.ptr;
            //如果发生事件的是listensock，表示有新的客户端连上来
            if(ch->islisten()==true)
            {
                struct sockaddr_in clientaddr;
                socklen_t len=sizeof(clientaddr);
                int clientfd=accept(listenfd,(struct sockaddr*)&clientaddr,&len);
                setnonblocking(clientfd);//客户端连接的fd必须设置为非阻塞的
                
                printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientfd,inet_ntoa(clientaddr.sin_addr),ntohs(clientaddr.sin_port));

                //为新客户准备可读事件，并添加到epoll中
                Channel*clientchannel=new Channel(clientfd);
                ev.data.ptr=clientchannel;
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
                    printf("client(eventfd=%d)disconnected.\n",ch->fd());
                    close(ch->fd());//关闭客户端的fd
                }
                else if(evs[i].events&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
                {
                    char buffer[1024];//存放从客户端读取的数据
                    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
                    {
                        bzero(&buffer,sizeof(buffer));
                        ssize_t nread = read(ch->fd(),buffer,sizeof(buffer));
                        if(nread>0)//成功读取到了数据
                        {
                            //把接收到的数据原封不动的返回回去
                            printf("recv(eventfd=%d);%s\n",ch->fd(),buffer);
                            send(ch->fd(),buffer,strlen(buffer),0);
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
                            printf("client(eventfd=%d) disconnected.\n",ch->fd());
                            close(ch->fd());
                            break;
                        }
                    }
                }
                else if(evs[i].events&EPOLLOUT)//有数据要写，暂时没代码，以后在说
                {

                }
                else //其他事件，都视为错误,或者对方关闭了链接。
                {
                    printf("client(eventfd=%d)error.\n",ch->fd());
                    close(ch->fd());
                }
                /////////////////////////////
                
            }
        }
    }

    return 0;
}
````

### 修改epoll头文件  
````
#pragma once
#include<stdio.h>
#include<stdlib.h>
#include<errno.h>
#include<strings.h>
#include<string.h>
#include<sys/epoll.h>
#include<vector>
#include<unistd.h>
#include"Channel.h"

class Channel;

//Epoll类
class Epoll
{
private:
    static const int MaxEvents=100;//epoll_wait()返回事件数组的大小
    int epollfd_ = -1;//epoll句柄，在构造函数中创建
    epoll_event events_[MaxEvents];//存放epoll返回的事件。
public:
    Epoll();        //构造函数创建epollfd_
    ~Epoll();       //析构函数关闭epollfd_
    //void addfd(int fd,uint32_t op); //把fd和他需要监视的事件添加到红黑树上
    void updatechannel(Channel *ch);//把channel添加/更新到红黑树上，channel中有fd，也有要监视的事件
    //std::vector<epoll_event> loop(int timeout=-1);//运行epoll_wait()，等待事件的发生，已发生的事件用vector容器返回
    std::vector<Channel*> loop(int timeout=-1);//运行epoll_wait()，等待事件的发生，已发生的事件用vector容器返回
};
````

### 修改epoll源文件  
````
#include"Epoll.h"

Epoll::Epoll()        //构造函数创建epollfd_
{
    if((epollfd_ = epoll_create(1))==-1) //创建epoll句柄（红黑树）
    {
        printf("epoll_create() failed(%d).\n",errno);
        exit(-1);
    }

}
Epoll::~Epoll()      //析构函数关闭epollfd_
{
    close(epollfd_);
}
/*
//把fd和他需要监视的事件添加到红黑树上
void Epoll::addfd(int fd,uint32_t op) 
{
    epoll_event ev;     //申明事件的数据结构
    ev.data.fd=fd;//指定事件的自定义数据，会随着epoll_wait()返回的事件一并返回
    ev.events=op;  //
    if(epoll_ctl(epollfd_,EPOLL_CTL_ADD,fd,&ev)==-1) //把需要监视的socket加入epollfd中。
    {
        printf("epoll_ctl() failed(%d).\n",errno);
        exit(-1);
    }
}
*/
//把channel添加/更新到红黑树上，channel中有fd，也有要监视的事件
void Epoll::updatechannel(Channel *ch)
{
    epoll_event ev;//申明事件的数据结构
    ev.data.ptr=ch;//指定channel
    ev.events=ch->events();//指定事件
    if(ch->inpoll())
    {
        if(epoll_ctl(epollfd_,EPOLL_CTL_MOD,ch->fd(),&ev)==-1)
        {
            perror("epoll_ctl() failed .\n");
            exit(-1);
        }
    }
    else
    {
        if(epoll_ctl(epollfd_,EPOLL_CTL_ADD,ch->fd(),&ev)==-1)
        {
            perror("epoll_ctl() failed .\n");
            exit(-1);
        }
        ch->setinepoll();
    }
}


std::vector<Channel*> Epoll::loop(int timeout)//运行epoll_wait()，等待事件的发生，已发生的事件用vector容器返回
{
    std::vector<Channel*> channels;//存放返回的epoll_wait()事件
    bzero(events_,sizeof(events_));
    
    //等待监视的socket有事件发生
    int infds = epoll_wait(epollfd_,events_,MaxEvents,timeout);

    
    //返回失败
    if(infds<0)
    {
        perror("epoll() failed\n");exit(-1);
    }
    //超时
    if(infds==0)
    {
        printf("epoll() timeout.\n");
        return channels;
    }
    //如果infds>0,表示有事件发生的socket的数量
    for(int i=0;i<infds;i++)
    {
        //evs.push_back(events_[i]);
        Channel *ch=(Channel*)events_[i].data.ptr;
        ch->setrevents(events_[i].events);
        channels.push_back(ch);
    }
    return channels;
}
````



### channel头文件    
````
#pragma once
#include<sys/epoll.h>
#include"Epoll.h"

class Epoll;

class Channel
{
private:
    int fd_=-1;        //channel和fd是一对一的关系
    Epoll * ep_=nullptr;//channel和epoll是一对一的关系,一个channel只在一个红黑树上
    bool inepoll_=false;//标记channel是否已经添加到epoll树上，如果未添加，调用epoll_ctl()的时候用EPOLL_CTL_ADD，否则用EPOLL_CTL_MOD
    uint32_t events_=0;//fd需要监视的事件，listenfd和clientfd需要监视EPOLLIN，clientfd还要监视EPOLLOUT
    uint32_t revents_=0;//存放fd已发生的事件
public:
    Channel(Epoll* ep,int fd);
    ~Channel();

    int fd();           //返回fd_成员
    void useet();        //采用边缘触发
    void enablereading();//让epoll_wait()监视fd_的读事件
    void setinepoll();  //把inepoll_成员的值设置为true
    void setrevents(uint32_t ev);//设置revents_成员的值为参数ev
    bool inpoll();      //返回inepoll_成员的值
    uint32_t events();  //返回events_成员的值
    uint32_t revents();  //返回revents_成员的值
};
````


### channel源文件    
````
#include"Channel.h"

Channel::Channel(Epoll* ep,int fd):ep_(ep),fd_(fd)
{

}

Channel::~Channel()
{
    //在析构函数中不要销毁ep_，也不能关闭fd_，因为这两个不属于Channel类，channel类只是需要他们，使用他们而已
}

int Channel::fd()           //返回fd_成员
{
    return fd_;
}
void Channel::useet()        //采用边缘触发
{
    events_=events_|EPOLLET;
}
void Channel::enablereading()//让epoll_wait()监视fd_的读事件
{
    events_=events_|EPOLLIN;
    ep_->updatechannel(this);
}
void Channel::setinepoll()  //把inepoll_成员的值设置为true
{
    inepoll_=true;
}
void Channel::setrevents(uint32_t ev)//设置revents_成员的值为参数ev
{
    revents_=ev;
}
bool Channel::inpoll()      //返回inepoll_成员的值
{
    return inepoll_;
}
uint32_t Channel::events()  //返回events_成员的值
{
    return events_;
}
uint32_t Channel::revents()  //返回revents_成员的值
{
    return revents_;
}
````

### Makefile   
````
all:client tcpepoll

client:client.cpp
	g++ -g client.cpp -o client

tcpepoll:tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp Channel.cpp
	g++ -g tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp Channel.cpp -o tcpepoll

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
#include"InetAddress.h"//TCP_NODELAY用于禁用Nagle算法
#include"Socket.h"
#include"Epoll.h"


int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }

    Socket servsock(createnonblocking());
    InetAddress servaddr(argv[1],atoi(argv[2]));
    servsock.setreuseaddr(true);
    servsock.settcpnodelay(true);
    servsock.setreuseport(true);
    servsock.setkeepalive(true);
    servsock.bind(servaddr);
    servsock.listen();

    Epoll ep;
    //ep.addfd(servsock.fd(),EPOLLIN);//水平触发，监听listenfd的读事件
    Channel * servchannel=new Channel(&ep,servsock.fd());
    servchannel->enablereading();
    

    while(1)//事件循环
    {
        std::vector<Channel*>channels=ep.loop();  //等待监视的fd有事件发生

        //如果infds>0,表示有事件发生的socket的数量
        for(auto& ch:channels)
        {
            //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
            //////////////////////////////
            if(ch->revents()&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
            {
                //如果客户端已经断开
                printf("client(eventfd=%d)disconnected.\n",ch->fd());
                close(ch->fd());//关闭客户端的fd
            }
            else if(ch->revents()&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
            {
                //如果发生事件的是listensock，表示有新的客户端连上来
                if(ch==servchannel)
                {
                    InetAddress clientaddr;
                    Socket* clientsock=new Socket(servsock.accept(clientaddr));

                    printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientsock->fd(),clientaddr.ip(),clientaddr.port());

                    //为新客户准备可读事件，并添加到epoll中
                    //ep.addfd(clientsock->fd(),EPOLLIN|EPOLLET);   //客户端采用边缘触发
                    Channel * clientchannel=new Channel(&ep,clientsock->fd());
                    clientchannel->useet();
                    clientchannel->enablereading();
                }
                else
                {
                    char buffer[1024];//存放从客户端读取的数据
                    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
                    {
                        bzero(&buffer,sizeof(buffer));
                        ssize_t nread = read(ch->fd(),buffer,sizeof(buffer));
                        if(nread>0)//成功读取到了数据
                        {
                            //把接收到的数据原封不动的返回回去
                            printf("recv(eventfd=%d);%s\n",ch->fd(),buffer);
                            send(ch->fd(),buffer,strlen(buffer),0);
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
                            printf("client(eventfd=%d) disconnected.\n",ch->fd());
                            close(ch->fd());
                            break;
                        }
                    }
                }
                
            }
            else if(ch->revents()&EPOLLOUT)//有数据要写，暂时没代码，以后在说
            {

            }
            else //其他事件，都视为错误,或者对方关闭了链接。
            {
                printf("client(eventfd=%d)error.\n",ch->fd());
                close(ch->fd());
            }
            /////////////////////////////
                
        }
    }


    return 0;
}

````

# 优化Channel类  
### 优化Channel头文件  
````
#pragma once
#include<sys/epoll.h>
#include"Epoll.h"
#include"InetAddress.h"
#include"Socket.h"

class Epoll;

class Channel
{
private:
    int fd_=-1;        //channel和fd是一对一的关系
    Epoll * ep_=nullptr;//channel和epoll是一对一的关系,一个channel只在一个红黑树上
    bool inepoll_=false;//标记channel是否已经添加到epoll树上，如果未添加，调用epoll_ctl()的时候用EPOLL_CTL_ADD，否则用EPOLL_CTL_MOD
    uint32_t events_=0;//fd需要监视的事件，listenfd和clientfd需要监视EPOLLIN，clientfd还要监视EPOLLOUT
    uint32_t revents_=0;//存放fd已发生的事件
    bool islisten_=false;//如果为listenfd，取值为true，客户端连上来的fd为false
public:
    Channel(Epoll* ep,int fd,bool islisten);
    ~Channel();

    int fd();           //返回fd_成员
    void useet();        //采用边缘触发
    void enablereading();//让epoll_wait()监视fd_的读事件
    void setinepoll();  //把inepoll_成员的值设置为true
    void setrevents(uint32_t ev);//设置revents_成员的值为参数ev
    bool inpoll();      //返回inepoll_成员的值
    uint32_t events();  //返回events_成员的值
    uint32_t revents();  //返回revents_成员的值

    void handleevent(Socket *servsock); //事件处理函数，epoll_wait()返回的时候，执行它
};
````

### 优化Channel源文件  
````
#include"Channel.h"

Channel::Channel(Epoll* ep,int fd,bool islisten):ep_(ep),fd_(fd),islisten_(islisten)
{

}

Channel::~Channel()
{
    //在析构函数中不要销毁ep_，也不能关闭fd_，因为这两个不属于Channel类，channel类只是需要他们，使用他们而已
}

int Channel::fd()           //返回fd_成员
{
    return fd_;
}
void Channel::useet()        //采用边缘触发
{
    events_=events_|EPOLLET;
}
void Channel::enablereading()//让epoll_wait()监视fd_的读事件
{
    events_=events_|EPOLLIN;
    ep_->updatechannel(this);
}
void Channel::setinepoll()  //把inepoll_成员的值设置为true
{
    inepoll_=true;
}
void Channel::setrevents(uint32_t ev)//设置revents_成员的值为参数ev
{
    revents_=ev;
}
bool Channel::inpoll()      //返回inepoll_成员的值
{
    return inepoll_;
}
uint32_t Channel::events()  //返回events_成员的值
{
    return events_;
}
uint32_t Channel::revents()  //返回revents_成员的值
{
    return revents_;
}

void Channel::handleevent(Socket *servsock)
{
    //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
    //////////////////////////////
    if(revents_&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
    {
        //如果客户端已经断开
        printf("client(eventfd=%d)disconnected.\n",fd_);
        close(fd_);//关闭客户端的fd
    }
    else if(revents_&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
    {
        //如果发生事件的是listensock，表示有新的客户端连上来
        if(islisten_)
        {
            InetAddress clientaddr;
            Socket* clientsock=new Socket(servsock->accept(clientaddr));

            printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientsock->fd(),clientaddr.ip(),clientaddr.port());

            //为新客户准备可读事件，并添加到epoll中
            Channel * clientchannel=new Channel(ep_,clientsock->fd(),false);
            clientchannel->useet();
            clientchannel->enablereading();
        }
        else
        {
            char buffer[1024];//存放从客户端读取的数据
            while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
            {
                bzero(&buffer,sizeof(buffer));
                ssize_t nread = read(fd_,buffer,sizeof(buffer));
                if(nread>0)//成功读取到了数据
                {
                    //把接收到的数据原封不动的返回回去
                    printf("recv(eventfd=%d);%s\n",fd_,buffer);
                    send(fd_,buffer,strlen(buffer),0);
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
                    printf("client(eventfd=%d) disconnected.\n",fd_);
                    close(fd_);
                    break;
                }
            }
        }
        
    }
    else if(revents_&EPOLLOUT)//有数据要写，暂时没代码，以后在说
    {

    }
    else //其他事件，都视为错误,或者对方关闭了链接。
    {
        printf("client(eventfd=%d)error.\n",fd_);
        close(fd_);
    }
    /////////////////////////////
}
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
#include"InetAddress.h"//TCP_NODELAY用于禁用Nagle算法
#include"Socket.h"
#include"Epoll.h"


int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }

    Socket servsock(createnonblocking());
    InetAddress servaddr(argv[1],atoi(argv[2]));
    servsock.setreuseaddr(true);
    servsock.settcpnodelay(true);
    servsock.setreuseport(true);
    servsock.setkeepalive(true);
    servsock.bind(servaddr);
    servsock.listen();

    Epoll ep;
    //ep.addfd(servsock.fd(),EPOLLIN);//水平触发，监听listenfd的读事件
    Channel * servchannel=new Channel(&ep,servsock.fd(),true);
    servchannel->enablereading();
    
    while(1)//事件循环
    {
        std::vector<Channel*>channels=ep.loop();  //等待监视的fd有事件发生

        //如果infds>0,表示有事件发生的socket的数量
        for(auto& ch:channels)
        {
            ch->handleevent(&servsock);
                
        }
    }
    return 0;
}
````

# 回调优化  
### 优化channel类头文件  
````
#pragma once
#include<sys/epoll.h>
#include<functional>
#include"Epoll.h"
#include"InetAddress.h"
#include"Socket.h"

class Epoll;

class Channel
{
private:
    int fd_=-1;        //channel和fd是一对一的关系
    Epoll * ep_=nullptr;//channel和epoll是一对一的关系,一个channel只在一个红黑树上
    bool inepoll_=false;//标记channel是否已经添加到epoll树上，如果未添加，调用epoll_ctl()的时候用EPOLL_CTL_ADD，否则用EPOLL_CTL_MOD
    uint32_t events_=0;//fd需要监视的事件，listenfd和clientfd需要监视EPOLLIN，clientfd还要监视EPOLLOUT
    uint32_t revents_=0;//存放fd已发生的事件
    
    std::function<void()> readcallback_;//fd_读事件的回调函数
public:
    Channel(Epoll* ep,int fd);
    ~Channel();

    int fd();           //返回fd_成员
    void useet();        //采用边缘触发
    void enablereading();//让epoll_wait()监视fd_的读事件
    void setinepoll();  //把inepoll_成员的值设置为true
    void setrevents(uint32_t ev);//设置revents_成员的值为参数ev
    bool inpoll();      //返回inepoll_成员的值
    uint32_t events();  //返回events_成员的值
    uint32_t revents();  //返回revents_成员的值

    void handleevent(); //事件处理函数，epoll_wait()返回的时候，执行它

    void newconnection(Socket *servsock); //处理客户端的新链接请求
    void onmessage();//处理对端发过来的报文
    void setreadcallback(std::function<void()> fn);//设置fd_读事件的回调函数
};
````

### 优化channel类源文件  
````
#include"Channel.h"

Channel::Channel(Epoll* ep,int fd):ep_(ep),fd_(fd)
{

}

Channel::~Channel()
{
    //在析构函数中不要销毁ep_，也不能关闭fd_，因为这两个不属于Channel类，channel类只是需要他们，使用他们而已
}

int Channel::fd()           //返回fd_成员
{
    return fd_;
}
void Channel::useet()        //采用边缘触发
{
    events_=events_|EPOLLET;
}
void Channel::enablereading()//让epoll_wait()监视fd_的读事件
{
    events_=events_|EPOLLIN;
    ep_->updatechannel(this);
}
void Channel::setinepoll()  //把inepoll_成员的值设置为true
{
    inepoll_=true;
}
void Channel::setrevents(uint32_t ev)//设置revents_成员的值为参数ev
{
    revents_=ev;
}
bool Channel::inpoll()      //返回inepoll_成员的值
{
    return inepoll_;
}
uint32_t Channel::events()  //返回events_成员的值
{
    return events_;
}
uint32_t Channel::revents()  //返回revents_成员的值
{
    return revents_;
}

void Channel::handleevent()
{
    //如果客户端连接的sock有事件，表示有报文发过来或者链接已经断开
    //////////////////////////////
    if(revents_&EPOLLRDHUP)//对方已经关闭连接，有些系统检测不到，可以使用EPOLLIN，recv()返回
    {
        //如果客户端已经断开
        printf("client(eventfd=%d)disconnected.\n",fd_);
        close(fd_);//关闭客户端的fd
    }
    else if(revents_&(EPOLLIN|EPOLLPRI))//接受区有数据可以读
    {
        //如果发生事件的是listensock，表示有新的客户端连上来
        readcallback_();
        
    }
    else if(revents_&EPOLLOUT)//有数据要写，暂时没代码，以后在说
    {

    }
    else //其他事件，都视为错误,或者对方关闭了链接。
    {
        printf("client(eventfd=%d)error.\n",fd_);
        close(fd_);
    }
    /////////////////////////////
}



void Channel::newconnection(Socket *servsock) //处理客户端的新链接请求
{
    InetAddress clientaddr;
    Socket* clientsock=new Socket(servsock->accept(clientaddr));

    printf("accept client(fd=%d,ip=%s,port=%d)ok.\n",clientsock->fd(),clientaddr.ip(),clientaddr.port());

    //为新客户准备可读事件，并添加到epoll中
    Channel * clientchannel=new Channel(ep_,clientsock->fd());
    clientchannel->setreadcallback(std::bind(&Channel::onmessage,clientchannel));
    clientchannel->useet();
    clientchannel->enablereading();
}
void Channel::onmessage()//处理对端发过来的报文
{
    char buffer[1024];//存放从客户端读取的数据
    while(true)//使用非阻塞IO，一次读取buffer大小数据，直到全部读取完
    {
        bzero(&buffer,sizeof(buffer));
        ssize_t nread = read(fd_,buffer,sizeof(buffer));
        if(nread>0)//成功读取到了数据
        {
            //把接收到的数据原封不动的返回回去
            printf("recv(eventfd=%d);%s\n",fd_,buffer);
            send(fd_,buffer,strlen(buffer),0);
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
            printf("client(eventfd=%d) disconnected.\n",fd_);
            close(fd_);
            break;
        }
    }
}


void Channel::setreadcallback(std::function<void()> fn)//设置fd_读事件的回调函数
{
    readcallback_=fn;
}
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
#include"InetAddress.h"//TCP_NODELAY用于禁用Nagle算法
#include"Socket.h"
#include"Epoll.h"


int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }

    Socket servsock(createnonblocking());
    InetAddress servaddr(argv[1],atoi(argv[2]));
    servsock.setreuseaddr(true);
    servsock.settcpnodelay(true);
    servsock.setreuseport(true);
    servsock.setkeepalive(true);
    servsock.bind(servaddr);
    servsock.listen();

    Epoll ep;
    //ep.addfd(servsock.fd(),EPOLLIN);//水平触发，监听listenfd的读事件
    Channel * servchannel=new Channel(&ep,servsock.fd());
    servchannel->setreadcallback(std::bind(&Channel::newconnection,servchannel,&servsock));
    servchannel->enablereading();
    

    while(1)//事件循环
    {
        std::vector<Channel*>channels=ep.loop();  //等待监视的fd有事件发生

        //如果infds>0,表示有事件发生的socket的数量
        for(auto& ch:channels)
        {
            ch->handleevent();
                
        }
    }


    return 0;
}
````

# 增加EventLoop事件循环类  
eventloop类封装了事件循环，每个eventloop都会创建一个epoll，事件循环对应一棵红黑树的生命周期；  
eventloop类通过run函数运行事件循环，接收红黑树监控的channel并且回调处理；  

### EventLoop头文件  
````
#pragma once
#include "Epoll.h"

//事件循环类
class EventLoop
{
private:
    Epoll *ep_;     //每个事件循环只有一个epoll

public:
    EventLoop();    //在构造函数中创建epoll对象ep_
    ~EventLoop();   //在析构函数中销毁ep_
    void run();     //运行事件循环
    Epoll* ep();    //返回ep_地址
};
````

### EventLoop源文件  
````
#include"EventLoop.h"

EventLoop::EventLoop():ep_(new Epoll)    //在构造函数中创建epoll对象ep_
{

}
EventLoop::~EventLoop()   //在析构函数中销毁ep_
{
    delete ep_;
}
void EventLoop::run()     //运行事件循环
{
    while(1)//事件循环
    {
        std::vector<Channel*>channels=ep_->loop();  //等待监视的fd有事件发生

        //如果infds>0,表示有事件发生的socket的数量
        for(auto& ch:channels)
        {
            ch->handleevent();
                
        }
    }
}

Epoll* EventLoop::ep()    //返回ep_地址
{
    return ep_;
}

````

### Makefile
````
all:client tcpepoll

client:client.cpp
	g++ -g client.cpp -o client

tcpepoll:tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp Channel.cpp EventLoop.cpp
	g++ -g tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp Channel.cpp EventLoop.cpp -o tcpepoll

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
#include"InetAddress.h"//TCP_NODELAY用于禁用Nagle算法
#include"Socket.h"
#include"Epoll.h"
#include"EventLoop.h"


int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }

    Socket servsock(createnonblocking());
    InetAddress servaddr(argv[1],atoi(argv[2]));
    servsock.setreuseaddr(true);
    servsock.settcpnodelay(true);
    servsock.setreuseport(true);
    servsock.setkeepalive(true);
    servsock.bind(servaddr);
    servsock.listen();


    EventLoop loop;
    
    Channel * servchannel=new Channel(loop.ep(),servsock.fd());
    servchannel->setreadcallback(std::bind(&Channel::newconnection,servchannel,&servsock));
    servchannel->enablereading();

    loop.run();
    
    return 0;
}
````


# EventLoop和Channel的连接处理  
这里eventloop包装了epoll类，channel直接与eventloop链接，eventloop内部处理epoll；  
channel和对应一个eventloop，也就对应一个epoll；  
### 修改eventloop类  
````
#pragma once
#include "Epoll.h"

class Channel;
class Epoll;

// 事件循环类。
class EventLoop
{
private:
    Epoll *ep_;                       // 每个事件循环只有一个Epoll。
public:
    EventLoop();                   // 在构造函数中创建Epoll对象ep_。
    ~EventLoop();                // 在析构函数中销毁ep_。

    void run();                      // 运行事件循环。

    void updatechannel(Channel *ch);      // 把channel添加/更新到红黑树上，channel中有fd，也有需要监视的事件。
};

///////////////////////////////////////////////
#include "EventLoop.h"

/*
// 事件循环类。
class EventLoop
{
private:
    Epoll *ep_;                       // 每个事件循环只有一个Epoll。
public:
    EventLoop();                   // 在构造函数中创建Epoll对象ep_。
    ~EventLoop();                // 在析构函数中销毁ep_。

    void run();                      // 运行事件循环。
};
*/


// 在构造函数中创建Epoll对象ep_。
EventLoop::EventLoop():ep_(new Epoll)                   
{

}

// 在析构函数中销毁ep_。
EventLoop::~EventLoop()
{
    delete ep_;
}

// 运行事件循环。
void EventLoop::run()                      
{
    while (true)        // 事件循环。
    {
       std::vector<Channel *> channels=ep_->loop();         // 等待监视的fd有事件发生。

        for (auto &ch:channels)
        {
            ch->handleevent();        // 处理epoll_wait()返回的事件。
        }
    }
}

// 把channel添加/更新到红黑树上，channel中有fd，也有需要监视的事件。
void EventLoop::updatechannel(Channel *ch)                        
{
    ep_->updatechannel(ch);
}
````

### 修改channel类  
其实就是之前用的epoll，现在改成eventloop的了；  
````
#pragma once
#include <sys/epoll.h>
#include <functional>
#include "EventLoop.h"
#include "InetAddress.h"
#include "Socket.h"
 
class EventLoop;

class Channel
{
private:
    int fd_=-1;                             // Channel拥有的fd，Channel和fd是一对一的关系。
    // Epoll *ep_=nullptr;                // Channel对应的红黑树，Channel与Epoll是多对一的关系，一个Channel只对应一个Epoll。
    EventLoop *loop_=nullptr;   // Channel对应的事件循环，Channel与EventLoop是多对一的关系，一个Channel只对应一个EventLoop。
    bool inepoll_=false;              // Channel是否已添加到epoll树上，如果未添加，调用epoll_ctl()的时候用EPOLL_CTL_ADD，否则用EPOLL_CTL_MOD。
    uint32_t events_=0;              // fd_需要监视的事件。listenfd和clientfd需要监视EPOLLIN，clientfd还可能需要监视EPOLLOUT。
    uint32_t revents_=0;             // fd_已发生的事件。 
    std::function<void()> readcallback_;         // fd_读事件的回调函数。

public:
    Channel(EventLoop* loop,int fd);      // 构造函数。
    ~Channel();                           // 析构函数。 

    int fd();                                            // 返回fd_成员。
    void useet();                                    // 采用边缘触发。
    void enablereading();                     // 让epoll_wait()监视fd_的读事件。
    void setinepoll();                            // 把inepoll_成员的值设置为true。
    void setrevents(uint32_t ev);         // 设置revents_成员的值为参数ev。
    bool inpoll();                                  // 返回inepoll_成员。
    uint32_t events();                           // 返回events_成员。
    uint32_t revents();                          // 返回revents_成员。 

    void handleevent();         // 事件处理函数，epoll_wait()返回的时候，执行它。

    void newconnection(Socket *servsock);    // 处理新客户端连接请求。
    void onmessage();                                     // 处理对端发送过来的消息。
    void setreadcallback(std::function<void()> fn);    // 设置fd_读事件的回调函数。
};
/////////////////////////////////////////////////////////////////
#include "Channel.h"

Channel::Channel(EventLoop* loop,int fd):loop_(loop),fd_(fd)      // 构造函数。
{

}

Channel::~Channel()                           // 析构函数。 
{
    // 在析构函数中，不要销毁loop_，也不能关闭fd_，因为这两个东西不属于Channel类，Channel类只是需要它们，使用它们而已。
}

int Channel::fd()                                            // 返回fd_成员。
{
    return fd_;
}

void Channel::useet()                                    // 采用边缘触发。
{
    events_=events_|EPOLLET;
}

void Channel::enablereading()                     // 让epoll_wait()监视fd_的读事件。
{
    events_|=EPOLLIN;
    // ep_->updatechannel(this);
    loop_->updatechannel(this);
}

void Channel::setinepoll()                           // 把inepoll_成员的值设置为true。
{
    inepoll_=true;
}

void Channel::setrevents(uint32_t ev)         // 设置revents_成员的值为参数ev。
{
    revents_=ev;
}

bool Channel::inpoll()                                  // 返回inepoll_成员。
{
    return inepoll_;
}

uint32_t Channel::events()                           // 返回events_成员。
{
    return events_;
}

uint32_t Channel::revents()                          // 返回revents_成员。
{
    return revents_;
} 

// 事件处理函数，epoll_wait()返回的时候，执行它。
void Channel::handleevent()
{
    if (revents_ & EPOLLRDHUP)                     // 对方已关闭，有些系统检测不到，可以使用EPOLLIN，recv()返回0。
    {
        printf("client(eventfd=%d) disconnected.\n",fd_);
        close(fd_);            // 关闭客户端的fd。
    }                                //  普通数据  带外数据
    else if (revents_ & (EPOLLIN|EPOLLPRI))   // 接收缓冲区中有数据可以读。
    {
        /*
        if (islisten_==true)   // 如果是listenfd有事件，表示有新的客户端连上来。
            newconnection(servsock);
        else                          // 如果是客户端连接的fd有事件。
            onmessage();
        */
        readcallback_();
    }
    else if (revents_ & EPOLLOUT)                  // 有数据需要写，暂时没有代码，以后再说。
    {
    }
    else                                                                   // 其它事件，都视为错误。
    {
        printf("client(eventfd=%d) error.\n",fd_);
        close(fd_);            // 关闭客户端的fd。
    }
}

// 处理新客户端连接请求。
void Channel::newconnection(Socket *servsock)    
{
    InetAddress clientaddr;             // 客户端的地址和协议。
    // 注意，clientsock只能new出来，不能在栈上，否则析构函数会关闭fd。
    // 还有，这里new出来的对象没有释放，这个问题以后再解决。
    Socket *clientsock=new Socket(servsock->accept(clientaddr));

    printf ("accept client(fd=%d,ip=%s,port=%d) ok.\n",clientsock->fd(),clientaddr.ip(),clientaddr.port());

    // 为新客户端连接准备读事件，并添加到epoll中。
    Channel *clientchannel=new Channel(loop_,clientsock->fd());   // 这里new出来的对象没有释放，这个问题以后再解决。
    clientchannel->setreadcallback(std::bind(&Channel::onmessage,clientchannel));
    clientchannel->useet();                 // 客户端连上来的fd采用边缘触发。
    clientchannel->enablereading();   // 让epoll_wait()监视clientchannel的读事件
}

// 处理对端发送过来的消息。
void Channel::onmessage()
{
    char buffer[1024];
    while (true)             // 由于使用非阻塞IO，一次读取buffer大小数据，直到全部的数据读取完毕。
    {    
        bzero(&buffer, sizeof(buffer));
        ssize_t nread = read(fd_, buffer, sizeof(buffer));
        if (nread > 0)      // 成功的读取到了数据。
        {
            // 把接收到的报文内容原封不动的发回去。
            printf("recv(eventfd=%d):%s\n",fd_,buffer);
            send(fd_,buffer,strlen(buffer),0);
        } 
        else if (nread == -1 && errno == EINTR) // 读取数据的时候被信号中断，继续读取。
        {  
            continue;
        } 
        else if (nread == -1 && ((errno == EAGAIN) || (errno == EWOULDBLOCK))) // 全部的数据已读取完毕。
        {
            break;
        } 
        else if (nread == 0)  // 客户端连接已断开。
        {  
            printf("client(eventfd=%d) disconnected.\n",fd_);
            close(fd_);            // 关闭客户端的fd。
            break;
        }
    }
}

 // 设置fd_读事件的回调函数。
 void Channel::setreadcallback(std::function<void()> fn)    
 {
    readcallback_=fn;
 }
````



# 封装TcpServer类  
tcpserver封装了服务端创建socket套接字，对套接字进行一系列的属性设置，以及运行事件循环的功能。  

### TcpServer头文件  
````
#pragma once 
#include"EventLoop.h"
#include"Socket.h"
#include"Channel.h"

class TcpServer
{
private:
    EventLoop loop_;    //一个TcpServer可以有多个事件循环，现在是单线程，暂时只用一个事件循环
public:
    TcpServer(const std::string &ip,const uint16_t port);
    ~TcpServer();
    void start();
};
````

### TcpServer源文件  
````
#include "TcpServer.h"

/*
class TcpServer
{
private:
    EventLoop loop_;         // 一个TcpServer可以有多个事件循环，现在是单线程，暂时只用一个事件循环。
public:
    TcpServer(const std::string &ip,const uint16_t port);
    ~TcpServer();
};*/

TcpServer::TcpServer(const std::string &ip,const uint16_t port)
{
    Socket *servsock=new Socket(createnonblocking());   // 这里new出来的对象没有释放，以后再说。
    InetAddress servaddr(ip,port);             // 服务端的地址和协议。
    servsock->setreuseaddr(true);
    servsock->settcpnodelay(true);
    servsock->setreuseport(true);
    servsock->setkeepalive(true);
    servsock->bind(servaddr);
    servsock->listen();

    Channel *servchannel=new Channel(&loop_,servsock->fd());       // 这里new出来的对象没有释放，这个问题以后再解决。
    servchannel->setreadcallback(std::bind(&Channel::newconnection,servchannel,servsock));
    servchannel->enablereading();       // 让epoll_wait()监视servchannel的读事件。
}

TcpServer::~TcpServer()
{

}

// 运行事件循环。
void TcpServer::start()          
{
    loop_.run();
}
````

### Makefile
````
all:client tcpepoll

client:client.cpp
	g++ -g client.cpp -o client

tcpepoll:tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp Channel.cpp EventLoop.cpp TcpServer.cpp
	g++ -g tcpepoll.cpp InetAddress.cpp Socket.cpp Epoll.cpp Channel.cpp EventLoop.cpp TcpServer.cpp -o tcpepoll

clean:
	rm -f client tcpepoll
````

### 修改服务程序  
````
/*
 *此程序用于演示epoll模型实现网络通信服务端
 */

#include"TcpServer.h"


int main(int argc,char* argv[])
{
    
    if(argc!=3)
    {
        printf("usage: ./tcpepoll ip port\n");
        printf("example: ./tcpepoll 192.168.157.128 5005\n");
        return -1;
    }

    TcpServer tcpserver(argv[1],atoi(argv[2]));

    tcpserver.start();

    return 0;
}

````

# 封装Acceptor类  
**WHY：**
* channel封装了监听fd和服务端连上来的fd
* 监听的fd和服务端连上来的fd的功能是不同的
* 监听的fd生命周期是永久的，连上来的客户端fd生命周期是有限的


Acceptor类是封装在channel之上的，Connection也是；  
Acceptor类和Connection都是TcpServer类管理的；  
Acceptor是负责监听的类；  

### Acceptor类头文件  
````
#pragma once
#include <functional>
#include "Socket.h"
#include "InetAddress.h"
#include "Channel.h"
#include "EventLoop.h"

class Acceptor
{
private:
    EventLoop *loop_;               // Acceptor对应的事件循环，在构造函数中传入。 
    Socket *servsock_;               // 服务端用于监听的socket，在构造函数中创建。
    Channel *acceptchannel_;   // Acceptor对应的channel，在构造函数中创建。
public:
    Acceptor(EventLoop *loop,const std::string &ip,const uint16_t port);
    ~Acceptor();
};

````

### Acceptor源头文件  
`````
#include "Acceptor.h"

Acceptor::Acceptor(EventLoop *loop,const std::string &ip,const uint16_t port):loop_(loop)
{
    servsock_=new Socket(createnonblocking());   
    InetAddress servaddr(ip,port);             // 服务端的地址和协议。
    servsock_->setreuseaddr(true);
    servsock_->settcpnodelay(true);
    servsock_->setreuseport(true);
    servsock_->setkeepalive(true);
    servsock_->bind(servaddr);
    servsock_->listen();

    acceptchannel_=new Channel(loop_,servsock_->fd());       
    acceptchannel_->setreadcallback(std::bind(&Channel::newconnection,acceptchannel_,servsock_));
    acceptchannel_->enablereading();       // 让epoll_wait()监视servchannel的读事件。 
}

Acceptor::~Acceptor()
{
    delete servsock_;
    delete acceptchannel_;
}
`````

### 修改TcpServer类  
````
#pragma once
#include "EventLoop.h"
#include "Socket.h"
#include "Channel.h"
#include "Acceptor.h"

// TCP网络服务类。
class TcpServer
{
private:
    EventLoop loop_;         // 一个TcpServer可以有多个事件循环，现在是单线程，暂时只用一个事件循环。
    Acceptor *acceptor_;   // 一个TcpServer只有一个Acceptor对象。
public:
    TcpServer(const std::string &ip,const uint16_t port);
    ~TcpServer();

    void start();          // 运行事件循环。
};
/////////////////////////////////////////////////////
#include "TcpServer.h"

TcpServer::TcpServer(const std::string &ip,const uint16_t port)
{
    /*
    Socket *servsock=new Socket(createnonblocking());   // 这里new出来的对象没有释放，以后再说。
    InetAddress servaddr(ip,port);             // 服务端的地址和协议。
    servsock->setreuseaddr(true);
    servsock->settcpnodelay(true);
    servsock->setreuseport(true);
    servsock->setkeepalive(true);
    servsock->bind(servaddr);
    servsock->listen();

    Channel *servchannel=new Channel(&loop_,servsock->fd());       // 这里new出来的对象没有释放，这个问题以后再解决。
    servchannel->setreadcallback(std::bind(&Channel::newconnection,servchannel,servsock));
    servchannel->enablereading();       // 让epoll_wait()监视servchannel的读事件。 
    */
    acceptor_=new Acceptor(&loop_,ip,port);
}

TcpServer::~TcpServer()
{
    delete acceptor_;
}

// 运行事件循环。
void TcpServer::start()          
{
    loop_.run();
}
````


# 封装Connection类  
* Connection类负责连接上来的fd，对应的channel
* Connection类负责将脸上来的fd放到对应eventloop

### Connection类头文件
````
#pragma once
#include <functional>
#include "Socket.h"
#include "InetAddress.h"
#include "Channel.h"
#include "EventLoop.h"

class Connection
{
private:
    EventLoop *loop_;               // Connection对应的事件循环，在构造函数中传入。 
    Socket *clientsock_;             // 与客户端通讯的Socket。
    Channel *clientchannel_;     // Connection对应的channel，在构造函数中创建。
public:
    Connection(EventLoop *loop,Socket *clientsock);
    ~Connection();
};

````

### Connection类源文件
````
#include "Connection.h"


Connection::Connection(EventLoop *loop,Socket *clientsock):loop_(loop),clientsock_(clientsock)
{
    // 为新客户端连接准备读事件，并添加到epoll中。
    clientchannel_=new Channel(loop_,clientsock_->fd());   
    clientchannel_->setreadcallback(std::bind(&Channel::onmessage,clientchannel_));
    clientchannel_->useet();                 // 客户端连上来的fd采用边缘触发。
    clientchannel_->enablereading();   // 让epoll_wait()监视clientchannel的读事件
}

Connection::~Connection()
{
    delete clientsock_;
    delete clientchannel_;
}
````

### 修改channel类  
`````
#include "Channel.h"

Channel::Channel(EventLoop* loop,int fd):loop_(loop),fd_(fd)      // 构造函数。
{

}

Channel::~Channel()                           // 析构函数。 
{
    // 在析构函数中，不要销毁loop_，也不能关闭fd_，因为这两个东西不属于Channel类，Channel类只是需要它们，使用它们而已。
}

int Channel::fd()                                            // 返回fd_成员。
{
    return fd_;
}

void Channel::useet()                                    // 采用边缘触发。
{
    events_=events_|EPOLLET;
}

void Channel::enablereading()                     // 让epoll_wait()监视fd_的读事件。
{
    events_|=EPOLLIN;
    loop_->updatechannel(this);
}

void Channel::setinepoll()                           // 把inepoll_成员的值设置为true。
{
    inepoll_=true;
}

void Channel::setrevents(uint32_t ev)         // 设置revents_成员的值为参数ev。
{
    revents_=ev;
}

bool Channel::inpoll()                                  // 返回inepoll_成员。
{
    return inepoll_;
}

uint32_t Channel::events()                           // 返回events_成员。
{
    return events_;
}

uint32_t Channel::revents()                          // 返回revents_成员。
{
    return revents_;
} 

// 事件处理函数，epoll_wait()返回的时候，执行它。
void Channel::handleevent()
{
    if (revents_ & EPOLLRDHUP)                     // 对方已关闭，有些系统检测不到，可以使用EPOLLIN，recv()返回0。
    {
        printf("client(eventfd=%d) disconnected.\n",fd_);
        close(fd_);            // 关闭客户端的fd。
    }                                //  普通数据  带外数据
    else if (revents_ & (EPOLLIN|EPOLLPRI))   // 接收缓冲区中有数据可以读。
    {
        readcallback_();
    }
    else if (revents_ & EPOLLOUT)                  // 有数据需要写，暂时没有代码，以后再说。
    {
    }
    else                                                                   // 其它事件，都视为错误。
    {
        printf("client(eventfd=%d) error.\n",fd_);
        close(fd_);            // 关闭客户端的fd。
    }
}

#include "Connection.h"

// 处理新客户端连接请求。
void Channel::newconnection(Socket *servsock)    
{
    InetAddress clientaddr;             // 客户端的地址和协议。
    // 注意，clientsock只能new出来，不能在栈上，否则析构函数会关闭fd。
    // 还有，这里new出来的对象没有释放，将在Connection类的析构函数中释放。
    Socket *clientsock=new Socket(servsock->accept(clientaddr));

    printf ("accept client(fd=%d,ip=%s,port=%d) ok.\n",clientsock->fd(),clientaddr.ip(),clientaddr.port());

    /*
    // 为新客户端连接准备读事件，并添加到epoll中。
    Channel *clientchannel=new Channel(loop_,clientsock->fd());   // 这里new出来的对象没有释放，这个问题以后再解决。
    clientchannel->setreadcallback(std::bind(&Channel::onmessage,clientchannel));
    clientchannel->useet();                 // 客户端连上来的fd采用边缘触发。
    clientchannel->enablereading();   // 让epoll_wait()监视clientchannel的读事件
    */
    Connection *conn=new Connection(loop_,clientsock);   // 这里new出来的对象没有释放，这个问题以后再解决。
}

// 处理对端发送过来的消息。
void Channel::onmessage()
{
    char buffer[1024];
    while (true)             // 由于使用非阻塞IO，一次读取buffer大小数据，直到全部的数据读取完毕。
    {    
        bzero(&buffer, sizeof(buffer));
        ssize_t nread = read(fd_, buffer, sizeof(buffer));
        if (nread > 0)      // 成功的读取到了数据。
        {
            // 把接收到的报文内容原封不动的发回去。
            printf("recv(eventfd=%d):%s\n",fd_,buffer);
            send(fd_,buffer,strlen(buffer),0);
        } 
        else if (nread == -1 && errno == EINTR) // 读取数据的时候被信号中断，继续读取。
        {  
            continue;
        } 
        else if (nread == -1 && ((errno == EAGAIN) || (errno == EWOULDBLOCK))) // 全部的数据已读取完毕。
        {
            break;
        } 
        else if (nread == 0)  // 客户端连接已断开。
        {  
            printf("client(eventfd=%d) disconnected.\n",fd_);
            close(fd_);            // 关闭客户端的fd。
            break;
        }
    }
}

 // 设置fd_读事件的回调函数。
 void Channel::setreadcallback(std::function<void()> fn)    
 {
    readcallback_=fn;
 }
`````

# 在Channel类中回调Acceptor类的成员函数

# 在Acceptor类中回调TcpServer类的成员函数


# buffer类  
![image](https://ryanaqu.github.io/img/inpost_cpp/image9.png)

![image](https://ryanaqu.github.io/img/inpost_cpp/image10.png)

![image](https://ryanaqu.github.io/img/inpost_cpp/image11.png)


# 目前的回调关系  
![image](https://ryanaqu.github.io/img/inpost_cpp/image12.png) 

# 多线程设计  
![image](https://ryanaqu.github.io/img/inpost_cpp/image13.png)  

# 增加工作线程  
![image](https://ryanaqu.github.io/img/inpost_cpp/image14.png)  


