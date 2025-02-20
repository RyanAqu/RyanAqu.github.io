---
layout:     post
title:      "C++ 设计模式"
subtitle:   " \"learning……\""
date:       2025-02-20 15:23:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 设计模式
---

# 单例模式  
单例模式确保某一个类只有一个实例，并提供一个全局访问点来访问该实例。它通常用于需要共享资源的场景，如数据库连接、日志管理等，避免频繁创建和销毁相同的对象。  

关键点：  
* 保证只有一个实例：无论创建多少个对象，类中只会有一个实例。
* 提供全局访问点：通过该实例可以访问到类的功能。
* 延迟实例化：实例化对象时通常是延迟加载的（懒汉模式）。

优点：  
* 节省内存资源：只会有一个实例，避免了重复的对象创建。
* 全局共享：可以在系统中多个地方共享同一个实例，方便管理资源。

缺点：  
* 不容易扩展：单例模式将类的实例化过程与具体实现绑定在一起，不容易扩展。
* 并发问题：在多线程环境中，需要确保实例化过程的线程安全，否则会出现多个实例。

### 代码  
````
class Singleton {
private:
    static Singleton* instance;
    Singleton() {}  // 私有化构造函数，禁止外部直接实例化
    
public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
};

// 初始化静态成员
Singleton* Singleton::instance = nullptr;

int main() {
    Singleton* singleton1 = Singleton::getInstance();
    Singleton* singleton2 = Singleton::getInstance();
    
    // 输出的两个实例地址相同
    cout << singleton1 << endl;
    cout << singleton2 << endl;
    return 0;
}

````









