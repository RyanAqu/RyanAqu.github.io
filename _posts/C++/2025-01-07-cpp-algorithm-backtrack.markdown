---
layout:     post
title:      "回溯算法"
subtitle:   " \"learning……\""
date:       2025-01-07 09:45:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 回溯算法
    - 基础算法
---

> 就算风吹散了冰雪，想念也会留存下来。

# 算法描述  
回溯法（backtracking）是优先搜索的一种特殊情况，又称为试探法，常用于需要记录节点状态的深度优先搜索。通常来说，排列、组合、选择类问题使用回溯法比较方便。顾名思义，回溯法的核心是回溯。在搜索到某一节点的时候，如果我们发现目前的节点（及其子节点）并不是需求目标时，我们回退到原来的节点继续搜索，并且把在目前节点修改的状态还原。这样的好处是我们可以始终只对图的总状态进行修改，而非每次遍历时新建一个图来储存状态。在具体的写法上，它与普通的深度优先搜索一样，都有 [修改当前节点状态]→[递归子节
点] 的步骤，只是多了回溯的步骤，变成了 [修改当前节点状态]→[递归子节点]→[回改当前节点状态]。

**两个小诀窍，一是按引用传状态，二是所有的状态修改在递归完成后回改。** 回溯法修改一般有两种情况，一种是修改最后一位输出，比如排列组合；一种是修改访问标
记，比如矩阵里搜字符串。

# 全排列  
leecode46  
````
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> result;
        vector<bool> used(nums.size(),false);
        vector<int> current;
        func(nums,used,current,result);

        return result;
    }
private:
void func(vector<int>& nums,vector<bool>& used,vector<int>& current,vector<vector<int>>& result)
{
    if(current.size()==nums.size())
    {
        result.push_back(current);
        return;
    }

    for(int i =0;i<nums.size();i++)
    {
        if(used[i]) continue;
        current.push_back(nums[i]);
        used[i]=true;
        func(nums,used,current,result);
        current.pop_back();
        used[i] = false;
    }
}
};
````

# 组合  
leetcode77  
````
class Solution {
private:
    void bt(int& n, int& k,int start,vector<int>& current,vector<bool>&used,vector<vector<int>>&result)
    {
        if(current.size()==k)
        {
            result.push_back(current);
            return;
        }
        for(int i=start;i<=n;++i)
        {
            if(used[i])continue;
            used[i]=true;
            current.push_back(i);
            bt(n,k,i+1,current,used,result);
            used[i]=false;
            current.pop_back();

        }
    }
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>>result;
        vector<int> current;
        vector<bool>used(n+1,false);
        bt(n,k,1,current,used,result);
        return result;
    }
};
````





# 搜索  
leetcode79  
````
class Solution {
private:
    bool bt(vector<vector<char>>& board, string& word,int index,int row,int column)
    {
        if(index==word.size()) return true;
        if(row<0||column<0||row>=board.size()||column>=board[0].size()||word[index]!=board[row][column]) return false;
        //找到了
        char temp=board[row][column];
        board[row][column]='#';
        bool find=bt(board,word,index+1,row-1,column)||bt(board,word,index+1,row+1,column)||bt(board,word,index+1,row,column-1)||bt(board,word,index+1,row,column+1);
        board[row][column]=temp;
        return find;
    }
public:
    bool exist(vector<vector<char>>& board, string word) {
        for(int i=0;i<board.size();i++)
        {
            for(int j=0;j<board[0].size();j++)
            {
                if(bt(board,word,0,i,j)) return true;
            }
        }
        return false;
    }
};
````




# 放置方案  
leetcode.51.N皇后  
````
class Solution {
    void bt(int& n,int row,vector<string>& board,unordered_set<int>& columns,unordered_set<int>& diagonals1,unordered_set<int>& diagonals2,vector<vector<string>>&result)
    {
        if(row==n)
        {
            result.push_back(board);
            return;
        }
        for(int col=0;col<n;++col)
        {
            if(columns.count(col)||diagonals1.count(row+col)||diagonals2.count(row-col)) continue;

            board[row][col]='Q';
            columns.insert(col);
            diagonals1.insert(row+col);
            diagonals2.insert(row-col);
            bt(n,row+1,board,columns,diagonals1,diagonals2,result);
            board[row][col]='.';
            columns.erase(col);
            diagonals1.erase(row+col);
            diagonals2.erase(row-col);
        }
    }
public:
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>>result;
        vector<string> board(n,string(n,'.'));//初始化棋盘
        unordered_set<int> columns;
        unordered_set<int> diagonals1;
        unordered_set<int> diagonals2;
        bt(n,0,board,columns,diagonals1,diagonals2,result);
        return result;
    }
};
````

# 练习  
leetcode.133.分割回文串  
leetcode.78.子集  


















