---
layout:     post
title:      "Linux 生产消费者模型实现"
subtitle:   " \"learning……\""
date:       2025-01-12 12:00:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - linux
    - 生产消费者模型
---

> 太年轻的人他总是不满足。

# 生产消费者模型 PV原语  
模型用循环队列+信号量（semaphore）实现。两个信号量，一个是互斥锁Mutex（用来锁共享内存），一个是条件变量condition（用来提示共享内存内有多少个数据资源）。
生产者-消费者问题是一种常见的多线程/多进程问题，通常用于解决生产和消费两者之间的协作关系。生产者负责生成数据（产品），消费者负责消费数据。它们通过共享一个缓冲区（循环队列+共享内存）来进行通信。缓冲区大小是有限的，生产者和消费者之间需要同步，以防止数据丢失或缓冲区溢出。使用互斥锁（std::mutex）来保护共享缓冲区，使用条件变量（std::condition_variable）来处理等待和通知机制。以下实现采用自定义实现，没有使用C++标准库。

使用以下命令查看和操作信号量和共享内存：  
````
ipcs -s        //查看信号量数组
ipcrm sem 信号量ID    //手动删除信号量数组
ipcs -m        //查看共享内存
ipcrm -m 共享内存ID //手动删除共享内存
````

### _public.h头文件和_public.cpp  
这个头文件包含了两个类，一个是循环队列类的实现（申明+定义）；一个是信号量类的申明，信号量类的定义在_public.cpp里面，用了一个联合体，能支持Mutex锁类型和条件变量condition类型的信号量。

_public.h：  
````
#ifndef __PUBLIC_HH 
#define __PUBLIC_HH 1

#include<iostream>
#include<cstdio>
#include<cstdlib>
#include<cstring>
#include<unistd.h>
#include<sys/ipc.h>
#include<sys/shm.h>
#include<sys/types.h>
#include<sys/sem.h>
using namespace std;

//循环队列
template<class TT,int MaxLength>
class squeue
{
private:
    bool m_inited; //队列被初始化标志，true-已初始化
    TT m_data[MaxLength];  //用数组存储循环队列中的元素
    int m_head;  //头指针
    int m_tail;  //尾指针
    int m_length;  //队列实际长度
    squeue(const squeue&)=delete;  //禁用拷贝构造函数
    squeue& operator=(const squeue&)=delete; //禁用赋值函数
public:
    squeue(){init();}  //构造函数
    //循环队列的初始化操作
    void init()
    {
        if(m_inited!=true)
        {
            m_head=0;  //头指针指向第一个元素
            m_tail=MaxLength-1; //尾指针指向最后一个元素
            m_length=0;
            memset(m_data,0,sizeof(m_data));  
            m_inited=true;
        }
    }

    //元素入队，返回值，成功-true
    bool push(const TT &ee)
    {
        if(full()==true)
        {
            cout<<"循环队列已满，入队失败"<<endl;
            return false;
        }

        //先移动队尾指针，然后再拷贝数据
        m_tail=(m_tail+1)%MaxLength;  
        m_data[m_tail]=ee;
        m_length++;
        return true;
    }

    //求循环队列长度
    int size()
    {
        return m_length;
    }
    
    //判断队列是否为空,true-空
    bool empty()
    {
        return m_length==0?true:false;
    }

    //判断循环队列是否已满，true-已满
    bool full()
    {
        return m_length==MaxLength?true:false;
    }

    //查看队头元素
    TT& front()
    {
        return m_data[m_head];
    }

    //元素出队，返回值：true-成功
    bool pop()
    {
        if(empty()==true)  return false;
        m_head=(m_head+1)%MaxLength; //头指针后移
        m_length--;
        return true;
    }

    //显示循环队列中全部的元素
    void printqueue()
    {
        for(int ii=0;ii<m_length;ii++)
        {
            cout<<"m_data["<<(m_head+ii)%MaxLength<<"],value="\
                <<m_data[(m_head+ii)%MaxLength]<<endl;
        }
    }
};

//信号量
class csemp
{
private:
    union semun  //用于信号量操作的共同体
    {
        int val;
        struct semid_ds *buf;
        unsigned short *arrt;
    };

    int m_semid;  //信号量id(描述符)
    short m_sem_flg; 
    //用于互斥锁 设置为SEM_UNDO
    //用于生产消费者模型，设置为0,此时操作系统不会将信号量恢复成初始值

    csemp(const csemp&)=delete;  //禁用拷贝构造函数
    csemp& operator=(const csemp&)=delete; //禁用赋值函数
public:
    csemp():m_semid(-1){}
  
    //用于互斥锁 value=1，sem_flg=SEM_UNDO
    //用于生产消费者模型 value=0,short sem_flg=0
    bool init(key_t key,unsigned short value=1,short sem_flg=SEM_UNDO);
    bool wait(short sem_op=-1);  //信号量的P操作 将信号量减一
    bool post(short sem_op=1);  //信号量的V操作，将信号量加一
    int getvalue();   //获取信号量的值，成功返回其值，失败返回-1
    bool destory();   //销毁信号量
    ~csemp(){}
};

#endif
````

_public.cpp：  
````
#include"_public.h"

//linux 中信号量函数 semget(),semop(),semctl()
bool csemp::init(key_t key,unsigned short value,short sem_flg)
{
    if(m_semid!=-1)  return false;  //如果已经初始化了，不必再次初始化

    m_sem_flg=sem_flg;

    //获取信号量
    if((m_semid=semget(key,1,0666))==-1)
    {
        //如何信号量不存在，创建它
        if(errno==ENOENT)
        {
            //用IPC_EXCL标志确保只有一个进程创建并初始化信号量，其他进程只能获取
            if((m_semid=semget(key,1,0666|IPC_CREAT|IPC_EXCL))==-1)
            {
                if(errno==EEXIST) //如果错误代码是信号量已存在，则再次获取信号量
                {
                    if((m_semid=semget(key,1,0666))==-1)
                    {
                        perror("init 1 semget()");
                        return false;
                    }
                    return true;
                }
                else //如果是其他错误，返回失败
                {
                    perror("init 2 semget()");
                    return false;                    
                }

            }
            //初始化
            union semun sem_union;
            sem_union.val=value; //设置信号量的初始值
            if(semctl(m_semid,0,SETVAL,sem_union)<0)
            {
                perror("init semctl()");
                return false;
            }

        }

        else
        {
            perror("init 3 semget()");
            return false;
        }

    }

    return true;
}

//信号量的P操作（把信号量的值减去value）
bool csemp::wait(short value)
{
    if(m_semid==-1) return false;
    struct sembuf sem_b;
    sem_b.sem_num=0;  //信号量编号，0代表第一个信号量
    sem_b.sem_op=value;  //p操作的value必须小于0
    sem_b.sem_flg=m_sem_flg;
    if(semop(m_semid,&sem_b,1)==-1)
    {
        perror("P semop()");
        return false;
    }
    return true;
}

//信号量的V操作（将信号量的值加上value）
bool csemp::post(short value)
{
    if(m_semid==-1) return false;
    struct sembuf sem_b;
    sem_b.sem_num=0;  //信号量编号，0代表第一个信号量
    sem_b.sem_op=value;  //p操作的value必须大于0
    sem_b.sem_flg=m_sem_flg;
    if(semop(m_semid,&sem_b,1)==-1)
    {
        perror("V semop()");
        return false;
    }
    return true;
}

//获取信号量的值，成功返回信号量的值，失败返回-1
int csemp::getvalue()
{
    return semctl(m_semid,0,GETVAL);
}

//销毁信号量
bool csemp::destory()
{
    if(m_semid==-1) return false;
    if(semctl(m_semid,0,IPC_RMID)==-1)
    {
        perror("destory semctl()");
        return false;
    }
    return true;
}
````

### 生产者incache.cpp  
生产者包含_public.h头文件，生产数据时会将共享内存加锁，然后在共享内存的循环队列里面加数据，加入成功就会把条件变量＋1.生产完成会解锁共享内存，然后把共享内存从程序中剥离。

````
//多进程的生产消费者模型的生产者程序
#include"_public.h"

int main()
{
    struct stgirl //循环队列的数据元素是超女结构体
    {
    int no; //编号
    char name[51]; //姓名 
    };

    using ElemType=stgirl;
    
    //初始化共享内存
    int shmid=shmget(0x5005,sizeof(squeue<ElemType,5>),0640|IPC_CREAT);
    if(shmid==-1)
    {
        cout<<"shmget(0x5005)failed"<<endl;
        return -1;
    }

    ///把共享内存连接到当前进程的地址空间
    squeue<ElemType,5>*QQ=(squeue<ElemType,5>*)shmat(shmid,0,0);
    if(QQ==(void*)-1)
    {
        cout<<"shmat()failed"<<endl;
        return -1;
    }

    QQ->init(); //初始化循环队列
    ElemType ee; //创建一个数据元素

    csemp mutex; mutex.init(0x5001);  //用于给共享内存加锁 使用默认参数
    csemp cond; cond.init(0x5002,0,0);  //信号量的值用于表示队列中数据元素的个数

    mutex.wait();  //加锁
    //生产3个数据
    ee.no=3; strcpy(ee.name,"西施"); if(QQ->push(ee)) cond.post(1);//push成功了才会加信号量
    ee.no=7; strcpy(ee.name,"冰冰"); if(QQ->push(ee)) cond.post(1);
    ee.no=8; strcpy(ee.name,"幂幂"); if(QQ->push(ee)) cond.post(1);
    mutex.post(); //解锁

    shmdt(QQ);  //把共享内存从当前进程中分离

}
````

### 消费者outcache.cpp  
消费者包含_public.h头文件，可以有多个消费者“共同消费”队列数据。不会剥离共享内存，只要程序在运行，就会循环查询，不断解锁->查询->加锁，直到有数据就消费数据。解锁共享内存（消费数据需要时间）。

````
//多进程的生产消费者模型的消费者程序
#include"_public.h"

int main()
{
    struct stgirl //循环队列的数据元素是超女结构体
    {
    int no; //编号
    char name[51]; //姓名
    };

    using ElemType=stgirl;

    //初始化共享内存
    int shmid=shmget(0x5005,sizeof(squeue<ElemType,5>),0640|IPC_CREAT);
    if(shmid==-1)
    {
        cout<<"shmget(0x5005)failed"<<endl;
        return -1;
    }

    ///把共享内存连接到当前进程的地址空间
    squeue<ElemType,5>*QQ=(squeue<ElemType,5>*)shmat(shmid,0,0);
    if(QQ==(void*)-1)
    {
        cout<<"shmat()failed"<<endl;
        return -1;
    }

    QQ->init();  //初始化循环队列

    ElemType ee;  //创建一个数据元素

    csemp mutex; mutex.init(0x5001);  //用于给共享内存加锁
    csemp cond; cond.init(0x5002,0,0);  //信号量的值用于表示队列中的数据元素的个数

    while(true) //消费者程序的核心流程
    {
        mutex.wait(); //加锁
        while(QQ->empty())  //如何队列空，进入循环，否则直接处理数据
        {
            mutex.post(); //解锁
            cond.wait(); //等待生产者的唤醒信号
            mutex.wait();  //加锁
        }

        //数据元素出队
        ee=QQ->front(); QQ->pop();
        mutex.post();

        //处理出队的数据（把数据消费掉）
        cout<<"no="<<ee.no<<",name="<<ee.name<<endl;
        usleep(100); //处理数据需要的时间
    }

}
````

### 获取信号量的程序getsemvalue.cpp  
使用方法：  
````
ipcs -s //查询信号量的key
./getsemvalue <key>
````

getsemvalue.cpp:
````
#include <sys/ipc.h>
#include <sys/sem.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <key>\n", argv[0]);
        return 1;
    }

    // 从命令行参数获取信号量的 key
    key_t key = (key_t) strtol(argv[1], NULL, 0);  // 转换为 key_t 类型

    // 获取信号量集的标识符
    int semid = semget(key, 1, 0);
    if (semid == -1) {
        perror("semget");
        return 1;
    }

    // 获取信号量的当前值
    int val = semctl(semid, 0, GETVAL);
    if (val == -1) {
        perror("semctl GETVAL");
        return 1;
    }

    printf("Semaphore value for key %x: %d\n", key, val); // 输出信号量值
    return 0;
}
````

### 项目makefile  
````
all:incache outcache getsemvalue

getsemvalue: getsemvalue.cpp
	g++ getsemvalue.cpp -o getsemvalue

incache: incache.o _public.o
	g++ incache.o _public.o -o incache

outcache: outcache.o _public.o
	g++ outcache.o _public.o -o outcache

_public.o: _public.h _public.cpp
	g++ -c _public.cpp -o _public.o

incache.o: incache.cpp _public.h
	g++ -c incache.cpp -o incache.o

outcache.o: outcache.cpp _public.h
	g++ -c outcache.cpp -o outcache.o

clean: 
	rm _public.o incache.o outcache.o
````





