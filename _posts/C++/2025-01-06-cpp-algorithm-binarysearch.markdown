---
layout:     post
title:      "二分查找"
subtitle:   " \"learning……\""
date:       2025-01-06 11:45:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 基础算法
---

> 两个囚犯从监狱里往外看，一个看到了泥土，一个看到了星星......

# 算法描述  
二分查找也常被称为二分法或者折半查找，每次查找时通过将待查找区间分成两部分并只取一部分继续查找，将查找的复杂度大大减少。对于一个长度为 O(n) 的数组，二分查找的时间复杂度为 O(log n)。

````
#include <iostream>
#include <vector>

using namespace std;

// 二分查找函数
int binarySearch(const vector<int>& arr, int target) {
    int left = 0;
    int right = arr.size() - 1;

    while (left <= right) {
        // 计算中间位置
        int mid = left + (right - left) / 2;

        // 检查中间元素是否是目标值
        if (arr[mid] == target) {
            return mid;  // 返回找到的元素的索引
        }
        // 如果目标值在右半部分
        else if (arr[mid] < target) {
            left = mid + 1;
        }
        // 如果目标值在左半部分
        else {
            right = mid - 1;
        }
    }

    // 如果没有找到目标值，返回 -1
    return -1;
}

int main() {
    vector<int> arr = {1, 3, 5, 7, 9, 11, 13, 15, 17, 19};  // 已排序数组
    int target = 7;

    int result = binarySearch(arr, target);

    if (result != -1) {
        cout << "元素 " << target << " 的索引是: " << result << endl;
    } else {
        cout << "元素 " << target << " 不在数组中。" << endl;
    }

    return 0;
}

````

# 查找区间  
leetcode.34.在排序数组中查找元素的第一个和最后一个位置：给定一个增序的整数数组和一个值，查找该值第一次和最后一次出现的位置。  

二分查找变形（找到目标值之后向右或者向左重新二分查找）：  
````
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int>result={-1,-1};
        return{binarys(nums,0,nums.size()-1,target,false),binarys(nums,0,nums.size()-1,target,true)};

    }
    int binarys(vector<int>& nums,int left,int right,int target,bool index)
    {
        int result=-1;
        while(left<=right)
        {
            int mid=(left+right)/2;
            if(nums[mid]==target)
            {
                result=mid;
                if(index)
                {
                    left=mid+1;
                }
                else
                {
                    right=mid-1;
                }
            }
            else if(nums[mid]>target)
            {
                right=mid-1;
            }
            else
            {
                left=mid+1;
            }
        }
        return result;
    }
};
````

# 旋转数组查数字  
leecode.81.搜索旋转排序数组II：一个原本增序的数组被首尾相连后按某个位置断开。给定一个值，判断这个值是否存在于这个为旋转数组中。  







