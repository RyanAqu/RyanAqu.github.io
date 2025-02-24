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





# 广度优先搜索  












