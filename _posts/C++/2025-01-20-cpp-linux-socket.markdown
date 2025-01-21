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

# 基于 Linux 的文件操作  
Linux一切皆文件，socket操作与文件操作没有区别。所以在网络传输数据的过程中，可以使用文件I/O函数。  
但实际开发中，一般使用C++流来操作文件，底层文件I/O操作大概理解就行。  

### C++流文件操作  
简单易用，面向对象的接口使得代码更具可读性和可维护性。自动管理缓冲区，减少了手动处理的麻烦。提供格式化的输入输出（如 <<, >> 运算符）。但是性能较低，尤其在需要大量I/O操作时。不够灵活，无法细粒度控制底层文件操作。  

````
#include <iostream>
#include <fstream>
#include <string>

int main() {
    // 写文件
    std::ofstream outfile("example.txt");
    if (outfile.is_open()) {
        outfile << "Hello, World!" << std::endl;
        outfile.close();
    }

    // 读文件
    std::ifstream infile("example.txt");
    std::string line;
    if (infile.is_open()) {
        while (std::getline(infile, line)) {
            std::cout << line << std::endl;
        }
        infile.close();
    }
    return 0;
}

````

### Linux底层I/O文件操作  
Linux底层I/O操作通过系统调用实现，直接与操作系统的内核交互。这些操作是基于文件描述符的，通常使用 open(), read(), write(), close() 等系统调用来进行文件操作。  

**创建和写文件**：  
````
//demo3.cpp:本程序演示linux底层文件操作-创建文件并写入数据
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<fcntl.h>
#include<unistd.h>
using namespace std;

int main()
{
    int fd;//定义一个文件描述符/文件句柄
    char buffer[1024];

    fd=open("data.txt",O_CREAT|O_RDWR|O_TRUNC);//打开文件
    if(fd==-1)
    {
        perror("open(data.txt)");
        return -1;
    }
    printf("文件描述符fd=%d\n",fd);

    strcpy(buffer,"hi\n");
    if(write(fd,buffer,strlen(buffer))==-1)
    {
        perror("write()");
        return -1;
    }
    close(fd);
    return 0;
}
````

**读操作**：  
````
//demo4.cpp:本程序演示linux底层文件操作-读取操作
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<fcntl.h>
#include<unistd.h>
using namespace std;


int main()
{
    int fd;//第年一一个文件描述符/文件句柄
    fd=open("data.txt",O_RDONLY); //打开文件
    if(fd==-1)
    {
        perror("open(data.txt)");return -1;
    }
    printf("文件描述符=%d\n",fd);
    char buffer[1024];
    memset(buffer,0,sizeof(buffer));

    if(read(fd,buffer,sizeof(buffer))==-1)
    {
        perror("read()");
        return -1;
    }
    printf("%s",buffer);

    close(fd);// 关闭文件
    return 0;
}

````

**文件描述符**  
在Linux内一切操作都是文件操作，利用ps -ef | grep demo3命令找到进程号，然后去根目录下的proc文件目录下找到对应进程号，其也是个文件目录，在其目录下有个fd文件目录，进入fd用ls即可以看到进程打开的文件描述符  
````
ps -ef | grep demo3  //譬如10370
cd /proc/10370/fd
ls
````
一般来说，每个进程都会打开0，1，2三个文件，其中0是标准输入（键盘cin），1是标准输出（显示器cout），2是标准错误（显示器cerr）。  
如果在程序中用close()函数关掉这三个文件，那么将无法执行相应的标准输入输出错误功能。  
所以sockfd其本质也是一个文件描述符，在对它使用send()函数和recv()函数时，可以等价替换为write()和read()函数。  

**socket()创建的也是个文件，返回的是文件描述符！！！**









