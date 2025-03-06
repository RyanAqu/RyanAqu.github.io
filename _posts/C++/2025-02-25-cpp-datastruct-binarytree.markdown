---
layout:     post
title:      "C++ 二叉树和搜索树"
subtitle:   " \"learning……\""
date:       2025-02-25 14:00:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 二叉树和搜索树
    - 数据结构
---

> 虽有遗憾，并无后悔。

# 二叉树的遍历  
### 递归实现三序遍历  
````
#include <iostream>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// **前序遍历（递归）**
void preorder(TreeNode* root) {
    if (!root) return;
    cout << root->val << " "; // 访问根节点
    preorder(root->left);     // 递归遍历左子树
    preorder(root->right);    // 递归遍历右子树
}

// **中序遍历（递归）**
void inorder(TreeNode* root) {
    if (!root) return;
    inorder(root->left);      // 递归遍历左子树
    cout << root->val << " "; // 访问根节点
    inorder(root->right);     // 递归遍历右子树
}

// **后序遍历（递归）**
void postorder(TreeNode* root) {
    if (!root) return;
    postorder(root->left);    // 递归遍历左子树
    postorder(root->right);   // 递归遍历右子树
    cout << root->val << " "; // 访问根节点
}

// 测试代码
int main() {
    TreeNode* root = new TreeNode(1);
    root->left = new TreeNode(2);
    root->right = new TreeNode(3);
    root->left->left = new TreeNode(4);
    root->left->right = new TreeNode(5);
    
    cout << "前序遍历: "; preorder(root); cout << endl;
    cout << "中序遍历: "; inorder(root); cout << endl;
    cout << "后序遍历: "; postorder(root); cout << endl;
    
    return 0;
}

````

### 非递归实现三序遍历  
````
//前序遍历
#include <stack>
void preorder(TreeNode* root) {
    if (!root) return;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        cout << node->val << " ";
        if (node->right) st.push(node->right); // 先右后左，保证左子树先遍历
        if (node->left) st.push(node->left);
    }
}
//中序遍历
#include <stack>
void inorder(TreeNode* root) {
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { // 先访问左子树
            st.push(cur);
            cur = cur->left;
        }
        cur = st.top(); st.pop();
        cout << cur->val << " "; // 访问根节点
        cur = cur->right; // 访问右子树
    }
}
//后序遍历
#include <stack>
void postorder(TreeNode* root) {
    if (!root) return;
    stack<TreeNode*> st;
    stack<int> output; // 辅助栈
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        output.push(node->val);
        if (node->left) st.push(node->left);
        if (node->right) st.push(node->right);
    }
    while (!output.empty()) {
        cout << output.top() << " ";
        output.pop();
    }
}

````

### 层序遍历  
````
//BFS
#include <queue>
void levelOrder(TreeNode* root) {
    if (!root) return;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        TreeNode* node = q.front(); q.pop();
        cout << node->val << " ";
        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
}

````


















