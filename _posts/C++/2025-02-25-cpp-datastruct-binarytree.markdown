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

# 二叉搜索树BST  
二叉搜索树是一种特殊的二叉树，它满足以下条件：
1. 对于树中的每一个节点 N，N 的左子树中所有节点的值小于 N 的值。
2. N 的右子树中所有节点的值大于 N 的值。
3. 每个节点的左子树和右子树也是二叉搜索树。
4. 这种结构的优点是能够在logn时间内进行查找、插入和删除操作。
5. 左<N<右  

### BST的基本操作  
1. 插入（Insert）：插入一个新节点时，从根节点开始遍历树。如果新节点的值小于当前节点的值，就进入左子树；否则，进入右子树，直到找到一个空位置插入新节点。
2. 查找（Search）查找一个节点时，也是从根节点开始遍历。根据值的大小关系，向左子树或右子树递归查找，直到找到目标节点或遍历完整棵树。
3. 删除（Delete）删除一个节点时，有三种情况：
   * 节点没有子节点：直接删除该节点。
   * 节点有一个子节点：用该子节点替代当前节点。
   * 节点有两个子节点：找到该节点的中序后继（右子树最小节点），将其替换为当前节点，再删除后继节点。
4. 遍历（Traversal）常见的遍历方式有：
   * 中序遍历：左子树 -> 根节点 -> 右子树（按照升序遍历）
   * 前序遍历：根节点 -> 左子树 -> 右子树
   * 后序遍历：左子树 -> 右子树 -> 根节点

### demo  
`````
#include <iostream>
using namespace std;

// 定义二叉搜索树节点结构体
struct TreeNode {
    int val;
    TreeNode *left, *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class BST {
public:
    TreeNode* root;

    BST() : root(nullptr) {}

    // 插入节点
    void insert(int value) {
        root = insert(root, value);
    }

    // 查找节点
    bool search(int value) {
        return search(root, value);
    }

    // 删除节点
    void remove(int value) {
        root = remove(root, value);
    }

    // 中序遍历
    void inorder() {
        inorder(root);
    }

private:
    // 插入节点的递归函数
    TreeNode* insert(TreeNode* node, int value) {
        if (node == nullptr) {
            return new TreeNode(value);
        }
        if (value < node->val) {
            node->left = insert(node->left, value);
        } else {
            node->right = insert(node->right, value);
        }
        return node;
    }

    // 查找节点的递归函数
    bool search(TreeNode* node, int value) {
        if (node == nullptr) {
            return false;
        }
        if (node->val == value) {
            return true;
        } else if (value < node->val) {
            return search(node->left, value);
        } else {
            return search(node->right, value);
        }
    }

    // 删除节点的递归函数
    TreeNode* remove(TreeNode* node, int value) {
        if (node == nullptr) {
            return node;
        }

        // 查找要删除的节点
        if (value < node->val) {
            node->left = remove(node->left, value);
        } else if (value > node->val) {
            node->right = remove(node->right, value);
        } else {
            // 找到节点
            if (node->left == nullptr) {
                TreeNode* temp = node->right;
                delete node;
                return temp;
            } else if (node->right == nullptr) {
                TreeNode* temp = node->left;
                delete node;
                return temp;
            }

            // 节点有两个子节点，找中序后继
            TreeNode* temp = minValueNode(node->right);
            node->val = temp->val;
            node->right = remove(node->right, temp->val);
        }
        return node;
    }

    // 找到最小节点
    TreeNode* minValueNode(TreeNode* node) {
        TreeNode* current = node;
        while (current && current->left != nullptr) {
            current = current->left;
        }
        return current;
    }

    // 中序遍历
    void inorder(TreeNode* node) {
        if (node == nullptr) {
            return;
        }
        inorder(node->left);
        cout << node->val << " ";
        inorder(node->right);
    }
};

int main() {
    BST tree;
    tree.insert(50);
    tree.insert(30);
    tree.insert(20);
    tree.insert(40);
    tree.insert(70);
    tree.insert(60);
    tree.insert(80);

    cout << "Inorder traversal: ";
    tree.inorder();  // 中序遍历

    cout << "\nSearch 40: " << (tree.search(40) ? "Found" : "Not Found") << endl;

    tree.remove(20);
    cout << "Inorder traversal after deleting 20: ";
    tree.inorder();  // 删除节点20后，进行中序遍历

    return 0;
}

`````


# 平衡二叉树AVL树  
平衡二叉树（AVL树）是一种自平衡的二叉搜索树，它要求树中任意一个节点的左右子树的高度差（平衡因子）不能大于1。换句话说，任何一个节点的左子树和右子树的高度差绝对值不超过1。  

### 平衡因子  
平衡因子：每个节点都有一个平衡因子（Balance Factor, BF），它是该节点的左子树的高度减去右子树的高度。  

### AVL的操作  
* 旋转操作：为了保持平衡，AVL树在插入或删除节点后，如果出现不平衡的情况，需要通过旋转操作来恢复平衡。旋转有四种类型：  
    * 左旋：当右子树比左子树高时，进行左旋转操作。该操作将不平衡节点的右子节点提升到该节点的位置，原来的节点成为左子节点。
    * 右旋：当左子树比右子树高时，进行右旋转操作。该操作将不平衡节点的左子节点提升到该节点的位置，原来的节点成为右子节点。
    * 左右旋：当节点的左子树不平衡，并且左子树的右子树比左子树的左子树高时，进行左右旋转。先对左子树进行左旋，再对当前节点进行右旋。
    * 右左旋：当节点的右子树不平衡，并且右子树的左子树比右子树的右子树高时，进行右左旋。先对右子树进行右旋，再对当前节点进行左旋。
* 插入操作
    * 插入一个新的节点时，遵循二叉搜索树的规则。
    * 插入节点后，向上回溯调整树的平衡。如果出现不平衡，执行相应的旋转操作来恢复平衡。（插入只需要调整最近的一个祖先）
* 删除操作
    * 删除一个节点时，遵循二叉搜索树的规则。
    * 删除节点后，同样需要向上回溯，检查每个节点的平衡因子并做必要的旋转。（删除后要依次向上查询调整所有祖先）

### demo  
`````
#include <iostream>
using namespace std;

// 二叉树节点定义
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    int height;
    
    TreeNode(int x) : val(x), left(nullptr), right(nullptr), height(1) {}
};

class AVLTree {
public:
    TreeNode* root;
    
    AVLTree() : root(nullptr) {}

    // 获取节点的高度
    int height(TreeNode* node) {
        if (!node) return 0;
        return node->height;
    }

    // 更新节点的高度
    void updateHeight(TreeNode* node) {
        node->height = max(height(node->left), height(node->right)) + 1;
    }

    // 获取节点的平衡因子
    int getBalance(TreeNode* node) {
        return height(node->left) - height(node->right);
    }

    // 右旋操作
    TreeNode* rightRotate(TreeNode* y) {
        TreeNode* x = y->left;
        TreeNode* T2 = x->right;
        
        // 进行旋转
        x->right = y;
        y->left = T2;

        // 更新节点的高度
        updateHeight(y);
        updateHeight(x);

        return x;
    }

    // 左旋操作
    TreeNode* leftRotate(TreeNode* x) {
        TreeNode* y = x->right;
        TreeNode* T2 = y->left;
        
        // 进行旋转
        y->left = x;
        x->right = T2;

        // 更新节点的高度
        updateHeight(x);
        updateHeight(y);

        return y;
    }

    // 插入节点
    TreeNode* insert(TreeNode* node, int val) {
        // 1. 进行普通的二叉搜索树插入
        if (!node) return new TreeNode(val);
        
        if (val < node->val) {
            node->left = insert(node->left, val);
        } else if (val > node->val) {
            node->right = insert(node->right, val);
        } else {
            return node;  // 重复值不插入
        }

        // 2. 更新节点的高度
        updateHeight(node);

        // 3. 获取平衡因子并判断是否需要旋转
        int balance = getBalance(node);

        // 如果节点不平衡，执行相应的旋转操作
        
        // 左左（Left-Left）情况
        if (balance > 1 && val < node->left->val) {
            return rightRotate(node);
        }

        // 右右（Right-Right）情况
        if (balance < -1 && val > node->right->val) {
            return leftRotate(node);
        }

        // 左右（Left-Right）情况
        if (balance > 1 && val > node->left->val) {
            node->left = leftRotate(node->left);
            return rightRotate(node);
        }

        // 右左（Right-Left）情况
        if (balance < -1 && val < node->right->val) {
            node->right = rightRotate(node->right);
            return leftRotate(node);
        }

        return node;
    }

    // 插入节点的接口
    void insert(int val) {
        root = insert(root, val);
    }

    // 中序遍历
    void inorder(TreeNode* node) {
        if (!node) return;
        inorder(node->left);
        cout << node->val << " ";
        inorder(node->right);
    }

    void inorder() {
        inorder(root);
    }
};

int main() {
    AVLTree tree;
    tree.insert(30);
    tree.insert(20);
    tree.insert(10);
    tree.insert(25);
    tree.insert(5);
    tree.insert(40);
    tree.insert(35);
    
    cout << "Inorder traversal of the AVL tree: ";
    tree.inorder();  // 输出中序遍历结果
    
    return 0;
}

`````

# 红黑树  
红黑树是一种自平衡的二叉查找树，它确保了树的高度不会超过2log⁡𝑛，从而保证了操作的最坏时间复杂度为𝑂(log𝑛)。与 AVL 树相比，红黑树允许不那么严格的平衡要求，从而使得插入和删除操作更高效。

### 红黑树的性质  
* 每个节点要么是红色，要么是黑色。
* 根节点是黑色的。
* 每个叶子节点（NIL节点）是黑色的。
* 如果一个红色节点有子节点，那么它的两个子节点都是黑色的。即没有两个连续的红色节点。
* 从任意节点到其所有后代叶子节点的路径上，黑色节点的个数必须相同。这就意味着，任何一条路径从根到叶子节点或 NIL 节点所经过的黑色节点的数量相同，这个数量被称为黑色高度。

### 红黑树的操作  
* 插入操作：
    * 插入一个新节点后，它会被插入为红色节点。
    * 插入后，如果不满足红黑树的性质，就通过旋转和重新着色来恢复这些性质。
* 删除操作：
    * 删除节点后，可能会破坏红黑树的平衡性，尤其是黑色节点的数量。
    * 需要通过旋转和着色来修复树的平衡。
* 旋转操作：
    * 左旋：将当前节点的右子树提升到当前节点的位置，将当前节点降为左子节点。
    * 右旋：将当前节点的左子树提升到当前节点的位置，将当前节点降为右子节点。


### 红黑树的插入修复
插入新节点后，红黑树的性质可能被破坏，特别是第 4 条和第 5 条。此时，我们需要通过旋转和着色操作来恢复平衡。插入的修复过程可以分为以下几种情况：
* 新插入的节点是根节点：只需要将其颜色设为黑色。
* 新插入的节点的父节点是黑色的：不需要任何修复。
* 新插入的节点的父节点是红色的：
    * 如果叔叔节点是红色：将父节点和叔叔节点的颜色改为黑色，将祖父节点的颜色改为红色，然后向上继续修复。
    * 如果叔叔节点是黑色或不存在：需要旋转和调整颜色来恢复红黑树的平衡

### demo  
````
#include <iostream>
using namespace std;

enum Color { RED, BLACK };

struct Node {
    int data;
    Node* left;
    Node* right;
    Node* parent;
    Color color;

    Node(int val) : data(val), left(nullptr), right(nullptr), parent(nullptr), color(RED) {}
};

class RedBlackTree {
private:
    Node* root;
    Node* TNULL;  // 辅助空节点，所有叶子节点和NULL节点都指向这个节点

    // 左旋操作
    void leftRotate(Node* x) {
        Node* y = x->right;
        x->right = y->left;
        if (y->left != TNULL) {
            y->left->parent = x;
        }
        y->parent = x->parent;
        if (x->parent == nullptr) {
            root = y;
        } else if (x == x->parent->left) {
            x->parent->left = y;
        } else {
            x->parent->right = y;
        }
        y->left = x;
        x->parent = y;
    }

    // 右旋操作
    void rightRotate(Node* x) {
        Node* y = x->left;
        x->left = y->right;
        if (y->right != TNULL) {
            y->right->parent = x;
        }
        y->parent = x->parent;
        if (x->parent == nullptr) {
            root = y;
        } else if (x == x->parent->right) {
            x->parent->right = y;
        } else {
            x->parent->left = y;
        }
        y->right = x;
        x->parent = y;
    }

    // 插入修复
    void fixInsert(Node* k) {
        Node* u;
        while (k->parent->color == RED) {
            if (k->parent == k->parent->parent->right) {
                u = k->parent->parent->left;
                if (u->color == RED) {
                    u->color = BLACK;
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    k = k->parent->parent;
                } else {
                    if (k == k->parent->left) {
                        k = k->parent;
                        rightRotate(k);
                    }
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    leftRotate(k->parent->parent);
                }
            } else {
                u = k->parent->parent->right;
                if (u->color == RED) {
                    u->color = BLACK;
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    k = k->parent->parent;
                } else {
                    if (k == k->parent->right) {
                        k = k->parent;
                        leftRotate(k);
                    }
                    k->parent->color = BLACK;
                    k->parent->parent->color = RED;
                    rightRotate(k->parent->parent);
                }
            }
            if (k == root) {
                break;
            }
        }
        root->color = BLACK;
    }

    // 插入节点
    void insert(int key) {
        Node* node = new Node(key);
        Node* y = nullptr;
        Node* x = root;

        while (x != TNULL) {
            y = x;
            if (node->data < x->data) {
                x = x->left;
            } else {
                x = x->right;
            }
        }

        node->parent = y;
        if (y == nullptr) {
            root = node;
        } else if (node->data < y->data) {
            y->left = node;
        } else {
            y->right = node;
        }
        node->left = TNULL;
        node->right = TNULL;
        node->color = RED;

        fixInsert(node);
    }

    // 中序遍历
    void inorderHelper(Node* root) {
        if (root != TNULL) {
            inorderHelper(root->left);
            cout << root->data << " ";
            inorderHelper(root->right);
        }
    }

public:
    RedBlackTree() {
        TNULL = new Node(0);
        TNULL->color = BLACK;
        root = TNULL;
    }

    // 插入接口
    void insertValue(int key) {
        insert(key);
    }

    // 打印中序遍历
    void inorder() {
        inorderHelper(root);
    }
};

int main() {
    RedBlackTree tree;

    tree.insertValue(55);
    tree.insertValue(40);
    tree.insertValue(65);
    tree.insertValue(60);
    tree.insertValue(50);
    tree.insertValue(45);

    cout << "Inorder traversal of the Red-Black Tree: ";
    tree.inorder();
    cout << endl;

    return 0;
}

````


























