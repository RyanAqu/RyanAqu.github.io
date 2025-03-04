---
layout:     post
title:      "数制&码制"
subtitle:   " \"learning……\""
date:       2024-11-13 22:57:00
author:     "Ryan"
header-img: "img/post-bg-dcb.jpg"
catalog: true
tags:
    - DigitalCircuitBasis
    - 数制&码制
---

> 情感激励大师&青春疼痛文学作者

# 数制
## 各进制
### 十进制
$(xxx)_{10}$ 或 $(xxx)_D$

数码：0，1，2，……，9  
位权： ${10}^{i}$  
基数：10  
进位：逢十进一

### 二进制
$(xxx)_{2}$ 或 $(xxx)_B$

数码：0，1  
位权： ${2}^{i}$  
基数：2  
进位：逢二进一

### 八进制
$(xxx)_{8}$ 或 $(xxx)_O$

数码：0，1，2，……，7  
位权： ${8}^{i}$  
基数：8  
进位：逢八进一

### 十六进制
$(xxx)_{16}$ 或 $(xxx)_H$

数码：0，1，2，……，9，A,……，F  
位权： ${16}^{i}$  
基数：16  
进位：逢十六进一

## 各进制转十进制
$(xxx)_D=\sum数码*位权$

## 十进制转其他进制
十进制转二进制遵循以下原则：
* 整数部分采用“除基取余法”
* 小数部分采用“乘基取整法”
![](https://ryanaqu.github.io/img/inpost_dcb/dec2any.png)

## 二、八、十六进制快转
注意：数左右两边补零

# 码制
## 8421BCD码&5421BCD码&2421BCD码  
![](https://ryanaqu.github.io/img/inpost_dcb/dcb_codes.png)

$(36)_D=(0011\ \ 0110) _{8421BCD}$  
$(4.79)_D=(0100.\ \ 0111\ \ 1001) _{8421BCD}$  
$(10)_D=(0001\ \ 0000) _{8421BCD}$  

## 格雷码
见Verilog格雷编码器
## 奇偶校验码
~~见Verilog~~

# 数电基础剩下的内容不再更新了
因为有很多课程内容讲述的很好，而作为一门基础课，我不想把时间浪费在为它“做课件”上，所以我只学习，但不会在做笔记。  
具体课件可见：[数字电路网课][1]



[1]:https://www.icourse163.org/spoc/learn/NJUST-1449961194?tid=1463291480#/learn/content

