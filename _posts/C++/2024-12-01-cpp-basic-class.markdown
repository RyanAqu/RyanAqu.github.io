---
layout:     post
title:      "C++ Basic & C With Classes"
subtitle:   " \"learning……\""
date:       2024-12-01 15:00:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog:    true
tags:
    - c++
---

> 冰柜为什么是个箱子，冰箱为什么是个柜子


# C++基础
![C++应用](https://ryanaqu.github.io/img/inpost_cpp/image0.png)  
![代码编译运行](https://ryanaqu.github.io/img/inpost_cpp/image1.png) 
## c++标准开头
#之后的内容是预处理，include代表引入程序库
```
#include <iostream>   //引入标准输入输出流库，用于实现输入输出功能。  
#using namespace std; //声明使用标准命名空间 std 中的所有标识符。

cin.get();            //等待键盘输入，可以防止窗口直接退出

system("pause");      //任意键以继续
```

## 标准输入输出
```
cout<<variable<<endl;   //标准输出，默认类型，默认十进制
cout<<hex;              //改为16进制输出
cout<<oct;              //改为8进制输出
cout<<dec;              //改为10进制输出

cin>>variable;          //输入内容到变量
```


## 编译和链接
### 编译
编译是将源代码（如 .c 或 .cpp 文件）转换为机器无法直接运行的中间文件（通常是目标文件 .o 或 .obj）。

**主要过程：**
* 预处理：由预处理器完成，处理#define，#include等指令，将结果传递给编译器
* 语法分析：检查代码的语法和语义是否正确
* 代码生成：将代码转化为对应平台的汇编代码
* 目标文件生成：将代码进一步转化为机器码，并存储为目标文件

编译的输出通常是目标文件（.o 或 .obj），它包含了机器指令，但这些指令是未链接的，无法独立运行。编译器（如 gcc、g++ 、clang 或 MSVC）。

### 链接
链接是将编译生成的目标文件与库文件结合在一起，生成最终的可执行文件（如 .exe 或 .out）。

**主要过程：**  
* 符号解析：确定所有函数和变量的定义位置（如在库或其他目标文件中）。
* 地址分配：将每个符号分配到实际内存地址。
* 合并段：将多个目标文件中不同类型的段（如 .text、.data）合并为单一的可执行文件。
* 库文件整合：链接静态库或动态库提供的代码。

链接的输出是最终的可执行文件，它可以直接运行在目标系统上。链接器（如 ld 或链接器程序内嵌于 gcc 和 clang 中）。

## 注释
```
//这是单行注释
//以下是多行注释
/**
 * @brief 这是函数描述
 * @param 这是变量描述
 * @return 这是返回值描述
 */
```

## 变量
程序中定义可以修改的量，变量有作用域（scope），一般以{}分隔，{}外定义的变量具有“全局作用域”（global scope），称为“全局变量”（block scope）；{}内具有“块作用域”，称为“局部变量”。

> 如果变量重名的话，作用域更小的变量申明能覆盖其他的变量。

**数值类型：**  
int：整形，4字节  
short：短整型，2字节，-32768~32767  
long：长整型，4字节，一般和int一样长  
long long:更长的整形，至少8字节  
signed：有符号  
unsigned：无符号  
float：单精度浮点型，4字节  
double：双精度浮点型，8字节  
**字符类型：**  
char：字符型，1字节，ASCII码（“A”为65，“0”为48）  
wchar_t：宽字符，2字节  
char16_t:Unicode字符，2字节，16位  
char32_t:Unicode字符，4字节，32位  
**bool类型：**  
true:真，1字节  
false:假，1字节  

```
int a = 10;     // 定义一个整型
float b = 3.14; // 定义一个浮点型
char c = 'A';   // 定义一个字符，通常占一个字节（8bit）
int a,b,c;      // 申明多个变量
bool b1 = true; // 定义布尔类型
sizeof(x);      // 查看变量字节数大小的函数
```
如果要跨平台开发，需要定义uint8_t之类的变量，需要引入头文件
```
#include<stdint.h> //c语言中的头文件，c++可兼容
#include<cstdint>  //c++中的头文件，与c++风格一致，但是需要命名域，譬如std:: uint8_t x = 2;
```
**数值计算原则：**
* 一般的整数计算全部用int
* 如果数值超出了int则用long long
* 确定数值不能为负则用unsigned

## 常量
用于记录程序中不可修改的量，有两种定义方法，①修饰词：const ②宏定义：#define
```
//以下两种都不能够更改变量内容，否则程序会报错
#define day 7       //宏
const int day = 10; //const常量
```
数值标签&字面值
```
cout<<36<<endl;     //输出十进制数36
cout<<036<<endl;    //在数字前面加一个0，为八进制，输出十进制数30
cout<<0x36<<endl;   //在数字前面加一个0x，为十六进制，输出十进制54

int a = 36l;        //l或L标签，表示long类型
int a = 36ll;       //ll或LL标签，表示long long类型
int a = 36u;        //u标签，表示unsigned类型
int a = 36ull;      //表示unsigned long long类型

float a = 36f;      //f或F标签，表示float类型
```
转义字符
```
\r  --  回车
\n  --  换行
\t  --  横向制表
\v  --  纵向制表符
\\  --  反斜杠\
\'  --  单引号'
\"  --  双引号"
\?  --  问号?
\b  --  退格符
```

## 运算符
优先级从高到低：

```
符号            功能            用法                   注意
----------------------------------------------------------------
::            作用域解析     std::cout
----------------------------------------------------------------
()             函数调用
[]             数组下标
.              成员访问
->             成员访问
++(后缀)        自加        (expr)++
--(后缀)        自减        (expr)--
----------------------------------------------------------------
++(前缀)        自加        ++(expr)     这一部分结合性都是从右到左
--(前缀)        自减        --(expr)
+               正号        + expr
-               负号        - expr
!              逻辑非       !(expr)
~              按位取反     ~(expr)
*              解引用       *ptr=&num
&              取地址       *ptr=&num
sizeof         字节大小     sizeof(para)
new            
delete
----------------------------------------------------------------
.*             成员指针访问
.->            成员指针访问
----------------------------------------------------------------
*               乘号        expr * expr
/               除号        expr / expr
%               取余        expr % expr      两个操作数必须都是整数
----------------------------------------------------------------
+               加号        expr + expr
-               减号        expr - expr
----------------------------------------------------------------
<<              左移
>>              右移
----------------------------------------------------------------
<               小于
<=             小于等于
>               大于
>=             大于等于
----------------------------------------------------------------
!=              不等于
==              等于
----------------------------------------------------------------
&               按位与
^              按位异或
|               按位或
----------------------------------------------------------------
&&              逻辑与
||              逻辑或
----------------------------------------------------------------
?:             条件运算符
----------------------------------------------------------------
= += -= *= /= %= <<= >>= &= ^= |=  赋值
```

测试自增自减运算符（前缀和后缀的差异）
```
int i=0,j=10;
j=i++;
cout<<"test i++ : i="<<i<<",j="<<j<<endl;

i=0;
j=10;
j=++i;
cout<<"test ++i : i="<<i<<",j="<<j<<endl;

//输出结果
test i++ : i=1,j=0
test ++i : i=1,j=1
```

短路原则：当逻辑符号左侧的布尔值已经能决定整体逻辑时，右侧的表达式将不被执行计算

## 隐式类型转换
* 整数提升，bool、char、short等变成int  
* 表达式中既有整型又有浮点则会变成浮点
* 逻辑表达式&逻辑判断中其他类型变成bool类型

## 强制类型转换
c++使用static_cast
```
int total = 20, num = 6;
double avg = total / num;           //最终输出3
```
```
int total = 20, num = 6;
double avg = double(total) / num;   //最终输出3.333

double avg = static_cast<double>(total) / num;//最终输出3.333
```

## 条件语句
```
//if 分支
if(condition){
    expr;
}  
else if(condition){
    expr;
}  
else{
    expr;
}  

//switch分支
switch(condition){
    case value1:expr1;break;
    case value2:expr2;break;
    ……
    default:break;
}

```

## 循环语句
```
//for 循环
for(int i=0;i<max_num;i++){
    expr;
}

//范围 for 循环
for(int num:{5,4,6,3,4,5}){
    expr;
}

//while循环  
while(condition){
    expr;
}

//do while
do{
    expr;
}while(condition)
```

## 跳转
```
break;
continue;//结束本次，开启下一次
tag:语句;  goto tag;
return;
```

## 数组
具有相同类型的数据对象有序排列，在内存中连续存放  
```
数据类型 数组名[元素个数]；         //定义数组
int a1[10];                         
const int n=4;double a2[n];       //元素个数可以是常量表达式，但不能是变量

int a3[4]={1,2,3,4};              //数组初始化
float a4[]={2.5,3.8,1.79};
short a5[10] = {3,6,9};
char c[10];

```


## 多维数组
{% raw %}
````
int arr[3][4];                    //多维数组定义
int arr2[2][3][4];

int arr3[3][4]={{1,2,3},{4,5,6}}；//申明
int arr4[][4] = {1,2,3,4,5,6};
int arr5[3][4] = {12};
````
{% endraw %}

## 字符串
1. 动态分配：std::string 能动态调整大小，不需要手动管理内存。
2. 提供丰富的操作方法：拼接、查找、插入、删除、替换等。
3. 自动处理字符串末尾的 \0，不需要手动添加。
4. 支持迭代器，能很好地与 STL 算法配合使用
```
#include <string>

//默认初始化
string s1;                        

//拷贝初始化
string s2 = s1;                   
string s3 = "hello world";

//直接初始化
string s4("hello world");
string s4(8,'h');

//访问字符
str[4];                             //第五个字符
str[str.size()-1];                    //最后一个字符

//转大写字符
s4[i] = toupper(s4[i]);

//字符串拼接
string s1 = "Hello";
string s2 = "World";
string s3 = s1 + ", " + s2 + "!";   // 拼接
s1 += " C++";                       // 追加

//字符串比较
string s1 = "abc";
string s2 = "abd";
if (s1 == s2) {
    cout << "s1 is equal to s2" << endl;
} else if (s1 < s2) {
    cout << "s1 is less than s2" << endl;
} else {
    cout << "s1 is greater than s2" << endl;
}

//字符串大小
s.size();

//查找子串
string s = "Hello, world!";
size_t pos = s.find("world");  // 返回子串的起始位置
if (pos != string::npos) {     // string::npos 表示未找到
    cout << "Found 'world' at position " << pos << endl;
} else {
    cout << "'world' not found" << endl;
}

//插入、删除和替换
string s = "Hello, !";
s.insert(7, "world");  // 在第 7 个位置插入 "world"
cout << s << endl;

s.erase(5, 2);         // 从第 5 个位置开始，删除 2 个字符
cout << s << endl;

s.replace(0, 5, "Hi"); // 从第 0 个位置开始替换 5 个字符为 "Hi"
cout << s << endl;

//子串
string s = "Hello, world!";
string sub = s.substr(7, 5);  // 从第 7 个位置开始，取 5 个字符
cout << "Substring: " << sub << endl;

//C语言字符数组
char str1[5]={'h','e','l','l','o'};
char str2[6]={'h','e','l','l','o','\0'};
cout<<"str1="<<str1<<endl;
cout<<"str2="<<str2<<endl;
char str3[6]="hello";
```

## 读取字符串
```
//cin输入的字符串会以空白符为结尾（包括空格回车换行等）
string str;
cin>>str; 
cout<<str; 

//读取一行信息
string str;
getline(cin,str);
cout<<str<<endl;
```

读取文件中的字符：
```
#include<iostream>
#include<fstream>
using namespace std;

int main(){
    ofstream out("out.txt");
    //按照单词读取
    ifstream file("readfile.txt");
    string word;
    while(file>>word){
        cout<<word<<endl;
        out<<word<<endl;
    }
    //按照行读取
    ifstream file1("readfile.txt");
    string line;
    while(getline(file1,line)){
        cout<<line<<endl;
    }

    //按照字母读取
    ifstream file2("readfile.txt");
    char ch;
    while(file2.get(ch)){
        cout<<ch<<" ";
    }cout<<endl;
    return 0;
}
```
## 结构体
```
struct 结构体名
{
    类型1 数据对象1;
    类型2 数据对象2;
    类型3 数据对象3;
    ...
};
```

```
#include <iostream>
using namespace std;

struct StudentInfo{
    //info
    string name;
    int age;
    double score;

    //operation
    void PrintStudentInfo(){
    cout<<"姓名："<<name<<"\t年龄："<<age<<"\t成绩："<<score<<endl;
    }
}stu3 = {"王五",19,65.0};


int main(){
    //创建对象并初始化
    StudentInfo stu1 = {"张三",18,85.5};
    StudentInfo stu2{"李四",20,90};

    //拷贝初始化
    StudentInfo stu4 = stu3;
    stu4.name = "赵六";

    //结构体数组
    StudentInfo s[3]={
        {"小红",17,85.5},
        {"小白",18,72.5},
        {"小黄",20,66.5}
    };

    //==========打印信息==========//
    stu1.PrintStudentInfo();
    stu2.PrintStudentInfo();
    stu3.PrintStudentInfo();
    stu4.PrintStudentInfo();
    
    for(StudentInfo stu:s) stu.PrintStudentInfo();

    return 0;
}
```
## 枚举
```
enum week{
    Mon,Tue,Wed,Thu,Fri,Sat,Sun
};
```
```
#include<iostream>
using namespace std;

enum Week{
    Mon,Tue,Wed,Thu = 10,Fri,Sat,Sun
};

int main(){
    Week w1 = Mon;
    cout<<"w1="<<w1<<endl;
    Week w2 = Thu;
    cout<<"w2="<<w2<<endl;
    Week w3 = Fri;
    cout<<"w3="<<w3<<endl;
    return 0;
}
```
## 指针
类型* 指针变量名;
```
//定义
int* p1;
long* p2;
long long* p3;

//空指针
int* p = nullptr;

//初始化
int* p = &a;

//解析指针
*p;

//二级指针
int i =1024;
int* p=&i;
int** pp = &p;

//指向常量的指针
const int c=10;
const int* p=&c;

//常量指针(指针不可变)
int* const p = &i;

//指向数组的指针
int(* ap)[5];

```
```
#include <iostream>
using namespace std;

int main(){

    int a = 42;
    int* p = &a;

    int& ref = a;  //ref相当于x的别名

    cout << "a's address: " << &a << endl;
    cout << "Value at p is a's address: " << p << endl;
    cout << "Pointing by value at p is value at a: " << *p << endl;

    cout << "Value at ref: " << ref << endl;
    cout << "ref's address: " << &ref << endl;

    
    return 0;
}
```

## 引用(别名)
```
int a=10;
int& ref = a;

//绑定指针的引用
int* ptr =&a;
int*& pref = ptr;
 
```

## 静态变量
生命周期为整个程序的生命周期，程序不中止则不会销毁  
作用域依旧遵循局部和全局原则
```
static int count = 0;
```

## 函数基础
函数原型：函数申明时只有形参类型，没有形参名
返回类型：函数返回结果的类型  
函数名：函数标识符  
参数列表：输入的数据类型+数据名，用“,”隔开  
函数体：花括号括起来 
传递方式：值传递(拷贝值，拷贝指针地址)，引用传递  
```
//传值 void function(int x);
//将值拷贝
int num = 0;
function(num);

//传指针 void function(int* x);
//将地址拷贝
int num = 0;
function(&num);

//传引用 void function(int& x);
//直接绑定地址操作
int num = 0;
function(num);

//值传数组 以下三种定义方法一样
void function(int* arr);
void function(int arr[]);
void function(int arr[5]);
//值传数组和数组大小
void function(int* arr, int size);

//引用传数组（引用传的时候能直接把数组大小也传进去）
void function(int(& arr)[5]){
    for(int num:arr)
}

//返回数组指针
int (*)[5] fun(int x);

```

## 函数递归
```
//斐波那契数列
#include<iostream>
using namespace std;

int fibo(int n){
    if(n==1||n==2) 
        return 1;
    else 
        return fibo(n-1)+fibo(n-2);
}

int main(){
    
    int num=8;
    int result = fibo(num);
    cout<<"斐波那契数列第"<<num<<"个数为："<<result<<endl;
    return 0;
}
```

## 内联函数
内联函数将函数内代码展开  
```
inline 一个函数
```

## 函数默认实参和调用
注意默认实参只能从右到左给出，不能断续给  
注意函数重载可能会有二义性错误  
```
// 定义函数，给参数 b 和 c 提供默认值
void exampleFunction(int a, int b = 5, int c = 10) {
    cout << "a: " << a << ", b: " << b << ", c: " << c << endl;
}

int main() {
    exampleFunction(1);      // 使用默认值，输出:a:1,b:5,c:10
    exampleFunction(1, 2);   // 覆盖b的默认值，输出:a:1,b:2,c:10
    exampleFunction(1, 2, 3);// 覆盖所有默认值，输出:a:1,b:2,c:3
    return 0;
}
```

## 函数重载
在同一个作用域下，同一个函数名可以定义多次，前提是形参列表不同（参数个数或者类型不同）


## 分离式编译和头文件

## 内存分区模型  
1. 代码区：存放函数体的二进制代码，由操作系统管理（共享、只读）  
2. 全局区：存放全局变量、静态static变量以及常量  
3. 栈区：编译器自动分配释放，存放函数的参数值，局部变量等  
4. 堆区：由程序员分配和释放，若程序员不释放，则程序结束时由操作系统回收,new delete

## 堆区new与delete
```
#include<iostream>
using namespace std;

int* func(){
    int* p = new int(10);
    return p;
}

int* func1(){
    int* arr = new int[10];
    for(int i=0;i<10;i++){
        arr[i]=i;
    }
    return arr;
}

int main(){
    int* p = func();
    cout<<"*p="<<*p<<endl;
    cout<<"p="<<p<<endl;
    delete p;

    int* arr = func1();
    cout<<"arr:";
    for(int i=0;i<10;i++){
        cout<<arr[i]<<" ";
    }
    cout<<endl;
    delete []arr;
    return 0;
}
```  





# Class类
面向对象编程：封装、继承、多态  
```
class 类名{
权限:
    //属性
    //成员属性
    //成员变量

    //行为
    //成员函数
    //成员方法
权限:
};
```  

## 权限
权限类型（默认权限是私有权限）：  
1. public 公共权限：成员 类内类外均可访问
2. protected 保护权限：不涉及继承，和私有没有区别
3. private 私有权限：成员 类内可以访问，类外不可访问  

## 构造函数和析构函数
每写一个类，编译器自动添加至少4个函数：默认无参构造、默认析构、默认拷贝构造（属性值拷贝）和赋值运算符重载operator=（属性值拷贝）  
构造函数生成规则如下：
* 如果用户定义有参构造函数，C++不提供默认无参构造函数，但是提供默认拷贝构造函数
* 如果用户定义拷贝构造函数，C++不会提供其他构造函数

构造函数：创建对象时为对象的成员属性赋值，编译器自动调用  
类名(){}  
1. 没有返回值，不写void  
2. 函数名称和类名相同  
3. 可以有参数，可以重载  
4. 构造对象时自动调用，只调用一次  

析构函数：对象销毁时自动调用，执行一些清理工作  
~类名(){}  
1. 没有返回值，不写void  
2. 函数名称和类名相同，，在名称前加上~  
3. 没有参数，不可以重载  
4. 对象销毁前自动调用析构，只调用一次  

```
#include<iostream>
#include<string>
using namespace std;

class Person{
public:
    Person(){
        cout<<"无参构造函数"<<endl;
    }
    Person(int a){
        age = a;
        cout<<"有参构造函数"<<endl;
    }
    Person(const Person &p){
        age = p.age;
        cout<<"拷贝构造函数"<<endl;
    }
    ~Person(){
        cout<<"析构函数"<<name<<endl;
    }
    string name;
    int age;
protected:
    string car_ID;
private:
    string phone_password;
public:
    void func_init(){
        name="zack";
        car_ID="HXN";
        phone_password="123456";
    }
    void show(){
        cout<<"姓名："<<name<<" 年龄："<<age<<" 车牌："<<car_ID<<" 手机密码："<<phone_password<<endl;

    }
    
};


int main(){

    Person p2(10);          //括号有参构造
    Person p1(p2);          //括号拷贝构造
    p1.func_init();
    p2.func_init();
    p2.name="ryan";
    p1.show();
    p2.show();

    Person p3;              //显示无参构造
    Person p4=Person(10);   //显示有参构造
    Person p5=Person(p3);   //显示拷贝构造

    Person(10);             //匿名构造，在程序结束时立即销毁，注意不要用拷贝构造函数初始化一个匿名对象

    Person p6=10;           //隐式有参构造
    Person p7=p3;           //隐式拷贝构造
    return 0;
}
```

## 深拷贝和浅拷贝
如果定义了一个有参构造函数，其传入的是一个指针，此时没有定义拷贝构造函数，编译器自动定义的拷贝构造函数会在拷贝构造时将指针直接拷贝过来，这样会导致析构函数会对同一个堆地址做多次析构，从而报错。  
总结：如果在堆区申请了空间，一定要自己定义一个拷贝构造函数，以解决浅拷贝带来的问题

## 初始化列表
构造函数形参列表后加一个冒号
```
Person():age(24),name("zack"),car_ID("HXN"),phone_password("123456")
    {
        cout<<"无参构造函数"<<endl;
    }

Person(int a,string b,string c,string d):age(a),name(b),car_ID(c),phone_password(d)
    {
        cout<<"有参构造函数"<<endl;
    }
```

## 类对象作为另一个类的成员
类B包含类A，类B定义对象b时，先构造A，在构造B；先析构B，再析构A。  
```
class A{
public:
    A()
    {
        cout<<"A的构造函数"<<endl;
    }
    ~A()
    {
        cout<<"A的析构函数"<<endl;
    }
};

class B{
public:
    B()
    {
        cout<<"B的构造函数"<<endl;
    }
    ~B()
    {
        cout<<"B的析构函数"<<endl;
    }
    A a;
};
```
## 静态成员static
1. 静态成员变量
    * 所有类的对象都共享一份数据  
    * 在编译阶段分配内存  
    * 类内申明，类外初始化  
2. 静态成员函数
    * 所有类的对象共用一个函数
    * 静态成员函数只能访问静态成员变量

```
//类内申明，类外初始化
class C
{
public: 
    static int xx;
};
int C::xx=100;
```

```
C c1,c2;
cout<<"xx="<<c1.xx<<endl;
c2.xx=200;
cout<<"xx="<<c1.xx<<endl;
```

## 类对象的空间计算  
成员变量和成员函数是分开存储的，非静态成员变量属于类的内存，静态成员变量和所有成员函数都不算入类的大小，用sizeof(对象名)函数观察  
1. 空的类占用一个字节
2. 有非静态成员变量，则空间就是所有非静态成员变量空间之和
3. 静态成员变量不属于类对象的空间占用
4. 非静态成员函数也不属于类对象的空间占用


## this指针
非静态成员函数调用时自动指向对象自己  
用法：
1. 解决名称冲突  
2. 返回对象自身*this  
3. 链式编程思想（返回类型为类型引用，返回*this）

```
//在有参构造函数内
public:
    Person(int age){
        //若写age = age;会错误
        this->age = age;
    }
    int age;
```
## const修饰成员函数
常函数：
1. 成员函数后加const被称为常函数，本质上修饰了this指针
2. 常函数不可以修改成员属性
3. 成员属性声明时加mutable后，在常函数中依然可以修改  

常对象：
1. 声明对象前加const称该对象为常对象
2. 常对象只能调用常函数
3. 成员属性声明时加mutable后，在常对象中依然可以修改 

## 友元friend
1. 全局函数做友元：让全局函数可以访问一个类的私有变量，定义一个全局函数，在类里面申明友元，即可以访问私有成员。friend void goodgay(Building& building);  
2. 类做友元：为了让一个类可以访问另一个类的私有变量friend class GoodGay;  
3. 成员函数做友元friend void GoodGay::visit();
```
class Building
{
    friend void goodgay(Building& building);
    friend class GoodGay;
public:
    Building()
    {   
        cout<<"创建建筑..."<<endl;
        m_SittingRoom="客厅";
        m_BedRoom = "卧室";
    }
public:
    string m_SittingRoom;
private:
    string m_BedRoom;
    
};
//全局函数
void goodgay(Building& building){
    cout<<"好基友全局函数正在访问："<<building.m_SittingRoom<<endl;
    cout<<"好基友全局函数正在访问："<<building.m_BedRoom<<endl;
}
void test03()
{
    Building b;
    goodgay(b);
}


class GoodGay
{
public:
    GoodGay();
    void visit();
    Building* building;
};
GoodGay::GoodGay(){
    cout<<"创建好基友..."<<endl;
    building = new Building;
}
void GoodGay::visit(){
    cout<<"好基友全局函数正在访问："<<building->m_SittingRoom<<endl;
    cout<<"好基友全局函数正在访问："<<building->m_BedRoom<<endl;

}

void test04()
{
    GoodGay gg;
    gg.visit();
}
```
```
#include<iostream>
#include<string>
using namespace std;

class Building;
class GoodGay
{
public:
    GoodGay();
    void visit();
    Building* building;
};


class Building
{
    friend void GoodGay::visit();
public:
    Building();
public:
    string m_SittingRoom;
private:
    string m_BedRoom;
};
Building::Building()
{
    m_SittingRoom="客厅";
    m_BedRoom="卧室";
}

GoodGay::GoodGay()
{
    building  = new Building;
}
void GoodGay::visit()
{
    cout<<"好基友全局函数正在访问："<<building->m_SittingRoom<<endl;
    cout<<"好基友全局函数正在访问："<<building->m_BedRoom<<endl;
}

void test01(){
    GoodGay gg;
    gg.visit();
}

int main(){
    test01();
    return 0;
} 
```

## 运算符重载
用“operator 运算符”替代原本的函数名字，可以重载运算符,运算符重载也可以进行函数重载

### 重载+：  
```
#include<iostream>
#include<string>
using namespace std;
//加号运算符重载
class Person 
{
public:
    int m_A;
    int m_B;
    //1.成员函数重载 + 号:本质是Person p3=p1.operator+(p2)
    //Person operator+(Person& person)
    //{
    //    Person temp;
    //    temp.m_A=this->m_A+person.m_A;
    //    temp.m_B=this->m_B+person.m_B;
    //    return temp;
    //}
    void show()
    {
        cout<<"m_A="<<m_A<<" m_B="<<m_B<<endl;
    }
};

//2.全局函数重载 + 号 本质是Person p3=operator+(p1,p2)
Person operator+(Person& p1,Person& p2)
{
    Person temp;
    temp.m_A=p1.m_A+p2.m_A;
    temp.m_B=p1.m_B+p2.m_B;
    return temp;
}
//函数重载
Person operator+(Person& p1,int num)
{
    Person temp;
    temp.m_A=p1.m_A+num;
    temp.m_B=p1.m_B+num;
    return temp;
}

void test01(){
    Person p1,p2;
    p1.m_A=10;p1.m_B=30;
    p2.m_A=20;p2.m_B=40;
    Person p3=p1+p2;
    p3.show();

    int num=50;
    Person p4=p1+10;
    p4.show();
}

int main(){
    test01();
    return 0;
}
```

### 重载<<：
```
#include<iostream>
using namespace std;
class Person
{
public:
    int m_A;
    int m_B;
};

ostream& operator<<(ostream &cout,Person &p){
    cout<<"m_A="<<p.m_A<<" m_B="<<p.m_B<<endl;
    return cout;
}

void test01(){
    Person p;
    p.m_A=10;
    p.m_B=20;
    cout<<p<<endl;
}
int main(){
    test01();
    return 0;
}
```

### 重载++
```
#include<iostream>
using namespace std;

class MyInterer
{
    friend ostream& operator<<(ostream& out,MyInterer myint);
public:
    MyInterer()
    {
        this->m_Num=0;
    }
    MyInterer(int num)
    {
        this->m_Num=num;
    }
    MyInterer(const MyInterer &myint)
    {
        this->m_Num=myint.m_Num;
    }
    //重载++
    //重载前置++ 返回引用
    MyInterer& operator++()
    {
        m_Num++;
        return *this;
    }
    //重载后置++ 返回值
    MyInterer operator++(int)
    {
        MyInterer temp = *this;
        m_Num++;
        return temp;
    }
private:
    int m_Num;
};

//重载<<
ostream& operator<<(ostream& out,MyInterer myint)
{
    cout<<myint.m_Num;
    return out;
}


void test01()
{
    cout<<"test01:";
    MyInterer myint;
    cout<<++myint<<endl;
    cout<<myint<<endl;
}
void test02()
{
    cout<<"test02:";
    MyInterer myint;
    cout<<myint++<<endl;
    cout<<myint<<endl;
}


int main(){
    test01();
    test02();
    return 0;
}
```

### 重载赋值运算符=
C++编译器在一个类产生的时候至少为其添加4个函数，其中第四个是赋值运算符重载operator=，其作用是对类的对象进行值拷贝，而值拷贝是一个浅拷贝，这就会带来深拷贝和浅拷贝的问题
```
#include<iostream>
using namespace std;

class Person
{
public:
    Person(int age)
    {
        m_Age=new int(age);
    }
    ~Person()
    {
        if(m_Age!=NULL){
            delete m_Age;
            m_Age = NULL;
        }
    }

    //重载=:要记得连等操作,所以要返回Person&
    Person& operator=(Person &p){
        if(m_Age!=NULL){
            delete m_Age;
            m_Age = NULL;
        }
        m_Age=new int(*p.m_Age);

        return *this;
    }

    int* m_Age;
};

void test01()
{
    Person p1(18);
    Person p2(20);
    Person p3(30);
    p3=p2=p1;
    cout<<"p1的年龄"<<*(p1.m_Age)<<endl;
    cout<<"p2的年龄"<<*(p2.m_Age)<<endl;
    cout<<"p3的年龄"<<*(p2.m_Age)<<endl;
}
int main(){
    test01();
    return 0;
}
```


### 重载比较运算符  
```
#include<iostream>
using namespace std;
class Person
{
public:
    Person(string name,int age)
    {
        m_Name=name;
        m_Age=age;
    }
    string m_Name;
    int m_Age;

    bool operator==(Person &p)
    {
        if((this->m_Age==p.m_Age)&&(this->m_Name==p.m_Name))
            return true;
        else
            return false;
    }
    bool operator!=(Person &p)
    {
        if((this->m_Age==p.m_Age)&&(this->m_Name==p.m_Name))
            return false;
        else
            return true;
    }
};

void test01()
{
    Person p1("Tom",18);
    Person p2("Jerry",18);
    if(p1!=p2)
        cout<<"p1和p2是不相等的"<<endl;
    else
        cout<<"p1和p2是相等的"<<endl;

    if(p1==p2)
        cout<<"p1和p2是不相等的"<<endl;
    else
        cout<<"p1和p2是相等的"<<endl;
}

int main(){
    test01();
    return 0;
}
```

### 重载()
非常灵活，记得匿名对象
```
#include<iostream>
#include<string>
using namespace std;

class MyPrint
{
public:
    //重载()
    void operator()(string test)
    {
        cout<<test<<endl;
    }
};

void test01()
{
    MyPrint myprint;
    myprint("hello world");
    MyPrint()("hello world");//匿名对象+运算符重载
}

int main()
{
    test01();
    return 0;
}
```

## 继承inheritance
语法："class 子类:public 父类"  
继承可以减少重复的代码，子类也称为派生类，父类也称为基类  
继承方式：  
1. 公有继承 public
2. 保护继承 protected
3. 私有继承 private

```
class A
{
public:
    int a;
protected:
    int b;
private:
    int c;
};
//公有继承
class B:public A
{
public:
    int a;
protected:
    int b;
不可访问:
    int c; 
}
//保护继承
class B:protected A
{
protected:
    int a;
    int b;
不可访问:
    int c; 
}
//私有继承
class B:private A
{
private:
    int a;
    int b;
不可访问:
    int c; 
}
```
父类中所有的非静态成员都被子类继承了，但是编译器隐藏了私有成员，不允许访问

构造和析构顺序（析构与构造顺序相反）：
1. 若是类嵌套，会先构造内部类，再构造外部类；先析构外部，再析构内部  
2. 若是子类构造，先构造父类，再构造子类；先析构子类，再析构子类  

### 继承中同名成员的处理方式
子类中出现同名成员函数时，会将所有同名成员函数全部隐藏，因此重载函数需要注意
1. 访问子类同名时，直接访问即可s.m_A  
2. 访问父类同名成员时，需要加作用域s.Base::m_A  
3. 静态成员和非静态成员出现同名，处理方式一致，注意类内申明，类外初始化  

### 多继承
语法：class 子类:继承方式1 父类1,继承方式 父类2,....
多继承中出现父类中出现同名成员，需要加作用域区分

### 菱形继承&虚继承
例子：动物类被羊和驼继承，羊驼同时继承羊和驼。但是此时羊驼继承了动物类两次，出现了问题  
解决方法：虚继承,虚继承相当于用指针指向父类  
```
//动物类
class Animal
{
    public:
    int m_Age;
};
//羊类
class Sheep:virtual public Animal
{};
//驼类
class Camel:virtual public Animal
{};
//羊驼类
class Alpaca:public Sheep,public Camel
{};

void test02()
{
    Alpaca yt;
    yt.m_Age = 18;
    cout<<"羊驼的年龄："<<yt.m_Age<<endl;
}
int main()
{
    test02();
    return 0;
}
```

## 多态
分类：  
1. 静态多态：函数重载和运算符重载属于静态多态，复用函数名。地址绑定早，编译阶段确定函数地址  
2. 动态多态：派生类和虚函数实现运行时多态。地址绑定晚，运行阶段确定函数地址，子类要重写父类中的虚函数  
```
//动物类
class Animal
{
public:
    int m_Age;
    virtual void speak()
    {
        cout<<"动物在说话"<<endl;
    }
};
//猫类
class Cat:public Animal
{
public:
    void speak()
    {
        cout<<"小猫在说话"<<endl;
    }
};
//狗类
class Dog:public Animal
{
public:
    void speak()
    {
        cout<<"小狗在说话"<<endl;
    }
};
void doSpeak(Animal &animal)
{
    animal.speak();
}
void test03()
{
    Cat cat;
    doSpeak(cat);
    Dog dog;
    doSpeak(dog);
}
int main()
{
    test03();
    return 0;
}
```

### 纯虚函数和抽象类
父类中定义的虚函数根本用不到，所以直接定义成纯虚函数；只要有纯虚函数，类就称为抽象类，那么这个类就无法实例化对象；抽象类的子类必须重写纯虚函数，否则继承了纯虚函数的类只能作为抽象类，也无法实例化对象，写法如下：
```
//动物类
class Animal
{
public:
    virtual void speak() = 0; //纯虚函数写法->虚函数直接等于0
    int m_Age;
};
```
### 虚析构函数和纯虚析构函数
用来解决父类指针指向子类对象出现的问题：在父类为抽象类的时候，使用父类指针指向子类产生多态；但是父类的指针在释放时无法调用子类的析构代码，当子类有内存开辟到堆区时就会产生风险。

用法：同样，在析构函数前加virtual修饰词，注意纯虚析构不仅需要申明，还需要实现。

