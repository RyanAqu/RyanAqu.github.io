---
layout:     post
title:      "双指针"
subtitle:   " \"learning……\""
date:       2025-01-05 10:45:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 基础算法
---

# 算法描述  
双指针主要用于遍历数组，两个指针指向不同的元素，从而协同完成任务。也可以延伸到多个数组的多个指针。  
若两个指针指向同一数组，遍历方向相同且不会相交，则也称为滑动窗口（两个指针包围的区域即为当前的窗口），经常用于区间搜索。  
若两个指针指向同一数组，但是遍历方向相反，则可以用来进行搜索，待搜索的数组往往是排好序的。  

双指针常有以下应用：  
 1. 滑动窗口
 2. 数组归并
 3. 快慢指针
 4. 高精度计算

# 两数之和题  
leetcode.167.两数之和：在一个增序的整数数组里找到两个数，使它们的和为给定值。因为数组已经排好序，我们可以采用方向相反的双指针来寻找这两个数字，一个初始指向最
小的元素，即数组最左边，向右遍历；一个初始指向最大的元素，即数组最右边，向左遍历。  
````
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int l=0,r=numbers.size()-1,sum;
        while(l<r)
        {
            sum=numbers[l]+numbers[r];
            if(sum==target)break;
            else if(sum>target) --r;
            else ++l;
        }
        return {l+1,r+1};
    }
};
````

# 归并两个有序数组EZ  
leetcode.88：输入是两个数组和它们分别的长度 m 和 n。其中第一个数组的长度被延长至 m + n，多出的n 位被 0 填补。题目要求把第二个数组归并到第一个数组上，不需要开辟额外空间.  
````
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int p=m+n-1,i=m-1,j=n-1;
        while(i>=0&&j>=0)
        {
            if(nums1[i]>nums2[j])
            {
                nums1[p]=nums1[i];
                i--;
                p--;
            }
            else
            {
                nums1[p]=nums2[j];
                j--;
                p--;
            }
        }
        while(i>=0)
        {
            nums1[p]=nums1[i];
            i--;
            p--;
        }
        while(j>=0)
        {
            nums1[p]=nums2[j];
            j--;
            p--;
        }
    }
};
````

# 快慢指针  
leetcode.142快慢指针：给定一个链表，如果有环路，找出环路的开始点。对于链表找环路的问题，有一个通用的解法——**快慢指针（Floyd 判圈法）**。给定两个指针，分别命名为 slow 和 fast，起始位置在链表的开头。每次 fast 前进两步，slow 前进一步。如果 fast可以走到尽头，那么说明没有环路；如果 fast 可以无限走下去，那么说明一定有环路，且一定存在一个时刻 slow 和 fast 相遇。当 slow 和 fast 第一次相遇时，我们将 fast 重新移动到链表开头，并让 slow 和 fast 每次都前进一步。当 slow 和 fast 第二次相遇时，相遇的节点即为环路的开始点。  
````
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if(head==NULL) return NULL;
        ListNode *slow=head,*fast=head;
        
        do
        {
            if(!fast||!fast->next) return NULL;
            fast=fast->next->next;
            slow=slow->next;
        }while(fast!=slow);
        fast=head;
        while(fast!=slow)
        {
            fast=fast->next;
            slow=slow->next;
        }
        return fast;
    }
};
````

# 滑动窗口(很难)  
leetcode.76.最小覆盖字串：给定两个字符串 S 和 T，求 S 中包含 T 所有字符的最短连续子字符串的长度，同时要求时间复杂度不得超过 O(n)。  
1. 统计目标频数，做成hash映射target，用于查询。
2. 创建窗口映射，用于实时记录。
3. 窗口**不断向右扩展**，直到到达源字符串末尾
   1. 每次扩展首先将新char记录进window，right指针右移。
   2. 做一个判断：去target找新char是否需要关注，若是且频数相等，说明窗口内已经包含target所需的一个字符，那么valid++（valid是target内字符达到计数的个数）。
   3. 当valid计数满足target要求（valid==target.size()），**不断尝试**缩减窗口
      1. 检查当前窗口是否比之前记录的短（minnum），如果短则更新minnum，更新最短窗口开始的位置start=left
      2. 保存left指向的char，left++缩短窗口，检查缩短是否造成valid减小
4. 返回结果


````
class Solution {
public:
    string minWindow(string s, string t) {
        //创建目标（目标集合中的字母顺序不重要，重要的是字母及其频数）
        unordered_map<char,int> target;
        for(char c:t)
        {
            target[c]++;
        }
        //创建窗口集合用来统计窗口内字母频数
        unordered_map<char,int> window;
        //窗口
        int left=0,right=0,valid=0,start=0,minnum=INT_MAX;
        while(right<s.size())
        {
            char newchar=s[right];
            window[newchar]++;
            right++;
            if(target.find(newchar)!=target.end()&&window[newchar]==target[newchar])
            {
                valid++;
            }
            while(valid==target.size())
            {
                if(minnum>right-left)
                {
                    minnum=right-left;
                    start=left;
                }
                //尝试缩短
                char c=s[left];
                left++;
                if(target.find(c)!=target.end())
                {
                    if(target[c]==window[c])
                    {
                        valid--;
                    }
                    window[c]--;
                }
            }
        }
        if(minnum==INT_MAX) return "";
        return s.substr(start,minnum);

    }
};
````

# 练习  
633.Two Sum 题目的变形题之一。  
680.Two Sum 题目的变形题之二。  
524.归并两个有序数组的变形题。  
340.至多包含 K 个不同字符的最长子串。  





