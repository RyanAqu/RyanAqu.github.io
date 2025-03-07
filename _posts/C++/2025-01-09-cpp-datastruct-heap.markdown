---
layout:     post
title:      "C++ 堆与优先级队列"
subtitle:   " \"learning……\""
date:       2025-01-09 15:47:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 堆
    - 数据结构
---

> 魔非魔，道非导，善恶在人心。欲非欲，情非情，因缘由天定。

# 堆  
堆是一个**完全二叉树**结构，但是它却存储在一个数组里面。所谓完全二叉树，即这棵树的元素是从左到右从上到下填充的，每一层都是连续存在的。  
将这颗完全二叉树从左到右从上到下填入数组，我们可以发现：一个结点为i的父节点，其子节点下标为2i+1和2i+2；对于一个下标为i的子结点，其父节点为（i-1）/2，由于整形会自动向下取整，所以不论是左子节点还是右子节点，都可以以这个代码得到父节点。  
如果堆每个节点都大于等于它的所有孩子，那么这个堆就是最大堆；如果每个节点都小于等于它的所有孩子，那么这个堆就是最小堆。  

![image](https://github.com/user-attachments/assets/ae1fc246-b29b-4442-906e-354762c47371)  


# 自定义堆  
堆的操作有建堆，堆化（堆下沉），pop  

首先是最核心的堆化操作（这里以**最大堆下沉**为例）：检查父节点是否比所有子节点大，否的话父节点与最大子节点交换，一直重复到堆的最下方  
````
void maxheapdown(vector<int>&nums,int i)
{
    int left=2*i+1;
    int right=2*i+2;
    int largest=i;
    if(left<nums.size()&&nums[largest]<nums[left])
    {
        largest=left;
    }
    if(right<nums.size()&&nums[largest]<nums[right])
    {
        largest=right;
    }
    if(largest!=i) 
    {
        swap(nums[largest],nums[i]);
        maxheapdown(nums,largest);
    }
}
````

建堆的操作就是对所有的父节点进行检查和堆化  
````
void makeheap(vector<int>& nums)
{
    for(int i=nums.size()/2-1;i>=0;i--)
    {
        maxheapdown(nums,i);
    }
}
````

出堆的操作就是交换堆顶到最下方（数组头和尾互换），然后排除数组尾，对堆顶进行一次堆化操作  
````
void heappop(vector<int>& nums)
{
    swap(nums[0],nums[nums.size()-1]);
    nums.pop_back();
    maxheapdown(nums,0);
}
````



# STL的优先级队列使用方法  
priority_queue 是一个容器适配器，默认使用最大堆。如果需要最小堆，可以通过传入一个比较器来改变堆的性质。  
### 默认最大堆  
````
#include <iostream>
#include <queue>

using namespace std;

int main() {
    // 创建一个最大堆
    priority_queue<int> maxHeap;

    // 插入元素
    maxHeap.push(10);
    maxHeap.push(20);
    maxHeap.push(15);

    // 输出堆顶元素（最大元素）
    cout << "最大堆的堆顶元素: " << maxHeap.top() << endl; // 输出: 20

    // 删除堆顶元素
    maxHeap.pop();
    cout << "删除堆顶元素后的堆顶元素: " << maxHeap.top() << endl; // 输出: 15

    return 0;
}

````

### 最小堆  
````
#include <iostream>
#include <queue>
#include <vector>

using namespace std;

int main() {
    // 创建一个最小堆
    priority_queue<int, vector<int>, greater<int>> minHeap;

    // 插入元素
    minHeap.push(10);
    minHeap.push(20);
    minHeap.push(15);

    // 输出堆顶元素（最小元素）
    cout << "最小堆的堆顶元素: " << minHeap.top() << endl; // 输出: 10

    // 删除堆顶元素
    minHeap.pop();
    cout << "删除堆顶元素后的堆顶元素: " << minHeap.top() << endl; // 输出: 15

    return 0;
}

````

# 配合vector使用的堆操作make_heap  
````
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main() {
    vector<int> arr = {10, 20, 15, 30, 40};

    // 创建最大堆
    make_heap(arr.begin(), arr.end());

    // 输出堆顶元素
    cout << "最大堆的堆顶元素: " << arr.front() << endl; // 输出: 40

    // 添加元素
    arr.push_back(50);
    push_heap(arr.begin(), arr.end()); // 将新元素添加到堆中

    cout << "添加元素后堆顶元素: " << arr.front() << endl; // 输出: 50

    // 移除堆顶元素
    pop_heap(arr.begin(), arr.end());
    arr.pop_back();

    cout << "移除堆顶元素后堆顶元素: " << arr.front() << endl; // 输出: 40

    return 0;
}

````



