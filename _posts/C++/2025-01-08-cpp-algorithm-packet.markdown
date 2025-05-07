---
layout:     post
title:      "背包问题"
subtitle:   " \"learning……\""
date:       2025-01-08 22:55:00
author:     "Ryan"
header-img: "img/bg-walle.jpg"
catalog: true
tags:
    - c++
    - 背包问题
    - 基础算法
---

> 因为生活，没有如果

# 01背包  
````
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    int N, W;
    cin >> N >> W;

    vector<int> weights(N), values(N);
    for (int i = 0; i < N; ++i) {
        cin >> weights[i] >> values[i];
    }

    // dp[i][j] 表示前 i 个物品在容量为 j 时的最大价值
    vector<vector<int>> dp(N + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= N; ++i) {
        for (int j = 0; j <= W; ++j) {
            if (j < weights[i - 1])
            {
                dp[i][j] = dp[i - 1][j]; // 不选第 i 个物品
            }
            else
            {
                dp[i][j] = max(dp[i - 1][j],                  // 不选
                               dp[i - 1][j - weights[i - 1]] + values[i - 1]); // 选
            }
        }
    }

    cout << dp[N][W] << endl;  // 输出最大价值
    return 0;
}

````



# 完全背包
`````
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    int N, W;
    cin >> N >> W;

    vector<int> weights(N), values(N);
    for (int i = 0; i < N; ++i) {
        cin >> weights[i] >> values[i];
    }

    // dp[i][j] 表示前 i 个物品在容量为 j 时的最大价值
    vector<vector<int>> dp(N + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= N; ++i) {
        for (int j = 0; j <= W; ++j) {
            if (j < weights[i - 1])
            {
                dp[i][j] = dp[i - 1][j]; // 不选第 i 个物品
            }
            else
            {
                dp[i][j] = max(dp[i - 1][j],                  // 不选
                               dp[i][j - weights[i - 1]] + values[i - 1]); // 选
            }
        }
    }

    cout << dp[N][W] << endl;  // 输出最大价值
    return 0;
}
`````


