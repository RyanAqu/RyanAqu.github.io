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
[C++ 11 新特性 lambda表达式](https://ryanaqu.github.io/2025/02/01/cpp-lambda/)  
[C++ 11 新特性 函数封装器和绑定器](https://ryanaqu.github.io/2025/02/01/cpp-function/)  
[C++ 设计模式](https://ryanaqu.github.io/2025/02/20/cpp-designpattern/)  

# 基础算法&数据结构  
## 算法  
[贪心算法](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-greedy/)  
[千奇百怪的排序算法](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-sort/)  
[双指针](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-twopointers/)  
[二分查找](https://ryanaqu.github.io/2025/01/06/cpp-algorithm-binarysearch/)  
[搜索算法](https://ryanaqu.github.io/2025/01/06/cpp-algorithm-search/)  
[回溯算法](https://ryanaqu.github.io/2025/01/07/cpp-algorithm-backtrack/)  
[动态规划](https://ryanaqu.github.io/2025/01/08/cpp-algorithm-dp/)  
[置换环](https://ryanaqu.github.io/2025/01/08/cpp-algorithm-sloop/)  

## 数据结构  
[二叉树和搜索树](https://ryanaqu.github.io/2025/01/09/cpp-datastruct-binarytree/)  
[堆和优先级队列](https://ryanaqu.github.io/2025/01/09/cpp-datastruct-heap/)  
[并查集](https://ryanaqu.github.io/2025/01/09/cpp-datastruct-unionfind/)  
[Trie前缀树](https://ryanaqu.github.io/2025/01/09/cpp-datastruct-trie/)  
[LRU最近最少使用](https://ryanaqu.github.io/2025/01/09/cpp-datastruct-lru/)  
[数据结构与基础算法](https://ryanaqu.github.io/2025/01/05/cpp-algorithm-datastructure/)  

# Linux 操作系统基础  
[CSDN上一篇关于APUE/内核编程的优质文章](https://blog.csdn.net/weixin_36750623/article/details/127408987)  
[Linux 下 Bash 命令基础](https://ryanaqu.github.io/2024/12/15/cpp-linux/)  
[Linux 下 g++ 编译基础和Makefile](https://ryanaqu.github.io/2025/01/10/cpp-compile/)  
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
[Linux 下 Reactor 高并发服务器](https://ryanaqu.github.io/2025/03/01/cpp-reactor/)  


# 基于RIS-V指令集架构的Xv6操作系统内核优化  
本项目以MIT6.S081课程为向导,基于RIS-V指令集架构的xv6操作系统为基础,对其系统调用,内存管理,进程管理,中断,文件系统等模块进行优化和扩展  

### 学习资料和环境搭建  
[一个比较好的学习资料](https://xv6.dgs.zone/)  
[Lab0：实验配置 by CSDN 上一篇关于Ubuntu环境下搭建实验环境的文章](https://blog.csdn.net/LostUnravel/article/details/120397168)  
[Lab0：实验配置 by 知乎 上一篇关于Windows环境下搭建实验环境的文章](https://zhuanlan.zhihu.com/p/449687883)  

### 版本控制和GDB
建议每个实验checkout到相应的分支，然后创建相应的测试分支，完成后没有问题在进行合并merge（也可以不合并）  
**注意：** 在切换分支的时候一定要跟踪远程分支，否则Git 不知道要从哪个远程分支拉取更新，因此需要手动指定  
````
git checkout util      # 切换到util分支（不跟踪）
git checkout -b util origin/util  # 切换到util分支（并跟踪）
git checkout -b util_test # 建立并切换到util的测试分支


git add .
git commit -m "完成了第一个作业"
git checkout util
git merge util_test
git push github util:util

git branch   //查看分支
````

启动GDB调试：  
````
make qemu        //直接make
ctrl+a->松开->x  //退出xv6系统
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

### Lab代码仓库  
fork了实验原始main分支，然后完成所有main下分支的lab，每一个完成的lab都在对应的“分支名+_test”的分支中。   

[Xv6操作系统个人实验代码仓库（更新中...）](https://github.com/RyanAqu/MIT6.S081-2020-labs)  

[Lab1：Makefile、基本系统调用、多进程编程、文件描述符、IPC管道、sleep、find、pingpong、primes、xargs](https://github.com/RyanAqu/MIT6.S081-2020-labs/tree/util_test)  
总结：学习了一些操作系统接口，用系统调用编写了一些实用的用户级程序。  

[Lab2：系统资源抽象（CPU抽象（线程、调度器）、内存抽象（虚拟内存、分页）、存储抽象（文件系统、块设备）、I/O抽象（设备驱动）、网络抽象（socket、VPN、SDN））；强隔离性（内核态用户态M/S/U）；内核组织（微内核、宏内核、混合内核、客户/服务器模式）；Xv6架构（模块化内核）；Xv6进程管理和隔离机制（进程抽象（独立地址空间、时间片、IPC、并发）、进程状态（运行、就绪、阻塞、等待、退出）、进程信息（页表、内核、寄存器状态））;trace、sysinfo](https://github.com/RyanAqu/MIT6.S081-2020-labs/tree/syscall_test)  
总结：学习了操作系统架构和进程，学习了系统调用是怎么进入的 ~~（usys.pl脚本使用entry函数自动生成汇编（交叉编译？），譬如fork会通过entry中的li a7, SYS_fork语句把 SYS_fork（即 fork 的系统调用号）加载到 a7 寄存器）~~ ~~（user.h暴露系统调用的接口，usy.pl将系统调用变成汇编并转到特定寄存器）~~，向xv6添加了一些新的系统调用，扩展了Xv6操作系统。  

[Lab3：内存映射（虚拟地址、物理地址、页内偏移、分页硬件）；内存管理MMU（进程页表、用户地址空间、内核地址空间、进程地址空间的独立性）；sbrk、exec；]() 





