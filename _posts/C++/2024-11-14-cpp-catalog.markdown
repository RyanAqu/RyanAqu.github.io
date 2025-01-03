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

# C++ 方向和学习路线

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
[Linux 下 Bash 命令基础][2]  
[Linux 下 C++ 编译基础][3]  

# 基础算法&数据结构
### 双指针
1. 快慢指针
2. 滑动窗口
3. 归并数组
4. 高精度计算  


### 千奇百怪的排序算法  

![image](https://github.com/user-attachments/assets/6520bf80-977f-4b7a-b0d1-95d88cf049d4)

![image](https://github.com/user-attachments/assets/0a194524-cf51-420a-9224-8c69c5a058f0)


1. 冒泡排序，无脑两两比较交换，稳定
2. 选择排序，每次遍历将目标值（最大/最小）筛选出来填到预留位置（最前/最后），指针后移，不稳定，选择比冒泡比较次数少
3. 插入排序，指针从第二个位置开始遍历，每次遍历将该数插入到前一段已经排好序的数组（数据后移），稳定，比选择排序比较次数少
4. 希尔排序，插入排序+分组，先定gap，一般是长度除二，然后每一轮（gap->全局分组->组内插排->gap/2），gap为1时，整个数组分为一组，相当于进行插排
5. 快速排序，分治递归（二叉树），左右指针不相遇，每次遍历选取基准数，将大于基准数的排到右边，小于的排到左边，返回基准数位置pivotindex，然后进行分治partition
6. 归并排序，分治递归（彻底的二分法分区），先序列化分（二分到只有一个），再进行元素的有序合并（数据合并时进行排序）
7. 堆排序，使用了最小堆（最大堆）优先级队列技术。先把数组最大堆化，然后每次循环取出最大堆堆顶放置末尾，末尾放置堆头，然后从堆头开始进行最大堆下沉（末尾排除）
8. 桶排序，先根据数据范围，划分多个桶，将数据分别入桶，桶内排序方式自主选择，排完后数据合并即可
9. 基数排序，数以链表形式排列，创建基数个桶（比如十进制数是0-9总共十个桶），遍历链表放入个位对应基数的桶，然后遍历桶，将链表重新连接；再对更高位进行分桶和合并
10. leetcode 75.颜色分类练习

### 一切皆可搜索  
1. 深搜DFS，分为树的搜索和图的搜索，一般通过递归实现，或者栈的实现  
2. 广搜BFS，分为树的搜索和图的搜索，一般用队列实现  
3. djkstra  
4. 回溯  


### 刷题记录
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
[3]:https://ryanaqu.github.io/2025/01/02/cpp-compile/



