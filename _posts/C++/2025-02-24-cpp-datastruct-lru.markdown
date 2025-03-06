---
layout:     post
title:      "C++ LRU"
subtitle:   " \"learning……\""
date:       2025-02-24 14:00:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - LRU
    - 数据结构
---

> 人总是贪婪的，就像最开始，我也只是想知道你的名字。


# LRU  
LRU（Least Recently Used，最近最少使用）缓存是一种常见的缓存淘汰策略，主要用于保持固定容量的缓存，当缓存满时，会淘汰最近最少使用的数据。  

### 实现  
实现思路：  
* 用哈希表（unordered_map）存储键值对，实现 O(1) 访问。
* 用双向链表（list）维护访问顺序，最新访问的元素放到链表头，最近最少使用的元素在链表尾部。
* 当缓存满时，删除链表尾部的元素。

基本操作：  
* get：如果 key 存在，返回值，并将其移动到链表头部。  
* put(key,value)：更新 key 值，如果 key 已存在，则移动到链表头；如果 key 不存在，则插入到头部，若超出容量，删除链表尾部元素。  

### demo  
````
class LRUCache {
private:
    int capacity;
    list<pair<int,int>> cacheList;
    unordered_map<int,list<pair<int,int>>::iterator>cacheMap;
public:
    LRUCache(int capacity) {
        this->capacity=capacity;
    }
    
    int get(int key) {
        if(cacheMap.find(key)!=cacheMap.end()) 
        {
            auto it=cacheMap[key];
            cacheList.push_front({it->first,it->second});
            cacheList.erase(it);
            cacheMap[key]=cacheList.begin();
            return cacheList.front().second;
        }
        return -1; 

    }
    
    void put(int key, int value) {
        if(cacheMap.find(key)!=cacheMap.end())
        {
            cacheList.erase(cacheMap[key]);
            cacheList.push_front({key,value});
            cacheMap[key]=cacheList.begin();
            
        }
        else
        {
            cacheList.push_front({key,value});
            cacheMap[key]=cacheList.begin();
            if(cacheList.size()>capacity)
            {
                cacheMap.erase(cacheList.back().first);
                cacheList.pop_back();
            }
        }
    }
};

````











