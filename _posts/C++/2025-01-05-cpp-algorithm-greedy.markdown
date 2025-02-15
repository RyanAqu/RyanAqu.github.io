---
layout:     post
title:      "贪心算法(Greedy Algorithm)"
subtitle:   " \"learning……\""
date:       2025-01-05 08:45:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 基础算法
---

# 算法描述  
贪心算法或贪心思想采用贪心的策略，保证每次操作都是局部最优的，从而使最后得到的结果是全局最优的。**不回溯，直接选择。**  

步骤：
1. 初始化：设定初始状态。
2. 选择：在当前状态下选择一个局部最优的解。
3. 问题约化：根据选择的局部最优解，减小问题的规模。
4. 终止条件：检查是否已经到达问题的终止条件，若是，则返回结果；否则，返回第2步。

贪心算法能应用的问题：  
1. 活动选择问题：给定多个活动的开始时间和结束时间，选择最大数量的活动，使得它们互不重叠。贪心策略：选择最早结束的活动。
2. 最小生成树（Kruskal / Prim 算法）：在图中找到一棵最小生成树。
3. 哈夫曼编码：用于数据压缩的编码方法。
4. 背包问题（部分可以用贪心）：比如分数背包问题，可以通过选择最大价值比（价值/重量）来进行贪心选择。
5. 最短路径问题（Dijkstra算法）： 在加权图中，Dijkstra算法通过每次选择当前最短路径的节点来找到最短路径，属于贪心算法。

# 分配问题  
leetcode.455.分配饼干：有一群孩子和一堆饼干，每个孩子有一个饥饿度，每个饼干都有一个大小。每个孩子只能吃最多一个饼干，且只有饼干的大小大于孩子的饥饿度时，这个孩子才能吃饱。求解最多有多少孩子可以吃饱。**贪心策略：** 给剩余孩子里最小饥饿度的孩子分配最小的能饱腹的饼干。  
````
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(),g.end());
        sort(s.begin(),s.end());
        int child=0;int cookies=0;
        while(child<g.size()&&cookies<s.size())
        {
            if(g[child]<=s[cookies]) child++;
            cookies++;
        }
        return child;

    }
};
````

leetcode.135.分发糖果：一群孩子站成一排，每一个孩子有自己的评分。现在需要给这些孩子发糖果，规则是如果一个孩子的评分比自己身旁的一个孩子要高，那么这个孩子就必须得到比身旁孩子更多的糖果；所有孩子至少要有一个糖果。求解最少需要多少个糖果。**贪心策略：** 把所有孩子的糖果数初始化为 1；先从左往右遍历一遍，如果右边孩子的评分比左边的高，则右边孩子的糖果数更新为左边孩子的糖果数加 1；再从右往左遍历一遍，如果左边孩子的评分比右边的高，且左边孩子当前的糖果数不大于右边孩子的糖果数，则左边孩子的糖果数更新为右边孩子的糖果数加 1。通过这两次遍历，分配的糖果就可以满足题目要求了。这里的贪心策略即为，从左到右贪心一遍，从右到左贪心一遍。  

````
class Solution {
public:
    int candy(vector<int>& ratings) {
        int result=ratings.size();
        if(result<2) return result;

        vector<int>num(result,1);
        for(int i=1;i<ratings.size();i++)
        {
            if(ratings[i]>ratings[i-1]) 
            {
                num[i]=num[i-1]+1;
            }
        }
        for(int i=ratings.size()-1;i>0;i--)
        {
            if(ratings[i]<ratings[i-1])
            {
                num[i-1]=max(num[i-1],num[i]+1);
            }
        }
        return accumulate(num.begin(),num.end(),0);
    }
};
````

# 区间问题  
leetcode.435.无重叠区间：给定多个区间，计算让这些区间互不重叠所需要移除区间的最少个数。起止相连不算重叠。  
**题解：** 在选择要保留区间时，区间的结尾十分重要：选择的区间结尾越小，余留给其它区间的空间就越大，就越能保留更多的区间。因此，我们采取的贪心策略为，优先保留结尾小且不相交的区间。具体实现方法为，先把区间按照结尾的大小进行增序排序，每次选择结尾最小且和前一个选择的区间不重叠的区间。我们这里使用 C++ 的Lambda，结合std::sort() 函数进行自定义排序。在样例中，排序后的数组为 [[1,2], [1,3], [2,4]]。按照贪心策略，首先初始化为区间[1,2]；由于 [1,3] 与 [1,2] 相交，我们跳过该区间；由于 [2,4] 与 [1,2] 不相交，我们将其保留。因此最终保留的区间为 [[1,2], [2,4]]。  
````
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        if(intervals.empty()) return 0;
        auto lambda = [](vector<int>&a,vector<int>&b){return a[1]<b[1];};
        sort(intervals.begin(),intervals.end(),lambda);
        int total=0;vector<int> temp=intervals[0];
        for(int i=1;i<intervals.size();++i)
        {
            if(intervals[i][0]<temp[1])
            {
                total++;
            }
            else
            {
                temp=intervals[i];
            }
        }
        return total;
    }
};
````

# 练习题  
**基础：**
* 605种花问题
* 452类435
* 763划分字母区间
* 122股票买卖

**进阶：**
* 406根据身高重建队列
* 665非递减数列











