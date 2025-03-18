---
layout:     post
title:      "各种排序算法"
subtitle:   " \"learning……\""
date:       2025-01-05 09:45:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 基础算法
---


# 千奇百怪的排序算法  

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
10. 拓扑排序，有向无环图的排序，从入度为0的结点开始，删除其节点以及节点所有的有向边，然后重复选择入度为零的节点以及删除的操作，直到没有节点（有环图会有节点剩余）
11. leetcode 75.颜色分类练习


# 常用排序算法  
以下是一些最基本的排序算法。虽然在 C++ 里可以通过 std::sort() 快速排序，而且刷题时很少需要自己手写排序算法，但是熟习各种排序算法可以加深自己对算法的基本理解，以及解出由这些排序算法引申出来的题目。

### STL中sort的底层实现  
快排优化  
* 当排的数过少时，当前递归采用插排（n<32）。
* 当递归深度过深时，转为堆排序。
* 为防止O(n2)退化，采用三数取中法。


### 冒泡排序  
````
void BubbleSort(vector<int>& nums)
{
    for(int i=0;i<nums.size()-1;++i)
    {
        int flag=1;
        for(int j=0;j<nums.size()-i-1;++j)
        {
            if(nums[j]>nums[j+1])
            {
                swap(nums[j],nums[j+1]);
                flag=0;
            }
        }
        if(flag) return;
    }
}
````

### 选择排序  
````
void SelectSort(vector<int>& nums)
{
    int n=nums.size();
    for(int i=0;i<n;i++)
    {
        int minindex=i;
        for(int j=i+1;j<n;j++)
        {
            if(nums[j]<nums[minindex]) minindex=j;
        }
        if(minindex!=i) swap(nums[minindex],nums[i]);
    }
}
````

### 插入排序  
````
void InsertSort(vector<int>& nums)
{
    for(int i =1;i<nums.size();++i)
    {
        int key=nums[i];
        int j=i-1;
        while(j>=0&&nums[j]>key)
        {
            nums[j+1]=nums[j];
            j--;
        }
        nums[j+1]=key;
    }
}
````

### 希尔排序  
````
void ShellSort(vector<int>& nums)
{
    for(int gap=nums.size()/2;gap>0;gap/=2)
    {
        for(int i = gap;i<nums.size();i++)
        {
            int key=nums[i];
            int j=i-gap;
            while(j>=0&&nums[j]>key)
            {
                nums[j+gap]=nums[j];
                j-=gap;
            }
            nums[j+gap]=key;
        }
    }
}
````

### 快排  
````
int partition(vector<int>& nums,int start,int end)
{
    int left=start+1,right=end;
    while(left<right)
    {
        while(nums[left]<=nums[start]&&left<right)
        {
            left++;
        }
        while(nums[right]>=nums[start]&&left<right)
        {
            right--;
        }
        swap(nums[left],nums[right]);
    }
    if(nums[left]<nums[start])
    {
        swap(nums[left],nums[start]);
        return left;
    }
    else
    {
        swap(nums[left-1],nums[start]);
        return left-1;
    }

}
void QuickSort(vector<int>& nums,int left,int right)
{
    if(left<right)
    {
        int pivotindex=partition(nums,left,right);
        QuickSort(nums,pivotindex+1,right);
        QuickSort(nums,left,pivotindex-1);
    }
}
````

### 快速选择  
````

````


### 堆排序  
以leetcode215为例，堆排序解决数组中的第K个最大元素（自己实现的一个堆）  
````
class Solution {
public:
    void maxheapdown(vector<int>& nums,int i,int heapsize)
    {
        int l=2*i+1,r=2*i+2,largest =i;
        if(l<heapsize&&nums[l]>nums[largest])
        {
            largest=l;
        }
        if(r<heapsize&&nums[r]>nums[largest])
        {
            largest=r;
        }
        if(largest!=i)
        {
            swap(nums[largest],nums[i]);
            maxheapdown(nums,largest,heapsize);
        }

    }
    void makeheap(vector<int>& nums,int heapsize)
    {
        for(int i=nums.size()/2-1;i>=0;--i)
        {
            maxheapdown(nums,i,heapsize);
        }
    }
    int findKthLargest(vector<int>& nums, int k) {
        int heapsize=nums.size();
        makeheap(nums,heapsize);

        for(int i=nums.size()-1;i>=nums.size()-k+1;--i)
        {
            swap(nums[i],nums[0]);
            --heapsize;
            maxheapdown(nums,0,heapsize);
        }
        return nums[0];
    }
};
````

### 基数排序  
````
class Solution {
public:
    void sortColors(vector<int>& nums) {
        //基数排序,得到桶使用次数
        int absmax=0;
        for(int num:nums)
        {
            absmax = abs(num)>absmax?abs(num):absmax;
        }
        int len=to_string(absmax).size();

        //开始排序
        vector<vector<int>> bucket;
        int mod=10,dev=1;
        for(int i=0;i<len;i++,dev*=10,mod*=10)
        {
            
            bucket.resize(20);
            for(int j=0;j<nums.size();j++)
            {
                int index=(nums[j]%mod)/dev+10;
                bucket[index].push_back(nums[j]);
            }
            int idx=0;
            for(auto vec:bucket)
            {
                for(int num:vec)
                {
                    nums[idx++] = num;
                }
            }
            bucket.clear();
        }
    }
};
````

### 归并排序  
````
#include <iostream>
#include <vector>

using namespace std;

// 合并两个有序子数组
void merge(vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;

    vector<int> L(n1), R(n2);
    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    int i = 0, j = 0, k = left;

    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) arr[k++] = L[i++];
        else arr[k++] = R[j++];
    }

    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

// 迭代归并排序
void mergeSortIterative(vector<int>& arr) {
    int n = arr.size();
    for (int size = 1; size < n; size *= 2) {  // 1, 2, 4, 8, ...
        for (int left = 0; left < n - size; left += 2 * size) {
            int mid = left + size - 1;
            int right = min(left + 2 * size - 1, n - 1);
            merge(arr, left, mid, right);
        }
    }
}

// 测试
int main() {
    vector<int> arr = {12, 11, 13, 5, 6, 7};
    mergeSortIterative(arr);

    cout << "排序后数组: ";
    for (int num : arr) cout << num << " ";
    return 0;
}
````




# 练习  
215  
347  
451  
75  
























