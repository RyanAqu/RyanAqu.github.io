---
layout:     post
title:      "C++ Catalog"
subtitle:   " \"learning……\""
date:       3024-11-14 20:06:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 基础算法
    - catalog
---

> 生活破破烂烂，小猫尝尝咸淡

这是一篇C++学习的笔记汇总帖，边复习边记录。

**C++ 嵌软**

![image](https://github.com/user-attachments/assets/d664db4e-d397-4c8c-926d-ecfb3097112b)  

**C++ 后端**

![image](https://github.com/user-attachments/assets/2c7ebfa1-32e2-4241-a5c2-405a00b2f7ac)

![image](https://github.com/user-attachments/assets/03e341b8-0629-45cc-a518-f6342cd60bb1)

![image](https://github.com/user-attachments/assets/a0926be0-dc3f-4cff-aba7-c561e2731be3)

![image](https://github.com/user-attachments/assets/cf02b07a-10ca-4754-8375-cd657deee09f)







# C++
[C Basic & C With Classes][0]  
[C++ 泛型 & STL 标准模板库][1]  

# Linux 操作系统  
[Linux 下 Bash 基础命令][2]  

# 基础算法&数据结构
## 双指针
1. 快慢指针
2. 滑动窗口
3. 归并数组
4. 高精度计算  


## 千奇百怪的排序算法  
1. 冒泡排序，无脑两两比较交换，稳定
2. 选择排序，每次遍历将目标值（最大/最小）筛选出来填到预留位置（最前/最后），指针后移，不稳定，选择比冒泡比较次数少
3. 插入排序，指针从第二个位置开始遍历，每次遍历将该数插入到前一段已经排好序的数组（数据后移），稳定，比选择排序比较次数少
4. 希尔排序，插入排序的改进版本，先定gap，一般是长度除二，然后每一轮（gap->全局分组->组内插排->gap/2），gap为1时，整个数组分为一组，相当于进行插排

5. 堆排序
6. 归并排序
7. 快速排序，分治递归，当左右指针不相遇的时候遍历，每次遍历找到支点应在的位置pivotindex，然后进行分治partition，不稳定
8. 桶排序
9. 计数排序

## 一切皆可搜索  
1. 深搜DFS  
2. 广搜BFS  
3. 回溯  


## 刷题记录
1. 二分查找.编号704
   * **递归**实现
   * 循环实现
2. 数组移除相同元素.编号27
   * STL库函数实现
   * **快慢指针**实现
3. 有序数组的平方.编号977
   * 暴力实现
   * 双指针实现
4. 长度最小的子数组.编号209
   * 暴力实现（超时）
   * **滑动窗口实现**
5. 螺旋矩阵.编号59
   * 循环不变量和边界条件
6. 移除链表元素.编号203
   * STL库
   * **创建虚拟表头**
7. 两数之和.编号1
   * 暴力求解n^2复杂度
   * 构建**哈希表**可以将查找复杂度n->1
8. 跳跃游戏.编号45/55
   * **贪心**（很难想到）





[0]:https://ryanaqu.github.io/2024/12/01/cpp-basic-class/
[1]:https://ryanaqu.github.io/2024/12/13/cpp-generic-stl/
[2]:https://ryanaqu.github.io/2024/12/15/cpp-linux/



