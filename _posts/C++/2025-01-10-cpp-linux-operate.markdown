---
layout:     post
title:      "Linux 基础操作"
subtitle:   " \"learning……\""
date:       2025-01-10 19:15:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - linux
---

> 岁月本长，而忙者自促……

# Linux的时间操作  

### time_t别名
time_t是long的别名，time_t类型和long是一样的，在<time.h>中定义，表示1970.01.01 00:00:00（linux纪元）开始到现在的秒数

### time库函数  
time函数获得的是从linux纪元到现在所经历的秒数，要通过localtime库函数转化  
````
#include<time.h>
time_t time(time_t* tloc);    //time函数的申明

time_t now=time(0);           //调用方式1：空指针传给time函数，然后返回time值给now

time_t now;
time(&now);                   //调用方式2：将time_t变量地质传给time函数
````

### localtime库函数  
![image](https://github.com/user-attachments/assets/95d4eaad-bfc0-4035-9602-89b6dccb2552)

````
#include<iostream>
#include<time.h>
using namespace std;

int main()
{
    time_t now=time(0);
    cout<<"now="<<now<<endl;

    tm tmnow;
    localtime_r(&now,&tmnow); // 把整数时间转化为tm结构体
    string stime = to_string(tmnow.tm_year+1900)+"-"
                +to_string(tmnow.tm_mon+1)+"-"
                +to_string(tmnow.tm_mday)+" "
                +to_string(tmnow.tm_hour)+":"
                +to_string(tmnow.tm_min)+":"
                +to_string(tmnow.tm_sec);
    cout<<"stime="<<stime<<endl;
    return 0;
}

````


### mktime库函数  

![image](https://github.com/user-attachments/assets/d10cdc1a-d66d-4e98-857e-07b3936d4d0c)


### 程序睡眠  

![image](https://github.com/user-attachments/assets/52d4d4c3-8fef-4dd4-ba0f-87d2133c6a74)




# Linux的目录操作  
linux系统中目录的最大长度为255  

### 当前目录地址获取  
````
#include<iostream>
#include<unistd.h>
using namespace std;

int main()
{
    //方法1
    char path1[256];//linux最大路径256,开辟栈空间
    getcwd(path1,256);//获取当前路径
    cout<<"path1="<<path1<<endl;

    //方法2
    char* path2=get_current_dir_name();
    cout<<"path2="<<path2<<endl;
    free(path2);
    
    return 0;
}
````

### 切换工作目录
![image](https://github.com/user-attachments/assets/12f93cb9-f515-471b-aa55-c6b34d53f16a)

### 创建目录  
![image](https://github.com/user-attachments/assets/dc3c643c-c519-48ce-bb71-3e4bfc93b861)


### 删除目录  
![image](https://github.com/user-attachments/assets/cace5e54-6b8e-4d12-9853-10e8ef686858)

### 读取目录下文件的demo  

* 包含目录头文件<dirent.h>
* 打开文件目录，并用指针保存目录
* 读取目录下内容并输出

````
#include<iostream>
#include<dirent.h> //目录操作头文件
using namespace std;

int main(int argc,char * argv[])
{
    if(argc!=2)
    {
        cout<<"格式错误：Using ./demo 目录名\n";
        return -1;
    }
    DIR *dir;  //定义目录指针
    //打开目录
    if((dir=opendir(argv[1]))==0) return -1;
    //存放从目录中读取的内容
    struct dirent *stdinfo=nullptr;

    while(1)
    {
        if((stdinfo=readdir(dir))==0) break;
        cout<<"文件名="<<stdinfo->d_name<<"\t\t文件类型="<<(int)stdinfo->d_type<<endl;
    }
    //关闭目录
    closedir(dir);

    return 0;
}
````

# Linux中的错误信息  
在<ctring>头文件中有133个错误类型，通过strerror函数可以显示出来  
````
#include<iostream>
#include<cstring>
using namespace std;

int main()
{
    //<cstring>只有133个错误代码
    for(int i=0;i<150;i++)
    {
        cout<<i<<":"<<strerror(i)<<endl;
    }
    return 0;
}
````

关于errno的使用，只有系统调用出错才会设置errno  
````
#include<iostream>
#include<cstring>
#include<cerrno>
#include<sys/stat.h>
using namespace std;

int main()
{
    int iret=mkdir("/temp",0755);//目录权限 
    
    if(iret!=0)
    {
        cout<<"iret="<<iret<<endl;
        cout<<errno<<":"<<strerror(errno)<<endl;
        perror("调用功能失败");
    }
    
    return 0;
}
````

# 文件或者目录的stat操作  
c/c++的stat库函数相当于shell的stat命令，可以查看文件或者目录的详细信息  
````
#include<iostream>
#include<cstring>
#include<unistd.h>
#include<sys/stat.h>
#include<stdio.h>
using namespace std;

int main(int argc,char* argv[])
{
    if(argc!=2)
    {
        cout<<"Using ./test_stat 文件或者目录名"<<endl;
        return -1;
    }
    struct stat st;
    if(stat(argv[1],&st)!=0)
    {
        cout<<"stat("<<argv[1]<<"):"<<strerror(errno)<<endl; return -1;
    }
    if(S_ISREG(st.st_mode))
    {
        cout<<"argv[1]"<<"是一个文件("<<"m_time="<<st.st_mtime<<",size="<<st.st_size<<")\n";
    }
    if(S_ISDIR(st.st_mode))
    {
        cout<<"argv[1]"<<"是一个目录("<<"m_time="<<st.st_mtime<<",size="<<st.st_size<<")\n";
    }

    return 0;
}
````

# utime库函数操作 
修改文件时间，只有atime和ctime

# rename库函数操作
相当于操作系统的mv命令  

# remove库函数操作
相当于操作系统的rm命令




