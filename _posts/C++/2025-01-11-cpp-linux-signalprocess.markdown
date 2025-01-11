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
















