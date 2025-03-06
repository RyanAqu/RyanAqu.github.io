---
layout:     post
title:      "C++ Trie前缀树"
subtitle:   " \"learning……\""
date:       2025-02-23 14:00:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 前缀树
    - 数据结构
---

> 只要耐心等待，裂缝总会出现。


# Trie前缀树  
Trie（前缀树）是一种用于高效存储和搜索字符串的树形数据结构，适用于字典、自动补全、IP路由表、拼写检查等应用。

### Trie基本结构  
* 每个节点：代表字符串的一个前缀。
* 根节点：为空，代表空前缀 ""。
* 边：代表字符，从根到某个节点的路径代表一个字符串。
* 终止标志：标记某个字符串是否完整存在于 Trie 中。

````
         (root)
        /      \
      a         b
      |         |
      p         a
      |         |
      p         t  (终止)
      |
      l
      |
      e  (终止)
````

### Trie基本操作  
* 插入单词
  * 从根节点开始，逐字符检查是否存在该字符的子节点。
  * 若不存在，则创建新节点。
  * 遍历完整个单词后，标记结尾节点 isEnd = true。
* 搜索单词
  * 从根节点出发，逐字符查找路径。
  * 若找到 isEnd = true，说明该单词存在，否则不存在。
* 判断是否存在前缀
  * 从根节点出发，逐字符查找路径。
  * 只要能走到该前缀的最后一个字符，就说明存在这个前缀。

### demo  
````
#include <iostream>
using namespace std;

struct TrieNode {
    TrieNode* children[26]; // 26 个英文字母
    bool isEnd; // 是否是一个完整单词的结尾

    TrieNode() { 
        isEnd = false; 
        for (int i = 0; i < 26; i++) 
            children[i] = nullptr;
    }
};

class Trie {
private:
    TrieNode* root;

public:
    Trie() { root = new TrieNode(); }

    // 插入单词
    void insert(string word) {
        TrieNode* node = root;
        for (char c : word) {
            int index = c - 'a'; // 计算字母索引
            if (!node->children[index]) 
                node->children[index] = new TrieNode();
            node = node->children[index];
        }
        node->isEnd = true; // 标记单词结尾
    }

    // 搜索单词是否存在
    bool search(string word) {
        TrieNode* node = root;
        for (char c : word) {
            int index = c - 'a';
            if (!node->children[index]) 
                return false;
            node = node->children[index];
        }
        return node->isEnd; // 只有到达终止节点才算找到完整单词
    }

    // 检查是否存在某个前缀
    bool startsWith(string prefix) {
        TrieNode* node = root;
        for (char c : prefix) {
            int index = c - 'a';
            if (!node->children[index]) 
                return false;
            node = node->children[index];
        }
        return true; // 只要能走到这个前缀的最后一个字符，就算前缀存在
    }
};

int main() {
    Trie trie;
    trie.insert("apple");
    cout << trie.search("apple") << endl;  // 1（找到）
    cout << trie.search("app") << endl;    // 0（未找到完整单词）
    cout << trie.startsWith("app") << endl; // 1（存在前缀）
    trie.insert("app");
    cout << trie.search("app") << endl;    // 1（现在找到了）
    return 0;
}

````


### 简易实施  
leetcode208  
````
class Trie {
private:
    struct TrieNode{
        unordered_map<char,TrieNode*>children;
        bool isEnd;
        TrieNode():isEnd(false){};
    };
    TrieNode * root;
public:
    Trie() {
        root=new TrieNode();
    }
    
    void insert(string word) {
        TrieNode* node=root;
        for(char c:word)
        {
            if(!node->children.count(c))
            {
                node->children[c]=new TrieNode();
            }
            node=node->children[c];
        }
        node->isEnd=true;
    }
    
    bool search(string word) {
        TrieNode* node =root;
        for(char c:word)
        {
            if(!node->children.count(c))
            {
                return false;
            }
            node=node->children[c];
        }
        return node->isEnd;
    }
    
    bool startsWith(string prefix) {
        TrieNode* node=root;
        for(char c:prefix)
        {
            if(!node->children.count(c)) return false;
            node=node->children[c];
        }
        return true;
    }
};

````




