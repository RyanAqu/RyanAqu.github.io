---
layout:     post
title:      "Linux 网络编程 Part1"
subtitle:   " \"learning……\""
date:       2025-01-20 19:29:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - linux
    - socket
    - 网络编程
---


> 我来到，我看见，我记录......

# 网络通信  
* 网络通信是指两台计算机中的程序进行传输数据的过程。  
* 客户程序：指主动发起通讯的程序。  
* 服务程序端：被动等待，然后为向他发起通讯的客户端提供服务。

# 第一个网络通信程序  
socket函数原型：  
````
int socket(int domain, int type, int protocol);
````

* domain：指定地址族，常用的地址族有AF_INET（IPv4网络协议）和AF_INET6（IPv6网络协议）。
* type：指定套接字类型，常用的类型有SOCK_STREAM（流式套接字，对应TCP协议）和SOCK_DGRAM（数据报套接字，对应UDP协议）。
* protocol：指定协议，常用的协议有IPPROTO_TCP（TCP传输协议）和IPPROTO_UDP（UDP传输协议）。通常情况下，如果只有一种协议支持特定的地址族和套接字类型，该参数可以设置为0，系统会自动选择合适的协议。  

### 客户端  
使用g++对demo1进行编译，然后使用bash命令查看服务端的ip和端口，利用./demo1 IP 端口号 运行程序  
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
 * demo1.cpp,用于演示socket通信的客户端
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
    //step3:与服务端通信，发送一个请求报文后等待回复，然后发送下一个请求报文
    char buffer[1024];
    for(int i=0;i<3;++i)//循环三次，与服务端进行三次通信。
    {
        int iret;
        memset(buffer,0,sizeof(buffer));
        sprintf(buffer,"这是第%d个报文，编号%03d",i+1,i+1);//生成报文内容
        //向服务端发送请求报文
        if((iret=send(sockfd,buffer,strlen(buffer),0))<=0)
        {
            perror("send");
            break;
        }
        cout<<"发送:"<<buffer<<endl;
        memset(buffer,0,sizeof(buffer));
        //接受服务端的回应报文，如果服务端没有发送回应报文，recv()函数将阻塞等待
        if((iret=recv(sockfd,buffer,sizeof(buffer),0))<=0)
        {
            cout<<"iret="<<iret<<endl;break;
        }
        cout<<"接收:"<<buffer<<endl;
        sleep(1);

    }
    //step4:关闭socket，释放资源
    close(sockfd);
    return 0;
}
````

### 服务端  
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
 * demo2.cpp,用于演示socket通信的服务端
 */
int main(int argc,char* argv[])
{
    if(argc!=2)
    {
        cout<<"Using ./demo2 通讯端口\nExample:./demo2 5005\n\n";//端口必须大于1024,不与其他的重复。
        cout<<"注意：运行服务端程序的Linux系统的防火墙必须要开通5005端口。\n";
        cout<<"   如果是云服务器，还要开通云平台的访问策略。\n\n";
        return -1;
    }
    //step1:创建服务端的socket
    int listenfd=socket(AF_INET,SOCK_STREAM,0);
    if(listenfd==-1)
    {
        perror("socket");
        return -1;
    }
    //step2:把服务端用于通信的IP和端口绑定到socket上。
    struct sockaddr_in servaddr;   //用于存放服务端IP和端口的数据结构。
    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family = AF_INET; //指定协议
    servaddr.sin_addr.s_addr=htonl(INADDR_ANY);//服务端任意网卡的IP都可以用于通讯。
    servaddr.sin_port=htons(atoi(argv[1]));   //指定通信端口，普通用户只能使用1024以上的端口。
    //绑定服务端的IP和端口
    if(bind(listenfd,(struct sockaddr *)&servaddr,sizeof(servaddr))!=0)
    {
        perror("bind");
        close(listenfd);
        return -1;
    }
    //step3:把socket设置为可连接的状态
    if(listen(listenfd,5)!=0)
    {
        perror("listen");
        close(listenfd);
        return -1;
    }
    //step4:受理客户端的连接请求，如果没有客户端连接，accept()函数将阻塞等待。
    int clientfd=accept(listenfd,0,0);
    if(clientfd==-1)
    {
        perror("accept");
        close(listenfd);
        return -1;
    }
    cout<<"客户端已连接。\n";
    //step5:与客户端进行通信，接受客户端发送过来的报文后，回复ok。
    char buffer[1024];
    while(true)
    {
        int iret;
        memset(buffer,0,sizeof(buffer));
        //接受客户端的请求报文，如果客户端没有发送请求报文，recv()函数将阻塞等待。
        //如果客户端已断开链接，revc()将返回0.
        if( (iret=recv(clientfd,buffer,sizeof(buffer),0))<=0 )
        {
            cout<<"iret="<<iret<<endl;
            break;
        }
        cout<<"接收:"<<buffer<<endl;

        strcpy(buffer,"ok");  //生成回应报文内容
        //向客户端发送回应报文
        if( (iret=send(clientfd,buffer,strlen(buffer),0))<=0 )
        {
            perror("send");
            break;
        }
        cout<<"发送:"<<buffer<<endl;
        
    }
    //step6:关闭socket，释放资源
    close(listenfd);
    close(clientfd);


    return 0;
}

````











