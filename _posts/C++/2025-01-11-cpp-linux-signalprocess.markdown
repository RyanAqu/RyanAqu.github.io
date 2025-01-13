---
layout:     post
title:      "Linux 信号与进程基础"
subtitle:   " \"learning……\""
date:       2025-01-11 15:00:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - linux
    - signal
    - process
---

> 天地本宽，而鄙者自隘……

# Linux的信号（Signal）基础  
在Linux中，信号（Signal）是一种用于进程间通信的机制（软件中断），允许操作系统通知进程发生了某些事件。信号可以被用于通知进程需要处理的异步事件，比如终止、暂停、继续运行等。信号在Linux和类Unix系统中广泛用于进程控制、系统管理和进程间通信。  

### 信号的基本概念  
* 信号是由内核或其他进程发送的通知，指示某个事件或状态的发生。
* 信号可以被发送到进程、进程组或线程。进程可以选择捕获信号并进行相应的处理，或者忽略它。
* 信号可以是同步的（如从程序内部触发）或异步的（如由内核或其他进程触发）。

### 信号的类型  
Linux中定义了很多信号，每个信号都有一个唯一的编号和名称。常见的信号有：  
* SIGINT（信号编号2）：由键盘中断（Ctrl+C）触发，默认行为是终止进程。
* SIGTERM（信号编号15）：终止进程的信号，通常用于正常的进程结束请求，默认行为是终止进程。
* SIGKILL（信号编号9）：**强制**终止进程的信号，无法被捕获或忽略。
* SIGSEGV（信号编号11）：段错误，通常是由于程序访问无效内存时触发。
* SIGQUIT（信号编号3）：由键盘退出（Ctrl+\）触发，默认行为是终止进程并生成核心转储。
* SIGSTOP（信号编号19）：停止进程的信号，无法被捕获、阻塞或忽略。
* SIGCONT（信号编号18）：继续一个已经被暂停的进程。
* SIGALRM（信号编号14）：定时器超时信号，通常由alarm()系统调用触发。


可以使用以下命令查看所有信号类型：  
````
kill -l
````


### 信号的发送和接收  
信号的发送可以通过以下方式：

* 由内核触发：例如，程序发生了错误（如段错误）、定时器到期等。
* 由其他进程发送：通过kill()系统调用，其他进程可以向目标进程发送信号。
* 通过命令行工具：如kill、pkill、killall等命令可以向进程发送信号。

### 信号的处理  
有三种处理方式：  
* 默认处理：每个信号都有默认的处理行为，进程可以选择忽略它（例如，SIGKILL），或者执行特定操作（如终止进程、生成核心转储等）。
* 自定义处理：通过信号处理函数（signal handler），进程可以自定义信号的处理行为。
* 忽略信号：进程可以选择忽略特定的信号（除SIGKILL和SIGSTOP外，其他信号都可以被忽略）。

以下代码定义了三种信号处理方式：  
````
#include<iostream>
#include<unistd.h>
#include<signal.h>
using namespace std;
void alarm_handler(int signum)
{
    cout<<"闹钟响了"<<endl;
    alarm(5);
}
void signal_handler(int signum)//自定义处理方式
{
    cout<<"收到了信号:"<<signum<<endl;
    signal(signum,SIG_DFL);//SIG_DFL宏将信号处理方法恢复为默认，所以第二次不会进入这个程序了
}
void bb(const int bc,const string bcn)
{
    for(int i=0;i<1000000;++i)
    {
        sleep(1);
        cout<<"i="<<i<<endl;
    }
}
void aa(const int no,const string name)
{
    bb(3,"skylar");
}
int main()
{
    signal(1, signal_handler);//回调的方法
    signal(15, signal_handler);//收到了1或者15的信号，则执行自定义的信号处理函数
    signal(2,SIG_IGN);//忽略信号2

    alarm(5);//闹钟定时器，只执行一次，在倒数5s后发送-14信号终止程序
    signal(14,alarm_handler);//自定义函数处理-14信号

    aa(8,"dylann");

    return 0;
}
````

使用以下命令对进程发送信号：
````
kill <pidof demo>
killall demo

killall -2 demo
killall -15 demo
killall -1 demo
````
2号信号会被忽略，15信号会被自定义处理函数输出，函数内将处理方法改回了默认处理方式，1信号进入后进程被终止。需要注意的是-9信号是AEF类型，它无法被捕获无法被忽略，会强制杀死程序，所以自定义函数无效。

### 信号的阻塞与解阻  
进程可以选择阻塞某些信号，阻塞的信号不会立即被处理，直到它们被解阻。例如，sigprocmask()函数可以用来阻塞或解除阻塞特定信号。

### 常用信号函数  
* signal()：设置信号处理程序，允许自定义信号的处理方式。
* kill()：向指定进程发送信号。
* raise()：向当前进程发送信号。
* sigprocmask()：阻塞或解阻特定信号。
* sigaction()：比signal()更强大的信号处理函数，提供了更多控制信号的选项。

### 信号的应用  
信号在Linux中应用广泛，尤其在进程管理和系统控制中。常见应用场景包括：  
* 进程终止：当程序运行出现错误或需要退出时，使用信号通知进程终止（如SIGTERM、SIGKILL）。
* 定时任务：定时器到期后发送信号（如SIGALRM）以执行定时操作。
* 进程暂停与恢复：通过SIGSTOP暂停进程，使用SIGCONT恢复进程。
* 进程间通信：通过信号实现简单的进程间通信机制。

### 信号的实际应用demo （有保护退出）  
在实际开发中，服务程序运行在后台，直接kill掉进程不是一个好的策略。因为程序往往占用了比较多的资源，直接kill掉是突然死亡，没有做好善后工作。如果向服务程序发送信号，程序接收到信号之后调用函数，在函数中编写善后的代码，程序就可以有计划的退出。

如果向服务程序发送-0信号，可以检测程序是否存活。

````
#include<iostream>
#include<unistd.h>
#include<signal.h>
using namespace std;

void EXIT(int signum)
{
    cout<<"收到了信号："<<signum<<endl;
    cout<<"正在释放资源，程序将退出......\n";
    //保存数据
    //关闭文件
    //断开数据库
    //断开网络连接
    //etc..
    cout<<"程序退出。"<<endl;
    exit(0);
}
int main(int argc,char* argv[])
{
    for(int i=0;i<=64;i++)
    {
        signal(i,SIG_IGN);//忽略全部信号，防止程序被信号异常终止
    }
    //如果收到2(ctrl+c)或者15(kill/killall)，主动退出
    signal(2,EXIT);
    signal(15,EXIT);

    while(1)
    {
        cout<<"执行了一次任务\n";
        sleep(1);
    }
}
````

# Linux 的进程终止
进程终止有8种方式，其中5种是正常终止  
## 5个正常终止  
### main函数中return
return 会调用全局和局部的析构，释放资源


### 使用exit()库函数  
exit() 是标准库函数，用于终止当前进程。会执行一定的清理工作，但是不会调用局部的析构函数  

````
#include <cstdlib>

int main() {
    // 终止进程，返回 0 表示成功退出，在任意位置都能直接退出
    std::exit(0);
}
````

exit可以使用atexit()函数登记终止时的自定义操作函数（最多32个），这些函数将由exit()自动调用，调用与登记顺序相反。这种方式可以在使用exit()函数的方式退出时实现清理工作  
````
atexit(func1);    //登记
````


### 使用_exit()  
_exit() 是更底层的函数，直接终止进程而不清理缓冲区

````
#include <unistd.h>

int main() {
    // 立即终止进程
    _exit(0);
}

````

### 最后一个线程从其启动例程（线程主函数）用return返回
这两种暂时不讨论，主要是涉及到线程的使用  

### 最后一个线程中调用pthread_exit()返回


## 3个异常终止  
### 使用 abort()  
abort() 会使程序异常终止并产生核心转储（core dump），便于调试。

````
#include <cstdlib>

int main() {
    // 异常终止
    std::abort();
}

````

### 信号捕获终止  

````
#include <csignal>
#include <iostream>
#include <cstdlib>

void signalHandler(int signal) {
    std::cout << "Received signal: " << signal << ". Cleaning up and exiting." << std::endl;
    std::exit(0);
}

int main() {
    // 捕获 SIGTERM 信号
    std::signal(SIGTERM, signalHandler);

    std::cout << "Process is running. Send SIGTERM to terminate." << std::endl;
    while (true) {
        // 模拟长时间运行
    }

    return 0;
}
````

### 线程对取消请求做响应  

### 总结  
* 优雅终止：建议使用 exit() 或 kill() 发送 SIGTERM 信号。
* 强制终止：使用 kill() 发送 SIGKILL 信号或 _exit()。
* 异常终止：使用 abort()。
* 信号捕获：结合 signal() 实现资源清理和退出逻辑。



# Linux 下C++程序调用其他程序（进程前置知识）  
Linux提供了system()函数和exec函数族，在C++程序中可以执行其他程序，包括二进制文件、操作系统命令或者shell脚本  

### system()函数  
在 C 和 C++ 中，system() 函数用于调用操作系统的命令解释器（shell）来执行一条命令或程序。它是标准库的一部分，定义在 <cstdlib> 或 <stdlib.h> 头文件中。 

system()执行后流程会回来  
````
#include <iostream>
#include <cstdlib>

int main() {
    int result = system("ls");
    std::cout << "Command exited with status: " << result << std::endl;
    return 0;
}
````

### exec()函数族  
exec() 函数族是 Unix/Linux 系统中用于替换当前进程镜像的一组函数。调用这些函数后，当前进程将被新程序取代，不会再回到调用的程序继续执行。因此，这些函数通常与 fork() 结合使用，用来实现进程的创建和替换。
![image](https://github.com/user-attachments/assets/dd030037-de95-4b59-a226-cd81055d2303)

##### execl()函数  
新进程的进程编号原进程相同，但是新进程取代了原进程代码段、数据段和堆栈  
````
#include <unistd.h>
#include <iostream>

int main() {
    // 替换当前进程为 `ls`，并传递参数
    execl("/bin/ls", "ls", "-l", "-a", nullptr);

    // 如果执行成功，上面代码后面的内容不会执行
    std::cerr << "Execution failed!" << std::endl;
    return 1;
}
````

##### execlp()函数  
支持搜索 PATH 环境变量，不需要提供全路径。  
````
#include <unistd.h>
#include <iostream>

int main() {
    // 使用 PATH 搜索 `ls` 命令
    execlp("ls", "ls", "-l", "-a", nullptr);

    std::cerr << "Execution failed!" << std::endl;
    return 1;
}

````

##### execlv()函数  
参数通过数组传递。  
````
#include <unistd.h>
#include <iostream>

int main() {
    char *args[] = { (char *)"ls", (char *)"-l", (char *)"-a", nullptr };
    execv("/bin/ls", args);

    std::cerr << "Execution failed!" << std::endl;
    return 1;
}

````


# Linux 进程基础  
查看进程树  
````
pstree -p 0        //查看0号进程树
pstree -p          //默认查看1号进程树
````

库函数查看进程ID  
````
getpid()        //当前进程ID
getppid()       //父进程ID
````

### 3个特殊进程  
##### 0号进程（swapper 或 idle 进程）  
* 功能  
    * 这是操作系统启动时创建的第一个进程，通常被称为 "idle process"（空闲进程）。
    * 它在没有其他任务运行时会执行，以保持 CPU 忙碌。
    * 主要用于 CPU 的资源管理，例如进入低功耗状态（C-state）。
* 特点
    * PID 为 0，是操作系统的核心部分。
    * 在现代 Linux 系统中，这部分功能通常是内核线程的一部分。
    * 永远不会被终止。

##### 1号进程（init 或 systemd）  
* 功能  
    * 是第一个用户态（user space）进程，由 0 号进程（内核）通过 fork() 和 exec() 创建.
    * 它是所有其他用户进程的祖先，并负责启动和管理系统中的其他服务和进程。
* 特点
    * PID 为 1。
    * 如果一个进程的父进程退出，其子进程会被 PID 1 的进程接管（即成为孤儿进程时由 init 或 systemd 管理）。
    * 在系统运行期间，PID 1 的进程非常重要，一旦它崩溃，系统通常会挂起或崩溃。

##### 2号进程（kthreadd）
* 功能  
    * 是内核线程管理器，负责管理和调度所有内核线程。
    * kthreadd 自身也是由 PID 0 的内核进程通过 fork() 创建的。
    * 内核中的许多后台线程都由 kthreadd 派生，例如：
        * kworker/*：处理内核任务队列。
        * ksoftirqd/*：处理软中断。
        * kswapd：负责内存换页操作。
* 特点
    * PID 为 2。
    * 运行在内核态，用户态进程无法直接与其交互。
    * 和 0 号进程一样，也不会终止。


### fork()创建进程  
使用fork()创建进程，fork()之后的内容两个进程（父进程和子进程）都会执行，但是fork()返回值不一样，子进程为0，父进程为进程ID  
````
#include<iostream>
#include<unistd.h>
using namespace std;

int main()
{
    cout<<"开始fork：\n";
    pid_t pid=fork();
    cout<<"pid="<<pid<<endl;
    return 0;
}
````

输出结果为:  
````
开始fork：
pid=0
pid=32688
````


可以用返回值判断当前进程为父还是子进程（这里用exec函数族将子进程替换成了另一个进程）  
````
pid_t pid = fork();
if (pid == 0) { // 子进程
    execlp("ls", "ls", "-l", nullptr);
} else if (pid > 0) { // 父进程
    wait(nullptr); // 等待子进程完成
}
````

注意：  
* 子进程获得的是父进程的所有拷贝副本，虽然打印变量地址可能相同，但是这只是虚拟地址，真实的物理地址不一样，不像线程    
* 创建子进程之后，两个进程谁快谁慢是不一定的  
* for()函数执行一次，但是返回值有两个
* 父进程和子进程共享一个文件偏移量，即如果打开了一个txt后fork()，fork()后写txt，那么他们写的内容可能会混合（而不是先后覆盖）

### vfork()创建进程    
vfork() 是 fork() 的一种变体，主要用于优化进程创建的性能，特别是在父进程立即调用 exec 函数时。  
![image](https://github.com/user-attachments/assets/baebf62d-fb6a-4539-82d5-c79445fc321c)


### 僵尸进程  
进程后台运行的方法有以下方法或者：父进程比子进程更早退出，子进程会被1号程序托管，即变成后台运行    
````
./demo &        //命令行运行程序时加&
````

但是，如果子进程比父进程先退出，而父进程没有处理子进程退出的信息，那么子进程将会变成**僵尸进程**，避免僵尸进程有以下三种方法  

##### 父进程调用 wait() 或 waitpid()  
* wait()：父进程可以使用 wait() 等待一个子进程的终止，回收其退出状态。
* waitpid()：类似 wait()，但 waitpid() 更灵活，允许父进程选择等待某个特定的子进程。

````
#include <iostream>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {  // 子进程
        std::cout << "Child process is running\n";
        exit(0);  // 子进程退出
    } else {  // 父进程
        wait(nullptr);  // 等待子进程退出并回收资源
        std::cout << "Parent process has collected child status\n";
    }
    return 0;
}

````

##### 使用 signal() 捕捉 SIGCHLD 信号，自动回收子进程  
父进程可以通过 signal() 捕捉 SIGCHLD 信号，当子进程退出时，父进程可以收到该信号，进而自动调用 wait() 或 waitpid() 来回收子进程的退出状态。这种方法避免了父进程主动调用 wait()，而是通过信号机制自动处理。  
````
#include <iostream>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <signal.h>

void sigchld_handler(int signum) {
    // 等待所有子进程结束并回收资源
    waitpid(-1, nullptr, WNOHANG);
}

int main() {
    signal(SIGCHLD, sigchld_handler);  // 捕捉 SIGCHLD 信号
    pid_t pid = fork();
    if (pid == 0) {  // 子进程
        std::cout << "Child process is running\n";
        exit(0);  // 子进程退出
    } else {  // 父进程
        // 父进程可以继续做自己的事情
        std::cout << "Parent process is running\n";
        sleep(5);  // 父进程休眠一段时间
    }
    return 0;
}

````

##### 子进程的父进程设置为init 1号进程  
孤儿进程回收，不常用  


# 多进程与信号  
在Linux中，可以用命令kill或者killall杀死进程，在程序中，可以用kill()函数来实现信号发送：int kill(pid_t pid,int signum);  
pid：目标进程的进程 ID（PID）。如果 pid 是负数，表示发送信号给指定组中的所有进程；如果 pid 为 0，表示发送信号给当前进程的所有同一进程组的进程；如果 pid 为 -1，则发送信号给所有进程（除了进程 1 和当前进程）。pid 也可以是特定的进程 ID。  
**应用：** 在多进程服务程序中，如果子进程收到退出信号，子进程自行退出；如果父进程收到信号，应该向所有子进程发送退出信号，然后自己再退出。  

以下代码父进程每10秒创建一个子进程，子进程每三秒打印输出进程号正在运行，使用ps -ef|grep demo查看所有父子进程，使用kill+子进程号可以杀死子进程，使用kill+父进程号可以杀死所有父子进程  
````
#include<iostream>
#include<unistd.h>
#include<signal.h>
using namespace std;

void FathExit(int signum)
{
    //防止信号处理过程中再次被中断，以进行有效的不被打断的退出
    signal(SIGINT,SIG_IGN);
    signal(SIGTERM,SIG_IGN);

    cout<<"父进程退出，sig="<<signum<<endl;
    kill(0,SIGTERM);//kill所有子进程

    //释放资源
    exit(0);
}

void ChildEXIT(int signum)
{
    //防止信号处理过程中再次被中断
    signal(SIGINT,SIG_IGN);
    signal(SIGTERM,SIG_IGN);

    cout<<"子进程"<<getpid()<<"退出，sig="<<signum<<endl;
    //释放子进程资源
    exit(0);
}

int main()
{
    for(int i=1;i<=64;++i)
    {
        signal(i,SIG_IGN);
    }
    signal(SIGTERM,FathExit);//15
    signal(SIGINT,FathExit);//2

    while(1)
    {
        if(fork()>0)
        {
            sleep(10);continue;//每10秒生成一个子进程

        }
        else
        {
            signal(SIGTERM,ChildEXIT);
            signal(SIGINT,SIG_IGN);
            while(1)
            {
                cout<<"子进程"<<getpid()<<"正在运行中。\n";
                sleep(3);
                continue;
            }
        }
    }

    return 0;
}
````


# 共享内存  
在多进程和多线程的编程中，共享内存是一个重要的概念。它使得多个进程或线程能够访问和修改同一块内存区域。不同的是，多进程和多线程在共享内存时的方式和机制有所不同。

### 进程间通信和共享内存  
进程间通信（IPC）：每个进程在操作系统中都运行在独立的地址空间内，它们通常不能直接访问彼此的内存。为了解决这个问题，操作系统提供了进程间通信（IPC）机制，允许不同进程之间进行数据交换。常见的 IPC 机制包括： 

* 管道（pipe）
* 消息队列（message queue）
* 信号量（semaphore）
* 套接字（socket）
* 共享内存（shared memory）

共享内存是一种效率较高的进程间通信方式，多个进程（不需要派生关系）可以直接访问同一块物理内存。这通常通过操作系统提供的共享内存区域实现。在共享内存的实现中，多个进程需要通过一些同步机制（如信号量、互斥锁等）来避免数据冲突。

### 共享内存的创建  
````
#include<iostream>
#include<unistd.h>
#include<signal.h>
#include<sys/shm.h>
using namespace std;

//本程序创建了一块共享内存
struct stgirl
{
    int no;
    char name[51];//注意不能用string
};

int main(int argc, char*argv[])
{
    //第1步，创建/获取共享内存，键值key为0x5005,也可以用其他值
    int shmid=shmget(0x5005,sizeof(stgirl),0640|IPC_CREAT);
    if(shmid==-1)
    {
        cout<<"shmget(0x5005) failed.\n";
        return -1;
    }
    cout<<"shmid="<<shmid<<endl;
    return 0;
}

````

用以下命令查看共享内存  
````
ipcs -m        //查看共享内存
ipcrm -m 共享内存ID //手动删除共享内存
````

### 共享内存的使用  
为了使用创建好的共享内存，需要使用shmat函数把共享内存连接到当前进程的地址空间。  
然后使用指针进行直接使用。  
使用后如果短时间内不再使用，就断开连接，释放资源。  
如果共享内存不再需要，就对其进行删除。  

````
#include<iostream>
#include<unistd.h>
#include<signal.h>
#include<sys/shm.h>
#include<cstring>
using namespace std;

//本程序创建了一块共享内存,链接并且使用
struct stgirl
{
    int no;
    char name[51];//注意不能用string
};

int main(int argc, char*argv[])
{
    if(argc!=3)
    {
        cout<<"Using ./test_sharedmemory age(int) name(string)\n";
        return -1;
    }
    //第1步，创建/获取共享内存，键值key为0x5005,也可以用其他值
    int shmid=shmget(0x5005,sizeof(stgirl),0640|IPC_CREAT);
    if(shmid==-1)
    {
        cout<<"shmget(0x5005) failed.\n";
        return -1;
    }
    cout<<"shmid="<<shmid<<endl;

    //第2步，把共享内存连接到当前进程的地址空间
    stgirl * ptr = (stgirl * )shmat(shmid,0,0);
    if(ptr==(void*)-1)
    {
        cout<<"shmat failed\n";
        return -1;
    }

    //第3步，使用共享内存，对内存进行读/写
    cout<<"原值：no="<<ptr->no<<".name="<<ptr->name<<endl;
    ptr->no=atoi(argv[1]);
    strcpy(ptr->name,argv[2]);
    cout<<"新值：no="<<ptr->no<<".name="<<ptr->name<<endl;
  
    //第4步，把共享内存从当前进程中分离
    shmdt(ptr);

    //第5步，如果共享内存不再使用，使用以下代码删除
    if(shmctl(shmid,IPC_RMID,0)==-1)
    {
        cout<<"shmctl failed\n";
        return -1;
    }

    return 0;
}

````

注意：  
* 共享内存不能自动扩展，只能使用C++内置的数据类型  
* 共享内存不能使用STL容器，也不能使用移动语义  
* 共享内存在使用时，前一个共享内存一定要删掉？


# 信号量  
信号量（Semaphore）是一种用于控制对共享资源的访问的同步工具，广泛应用于多进程和多线程的编程中。信号量由一个计数器、一个进程或线程队列以及一组操作组成。信号量可以帮助我们在并发环境中控制资源的访问，避免数据竞态、死锁等问题。 

### 信号量的基本原理和核心操作  
信号量通过维护一个整数值来管理对共享资源的访问：  
* 当信号量的值大于 0 时，进程/线程可以访问共享资源。  
* 当信号量的值为 0 时，进程/线程需要等待，直到其他进程/线程释放资源，信号量的值增加，才可以访问共享资源。  

信号量的核心操作包括：  
* P 操作（等待操作，或称为减操作）：将信号量的值减 1。如果信号量的值为 0，进程/线程会被阻塞，直到信号量值大于 0。  
* V 操作（释放操作，或称为加操作）：将信号量的值加 1。如果有进程/线程因为信号量值为 0 被阻塞，V 操作会唤醒一个等待的进程/线程。  

### 信号量的应用场景  
* 互斥：使用二值信号量来确保某些共享资源在同一时刻只能被一个进程/线程访问。信号量值为 1 表示资源可用，值为 0 表示资源已被占用。  
* 进程/线程同步：信号量用于协调多个进程或线程的执行顺序。例如，线程 A 完成某项任务后，线程 B 才能开始执行。  
* 资源管理：例如，线程池、数据库、连接池等，其中信号量的值表示可用资源的数量，进程/线程在访问这些资源时需要通过信号量来同步。
* 例如生产消费者模型中用了两个信号量，一个共享内存（循环队列），一个信号量作为互斥锁，一个作为条件变量的资源数量。


### 信号量类创建以及使用  
````
#include<iostream>
#include<unistd.h>
#include<signal.h>
#include<sys/shm.h>
#include<sys/sem.h>
#include<cstring>
using namespace std;

//本程序创建了一块共享内存,链接并且使用，并创建了一个信号量互斥锁

// 信号量类
class csemp
{
    private:
        union semun // 用于信号量操作的共同体
        {
            int val;
            struct semid_ds *buf;
            unsigned short *arry;
        };
        
        int m_semid; // 信号量id（描述符）
        
        /*
        如果把sem _flg设置为SEM_UNDO，操作系统将跟踪进程对信号量的修改情况，
        在全部修改过信号量的进程（正常或异常）终止后，操作系统将把信号量恢
        复为初始值（就像撤消了全部进程对信号的操作)。
        如果信号量用于互斥锁，设置为SEM_UNDO。
        如果信号量用于生产消费者模型，设置为0。
        */
        short m_sem_flg;
        
        csemp(const csemp &) = delete; // 禁用拷贝构造函数
        csemp &operator = (const csemp &) = delete; // 禁用赋值函数
        
    public:
        csemp(): m_semid(-1) {} // 构造函数，初始化信号量的id为-1，表示信号量没有被初始化，已经初始化了id >= 0。
        /*
        如果信号量已存在，获取信号量;如果信号量不存在，则创建它并初始化为
        valueo如果用于互斥锁，value填1，sem_flg填SEM_UNDO。
        如果用于生产消费者模型，value填0，sem_flg填0。
        */
        bool init(key_t key, unsigned short value = 1, short sem_flg = SEM_UNDO);
        bool wait(short sem_op = -1); // 信号量的P操作
        bool post(short sem_op = 1); // 信号量的V操作
        int getvalue(); // 获取信号量的值，成功返回信号量的值，失败返回-1
        bool destroy(); // 销毁信号量
        ~csemp();      
};

// 信号量的一些函数
bool csemp::init(key_t key, unsigned short value, short sem_flg)
{
    if(m_semid != -1) // 如果初始化了，则不必再初始化
        return false;
    m_sem_flg = sem_flg;
    /*
    信号量的初始化不能直接用semget(key,1,0666|IPC_CREAT)
    因为信号量创建后，初始值是0，如果用于互斥锁，需要把它的初始值设置为1，
    而获取信号量则不需要设置初始值，所以，创建信号量和获取信号量的流程不同。
    信号量的初始化分三个步骤:
    1)获取信号量，如果成功，函数返回。
    2)如果失败，则创建信号量。
    3)设置信号量的初始值。
    */
    
    //获取信号量
    if((m_semid = semget(key, 1, 0666)) == -1)
    {
        if(errno == ENOENT) // 如果信号量不存在
        {
            if((m_semid = semget(key, 1, 0666 | IPC_CREAT | IPC_EXCL)) == -1) // 用IPC_EXCL标志确保只有一个进程创建
                                                                              // 并初始化信号量，其他进程只能获取。
            {
                if(errno == EEXIST) // 如果错误代码是信号量已存在，则再次获取
                {
                    if((m_semid = semget(key, 1, 0666)) == -1) // 再次获取信号量
                    {
                        perror("init 1 semget()");
                        return false;
                    }
                    return true;
                }
                else
                {
                    perror("init 2semget()");
                    return false;
                }
            }
            // 运行下面几行说明，信号量创建成功，需要初始化成value
            union semun sem_union;
            sem_union.val = value;
            if(semctl(m_semid, 0, SETVAL, sem_union) < 0)
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

// 信号量的P操作，wait函数（把信号量的值加value），如果信号量的值为0，将阻塞等待，直到信号量的值大于0.
bool csemp::wait(short value)
{
    if(m_semid == -1)
        return false;
    
    struct sembuf sem_b;
    sem_b.sem_num = 0; // 信号量的编号，0代表第一个信号量
    sem_b.sem_op = value; // P操作的value 必须小于0
    sem_b.sem_flg = m_sem_flg;
    if(semop(m_semid, &sem_b, 1) == -1)
    {
        perror("P semop()");
        return false;
    }
    return true;
}

// 信号量的V操作（把信号量的值加value）
bool csemp::post(short value)
{
    if(m_semid == -1)
        return false;
    
    struct sembuf sem_b;
    sem_b.sem_num = 0; // 信号量的编号，0代表第一个信号量
    sem_b.sem_op = value; // V操作的value 必须大于0
    sem_b.sem_flg = m_sem_flg;
    if(semop(m_semid, &sem_b, 1) == -1)
    {
        perror("V semop()");
        return false;
    }
    return true;
}

// 获取信号量的值，成功返回值，否则返回-1
int csemp::getvalue()
{
    return semctl(m_semid, 0, GETVAL);
}

// 销毁信号量
bool csemp::destroy()
{
    if(m_semid == -1)
        return false;
    if(semctl(m_semid, 0, IPC_RMID) == -1)
    {
        perror("destroy semctl()");
        return false;
    }
    return true;
}



struct stgirl
{
    int no;
    char name[51];//注意不能用string
};

int main(int argc, char*argv[])
{
    if(argc!=3)
    {
        cout<<"Using ./test_sharedmemory age(int) name(string)\n";
        return -1;
    }
    //第1步，创建/获取共享内存，键值key为0x5005,也可以用其他值
    int shmid=shmget(0x5005,sizeof(stgirl),0640|IPC_CREAT);
    if(shmid==-1)
    {
        cout<<"shmget(0x5005) failed.\n";
        return -1;
    }
    cout<<"shmid="<<shmid<<endl;

    //第2步，把共享内存连接到当前进程的地址空间
    stgirl * ptr = (stgirl * )shmat(shmid,0,0);
    if(ptr==(void*)-1)
    {
        cout<<"shmat failed\n";
        return -1;
    }

    //创建、初始化二元信号量(mutex),加锁
    csemp mt;
    if(mt.init(0x5005)==false)
    {
        cout<<"mt.init(0x5005) failed.\n";
        return -1;
    }
    cout<<"申请加锁...\n";
    mt.wait();
    cout<<"加锁成功.\n";


    //第3步，使用共享内存，对内存进行读/写
    cout<<"原值：no="<<ptr->no<<".name="<<ptr->name<<endl;
    ptr->no=atoi(argv[1]);
    strcpy(ptr->name,argv[2]);
    cout<<"新值：no="<<ptr->no<<".name="<<ptr->name<<endl;
    sleep(10);

    //解锁
    mt.post();
    cout<<"解锁。\n";


    //第4步，把共享内存从当前进程中分离
    shmdt(ptr);

    //第5步，如果共享内存不再使用，使用以下代码删除
    //if(shmctl(shmid,IPC_RMID,0)==-1)
    //{
    //    cout<<"shmctl failed\n";
    //    return -1;
    //}

    return 0;
}

````

以下命令查看和操作信号量  
````
ipcs -s        //查看信号量数组
ipcrm sem 信号量ID    //手动删除信号量数组
````



# 生产消费者模型 PV原语  
循环队列+信号量  











