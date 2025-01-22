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

### 客户端  
使用g++对demo1进行编译，然后使用bash命令查看服务端的ip和端口，利用./demo1 IP 端口号 运行程序  

````
  +--------------------------+
  |     创建套接字 (socket)    |
  +--------------------------+
               |
               v
  +--------------------------+
  | 设置目标服务端地址 (connect)|
  +--------------------------+
               |
               v
  +--------------------------+
  |     发送数据 (send)       |
  +--------------------------+
               |
               v
  +--------------------------+
  |     接收数据 (recv)       |
  +--------------------------+
               |
               v
  +--------------------------+
  |       关闭连接 (close)    |
  +--------------------------+

````

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
  +--------------------------+
  |     创建套接字 (socket)    |
  +--------------------------+
               |
               v
  +--------------------------+
  |    设置地址并绑定 (bind)   |
  +--------------------------+
               |
               v
  +--------------------------+
  |   监听连接请求 (listen)   |
  +--------------------------+
               |
               v
  +--------------------------+
  |   接受客户端连接 (accept) |
  +--------------------------+
               |
               v
  +--------------------------+
  |    读写数据 (read/write)  |
  +--------------------------+
               |
               v
  +--------------------------+
  |       关闭连接 (close)    |
  +--------------------------+

````

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

**文件描述符：**  
在Linux内一切操作都是文件操作，利用ps -ef | grep demo3命令找到进程号，然后去根目录下的proc文件目录下找到对应进程号，其也是个文件目录，在其目录下有个fd文件目录，进入fd用ls即可以看到进程打开的文件描述符  
````
ps -ef | grep demo3  //譬如10370
cd /proc/10370/fd
ls
````
一般来说，每个进程都会打开0，1，2三个文件，其中0是标准输入（键盘cin），1是标准输出（显示器cout），2是标准错误（显示器cerr）。  
如果在程序中用close()函数关掉这三个文件，那么将无法执行相应的标准输入输出错误功能。  
所以sockfd其本质也是一个文件描述符，在对它使用send()函数和recv()函数时，可以等价替换为write()和read()函数。  

**socket()创建的sockfd也是个文件，返回的是文件描述符！！！**

 
# socket函数详解  
网络编程的每个细节都很重要，不能一知半解！！！  

### 创建socket  
套接字（Socket）是操作系统提供的用于网络通信的抽象层。网络编程的基本概念包括：  
* IP 地址：标识网络中每个设备的地址。
* 端口号：标识设备上某个特定的服务或进程。
* 套接字：提供应用程序与操作系统内核之间的接口，用于发送和接收数据。

socket函数原型：  
````
int socket(int domain, int type, int protocol);  //失败返回值-1，错误代码被设置
````

* domain：指定协议族，常用的地址族有AF_INET（IPv4网络协议）和AF_INET6（IPv6网络协议）。
* type：指定套接字类型，常用的类型有SOCK_STREAM（流式套接字，对应TCP协议）和SOCK_DGRAM（数据报套接字，对应UDP协议）。
* protocol：指定协议，常用的协议有IPPROTO_TCP（TCP传输协议）和IPPROTO_UDP（UDP传输协议）。通常情况下，如果只有一种协议支持特定的地址族和套接字类型，该参数可以设置为0，系统会自动选择合适的协议。

协议族：  
* AF_INET (IPv4) - 适用于 IPv4 地址  
* AF_INET6 (IPv6) - 适用于 IPv6 地址  
* AF_UNIX (Unix Domain Sockets) - 适用于本地进程间通信  
* AF_PACKET - 适用于链路层通信

套接字类型：
* SOCK_STREAM：面向连接的套接字，通常用于 TCP 协议，提供可靠的字节流服务。  
* SOCK_DGRAM：无连接的套接字，通常用于 UDP 协议，提供不可靠的消息传输。  
* SOCK_RAW：原始套接字，允许访问低层协议（如 IP、ICMP 等），常用于网络监控和分析。  
* SOCK_SEQPACKET：提供可靠的数据包传输，不同于流套接字，数据包有明确的边界，常用于某些特定协议。  

![image](https://github.com/user-attachments/assets/5c2d6351-b81b-4578-a507-b15cde28ec23)

套接字的创建与使用：  
* 使用 socket() 创建套接字。
* 使用 bind() 将套接字绑定到一个端口（服务器端）。
* 使用 connect() 建立与远程主机的连接（客户端）。
* 使用 listen() 设置套接字为监听状态，等待客户端连接（服务器端）。
* accept() 接受客户端连接（服务器端）。
* send() 和 recv() 发送和接收数据。
* close() 关闭套接字。

### 套接字类型  
* SOCK_STREAM：面向连接的套接字，通常用于 TCP 协议。它提供可靠的字节流服务。
* SOCK_DGRAM：无连接的套接字，通常用于 UDP 协议。它提供不可靠的数据包传输。
* SOCK_RAW：原始套接字，允许应用程序访问底层协议（如 ICMP）。

### TCP/UDP协议  
TCP（Transmission Control Protocol）  
* 面向连接的协议。
* 可靠性：保证数据完整性，确保数据按顺序到达。
* 适用于需要高可靠性的数据传输，如网页浏览、文件传输等。
* 需要三次握手建立连接，四次挥手关闭连接。

UDP（User Datagram Protocol）
* 无连接的协议。
* 不保证数据可靠性，数据包可能丢失、乱序或重复。
* 适用于实时性要求较高、不需要保证可靠性的应用，如视频流、语音通信等。

### TCP 连接的三次握手与四次挥手  
三次握手：  
1. 客户端发送 SYN 请求连接。  
2. 服务器响应 SYN-ACK。
3. 客户端发送 ACK，连接建立成功。  

四次挥手：
1. 客户端发送 FIN，表示关闭连接。  
2. 服务器响应 ACK，表示收到关闭请求。  
3. 服务器发送 FIN，表示准备关闭连接。  
4. 客户端响应 ACK，连接完全关闭。  

# 主机字节序和网络字节序  
### 大端序和小端序  
在大端序中，数据的高位字节存储在低地址，而数据的低位字节存储在高地址。即，最重要的字节存储在最前面。  
````
地址:   0x00    0x01    0x02    0x03
数据:   0x12    0x34    0x56    0x78
````

在小端序中，数据的低位字节存储在低地址，而数据的高位字节存储在高地址。即，最不重要的字节存储在最前面。  
````
地址:   0x00    0x01    0x02    0x03
数据:   0x78    0x56    0x34    0x12
````

### 不同平台的字节序  
不同的硬件平台有不同的字节序：  
* x86 架构 (Intel): 采用 小端序（Little-Endian）。  
* ARM 架构: 默认采用 小端序，但也可以切换到 大端序。  
* PowerPC: 可选择大端序或小端序，取决于硬件和操作系统的配置。  

### 字节序转换  
在网络通信中，尤其是在 TCP/IP 协议栈中，标准规定使用 大端序 进行数据传输。所以，不同的系统在进行网络通信时，通常需要进行 字节序转换，即：  
* htonl (host to network long): 将主机字节序转换为网络字节序（大端序）。 //4字节整数uint32_t  
* htons (host to network short): 将主机字节序转换为网络字节序（大端序）。//2字节整数uint16_t
* ntohl (network to host long): 将网络字节序转换为主机字节序。  
* ntohs (network to host short): 将网络字节序转换为主机字节序。  

# 网络通信的结构体  
### sockaddr结构体  
struct sockaddr 是一个用于表示地址的结构体，它是各种协议族（如 IPv4、IPv6 和 UNIX 域套接字）的基础结构体。sockaddr 结构体是一个通用的套接字地址结构体，其他协议族（如 sockaddr_in、sockaddr_in6）都可以看作是从它派生出来的。**客户端的connect()函数和服务端的bind()函数都会用到这个结构体。**  
````
struct sockaddr {
    sa_family_t sa_family;   // 地址族 (protocol family),定义了地址的类型，例如 AF_INET（IPv4），AF_INET6（IPv6）
    char sa_data[14];        // 套接字地址的具体数据,存储地址信息（如 IP 地址、端口号等），其长度为 14 字节。对于不同协议族，该字段的具体内容和格式不同。
};
````

* sockaddr 结构体通常作为函数参数来表示地址信息，尤其是函数如 bind(), connect(), accept(), sendto(), recvfrom() 等都需要将 sockaddr 类型作为参数。
* 在实际使用中，通常会使用具体协议族的派生结构体（如 sockaddr_in 或 sockaddr_un）来填充数据，而 sockaddr 本身通常作为通用地址容器使用。


### IPv4 地址结构体 sockaddr_in  
````
struct sockaddr_in {
    sa_family_t sin_family;    // 地址族，通常为 AF_INET
    in_port_t sin_port;        // 16 位端口号，使用网络字节顺序（大端字节序），需要使用 htons() 函数进行转换。
    struct in_addr sin_addr;   // IPv4 地址，通常使用 inet_pton() 或 inet_addr() 来填充。
    char sin_zero[8];          // 填充字段，通常不会使用，但它的存在是为了与 sockaddr 结构体大小对齐。
};
````

### gethostbyname()函数  
函数原型：  
````
struct hostent* gethostbyname(const char *hostname);
````

gethostbyname 返回一个 struct hostent* 类型的指针，表示查询结果。struct hostent 中包含了该主机的多个信息，如主机名、别名、IP 地址等。如果查询失败，返回 NULL，并且可以通过 h_errno 获取错误码。  

gethostbyname 返回的 hostent 结构体包含了关于主机的信息：  
````
struct hostent {
    char  *h_name;       // 主机的正式名称
    char  **h_aliases;   // 主机的别名列表
    int    h_addrtype;   // 地址类型，通常是 AF_INET（IPv4）或 AF_INET6（IPv6）
    int    h_length;     // 地址长度（IPv4 是 4 字节，IPv6 是 16 字节）
    char  **h_addr_list; // 主机的 IP 地址列表
};

````

# 封装socket成类

### 封装客户端程序成ctcpclient类  
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


class ctcpclient
{
public:
    int m_clientfd;
    string m_ip;
    unsigned short m_port;

    ctcpclient()
    {
        m_clientfd = -1;
    }
    ~ctcpclient()
    {
        m_close();
    }

    //向服务端发起连接请求的成员函数，成功返回true，失败返回false
    bool m_Connect(const string & in_ip,const unsigned short in_port)
    {
        if(m_clientfd!=-1)
        {
            //如果socket已经连接，直接返回失败
            return false;
        }
        m_ip=in_ip;
        m_port=in_port;
        
        //step1:create socket for client
        m_clientfd = socket(AF_INET,SOCK_STREAM,0);
        if(m_clientfd==-1)
        {
            return false;
        }
        //step2:send a request to the server
        struct sockaddr_in servaddr;//存放服务端IP和端口的结构体
        memset(&servaddr,0,sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        servaddr.sin_port=htons(m_port);//指定服务端的通信端口

        struct hostent* h;  //存放服务端IP的结构体
        if((h=gethostbyname(m_ip.c_str()))==0)//用于存放服务端IP和端口的结构体
        {
            ::close(m_clientfd);
            m_clientfd=-1;
            return false;
        }
        memcpy(&servaddr.sin_addr,h->h_addr,h->h_length);//指定服务端的IP地址
        if(connect(m_clientfd,(struct sockaddr*)&servaddr,sizeof(servaddr))!=0)//向服务端发起请求连接
        {
            ::close(m_clientfd);
            m_clientfd=-1;
            return false;
        }
        return true;
    }
    //发送报文的成员函数
    bool m_Send(const string &buffer)
    {
        if(m_clientfd==-1) return false;
        //向服务端发送请求报文
        if(::send(m_clientfd,buffer.data(),buffer.size(),0)<=0)
        {
            return false;
        }
        return true;
    }
    //接收服务端的报文的成员函数
    bool m_Receive(string & buffer,const size_t maxlen)
    {
        buffer.clear();
        buffer.resize(maxlen);
        int iret = ::recv(m_clientfd,&buffer[0],buffer.size(),0);
        if(iret<=0)
        {
            buffer.clear();
            return false;
        }
        buffer.resize(iret);
        return true;
    }
    //断开与服务端的连接
    bool m_Close()
    {
        if(m_clientfd==-1) return false;
        ::close(m_clientfd);
        m_clientfd=-1;
        return true;
    }
    
};

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

    ctcpclient tcpclient;
    if(tcpclient.m_Connect(argv[1],atoi(argv[2]))==false)
    {
        perror("connect()");
        return -1;
    }


    string buffer;
    for(int i=0;i<3;++i)//循环三次，与服务端进行三次通信。
    {
        int iret;
        buffer="这是第"+to_string(i+1)+"个报文，编号"+to_string(i+1)+"。";//生成报文内容
        if(tcpclient.m_Send(buffer)==0)
        {
            perror("send");
            break;
        }

        cout<<"发送:"<<buffer<<endl;
        //接受服务端的回应报文，如果服务端没有发送回应报文，recv()函数将阻塞等待
        if(tcpclient.m_Receive(buffer,1024)==false)
        {
            perror("recv()");
            break;
        }
        cout<<"接收:"<<buffer<<endl;
        sleep(1);

    }
    //step4:关闭socket，释放资源
    tcpclient.m_Close();
    return 0;
}
````

### 封装服务端程序成ctcpserver类  
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

class ctcpserver
{
public:
    int m_listenfd; //监听socket，-1表示未初始化
    int m_clientfd; //客户端连上来的socket，-1表示客户端未连接
    string m_clientip; //客户端字符串格式的IP
    unsigned short m_port; //服务端用于通信的端口
public:
    ctcpserver():m_listenfd(-1),m_clientfd(-1)
    {

    }
    
    //初始化用于监听的socket  
    bool initserver(const unsigned short in_port)
    {
        //第1步：创建服务端的socket
        if((m_listenfd=socket(AF_INET,SOCK_STREAM,0))==-1)
        {
            return false;
        }
        m_port=in_port;
        //第2步：把服务端用于通信的IP和端口绑定到socket上
        struct sockaddr_in servaddr;   //用于存放服务端IP和端口的数据结构。
        memset(&servaddr,0,sizeof(servaddr));
        servaddr.sin_family = AF_INET; //指定协议
        servaddr.sin_addr.s_addr=htonl(INADDR_ANY);//服务端任意网卡的IP都可以用于通讯。
        servaddr.sin_port=htons(m_port);   //指定通信端口，普通用户只能使用1024以上的端口。

        //绑定服务端的IP和端口   
        if(bind(m_listenfd,(struct sockaddr *)&servaddr,sizeof(servaddr))==-1)
        {
            close(m_listenfd);
            m_listenfd=-1;
            return false;
        }

        //step3:把socket设置为可连接的状态
        if(listen(m_listenfd,5)==-1)
        {
            close(m_listenfd);
            m_listenfd=-1;
            return false;
        }
        return true;
    }
    //受理客户的连接（从已连接的客户端中取出一个客户端）
    //如果没有已连接的客户端，accept()函数将阻塞等待
    bool m_Accept()
    {
        struct sockaddr_in caddr;//客户端地址信息
        socklen_t addrlen=sizeof(caddr); //struct sockaddr_in的大小 
        
        if((m_clientfd=::accept(m_listenfd,(struct  sockaddr *)&caddr,&addrlen))==-1)
        {
            return false;
        }
        m_clientip=inet_ntoa(caddr.sin_addr); //把客户端的地址从大端序转换成字符串

        return true;
    }
    //获取客户端的IP（字符串格式）
    const string & m_Clientip() const
    {
        return m_clientip;
    }
    //向服务端发送报文，成功返回true，失败返回false
    bool m_Send(const string & buffer)
    {
        if(m_clientfd==-1)
        {          
            return false;
        }
        if((::send(m_clientfd,buffer.data(),buffer.size(),0))<=0)
        {
            return false;
        }
        return true;
    }
    //接收报文，成功返回true，失败返回false
    //buffer存放接收报文内容，maxlen为本次接收报文的最大长度
    bool m_Receive(string &buffer,const size_t maxlen)
    {
        buffer.clear();
        buffer.resize(maxlen);
        int readn =::recv(m_clientfd,&buffer[0],buffer.size(),0);
        if(readn<=0)
        {
            buffer.clear();
            return false;
        }
        buffer.resize(readn);

        return true;
    }
    //关闭监听的socket
    bool m_CloseListen()
    {
        if(m_listenfd==-1)
        {
            return false;
        }
        ::close(m_listenfd);
        m_listenfd=-1;
        return true;
    }
    //关闭客户端连接的socket
    bool m_CloseClient()
    {
        if(m_clientfd==-1)
        {
            return false;
        }
        ::close(m_clientfd);
        m_clientfd=-1;
        return true;
    }

    ~ctcpserver()
    {
        m_CloseListen();
        m_CloseClient();
    }
};


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

    //创建tcp
    
    ctcpserver tcpserver;
    
    if(tcpserver.initserver(atoi(argv[1]))==false)
    {
        perror("initserver()");
        return -1;
    }

    //受理客户端的连接请求，如果没有客户端连接，accept()函数将阻塞等待。
    if(tcpserver.m_Accept()==false)
    {
        perror("accept()");
        return -1;
    }
    cout<<"客户端已连接("<<tcpserver.m_Clientip()<<")\n";


    //与客户端进行通信，接受客户端发送过来的报文后，回复ok。
    string buffer;
    while(true)
    {
        //接受客户端的请求报文，如果客户端没有发送请求报文，recv()函数将阻塞等待。
        //如果客户端已断开链接，revc()将返回0.
        
        if((tcpserver.m_Receive(buffer,1024))==false)
        {
            
            perror("recv()");
            break;
        }
        cout<<"接收:"<<buffer<<endl;
        buffer="ok";
        //向客户端发送回应报文
        if(tcpserver.m_Send(buffer)==false)
        {
            perror("send");
            break;
        }
        cout<<"发送:"<<buffer<<endl;
        
    }

    return 0;
}

````

# 多进程的socket服务端  
![image](https://github.com/user-attachments/assets/ea9bd7a4-7178-4293-abe9-377b5df9c545)

其实就是之前的多进程fork()，每当多一个客户端请求通信，父进程就会fork出一个子进程进行受理；父进程只监听不受理，子进程只受理不监听。所以只有父进程使用listenfd，会将clientfd关闭；子进程使用clientfd，会将listenfd关闭。同时，为了防止僵尸进程的产生，父进程关闭前会kill掉所有子进程。  

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
#include<signal.h>
using namespace std;

/*
 * demo8.cpp,多进程的socket通信的服务端
 */

class ctcpserver
{
public:
    int m_listenfd; //监听socket，-1表示未初始化
    int m_clientfd; //客户端连上来的socket，-1表示客户端未连接
    string m_clientip; //客户端字符串格式的IP
    unsigned short m_port; //服务端用于通信的端口
public:
    ctcpserver():m_listenfd(-1),m_clientfd(-1)
    {

    }
    
    //初始化用于监听的socket  
    bool initserver(const unsigned short in_port)
    {
        //第1步：创建服务端的socket
        if((m_listenfd=socket(AF_INET,SOCK_STREAM,0))==-1)
        {
            return false;
        }
        m_port=in_port;
        //第2步：把服务端用于通信的IP和端口绑定到socket上
        struct sockaddr_in servaddr;   //用于存放服务端IP和端口的数据结构。
        memset(&servaddr,0,sizeof(servaddr));
        servaddr.sin_family = AF_INET; //指定协议
        servaddr.sin_addr.s_addr=htonl(INADDR_ANY);//服务端任意网卡的IP都可以用于通讯。
        servaddr.sin_port=htons(m_port);   //指定通信端口，普通用户只能使用1024以上的端口。

        //绑定服务端的IP和端口   
        if(bind(m_listenfd,(struct sockaddr *)&servaddr,sizeof(servaddr))==-1)
        {
            close(m_listenfd);
            m_listenfd=-1;
            return false;
        }

        //step3:把socket设置为可连接的状态
        if(listen(m_listenfd,5)==-1)
        {
            close(m_listenfd);
            m_listenfd=-1;
            return false;
        }
        return true;
    }
    //受理客户的连接（从已连接的客户端中取出一个客户端）
    //如果没有已连接的客户端，accept()函数将阻塞等待
    bool m_Accept()
    {
        struct sockaddr_in caddr;//客户端地址信息
        socklen_t addrlen=sizeof(caddr); //struct sockaddr_in的大小 
        
        if((m_clientfd=::accept(m_listenfd,(struct  sockaddr *)&caddr,&addrlen))==-1)
        {
            return false;
        }
        m_clientip=inet_ntoa(caddr.sin_addr); //把客户端的地址从大端序转换成字符串

        return true;
    }
    //获取客户端的IP（字符串格式）
    const string & m_Clientip() const
    {
        return m_clientip;
    }
    //向服务端发送报文，成功返回true，失败返回false
    bool m_Send(const string & buffer)
    {
        if(m_clientfd==-1)
        {          
            return false;
        }
        if((::send(m_clientfd,buffer.data(),buffer.size(),0))<=0)
        {
            return false;
        }
        return true;
    }
    //接收报文，成功返回true，失败返回false
    //buffer存放接收报文内容，maxlen为本次接收报文的最大长度
    bool m_Receive(string &buffer,const size_t maxlen)
    {
        buffer.clear();
        buffer.resize(maxlen);
        int readn =::recv(m_clientfd,&buffer[0],buffer.size(),0);
        if(readn<=0)
        {
            buffer.clear();
            return false;
        }
        buffer.resize(readn);

        return true;
    }
    //关闭监听的socket
    bool m_CloseListen()
    {
        if(m_listenfd==-1)
        {
            return false;
        }
        ::close(m_listenfd);
        m_listenfd=-1;
        return true;
    }
    //关闭客户端连接的socket
    bool m_CloseClient()
    {
        if(m_clientfd==-1)
        {
            return false;
        }
        ::close(m_clientfd);
        m_clientfd=-1;
        return true;
    }

    ~ctcpserver()
    {
        m_CloseListen();
        m_CloseClient();
    }
};
ctcpserver tcpserver;
void FathExit(int signum)//父进程的信号处理函数
{
    signal(SIGTERM,SIG_IGN);
    signal(SIGINT,SIG_IGN);//防止被打断

    cout<<"父进程退出，sig="<<signum<<endl;
    kill(0,SIGTERM);//向子进程发送15信号，通知退出
    //释放资源
    tcpserver.m_CloseListen();
    exit(0);
}
void ChldExit(int signum)//子进程的信号处理函数
{
    signal(SIGTERM,SIG_IGN);
    signal(SIGINT,SIG_IGN);//防止被打断
    cout<<"子进程"<<getpid()<<"退出，sig="<<signum<<endl;
    tcpserver.m_CloseClient();
    exit(0);
}


int main(int argc,char* argv[])
{
    if(argc!=2)
    {
        cout<<"Using ./demo2 通讯端口\nExample:./demo2 5005\n\n";//端口必须大于1024,不与其他的重复。
        cout<<"注意：运行服务端程序的Linux系统的防火墙必须要开通5005端口。\n";
        cout<<"   如果是云服务器，还要开通云平台的访问策略。\n\n";
        return -1;
    }

    //忽略全部信号，不希望被打扰
    for(int i=1;i<=64;i++)
    {
        signal(i,SIG_IGN);
    }
    //设置sell状态下可用kill进程号或ctrl+c正常终止进程
    signal(SIGTERM,FathExit);
    signal(SIGINT,FathExit);

    //创建tcp
    
    if(tcpserver.initserver(atoi(argv[1]))==false)
    {
        perror("initserver()");
        return -1;
    }

    while(true)
    {
        //受理客户端的连接请求，如果没有客户端连接，accept()函数将阻塞等待。
        if(tcpserver.m_Accept()==false)
        {
            perror("accept()");
            return -1;
        }
        //每有一个客户端连接，就分裂一个子进程出来受理，父进程继续受理
        int pid=fork();
        if(pid==-1)
        {
            perror("fork()");
            return -1;//系统资源不足
        }
        if(pid>0) 
        {
            tcpserver.m_CloseClient();//父进程关闭客户端连接的socket
            continue;//父进程返回开头
        }

        tcpserver.m_CloseListen();//子进程关闭监听的socket

        //子进程重新设置信号
        signal(SIGTERM,ChldExit);
        signal(SIGINT,SIG_IGN);
        

        cout<<"客户端已连接("<<tcpserver.m_Clientip()<<")\n";

        //与客户端进行通信，接受客户端发送过来的报文后，回复ok。
        string buffer;
        while(true)
        {
            //接受客户端的请求报文，如果客户端没有发送请求报文，recv()函数将阻塞等待。
            //如果客户端已断开链接，revc()将返回0.
            
            if((tcpserver.m_Receive(buffer,1024))==false)
            {
                
                perror("recv()");
                break;
            }
            cout<<"接收:"<<buffer<<endl;
            buffer="ok";
            //向客户端发送回应报文
            if(tcpserver.m_Send(buffer)==false)
            {
                perror("send");
                break;
            }
            cout<<"发送:"<<buffer<<endl;
            
        }

    }

    return 0;
}

````









