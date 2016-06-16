---
title: LeetCode--动态规划
tags: LeetCode,算法,动态规划
grammar_cjkRuby: true
---

[TOC]

##  115. Distinct Subsequences
题目：对于字符串 `s`，有多少种方法移除几个字符串能得到 `t`
思路：
```bash?linenums
/*
Solution (DP):
We keep a m*n matrix and scanning through string S, while m = T.length() + 1 and n = S.length() + 1 and each cell in matrix Path[i][j] means the number of distinct subsequences of 
T.substr(1...i) in S(1...j)
  
Path[i][j] = Path[i][j-1]            (discard S[j])
               +     Path[i-1][j-1]    (S[j] == T[i] and we are going to use S[j])
                  or 0                 (S[j] != T[i] so we could not use S[j])
while Path[0][j] = 1 and Path[i][0] = 0
*/
class Solution {
public:
    int numDistinct(string S, string T) {
        int m = T.length();
        int n = S.length();
        if (m > n) return 0;    // impossible for subsequence
        vector<vector<int>> path(m+1, vector<int>(n+1, 0));
        for (int k = 0; k <= n; k++) path[0][k] = 1;    // initialization
    
        for (int j = 1; j <= n; j++) {
            for (int i = 1; i <= m; i++) {
                path[i][j] = path[i][j-1] + (T[i-1] == S[j-1] ? path[i-1][j-1] : 0);
            }
        }
    
        return path[m][n];
    }
};
```



















































