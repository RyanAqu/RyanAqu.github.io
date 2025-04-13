---
layout:     post
title:      "置换环"
subtitle:   " \"learning……\""
date:       2025-01-08 22:45:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 置换环
    - 基础算法
---

> 不想前进的时候，就暂且停下脚步吧。

# 基于置换环的最小交换次数计算  
**置换环：一般用于解决数组排序元素间所需最小交换次数这类问题。**
图论中的置换问题。数组元素的位置错乱可以看作是一种“置换”，每个置换可以分解成若干个置换环（permutation cycles）。置换环思想：置换环将每个元素指向其排序后应在的位置，最终首位相连形成一个环（若数字在最终位置，则其自身成环），可知元素之间的交换只会在同一个环内进行。    
* 对于一个长度为𝑘的环，只需交换𝑘−1次即可排好序。
* 所以总交换次数 = 所有环的(长度−1)之和。

### 例子
````
arr = [4, 3, 2, 1]   // 下标：0  1  2  3

位置 0 → 元素 4 → 应该去位置 3
位置 1 → 元素 3 → 应该去位置 2
位置 2 → 元素 2 → 应该去位置 1
位置 3 → 元素 1 → 应该去位置 0

即位置
0 → 3 → 0    // 第一个环（环中有两个元素，需要交换1次）
1 → 2 → 1    // 第二个环（环中有两个元素，需要交换1次）
所以排序这个数组最少需要1+1=2次交换

````

### demo  
只需要遍历一遍，就可以计算出排序最少所需交换数。  
````
int minSwapsToSort(const vector<int>& arr) { 
    int n = arr.size();
    vector<bool> visited(n, false);
    int swaps = 0;
    
    for (int i = 0; i < n; i++) {
        if (visited[i])
            continue;
        int cycleSize = 0;
        int j = i;
        while (!visited[j]) {
            visited[j] = true;
            j = arr[j] - 1;
            cycleSize++;
        }
        if (cycleSize > 0)
            swaps += (cycleSize - 1);
    }
    return swaps;
}
````








