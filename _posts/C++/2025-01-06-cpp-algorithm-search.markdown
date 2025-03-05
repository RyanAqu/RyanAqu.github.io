---
layout:     post
title:      "搜索算法"
subtitle:   " \"learning……\""
date:       2025-01-06 09:45:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 搜索算法
    - 基础算法
---

> 人类的赞歌就是勇气的赞歌。  

# 算法描述  
深度优先搜索（DFS）和广度优先搜索（BFS）是两种最常见的优先搜索方法，它们被广泛地运用在图和树等结构中进行搜索。  
DFS非递归实现可以利用stack，BFS非递归实现可以利用queue。  

# 深度优先搜索  
深度优先搜索（depth-first seach，DFS）在搜索到一个新的节点时，立即对该新节点进行遍历；因此遍历需要用先入后出的栈来实现，也可以通过与栈等价的递归来实现。对于树结构而言，由于总是对新节点调用遍历，因此看起来是向着“深”的方向前进。考虑如下一颗简单的树。我们从 1 号节点开始遍历，假如遍历顺序是从左子节点到右子节点，那么按照优先向着“深”的方向前进的策略，假如我们使用递归实现，我们的遍历过程为 1（起始节点）->2（遍历更深一层的左子节点）->4（遍历更深一层的左子节点）->2（无子节点，返回父结点）->1（子节点均已完成遍历，返回父结点）->3（遍历更深一层的右子节点）->1（无子节点，返回父结点）-> 结束程序（子节点均已完成遍历）。如果我们使用栈实现，我们的栈顶元素的变化过程为 1->2->4->3。  
````
     1
    / \
   2   3
  /
 4
````

深度优先搜索也可以用来**检测环路**：记录每个遍历过的节点的父节点，若一个节点被再次遍历且父节点不同，则说明有环。我们也可以用之后会讲到的**拓扑排序**判断是否有环路，若最后存在入度不为零的点，则说明有环。有时我们可能会需要对已经搜索过的节点进行标记，以防止在遍历时重复搜索某个节点，这种做法叫做状态记录或记忆化（memoization）。  

### 最大岛屿面积  
leecode695：ez，先写dfs，返回条件是当前坐标不在二维数组范围内，或者当前区域已被探索；然后不断上下左右扩张探索，将探索过的区域标记，并且面积++，面积是一个引用传递。边界判定判断当前点是否在区域内的函数很简单，随便写写。最后主函数内遍历整个数组。  

递归写法：  
````
class Solution {
public:
    bool inArea(vector<vector<int>>& grid,int row,int column)
    {
        return row>=0&&row<grid.size()&&column>=0&&column<grid[0].size();
    }

    void dfs(vector<vector<int>>& grid,int row,int column,int& maxsize)
    {
        if(!inArea(grid,row,column))
        {
            return;
        }
        if(grid[row][column]!=1)
        {
            return;
        }
        grid[row][column]=2;
        maxsize++;
        dfs(grid,row-1,column,maxsize);
        dfs(grid,row+1,column,maxsize);
        dfs(grid,row,column-1,maxsize);
        dfs(grid,row,column+1,maxsize);

    }
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int res=0;
        for(int i=0;i<grid.size();++i)
        {
            for(int j=0;j<grid[0].size();++j)
            {
                if(grid[i][j]==1)
                {
                    int maxsize=0;
                    dfs(grid,i,j,maxsize);
                    res=max(res,maxsize);
                }
            }
        }
        return res;
    }
};
````

栈写法：  
````
class Solution {
public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        vector<int> direction{-1, 0, 1, 0, -1};
        int m=grid.size();
        int n=m?grid[0].size():0;
        int local_area,area=0,x,y;
        for (int i = 0; i < m; ++i) 
        {
            for (int j = 0; j < n; ++j) 
            {
                if(grid[i][j])
                {
                    local_area = 1;
                    grid[i][j]=0;
                    stack<pair<int,int>> island;
                    island.push({i,j});
                    while(!island.empty())
                    {
                        auto[r,c]=island.top();
                        island.pop();
                        for(int k=0;k<4;++k)
                        {
                            x = r + direction[k], y = c + direction[k+1];
                            if(x >= 0 && x < m &&y >= 0 && y < n && grid[x][y] == 1) 
                            {
                                grid[x][y] = 0;
                                ++local_area;
                                island.push({x, y});
                            }

                        }
                    }
                    area = max(area, local_area);
                }
            }
        }
        return area;
    }
};
````  
### 练习  
547  
417  


# 广度优先搜索  
广度优先搜索（breadth-first search，BFS）不同与深度优先搜索，它是一层层进行遍历的，因此需要用先入先出的队列而非先入后出的栈进行遍历。由于是按层次进行遍历，广度优先搜索时按照“广”的方向进行遍历的，也常常用来处理最短路径等问题。考虑如下一颗简单的树。我们从 1 号节点开始遍历，假如遍历顺序是从左子节点到右子节点，那么按照优先向着“广”的方向前进的策略，队列顶端的元素变化过程为 [1]->[2->3]->[4]，其中方括号代表每一层的元素。  
````
     1
    / \
   2   3
  /
 4
````
 
这里要注意，深度优先搜索和广度优先搜索都可以处理可达性问题，即从一个节点开始是否能达到另一个节点。因为深度优先搜索可以利用递归快速实现，很多人会习惯使用深度优先搜索刷此类题目。实际软件工程中，笔者很少见到递归的写法，因为一方面难以理解，另一方面可能产生栈溢出的情况；而用栈实现的深度优先搜索和用队列实现的广度优先搜索在写法上并没有太大差异，因此使用哪一种搜索方式需要根据实际的功能需求来判断。  

### 基础BFS  
leetcode.934.最短的桥  
给定一个二维 0-1 矩阵，其中 1 表示陆地，0 表示海洋，每个位置与上下左右相连。已知矩阵中有且只有两个岛屿，求最少要填海造陆多少个位置才可以将两个岛屿相连。  

`````
class Solution {
    vector<vector<int>> directions={  { -1,0},{1,0},{0,-1},{0,1}};
public:
    int shortestBridge(vector<vector<int>>& grid) {
        int steps=0;
        queue<pair<int,int>>q;
        //找第一个岛
        for(int i=0;i<grid.size();++i)
        {
            for(int j=0;j<grid[0].size();++j)
            {
                if(grid[i][j]==1) 
                {
                    dfs(grid,i,j,q);
                    goto end;
                }
            }
        }
        end://跳出多重for循环  
        while(!q.empty())
        {
            int size=q.size();
            while(size--)
            {
                auto [x, y] = q.front();
                q.pop();
                for(auto& d:directions)
                {
                    int nx=x+d[0],ny=y+d[1];
                    if(nx>=0&&ny>=0&&nx<grid.size()&&ny<grid[0].size())//未越界
                    {
                        if(grid[nx][ny]==1)return steps;
                        if(grid[nx][ny]==0)
                        {
                            grid[nx][ny]=2;
                            q.push({nx,ny});
                        }
                        
                    }
                }
            }
            steps++;
        }
        return steps;
        
    }
    void dfs(vector<vector<int>>& grid,int i,int j,queue<pair<int,int>>&q)
    {
        if(i<0||j<0||i>=grid.size()||j>=grid[0].size()||grid[i][j]!=1)return;//越界或者是水
        grid[i][j]=2;//标记已访问
        q.push({i,j});
        dfs(grid,i-1,j,q);
        dfs(grid,i+1,j,q);
        dfs(grid,i,j-1,q);
        dfs(grid,i,j+1,q);
    }
};

`````

### 单词接龙  
leetcode.126.单词接龙  
给定一个起始字符串和一个终止字符串，以及一个单词表，求是否可以将起始字符串每次改一个字符，直到改成终止字符串，且所有中间的修改过程表示的字符串都可以在单词表里找到。若存在，输出需要修改次数最少的所有更改方式。  

`````
class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string> &wordList) {
        vector<vector<string>> res;
        // 因为需要快速判断扩展出的单词是否在 wordList 里，因此需要将 wordList 存入哈希表，这里命名为「字典」
        unordered_set<string> dict = {wordList.begin(), wordList.end()};
        // 修改以后看一下，如果根本就不在 dict 里面，跳过
        if (dict.find(endWord) == dict.end()) {
            return res;
        }
        // 特殊用例处理
        dict.erase(beginWord);

        // 第 1 步：广度优先搜索建图
        // 记录扩展出的单词是在第几次扩展的时候得到的，key：单词，value：在广度优先搜索的第几层
        unordered_map<string, int> steps = {  {beginWord, 0} };
        // 记录了单词是从哪些单词扩展而来，key：单词，value：单词列表，这些单词可以变换到 key ，它们是一对多关系
        unordered_map<string, set<string>> from = {{beginWord, {} } };
        int step = 0;
        bool found = false;
        queue<string> q = queue<string>{{beginWord}};
        int wordLen = beginWord.length();
        while (!q.empty()) {
            step++;
            int size = q.size();
            for (int i = 0; i < size; i++) {
                const string currWord = move(q.front());
                string nextWord = currWord;
                q.pop();
                // 将每一位替换成 26 个小写英文字母
                for (int j = 0; j < wordLen; ++j) {
                    const char origin = nextWord[j];
                    for (char c = 'a'; c <= 'z'; ++c) {
                        nextWord[j] = c;
                        if (steps[nextWord] == step) {
                            from[nextWord].insert(currWord);
                        }
                        if (dict.find(nextWord) == dict.end()) {
                            continue;
                        }
                        // 如果从一个单词扩展出来的单词以前遍历过，距离一定更远，为了避免搜索到已经遍历到，且距离更远的单词，需要将它从 dict 中删除
                        dict.erase(nextWord);
                        // 这一层扩展出的单词进入队列
                        q.push(nextWord);
                        // 记录 nextWord 从 currWord 而来
                        from[nextWord].insert(currWord);
                        // 记录 nextWord 的 step
                        steps[nextWord] = step;
                        if (nextWord == endWord) {
                            found = true;
                        }
                    }
                    nextWord[j] = origin;
                }
            }
            if (found) {
                break;
            }
        }
        // 第 2 步：回溯找到所有解，从 endWord 恢复到 beginWord ，所以每次尝试操作 path 列表的头部
        if (found) {
            vector<string> Path = {endWord};
            backtrack(res, endWord, from, Path);
        }
        return res;
    }

    void backtrack(vector<vector<string>> &res, const string &Node, unordered_map<string, set<string>> &from,
             vector<string> &path) {
        if (from[Node].empty()) {
            res.push_back({path.rbegin(), path.rend()});
            return;
        }
        for (const string &Parent: from[Node]) {
            path.push_back(Parent);
            backtrack(res, Parent, from, path);
            path.pop_back();
        }
    }
};

`````

### 练习  
130  
257  
47  
40  
37  
310  




