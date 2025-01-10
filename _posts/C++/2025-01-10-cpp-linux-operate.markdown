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













