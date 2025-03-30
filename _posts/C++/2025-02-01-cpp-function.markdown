---
layout:     post
title:      "C++11 function&bind"
subtitle:   " \"learning……\""
date:       2025-02-01 14:18:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - c++11
---

> 真正的大师，永远都怀着一颗学徒的心......

# 函数封装器和绑定器  
C++11 引入了 std::function 和 std::bind，主要用于函数对象的存储和调用，提供了一种更灵活的方式来管理回调函数、延迟执行和绑定参数。  

# 函数封装器  
std::function 是 C++ 标准库中的一个通用函数包装器，它可以存储普通函数、lambda 表达式、函数指针、成员函数指针、仿函数等。类似函数指针。  

### 存储普通函数  
````
#include <iostream>
#include <functional>

void hello() {
    std::cout << "Hello, world!\n";
}

int main() {
    std::function<void()> func = hello;  // 存储普通函数
    func();  // 调用
}
````

### 存储lambda表达式
````
std::function<int(int, int)> add = [](int a, int b) { return a + b; };
std::cout << add(2, 3);  // 输出 5
````

### 存储函数成员  
````
class Test {
public:
    void say() { std::cout << "Hello from Test\n"; }
};

Test obj;
std::function<void()> f = std::bind(&Test::say, &obj);
f();  // 调用成员函数
````
# 绑定器  
std::bind 用于绑定函数的部分或全部参数，返回一个新的可调用对象，常用于回调函数或参数固定的函数调用。  

### 绑定普通函数  
````
#include <iostream>
#include <functional>

int add(int a, int b) { return a + b; }

int main() {
    auto add5 = std::bind(add, 5, std::placeholders::_1);  // 绑定第一个参数为 5,std::placeholders::_1 代表占位符，表示调用时再传递的参数。
    std::cout << add5(10);  // 输出 15
}
````

### 绑定成员函数  
````
class Test {
public:
    void show(int x) { std::cout << "Value: " << x << '\n'; }
};

int main() {
    Test obj;
    auto f = std::bind(&Test::show, &obj, std::placeholders::_1);
    f(42);  // 输出 Value: 42
}
````

### 绑定成员变量  
````
#include <iostream>
#include <functional>

struct Data {
    int value;
};

int main() {
    Data d{10};
    auto get_value = std::bind(&Data::value, &d);
    std::cout << get_value();  // 输出 10
}
````

![image](https://github.com/user-attachments/assets/2eb79dac-2c02-4dfc-8cfc-0bdc4625ce99)








