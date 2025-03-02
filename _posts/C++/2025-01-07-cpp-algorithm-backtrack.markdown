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






