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

### 单例模式  
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

但是当前的单例模式可能**线程不安全**，需要以下方法实现线程安全。  

### 线程安全的懒汉模式  
线程安全的单例模式是指确保在多线程环境下，单例类的实例只会被创建一次，并且在不同线程中共享同一个实例。为了避免多线程环境下可能出现的竞争条件和不一致性问题，需要使用适当的同步机制来确保线程安全。  
````
#include <iostream>
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::mutex mtx;  // 用于保护实例化过程的互斥锁

    // 私有构造函数，防止外部直接创建实例
    Singleton() {}

public:
    // 禁止拷贝构造和赋值
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    // 获取唯一实例
    static Singleton* getInstance() {
        // 锁住实例化过程，保证线程安全
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);  // 防止多个线程同时进入
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }

    void showMessage() {
        std::cout << "Hello, I am a Singleton!" << std::endl;
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;

int main() {
    Singleton* s1 = Singleton::getInstance();
    s1->showMessage();
    return 0;
}

````

### 线程安全的饿汉模式  
饿汉式在程序启动时就创建实例，通常是在静态初始化时完成实例化，因此不需要额外的同步机制，天然线程安全。  
````
#include <iostream>

class Singleton {
private:
    // 在类内部就进行静态实例化，保证线程安全
    static Singleton instance;

    // 私有构造函数，防止外部创建实例
    Singleton() {}

public:
    // 禁止拷贝构造和赋值
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    // 获取唯一实例
    static Singleton& getInstance() {
        return instance;
    }

    void showMessage() {
        std::cout << "Hello, I am a Singleton!" << std::endl;
    }
};

// 静态成员初始化
Singleton Singleton::instance;

int main() {
    Singleton& s1 = Singleton::getInstance();
    s1.showMessage();
    return 0;
}

````





# 工厂模式  
工厂模式提供了一种创建对象的方式，而不需要明确指定具体类的实例化过程。通过工厂方法，客户端能够通过简单的接口创建各种对象，而不需要知道具体的实现类。这种模式帮助我们将对象的创建与对象的使用解耦。  

适用场景：
* 系统中有多个类，而且这些类都属于同一类型，并且需要在运行时选择具体类时。
* 当我们需要根据输入的不同，创建不同种类的对象时。

实现方式：
* 工厂模式一般有两种实现方式：简单工厂模式和工厂方法模式。
* 简单工厂模式：通过一个工厂类来创建对象，根据传入的参数来决定创建哪个对象。
* 工厂方法模式：定义一个创建对象的接口，由子类来决定实例化哪个类。

### 简单工厂模式  
````
#include <iostream>
using namespace std;

// 产品类
class Product {
public:
    virtual void operation() = 0;
};

// 具体产品A
class ProductA : public Product {
public:
    void operation() override {
        cout << "ProductA operation" << endl;
    }
};

// 具体产品B
class ProductB : public Product {
public:
    void operation() override {
        cout << "ProductB operation" << endl;
    }
};

// 工厂类
class Factory {
public:
    // 根据类型创建不同的产品
    static Product* createProduct(char type) {
        if (type == 'A') {
            return new ProductA();
        } else if (type == 'B') {
            return new ProductB();
        }
        return nullptr;
    }
};

int main() {
    Product* productA = Factory::createProduct('A');
    productA->operation();
    
    Product* productB = Factory::createProduct('B');
    productB->operation();
    
    delete productA;
    delete productB;

    return 0;
}

````

### 工厂方法模式  
````
#include <iostream>
using namespace std;

// 产品类
class Product {
public:
    virtual void operation() = 0;
};

// 具体产品A
class ProductA : public Product {
public:
    void operation() override {
        cout << "ProductA operation" << endl;
    }
};

// 具体产品B
class ProductB : public Product {
public:
    void operation() override {
        cout << "ProductB operation" << endl;
    }
};

// 抽象工厂
class Creator {
public:
    virtual Product* createProduct() = 0;
};

// 具体工厂A
class CreatorA : public Creator {
public:
    Product* createProduct() override {
        return new ProductA();
    }
};

// 具体工厂B
class CreatorB : public Creator {
public:
    Product* createProduct() override {
        return new ProductB();
    }
};

int main() {
    Creator* creatorA = new CreatorA();
    Product* productA = creatorA->createProduct();
    productA->operation();
    
    Creator* creatorB = new CreatorB();
    Product* productB = creatorB->createProduct();
    productB->operation();

    delete creatorA;
    delete creatorB;
    delete productA;
    delete productB;

    return 0;
}

````













