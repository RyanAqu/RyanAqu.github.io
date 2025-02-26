---
layout:     post
title:      "C++ Catalog"
subtitle:   " \"learning……\""
date:       3024-11-14 20:06:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 基础算法
    - catalog
---

> 生活破破烂烂，小猫尝尝咸淡

这是一篇C++学习的笔记汇总帖，边复习边记录。  
* 绝大部分是hands-on实操写代码。
* 一部分是knowledge知识&问答。
* 一点点有用的reference参考链接。
* 全部的markdown编写。
* 有的note可能有点“巨长”。


# C++  
[C++ 学习路线和就业方向](https://ryanaqu.github.io/2024/11/15/cpp-careerdirection/)  
[C Basic & C With Classes](https://ryanaqu.github.io/2024/12/01/cpp-basic-class/)  
[C++ 泛型 & STL 标准模板库](https://ryanaqu.github.io/2024/12/13/cpp-generic-stl/)  
[C++ 11新特性 lambda表达式](https://ryanaqu.github.io/2025/02/01/cpp-lambda/)  
[C++ 设计模式](https://ryanaqu.github.io/2025/02/20/cpp-designpattern/)  

# 基础算法&数据结构  
## 算法  
[贪心算法](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-greedy/)  
[千奇百怪的排序算法](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-sort/)  
[双指针](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-twopointers/)  
[二分查找](https://ryanaqu.github.io/2025/01/06/cpp-algorithm-binarysearch/)  
[搜索算法](https://ryanaqu.github.io/2025/01/06/cpp-algorithm-search/)  

## 数据结构  
[堆和优先级队列](https://ryanaqu.github.io/2025/02/22/cpp-datastruct-heap/)  
[并查集](https://ryanaqu.github.io/2025/02/22/cpp-datastruct-unionfind/)  
[数据结构与基础算法](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-datastructure/)  

# Linux 操作系统基础  
[CSDN上一篇关于APUE/内核编程的优质文章](https://blog.csdn.net/weixin_36750623/article/details/127408987)  
[Linux 下 Bash 命令基础](https://ryanaqu.github.io/2024/12/15/cpp-linux/)  
[Linux 下 g++ 编译基础和Makefile](https://ryanaqu.github.io/2025/01/02/cpp-compile/)  
[Linux 下 gdb 调试](https://ryanaqu.github.io/2025/01/10/cpp-gdb/)  
[Linux 系统操作基础](https://ryanaqu.github.io/2025/01/10/cpp-linux-operate/)  
[Linux 信号与进程](https://ryanaqu.github.io/2025/01/11/cpp-linux-signalprocess/)  
[Linux 生产消费者模型实现](https://ryanaqu.github.io/2025/01/12/cpp-linux-pcmodel/)  
[Linux Socket网络编程](https://ryanaqu.github.io/2025/01/20/cpp-linux-socket/)  
[Linux 下 TCP/IP协议](https://ryanaqu.github.io/2025/01/23/cpp-linux-netioreuse/)  
[Linux 下 select网络I/O复用模型](https://ryanaqu.github.io/2025/01/23/cpp-linux-netioreuse1/)  
[Linux 下 poll/epoll模型](https://ryanaqu.github.io/2025/01/23/cpp-linux-netioreuse2/)  
[Linux 下 MySql实操](https://ryanaqu.github.io/2025/02/17/cpp-mysql/)  
[Linux 下 Redis实操](https://ryanaqu.github.io/2025/02/18/cpp-redis/)  


# 基于RIS-V指令集架构的Xv6操作系统内核优化  
本项目以MIT6.S081课程为向导,基于RIS-V指令集架构的xv6操作系统为基础,对其系统调用,内存管理,进程管理,中断,文件系统等模块进行优化和扩展  
[一个比较好的学习资料](https://xv6.dgs.zone/)  
[Lab0：实验配置 by CSDN 上一篇关于Ubuntu环境下搭建实验环境的文章](https://blog.csdn.net/LostUnravel/article/details/120397168)  
[Lab0：实验配置 by 知乎 上一篇关于Windows环境下搭建实验环境的文章](https://zhuanlan.zhihu.com/p/449687883)  

启动GDB调试：  
````
make qemu        //直接make
make qemu-gdb    //带GDB调试的make
make qemu-gdb CPUS=1    //需要单核启动可以设置此参数
gdb-multiarch    //在另一个窗口打开gdb进行调试
有如下结果说明进入成功
0x0000000000001000 in ?? ()
````

系统运行（gdb中输入continue命令）：  
````
gdb中输入c即启动，此时Xv6操作系统可以正常使用，bt有如下结果
(gdb) bt
#0  scheduler () at kernel/proc.c:465
#1  0x0000000080000f1c in main () at kernel/main.c:44
````

GDB常规调试：
````
break main        //打断点
info register     //查看寄存器
step              //单步执行
step n            //执行n步
````










