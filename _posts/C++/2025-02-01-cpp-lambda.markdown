---
layout:     post
title:      "C++11 lambda"
subtitle:   " \"learning……\""
date:       2025-02-01 13:18:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - c++11
---

> 医院的白墙听过比教堂更多的祈祷


Lambda 表达式是 C++11 引入的一种特性，用于定义匿名函数对象（即没有名字的函数，或者称为闭包）。这种表达式使得函数作为参数传递更加简洁，并且能够访问外部作用域的变量。  
# 基础语法  
````
[capture](parameter_list) -> return_type { body }
````

* capture：指定外部变量的捕获方式。可以是值捕获或引用捕获。
  *  \[ \]：空捕获列表，表示不捕获任何外部变量。
  *  \[x\]：按值捕获外部变量 x。
  *  \[&\]：按引用捕获外部所有变量。
  *  \[this\]：捕获当前对象指针 this。
  *  \[=\]：按值捕获所有外部变量。
  *  \[&x\]：按引用捕获 x。
* parameter_list：函数的参数列表，类似普通函数的参数列表。  
* return_type：返回类型，如果能够推导出来可以省略。C++14 中支持推导返回类型，如果不显式声明返回类型，编译器会根据函数体推导出返回类型。  
* body：函数体，包含具体的实现。  

# 一些简单的示例  
**空捕获无参：**
````
#include <iostream>

int main() {
    auto lambda = []() { std::cout << "Hello, Lambda!" << std::endl; };
    lambda(); // 调用 lambda 表达式
    return 0;
}
````

**带参数的lambda表达式：**  
````
#include <iostream>

int main() {
    auto add = [](int a, int b) { return a + b; };
    std::cout << "Sum: " << add(3, 5) << std::endl;
    return 0;
}
````

**按值捕获的lambda表达式：**  
````
#include <iostream>

int main() {
    int x = 10, y = 20;
    auto lambda = [x, y]() { return x + y; };
    std::cout << "Sum: " << lambda() << std::endl;
    return 0;
}
````

**按引用捕获的lambda表达式：**  
````
#include <iostream>

int main() {
    int x = 10, y = 20;
    auto lambda = [&x, &y]() { x += 5; y += 5; };
    lambda(); // 修改了外部变量
    std::cout << "x: " << x << ", y: " << y << std::endl; // 输出 x: 15, y: 25
    return 0;
}

````

**捕获this指针：**  
````
#include <iostream>

class MyClass {
public:
    int x = 10;
    
    void print() {
        auto lambda = [this]() { std::cout << "x = " << x << std::endl; };
        lambda();
    }
};

int main() {
    MyClass obj;
    obj.print();  // 输出 x = 10
    return 0;
}

````

# lambda表达式的用途  
1. 作为回调函数：Lambda 表达式常常用作回调函数传递给算法或线程等。  
2. 函数式编程：在 STL 算法（如 std::sort, std::for_each 等）中，Lambda 表达式用于指定自定义操作。  
3. 临时操作：可以用于定义不需要命名的、只在某一小段代码中使用的函数。  

* Lambda 表达式使得函数可以被定义为匿名函数对象，减少了代码冗余。  
* 它的捕获机制允许访问外部作用域的变量，可以按值或按引用捕获。  
* Lambda 表达式在 STL 算法和多线程编程中非常有用。  


















