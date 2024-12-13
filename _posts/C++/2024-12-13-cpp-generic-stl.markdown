---
layout:     post
title:      "C++ 泛型 & STL 标准模板库"
subtitle:   " \"learning……\""
date:       2024-12-13 18:13:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
    - STL
---

> 大器藏拙，出手即伤

# 模板（泛型编程）
分类：
1. 函数模板
2. 类模板

语法：  
````
template<typename T>        //template申明创建模板
函数申明或者定义
````

## 两种模板类型

数据类型typename模板示例：  
````
//不用模板
void swapT(int &a,int &b)
{
    int temp =a;
    a=b;
    b=temp;
}
//用模板
template<typename T>
void swapT(T &a,T &b)
{
    T temp =a;
    a=b;
    b=temp;
}
void test01()
{
    double a=2,b=1;
    swapT(a,b);                 //自动类型推导
    swapT<double>(a,b);         //显式指定类型
    cout<<"a="<<a<<" b="<<b<<endl;
}
````

class模板示例(class也可以传typename)：
````
template<class T>
void swapT(T& a,T& b)
{
    T temp=a;
    a=b;
    b=temp;
}
void test01()
{
    double a=2,b=1;
    swapT(a,b);
    cout<<"a="<<a<<" b="<<b<<endl;
    swapT<double>(a,b);
    cout<<"a="<<a<<" b="<<b<<endl;
}
````

遇事不决用class模板

## 模板实现数组排序-选择排序  
````
#include<iostream>
using namespace std;

template<class T>
void QuickSort(T arr[],int len)
{
    for(int i=0;i<len-1;++i)
    {
        int minindex=i;
        for(int j=i+1;j<len;++j)
        {
            if(arr[minindex]>arr[j]) minindex=j;
        }
        if(minindex!=i) swap(arr[minindex],arr[i]);
    }
}
template<class T>
void printArray(T arr[],int len)
{
    for(int i=0;i<len;++i)
    {
        cout<<arr[i]<<" ";
    }
    cout<<endl;
}
void test01()
{
    char charArr[]="badcfe";        
    int num =sizeof(charArr)/sizeof(char);
    QuickSort(charArr,num-1);
    printArray(charArr,num-1);

    int intArr[] = {7,5,1,3,9,2,4,6,8};
    int num1 =sizeof(intArr)/sizeof(int);
    QuickSort(intArr,num1);
    printArray(intArr,num1);
}
int main()
{

    test01();
    return 0;
}
````

## 隐式类型转换
普通函数和函数模板的区别：
* 普通函数调用时可以发生自动类型转换
* 函数模板调用时，如果利用自动类型推导，不会发生隐式类型转换
* 如果利用显式指定类型的方式，可以发生隐式类型转换

## 普通函数和函数模板的调用规则
调用规则如下：  
1. 如果函数模板和普通函数都可以实现，则优先调用普通函数
2. 可以通过空模板参数列表来强制调用函数模板
3. 函数模板也可以重载
4. 如果函数模板可以产生更好的匹配则优先调用函数模板  

## 类模板
类模板无法自动推导类型，但是模板名在申明的时候可以有默认参数  
````
#include<iostream>
#include<string>
using namespace std;

template<class NameType,class AgeType>
class Person
{
public:
    Person(NameType name,AgeType age)
    {
        this->m_Age = age;
        this->m_Name = name;
    }
    NameType m_Name;
    AgeType m_Age;
    void showPerson(){
        cout<<"姓名："<<m_Name<<" 年龄："<<m_Age<<endl;
    }
};

void test01()
{
    Person<string,int> p1("ryan",23);
    p1.showPerson();
}
int main()
{
    test01();
    return 0;
}
````

## 类模板中成员函数创建时机
* 普通类中的成员函数一开始就可以创建
* 类模板中的成员函数在调用时才创建

## 类模板的对象做函数的参数
1. 指定传入类型(最常用)  
2. 参数模板化  
3. 整个类模板化  

````
#include<iostream>
#include<string>
using namespace std;

template<class T1,class T2>
class Person
{
public:
    Person(T1 name,T2 age)
    {
        this->m_Name=name;
        this->m_Age=age;
    }
    T1 m_Name; 
    T2 m_Age;
    void show(){
        cout<<"name:"<<m_Name<<" age:"<<m_Age<<endl;
    }
};
void printPerson1(Person<string,int>& p)//指定传入类型
{
    p.show();
}
template<class T1,class T2>
void printPerson2(Person<T1,T2>& p)//参数模板化
{
    p.show();
}
template<class T>
void printPerson3(T& p)//整个类都模板化
{
    p.show();
}
void test01()//指定传入类型
{
    Person<string,int> p1("ryan",100); 
    printPerson1(p1);
}
void test02()//参数模板化
{
    Person<string,int> p2("zack",200); 
    printPerson2(p2);
}
void test03()//整个类都模板化
{
    Person<string,int> p3("hxn",300); 
    printPerson3(p3);
}
int main()
{
        
    test01();
    test02();
    test03();
    return 0;
}
````

## 类模板与继承  
注意点：
* 当子类继承的父类是一个类模板时，子类在申明的时候，需要指出父类中T的类型
* 如果不确定，编译器无法给子类分配内存
* 如果要灵活指出父类中的类型，子类也要变成类模板

````
#include<iostream>
#include<string>
using namespace std;

template<class T>
class Parent
{
public:
    T m;
};
class Son1:public Parent<int>
{

};
template<class T1,class T2>
class Son2:public Parent<T2>
{
    T1 obj;
};

void test01()
{
    Son1 s1;
}
void test02()
{
    Son2<int,char> s2;
}
int main()
{
    test01();
    test02();
    return 0;
}
````

## 类模板成员函数类外实现 
````
template<class T1,class T2>
class Person
{
public:
    Person(T1 name,T2 age);
    T1 m_Name;
    T2 m_Age;
    void show();
};
template<class T1,class T2>
Person<T1,T2>::Person(T1 name,T2 age)
{
    this->m_Name=name;
    this->m_age=age;
}
template<class T1,class T2>
void Person<T1,T2>::show()
{
    //
}

````


## 类模板分文件编写
由于模板类的头文件中的类成员不会在编译的时候创建，所以只包含.h文件无法解析接口。所以将.h和.cpp的内容放在一起，改后缀为.hpp，包含这个文件即可。


## 类模板与友元函数
有类内实现和类外实现，类外实现烦的一，建议类内实现  
````
#include<iostream>
#include<string>
using namespace std;

//////////类外实现 烦的一////////
template<class T1,class T2>
class Person;
template<class T1,class T2>
void printPerson2(Person<T1,T2>& p)
{
    cout<<"name:"<<p.m_Name<<" age:"<<p.m_Age<<endl;
}
//////////

template<class T1,class T2>
class Person
{
    //全局函数 类内实现
    friend void printPerson(Person<T1,T2>& p)
    {
        cout<<"name:"<<p.m_Name<<" age:"<<p.m_Age<<endl;
    }
    //全局函数 类外实现
    //需要加一个空模板参数列表
    friend void printPerson2<>(Person<T1,T2>& p);
public:
    Person(T1 name,T2 age);
    void showPerson();
private:
    T1 m_Name;
    T2 m_Age;
};


template<class T1, class T2>
Person<T1,T2>::Person(T1 name,T2 age)
{
    this->m_Age=age;
    this->m_Name=name;
}
template<class T1,class T2>
void Person<T1,T2>::showPerson()
{
    cout<<"name:"<<m_Name<<" age:"<<m_Age<<endl;
}



void test01()
{
    Person<string,int> p("ryan",23);
    printPerson(p);
}
void test02()
{
    Person<string,int> p("zack",25);
    printPerson2(p);
}
int main()
{
    test01();
    test02();
    return 0;
}
````




# C++进阶：标准模板库(Standard Template Library,STL)  
## STL基本概念  
* 标准模板库
* 广义上分为容器、算法、迭代器  
* 容器与算法通过迭代器进行无缝链接
* STL所有代码几乎都采用了模板类或者模板函数

## STL六大组件
STL大体分为六大组件：容器、算法、迭代器、仿函数、适配器、空间配置器  
1. 容器：各种数据结构，如vector、list、deque、set、map等  
2. 算法：各种常用算法，如sort、find、copy、for_each等  
3. 迭代器：扮演了容器与算法之间的胶合剂  
4. 仿函数：小括号重载，行为类似函数，可作为算法的某种策略  
5. 适配器：一种用于修饰容器或者仿函数或者迭代器接口的东西  
6. 空间配置器：负责空间的配置与管理  

## STL中容器、算法、迭代器  
常用数据结构：数组、链表、树、栈、队列、集合、映射表等  
* 序列式容器：强调值的顺序，数据有固定的位置  
* 关联式容器：二叉树结构，哈希表，没有严格物理顺序  

![AC](https://ryanaqu.github.io/img/inpost_cpp/image7.jpg)

算法分为质变算法和非质变算法：  
* 质变算法：拷贝、替代、删除  
* 非质变算法：查找、遍历、计数、寻找极值  
* 有些容器不支持迭代器随机访问，因此不能使用sort函数

### string
**容器 -- 字符串**  
string本质上是一个类，内部维护着一个char\*字符串，是一个char\*的容器  
```
#include <string>           //包含头文件<string>

//构造
string s1;                  //构造空字符串，原型string()
string s2("hello world!");  //字符串数组s初始化，原型string(const char* s);
string s1(s2);              //一个string对象初始化另一个对象，原型string(const string& str)
string(5,'A');              //初始化n个字符，string(int n,char c)

//赋值
string s1 = "hello world!"; //等号运算符重载
string s2 = s1;             //等号运算符重载

//追加
str1 += str2;               //拼接
str1.append("LOL");         //追加
str1.append(str2,0,3)       //追加一个字符串，从第0位开始，个数为3

//字符串的查找
string s1 = "abcdefg";
int pos = s1.find("de");    //返回找到的第一个位置，没找到返回-1
cout<<"pos="<<pos<<endl;

pos = s1.rfind("de");       //rfind是从右往左查
cout<<"pos="<<pos<<endl;

//字符串的替换
string s1 = "abcdefg";
s1.replace(1,3,"1111");     //将指定位置内容擦除，把指定内容填入
cout<<"s1="<<s1<<endl;

//字符串的比较
string s1 = "hello";
string s2 = "xello";
if(s1.compare(s2)==0)
{
    cout<<"s1=s2"<<endl;
}
else if(s1.compare(s2)<0)
{
    cout<<"s1<s2"<<endl;
}
else
{
    cout<<"s1>s2"<<endl;
}

//字符串中字符的读和写
string s1 = "hello";
cout<<"位置1的字符为:"<<s1[1]<<endl; //通过下标访问

for(int i=0;i<s1.size();++i){
    cout<<s1.at(i)<<" ";
}cout<<endl;                        //通过at函数访问

s1[1]=x;                            //通过下标修改

s1.at(1)=x;                         //通过at函数修改 


//字符串的插入和删除
string s1 = "hello";
cout<<"s1:"<<s1<<endl;
s1.insert(1,"111");                 //指定位置插入指定内容
cout<<"s1:"<<s1<<endl;

s1.erase(1,3);                      //擦除指定位置、指定个数
cout<<"s1:"<<s1<<endl;

//子串
string s3 = s2.substr(6,5);         //子串操作

``` 


### vector
**容器 -- 动态数组**  
![vector](https://ryanaqu.github.io/img/inpost_cpp/image2.png)  

vector容器常用成员：  
1. front()第一个元素  
2. back()最后一个元素  
3. push_back()尾插法  
4. pop_back()尾删法  
5. 迭代器begin()指向第一个元素  
6. 迭代器end()指向最后一个元素的后一个位置  
7. 迭代器rbegin()指向倒数第一个元素  


```
#include <vector>           //包含头文件<vector>

//构造
vector<type> variable_name; //创建空 vector 默认构造
vector<int> myVector;       //创建一个存储整数的 vector
vector<char> myVector;      //创建一个存储字符的 vector
vector<string> myVector;    //创建一个存储字符串的 vector

//构造
vector<int> vec1(5);        //指定初始大小并填充默认值（0） [0, 0, 0, 0, 0] 
vector<int> vec2(5, 42);    //指定大小并填充值[42, 42, 42, 42, 42]
vector<int> vec3 = {1, 2, 3, 4, 5};//数组形式初始化
vector<int> v2(v1.begin(),v1.end());//拷贝构造
vector<int> v5(v4);                 //拷贝构造

//赋值
vector<int> v2 = v1;                //等号赋值
v3.assign(v2.begin(),v2.end());     //assign赋值，前闭后开
v3.assign(10,100);                  //assign赋值，10个100

//访问元素
int x = vec[0];             // 使用下标访问第一个元素
int y = vec.at(1);          // 安全访问第二个元素，带边界检查
int z = vec.front();        // 获取第一个元素
int w = vec.back();         // 获取最后一个元素


//获取信息
int size = vec.size();      // 获取元素个数
int capacity = vec.capacity(); // 获取当前容量
bool isEmpty = vec.empty(); // 判断是否为空,空为1
vec.resize(10);             //重新指定大小

//插入与删除
vec.push_back(10);          // 在末尾插入元素 10
vec.pop_back();             // 删除末尾元素
vec.insert(vec.begin(), 20);// 在开头插入 20
vec.insert(vec.begin(),2,20);// 在开头插入2个20
vec.erase(vec.begin());     // 删除开头元素
vec.erase(vec.begin(),vec.begin()+4);// 删除区间，前闭后开
vec.clear();                // 清空所有元素

//互换swap
vec1.swap(vec2);            //互换容器
vector<int> (v).swap(v);    //匿名对象互换可以收缩capacity

//预留空间
vec.reserve(100);           //分配内存但不初始化

//遍历
// 使用索引遍历
for (int i = 0; i < vec.size(); ++i) {
    cout << vec[i] << " ";
}

// 使用范围循环
for (int val : vec) {
    cout << val << " ";
}

// 使用迭代器
for (auto it = vec.begin(); it != vec.end(); ++it) {
    cout << *it << " ";
}
```

### deque
**容器 -- 双端数组 -- 中控器实现**  
![deque](https://ryanaqu.github.io/img/inpost_cpp/image3.png)  

1. 双端数组可以在头端插入和删除  
2. vector对于头部的插入删除效率低，数据量越大效率越低  
3. deque相对而言对头部插入删除速度比vector快  
4. vector访问速度比deque快，这与内部实现有关  
5. deque没有容量的概念  


````
#include<deque>             //包含头文件

//构造函数
deque<int> dq;              //构造空 dq 默认构造
deque<int> dq(5);           //构造指定大小，初始值为0
deque<int> dq(5, 42);       //构造指定大小，指定填充值
deque<int> dq2{1,2,3,4,5,6,7,8,9};//{}构造
deque<int> dq3(dq2);        //拷贝构造

//迭代器范围构造
vector<int> vec = {1, 2, 3, 4, 5};
deque<int> dq(vec.begin(), vec.end());

//赋值
deque<int> dq1{1,2,3,4,5,6,7,8,9};
deque<int> dq2=dq1;         //等号赋值
dq3.assign(dq1.begin()+1,dq1.begin()+4);//assign赋值

//容器大小操作
dq1.size();                 //返回容器大小
dq1.resize(10);             //重新定义大小，默认0填充
dq1.resize(15,1);           //重新定义大小，超出部分填充1
dq1.clear();                //容器清空
dq1.empty();                //判断容器是否为空

//deque插入和删除  
dq.push_back(10);           //尾插法
dq.push_front(12);          //头插法
dq.pop_back();              //尾删法
dq.pop_front();             //头删法
dq.insert(pos,elem);        //确定位置插入确定元素
dq.insert(pos,n,elem);      //确定位置插入n个元素
dq.insert(pos,d.begin(),d.end());//固定位置插入迭代器指定段
erase(begin,end);           //擦除特定区域，前闭后开，返回下一个数据位置
erase(pos);                 //擦除特定位置，返回下一个数据位置

//数据读写
dq.front();                 //第一个元素
dq.back();                  //最后一个元素
dq[5];                      //通过下标访问
dq.at(5);                   //at()函数访问

//deque容器排序  
sort(dq.begin(),dq.end());

````

### stack
**容器 -- 栈**  
![stack](https://ryanaqu.github.io/img/inpost_cpp/image4.png)  

先进后出，压栈，出栈，顶端可访问。  

````
#include<stack>         //头文件

stack<int>s;            //构造
s.push(10);             //压栈
s.pop();                //出栈
s.top();                //访问栈顶元素
s.size();               //栈的大小
s.empty();              //是否为空

````

### queue
**容器 -- 队列**  
![queue](https://ryanaqu.github.io/img/inpost_cpp/image5.png)

先进先出，头尾可访问。

````
#include<queue>         //头文件

q.push(10);             //入队
q.pop();                //出队
q.front();              //访问队头元素
q.back();               //访问队尾元素
q.size();               //队列大小
q.empty();              //是否为空

````

### list
**容器 -- 链表**  
![list](https://ryanaqu.github.io/img/inpost_cpp/image6.png)

STL中的链表由节点构成，节点有指针域和数据域，是双向循环链表。

````
#include<list>                  //头文件

//构造
list<int> mylist;               //默认构造
list<int> mylist1(mylist.begin(),mylist.end());//区间构造
list<int> mylist2(mylist1);     //拷贝构造
list<int> mylist3(10,10);       //n个elem构造
list<int> mylist{10,20,30,40,50};//{}构造

//list赋值
list<int>mylist1=mylist;        //等号赋值
mylist2.assign(mylist.begin(),mylist.end());//区间赋值
mylist2.assign(10，10);         //n个elem赋值

//list交换
mylist3.swap(mylist2);          //内容互换

//list大小
mylist.size();                  //list中的node数
mylist.empty();                 //判断是否为空
mylist.resize(10);              //重新指定大小,默认补零
mylist.resize(10,10);           //重新指定大小,补充指定元素

//list插入和删除
mylist.push_back(60);           //尾插法
mylist.push_front(0);           //头插法
mylist.pop_back();              //尾删法
mylist.pop_front();             //头删法

list<int>::iterator it=mylist.begin();
mylist.insert(++it,0);          //插入
mylist.erase(it);               //删除
mylist.remove(elem);            //将所有等于这个elem的值都删除
mylist.clear();                 //清空

//数据存取
mylist.front();                 //第一个元素
mylist.back();                  //最后一个元素

//双向迭代器，但不支持随机访问
it++;                           //向后走
it--;                           //向前走

//反转
mylist.reverse();               //反转  

//排序
mylist.sort();                  //排序，默认升序

bool myCompare(int val1,int val2)
{
    return val1>val2;
}
mylist.sort(myCompare);         //降序排序

````

### set
**容器 -- 集合**  
底层是由二叉树实现的，set/multiset属于关联式容器，所有元素在插入时会自动排序。  

set和multiset区别：  
* set不允许有重复的元素
* multiset允许容器有重复的元素

````
#include<set>                   //一个头文件包含set和multiset

//构造
set<int> s;                     //默认构造
set<int> s{1,2,3,6,5,4,9,8,7};  //{}构造，会自动排序

//插值和删除
s.insert(elem);                 //只能用insert，自动排序
s.erase(elem);                  //擦除数据，自动查找
s.erase(s.bgin());              //擦除迭代器指向的内容
s.erase(s.begin(),++s.begin()); //擦除特定区间的内容，前闭后开
s.clear();                      //清空set

//交换
s.swap(s1);

//信息
s.empty()；                     //判断是否为空
s.size();                       //输出大小

//查找和统计
s.find(elem);                   //找到会返回对应的迭代器，否则返回s.end()
s.count(elem);                  //对set而言，统计结果只有0和1

//insert插入函数返回键值对，以观察是否插入成功,插入失败会返回s.end()迭代器
pair<set<int>::iterator,bool> ret = s.insert(10);
cout<<"插入:"<<*ret.first<<(ret.second ? "成功":"失败")<<endl;

//通过仿函数改set排序顺序
class MyCompare
{
public:
    bool operator()(int v1,int v2) const
    {
        return v1>v2;
    }
};

void test04()
{
    //指定排序规则 -- 降序
    set<int,MyCompare> s{5,6,9,8,7,4,1,2,3};
    
    for(set<int,MyCompare>::iterator it=s.begin();it!=s.end();++it)
    {
        cout<<*it<<" ";
    }
    cout<<endl;
}

//自定义数据类型必须提供仿函数
class PersonCompare
{
public:
    bool operator()(const Person& p1,const Person& p2) const
    {
        return p1.Age>p2.Age;
    }
};

void test05()
{
    //指定排序规则 -- 降序
    Person p1("ryan",23);
    Person p2("zack",22);
    Person p3("hxn",26);
    Person p4("baiye",21);

    set<Person,PersonCompare> s;
    s.insert(p1);
    s.insert(p2);
    s.insert(p3);
    s.insert(p4);
    
    for(set<Person,PersonCompare>::iterator it=s.begin();it!=s.end();++it)
    {
        cout<<"name:"<<it->m_Name<<"\tage:"<<it->Age<<endl;
    }
}

````

### pair
**容器 -- 键值对（队组）**   
通常和set或者map一起使用。  

````
//构造
pair<int, string> p1(1, "apple");
pair<int, string> p2 = make_pair(2, "banana");
````

### map
**容器 -- map**  
map底层是红黑树。  
* map中所有元素都是pair  
* pair中第一个元素是key（键值），起索引作用，第二个元素为value  
* 所有元素根据键值自动排序  

map和multimap区别：  
* map不允许有重复的元素  
* multimap允许容器有重复的元素  

````
#include<map>               //包含头文件

//构造
map<int,int> m;             //默认构造
map<int,int> m1(m);         //拷贝构造

//赋值
map<int,int> m1 = m;        //等号赋值


//插值和删除
m.insert(pair<int,int>(2,3));//插入键值对
m[5]=50;                    //下标插入,不建议用来插数
m.clear();                  //清空
m.erase(pos);               //删除pos迭代器所指位置，返回下一个
m.erase(begin,end);         //删除区间
m.erase(key);               //根据键值删除 

//信息
m.empty();                  //返回是否为空
m.size();                   //输出键值对个数

//交换
m.swap(m1);                 //交换

//查找和统计
m.find(key);                //根据键值查找，查到返回迭代器，查不到返回m.end()
m.count(key);               //统计键值出现次数，map至多一次

//利用仿函数改变排序规则
class mycompare
{
public:
    bool operator()(int v1,int v2) const
    {
        return v1>v2;
    }
};
void test04()
{
    map<int,int,mycompare> m;
    m.insert(pair<int ,int>(1,40));
    m.insert(pair<int ,int>(2,30));
    m.insert(pair<int ,int>(3,20));
    m.insert(pair<int ,int>(4,10));
    for(map<int,int,mycompare>::const_iterator it = m.begin();it!=m.end();it++)
    {
        cout<<it->first<<":"<<it->second<<"  ";
    }
    cout<<endl;
}
````

### bitset
**二进制运算**
```
#include <bitset>           //包含头文件

//创建
bitset<N> variable          //N表示bit长度

//初始化
bitset<8> bits("10110101"); //0b10110101

//操作
int bit = 0xb5;
bitset<8> bits(bit);        //转换

bits<<1;                    //左移1位
bits>>2;                    //右移2位
bits.flip();                //按位取反
bits.reset(1);              //特定位置0
bits.set(1);                //特定位置1
```



## 函数对象
即仿函数，其特点如下：  
* 函数对象在使用时，可以像普通函数那样调用，可以有参数、可以有返回值  
* 函数对象有自己的状态  
* 函数对象可以作为参数传递  

## 谓词  
* 返回bool类型的仿函数统称为谓词  
* 如果operator()接受一个参数，那么称为一元谓词  
* 如果operator()接受两个参数，那么称为二元谓词  

## 内建仿函数functional  
分类：
* 算数仿函数  
* 关系仿函数  
* 逻辑仿函数  

````
#include<functional>            //包含头文件

//算数仿函数+-*/~%
plus<int> p;                    //+
cout<<p(10,20)<<endl;
minus<int>m;                    //-
cout<<m(10,20)<<endl;
multiplies<int> mu;             //*
cout<<mu(10,20)<<endl;
divides<int>di;                 ///
cout<<di(20,10)<<endl;
modulus<int>mo;                 //%
cout<<mo(16,10)<<endl;
negate<int> n;                  //~
cout<<n(50)<<endl;

//关系仿函数  
less<T> name;                   
name(a,b);                      //<
greater<T> name;
name(a,b);                      //>
less_equal<T> name;
name(a,b);                      //<=
greater_equal<T> name;
name(a,b);                      //>=
equal_to<T> name;
name(a,b);                      //==
not_equal_to<T> name;
name(a,b);                      //!=
sort(v.begin(),v.end(),greator<T>())//排序

//逻辑仿函数
logical_and<T> name;
name(num);                      //逻辑与
logical_or<T> name;
name(num);                      //逻辑或
logical_not<T> name;
name(num);                      //逻辑非

````


## STL常用algorithm算法  
````
#include<algorithm>         //头文件

//遍历容器：
for_each(itrator begin,iterator end, _func);

//搬运算法
transform(itrator begin1,iterator end1,itrator begin2,iterator end2, _func);            //返回值

//查找算法
find(iterator begin,iterator end,value); //查找元素
find_if(iterator begin,iterator end,一元谓词); //按照条件查找
adjacent_find(v.begin(),v.end());//查找相邻重复,返回第一个迭代器
binary_search();                //二分查找，返回真假,有序序列
count(v.begin(),v.end(),elem)     //统计元素个数
count_if(v.begin(),v.end(),谓词);   //按条件统计元素个数

//排序算法
sort(v.begin(),v.end());            //默认升序
sort(v.begin(),v.end(),greator<int>());//可以自己写降序 
random_shuffle(v1.begin(),v1.end());  //洗牌打乱
merge(v1.begin(),v1.end(),v1.begin(),v2.end(),vtarget.begin());                   //必须是合并有序序列
reverse(v.begin(),v.end()); //反转

//拷贝和替换
copy(v1.begin(),v1.end(),v2.begin()); //拷贝
replace(v1.begin(),v1.end(),旧值,新值);  //查找替换，新值换旧值
replace_if(v1.begin(),v1.end(),谓词,新值);
swap(容器名1,容器名2);

````


## STL常用numeric算法  
````
#include<numeric>               //头文件

//sum
int total = accumulate(v.begin(),v.end(),0);/第三个参数是起始累加值

//fill
fill(v.begin(),v.end(),value)   //用特定值填充特定区间
````

## STL常用集合算法

````
//求交集，将v1和v2容器中元素求交集并放到目标容器中
vector<int>::iterator itend=set_intersection(v1.begin(),v1.end(),v2.begin(),v2.end(),vt.begin());

//求并集，将v1和v2容器中元素求并集并放到目标容器中
vector<int>::iterator itend1=set_union(v1.begin(),v1.end(),v2.begin(),v2.end(),vt2.begin());

//求差集，将v1和v2容器中元素求差集并放到目标容器中
vector<int>::iterator itend2 = set_difference(v1.begin(),v1.end(),v2.begin(),v2.end(),vt3.begin());

````



