---
layout:     post
title:      "Compile Command"
subtitle:   " \"learning……\""
date:       2025-01-02 17:16:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - compile
---

> Hey,亲爱的工作狂先生，新的一年不要对自己太苛刻了～

# 编译过程  
C++ 编译过程可以分为预处理、编译、汇编、链接等几个主要的阶段，每个阶段的输出都会成为下一阶段的输入。

## 预处理  
预处理（Preprocessing），将项目中所有.cpp文件展开，转化为.i文件，前提是所有的.cpp和.h文件没有语法错误且进行了合理的包含。在这个阶段，C++ 预处理器（cpp）会处理所有的预处理指令，如 #include、#define 和 #ifndef 等。预处理的目的是生成一个没有宏定义、头文件包含等指令的源代码文件。预处理的结果通常是一个纯净的 .i 文件，包含了展开后的代码。

* 宏展开
* 文件包含
* 条件编译
* 去除注释

常用命令：
````
g++ -E hello.cpp -o hello.i        //将源文件变成.i文件
g++ -E main.cpp -o main.i
````

## 编译  

## 汇编
## 链接
