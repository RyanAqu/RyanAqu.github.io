---
layout:     post
title:      "C++ 并查集"
subtitle:   " \"learning……\""
date:       2025-02-22 14:11:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 并查集
    - 数据结构
---

> 你有一千个名字念在嘴边，却只是为了掩盖心中的那一个......


# 并查集  
并查集（Union-Find）是一种数据结构，主要用于处理一些不交集（Disjoint Sets）的合并及查询问题。并查集通常用于图论中的连通性问题，如判断两个元素是否属于同一集合、合并两个集合等。并查集通常使用**路径压缩**和**按秩合并（按大小合并）** 两种优化技术，使得查询和合并操作的时间复杂度接近常数时间，通常为 O(α(n))，其中α为阿克曼函数的反函数，增长速度非常慢。  

核心思想：在最初，每个元素都是一个独立的“帮派”，即每个元素的parent都是自己（**初始化**）。然后根据个体间的联系，独立的个体合并成一个大的“帮派”，合并的方式是，所有的个体元素认一个元素为“帮派老大”，即parent\[i\]指向唯一的“老大”（**合并操作**），具体表现为两个元素之间竞争“帮派老大”。在合并的过程中，需要查询合并的两个独立个体元素是否在一个“帮派”中，也就是是否有同一个“老大”，查询的同时进行路径压缩，即所有人的“老大”更新为同一个（**查询操作和路径压缩**）。

### 并查集的基本操作  
* find(x)：查找元素 x 所在的集合，返回该集合的代表元。
* union(x, y)：将元素 x 和元素 y 合并到同一集合中。
* connected(x, y)：判断元素 x 和元素 y 是否在同一集合中。

### demo  
````
#include <iostream>
#include <vector>
using namespace std;

class UnionFind {
private:
    vector<int> parent;
    vector<int> rank;  // 用于按秩合并，避免树形结构过高

public:
    // 构造函数，初始化 parent 数组和 rank 数组
    UnionFind(int size) {
        parent.resize(size);
        rank.resize(size, 0);  // 初始时所有节点的秩为 0
        for (int i = 0; i < size; ++i) {
            parent[i] = i;  // 每个节点的父节点是它自己
        }
    }

    // 查找操作，带路径压缩
    int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);  // 路径压缩
        }
        return parent[x];
    }

    // 合并操作，按秩合并
    void unionSets(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);

        if (rootX != rootY) {
            // 按秩合并，确保树的高度最小
            if (rank[rootX] > rank[rootY]) {
                parent[rootY] = rootX;
            } else if (rank[rootX] < rank[rootY]) {
                parent[rootX] = rootY;
            } else {
                parent[rootY] = rootX;
                rank[rootX]++;  // 如果秩相同，合并时根的秩加 1
            }
        }
    }

    // 判断两个元素是否属于同一集合
    bool connected(int x, int y) {
        return find(x) == find(y);
    }
};

int main() {
    UnionFind uf(10);  // 创建一个包含 10 个元素的并查集

    // 合并一些集合
    uf.unionSets(1, 2);
    uf.unionSets(2, 3);
    uf.unionSets(4, 5);
    uf.unionSets(6, 7);

    // 判断集合的连接性
    cout << "1 and 3 connected: " << uf.connected(1, 3) << endl;  // 输出 1 (true)
    cout << "1 and 4 connected: " << uf.connected(1, 4) << endl;  // 输出 0 (false)

    uf.unionSets(3, 4);
    cout << "1 and 4 connected after union: " << uf.connected(1, 4) << endl;  // 输出 1 (true)

    return 0;
}

````








