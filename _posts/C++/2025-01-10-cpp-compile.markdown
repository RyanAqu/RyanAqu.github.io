---
layout:     post
title:      "g++ 编译细节、库制作和Makefile"
subtitle:   " \"learning……\""
date:       2025-01-10 06:16:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - compile
    - Makefile
---

> Hey,亲爱的工作狂先生，新的一年不要对自己太苛刻了～

# VScode和g++11下载  
用以下命令下载VScode，下载完之后安装chinese拓展、C++拓展、makefile拓展  
````
sudo snap install code --classic	//下载code
````

用以下命令下载g++编译工具  
````
sudu apt install g++
````



# 编译过程  
C++ 编译过程可以分为预处理、编译、汇编、链接等几个主要的阶段，每个阶段的输出都会成为下一阶段的输入。

一步到位的编译命令：
````
g++ main.cpp hello.cpp -o main        //将项目所需的所有.cpp文件编译链接成二进制可执行文件
````

### 预处理  
预处理（Preprocessing），将项目中所有.cpp文件展开，转化为.i文件，前提是所有的.cpp和.h文件没有语法错误且进行了合理的包含。在这个阶段，C++ 预处理器（cpp）会处理所有的预处理指令，如 #include、#define 和 #ifndef 等。预处理的目的是生成一个没有宏定义、头文件包含等指令的源代码文件。预处理的结果通常是一个纯净的 .i 文件，包含了展开后的代码。

* 宏展开：对宏进行替换。
* 文件包含：替换 #include 指令，插入相应的头文件内容。
* 条件编译：处理 #if、#ifdef 等条件编译指令，决定哪些代码会被编译，哪些代码会被忽略。
* 去除注释

预处理命令：
````
g++ -E hello.cpp -o hello.i        //将源文件变成.i文件
g++ -E main.cpp -o main.i
````

### 编译  
编译（Compilation），编译器将预处理后的源代码转换成汇编代码。这个阶段的工作是语法分析、语义分析，并生成与平台相关的低级指令。编译后产生的文件通常是 .s 格式的汇编源文件。  

* 语法分析：检查程序是否符合语法规则，生成抽象语法树（AST）。
* 语义分析：检查程序中的类型是否正确、变量是否已声明等。
* 生成汇编：生成平台相关的汇编代码。

编译命令：
````
g++ -S main.i -o main.s            //将.i文件编译成.s汇编文件
g++ -S hello.i -o hello.s
````

### 汇编  
汇编（Assembly），汇编器（as）将汇编源文件（.s）转化为目标文件（.o 或 .obj）。在这个阶段，汇编器将汇编代码转换成机器码，生成的目标文件通常是二进制的，但仍然包含符号和调试信息。

汇编命令：
````
g++ -c hello.s -o hello.o         //将.s汇编文件生成.o目标文件
g++ -c main.s -o main.o
````

通常.o文件和可执行文件都是二进制文件，如果强制打开就是一串乱码，使用以下命令进行查看：
````
objdump -s main.o                 //查看.o目标文件
objdump -s main                   //查看.exe执行文件
objdump -d main                   //反汇编指令，将.exe变成汇编文件进行查看
````

### 链接  
链接器（ld）将目标文件（.o）和库文件（如标准库或其他自定义库）进行链接，生成最终的可执行文件（如 .exe 或其他平台的可执行格式）。链接的主要任务是解决不同目标文件之间的符号引用问题，将程序中未定义的函数、变量链接到合适的地方。链接过程中，链接器会生成一个最终的可执行文件，通常是 .exe（Windows）或无扩展名的可执行文件（Linux 和 macOS）。

* 静态链接：将所有目标文件和库文件直接组合成一个大的二进制文件，生成最终的可执行文件。
* 动态链接：生成一个可执行文件，但某些库（如共享库 .dll 或 .so）将在程序运行时加载，而不是编译时静态链接。

链接命令：
````
g++ -static main.o hello.o -o main        //静态链接，将所有内容合到一个程序内，文件较大但执行效率高  
g++ main.o hello.o -o main                //默认动态链接，文件较小，要用到相应函数时再去库立面找  
````


# 静态库和动态库  
在实际开发中，把通用的函数和类分文件编写，称之为库。在其他程序中，可以使用库中的函数和类。一般来说，一个项目文件夹下有./include ./src ./lib 目录，其中 ./include 文件夹存放头文件， ./src 存放源文件， ./lib 存放即将要做的库文件。  

![image](https://github.com/user-attachments/assets/7b6145f8-ba66-4de9-82d2-c5a29791252e)

### 动态库制作  
1. 找到函数定义的地方，也就是src处，直接将.c或者.cpp文件制作成动态库

````
gcc -fPIC -shared add.c -o libadd.so        //注意lib是前缀，add才是库名，.so是后缀
gcc -fPIC -shared add.o -o libadd.so -I ../include    //注意这里头文件没有跟.c文件放同一个目录，所以要指定一下头文件的位置
````

2. 将生成的动态库移动到.lib文件夹下

![image](https://github.com/user-attachments/assets/bdfeef15-2c75-4254-afab-4af5721d5332)

3. 编译命令  

````
gcc main.c -o main -Iinclude/ -Llib/ -ladd
````

* -I（大写的i Include）：告诉编译器在哪里找头文件
* -L（库Lib）：告诉编译器在哪里找库文件
* -l（小写的L）：要链接哪个动态库，写库名（即add）

4. 添加环境变量

![image](https://github.com/user-attachments/assets/faf23505-a92b-432b-914e-5684fe354a52)

可以把库路径添临时加进去：  

![image](https://github.com/user-attachments/assets/34317196-fe00-45ef-99ef-6b4ebd632dca)

或者直接把库复制到~/lib下：

![image](https://github.com/user-attachments/assets/3499a628-43de-441f-ace8-b1a8a1830fca)




### 静态库制作  
1. 找到函数定义的地方，也就是src处，将.c或者.cpp文件编译成.o文件  

````
gcc -c add.c -o add.o
````

2. 把.o文件编译生成libadd.a（注意不能一步到位，只能从.o制作静态库）

````
ar -rc libadd.a add.o
````

3. 把库文件放到与main.c一个文件夹，编译

````
gcc main.c libadd.a -o main -Iinclude    //因为头文件不在一个地方，所以得指定
````

# Makefile  
我们在嵌入式开发的时候，有时在单片机上跑linux系统，或者在服务器上跑linux系统，常常是没有图形化界面的。我们无法使用便捷的vscode工具编译代码，所以需要这样一个工具去统一编译整个项目，即Makefile  

### 工具下载  
````
sudo apt install -y build-essential
````

### make  
控制台实现make，首先在项目内添加Makefile文件，文件名固定为**Makefile**，然后开始编写  
````
# Makefile内容通常有以下3部分内容组成
# <目标名称>:<前置依赖>
# \t<需要执行的命令>

# 放在首位的是默认目标，名称为main
# 目标是编译出mian文件，依赖hello.o main.o
# 编译的命令是g++ hello.o main.o -o main
# 如果前置依赖不存在，会先编译出依赖的.o文件
main: hello.o main.o
	g++ hello.o main.o -o main
````
编写完成后控制台输入**make**既可以完成编译,本来应该加上前置依赖的生成命令，但make可以自动检测前置依赖的有无并且补全命令,但是此方法不推荐，因为无法监控依赖的.h文件的变化，只能自动推导出hello.o: hello.cpp  

以下是带前置依赖的Makefile,-o以及后面的内容可以省略，工具会自动推导  
````
main: hello.o main.o
	g++ hello.o main.o -o main

main.o: main.cpp hello.h
	g++ -c main.cpp -o main.o

hello.o: hello.cpp hello.h
	g++ -c hello.cpp -o hello.o

````

带自动推导的简化版本  
````
main: hello.o main.o
	g++ hello.o main.o -o main

main.o: hello.h

hello.o: hello.h
````


## clean  
在Makefile中添加clean语句  
````
main: hello.o main.o
	g++ hello.o main.o -o main

main.o: main.cpp hello.h
	g++ -c main.cpp -o main.o

hello.o: hello.cpp hello.h
	g++ -c hello.cpp -o hello.o

clean:
	rm main.o hello.o
````

在make之后执行**make clean**命令可以将不需要的.o文件清除  

### 伪目标  
在写clean的时候，clean本身就是一个伪目标,但是如果目录下有一个clean，那么make clean会出现问题，所以需要显示申明clean是一个伪目标  
````
.PHONY: clean
````

# main函数的参数  
main函数一般有三个参数，argc,argv,envp  
* argc：自动检测输入的参数个数，其中第一个是程序本身的路径
* argv：自动保存所有输入的参数，后续可以处理
* envp：自动保存环境变量

````
#include<iostream>
using namespace std;

int main(int argc,char* argv[],char* envp[])
{
    /*
    cout<<"There is "<<argc<<" arguments"<<endl;
    for(int i=0;i<argc;++i)
    {
        cout<<argv[i]<<" ";
    }cout<<endl;
    */

    for(int i=0;envp[i]!=0;++i)
    {
        cout<<envp[i]<<endl;
    }

    return 0;
}
````










