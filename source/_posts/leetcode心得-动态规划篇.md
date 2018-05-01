---
title: leetcode心得 - 动态规划篇
date: 2018-04-30 20:14:59
categories:
	- Algorithms
tags:
    - leetcode
    - 数据结构与算法
    - 动态规划
---

动态规划（dynamic programming）
-----
动态规划在我看来主要是找到递推关系，要善于归纳总结

word-break
----------------

### 题目描述
> Given a string s and a dictionary of words dict, determine if s can be segmented into a space-separated sequence of one or more dictionary words. 
For example, given
> s ="leetcode",
> dict =["leet", "code"]. 
> Return true because "leetcode" can be segmented as"leet code". 
<!--more-->
### 解题过程
对一个长度为(n + 1)的字符串s，如果知道s.substring(0, i)（其中0 < i <= n）是否可分，那么怎么确定这个字符串是否可分呢？

``` java
    public boolean wordBreak(String s, Set<String> dict) {
        if (s == null || s.length() == 0) {
            return dict.contains(s);
        }
        int len = s.length();
        // dp[i]表示s.substring(0, i)是否可以切分
        // dp[s.length()]的取值即为问题的答案
        boolean[] dp = new boolean[len + 1];
        dp[0] = true;
        for (int i = 1; i <= len; i++) {
            // 确定dp[i]的值，要用到所有dp[0...i - 1]的值
            for (int j = 0; j < i; j++) {
                if (dp[j] && dict.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
                    
            }
        }
        return dp[len];
    }
```

palindrome-partitioning-ii 
--
### 题目描述

> Given a string *s*, partition *s* such that every substring of the
> partition is a palindrome.  Return the minimum cuts needed for a
> palindrome partitioning of *s*. 
> For example, 
> given *s* = "aab",
> Return 1 since the palindrome partitioning ["aa","b"] could be produced
> using 1 cut.


### 解题过程
对于一个长度为(n + 1)的字符串*s*，如果知道*s*.substring(0, i)（其中0 < i <= n）各自的最小切分数，那么怎么确定字符串*s*的最小切分数呢？

``` java
    public int minCut(String s) {
        // dp[i]表示s.substring(0, i)的最小切分数
        // dp[s.length()]即为问题的答案
        int[] dp = new int[s.length() + 1];
        dp[0] = 0;
        for (int i = 1; i <= s.length(); i++) {
            // 如果s.substring(0, i)是回文串，那么不需要切分
            if (isPalindrome(s.substring(0, i))) {
                dp[i] = 0;
                continue;
            }
            // 否则，最多可能要切分(i - 1)次
            dp[i] = i - 1;
            for (int j = 1; j < i; j++) {
                // 每次都取最小值，遍历1...(i - 1)后，获取全局最小值
                if (isPalindrome(s.substring(j, i))) {
                    dp[i] = Math.min(dp[i], dp[j] + 1);
                }
                else {
                    dp[i] = Math.min(dp[i], dp[j] + i - j);
                }
            }
        }
        return dp[s.length()];
    }
    public boolean isPalindrome(String s) {
        StringBuilder sb = new StringBuilder(s);
        return (s.equals(sb.reverse().toString()));
    }
```

candy
--
### 题目描述

> There are *N* children standing in a line. Each child is assigned a rating value. 
You are giving candies to these children subjected to the following requirements: 
> Each child must have at least one candy. 
> Children with a higher rating get more candies than their neighbors. 
> What is the minimum candies you must give? 

### 解题过程

正向遍历一次，计算每个人应该得到多少糖果；反向再遍历一次，计算每个人应该得到多少糖果；取再次计算结果较大的值。

``` java
    public int candy(int[] ratings) {
        int[] dp = new int[ratings.length];
        for (int i = 1; i < ratings.length; i++) {
            if (ratings[i] > ratings[i - 1]) {
                dp[i] = dp[i - 1] + 1;
            }
        }
        for (int i = ratings.length - 2; i >= 0; i--) {
            if (ratings[i] > ratings[i + 1] && dp[i] <= dp[i + 1]) {
                dp[i] = dp[i + 1] + 1;
            }
        }
        int sum = ratings.length;
        for (int item : dp) {
            sum += item;
        }
        return sum;
    }
```



## dictinct-subsequences

### 题目描述
> Given a string *S* and a string *T*, count the number of distinct subsequences of *T* in *S*. 
> A subsequence of a string is a new string which is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e.,"ACE" is a subsequence of "ABCDE" while "AEC" is not). 
> Here is an example:
> *S* = "rabbbit", T = "rabbit" 
> Return 3. 


### 解题过程
设想一个长度为(j + 1)的字符串*S*，和一个长度为(i + 1)的字符串*T*。
我们用S[j + 1]表示*S*的第(j + 1)个字符，T[i + 1]表示*T*的第(i + 1)个字符。用S[m...n]表示*S*的子序列（从m到n的子字符串，包括m和n）。

1. 如果S[j + 1] =/= T[i + 1]，那么S[j + 1]应该是无关紧要的，因为S[j + 1]在生成子序列*T*的时候一定会被剔除。于是，问题转化为S[0...j]可以生成多少个*T*。

2. 如果S[j + 1] == T[i + 1]，我们依然可以去除S[j + 1]，因为*S*中可能包括不止一个T[i + 1]；我们也可以选择保留S[j + 1]，这样，S[j + 1]与T[i + 1]已经匹配成功，我们只需要找出S[0...j]可以生成多少个T[0...i]就可以了。

最重要的还是建立递推关系。

``` java
    public int numDistinct(String S, String T) {
        int sLen = S.length();
        int tLen = T.length();
        int[][] dp = new int[tLen + 1][sLen + 1];
        for (int j = 0; j < sLen; j++) {
            dp[0][j] = 1;
        }
        for (int i = 0; i < tLen; i++) {
            for (int j = 0; j < sLen; j++) {
                if (T.charAt(i) == S.charAt(j)) {
                    dp[i + 1][j + 1] = dp[i][j] + dp[i + 1][j];
                }
                else {
                    dp[i + 1][j + 1] = dp[i + 1][j];
                }
            }
        }
        return dp[tLen][sLen];
    }
```

decode-ways
--
### 题目描述
> A message containing letters fromA-Zis being encoded to numbers using the following mapping: 
'A' -> 1
'B' -> 2
...
'Z' -> 26
Given an encoded message containing digits, determine the total number of ways to decode it. 
For example,
Given encoded message "12", it could be decoded as "AB" (1 2) or "L"(12). 
The number of ways decoding "12" is 2. 

### 解题过程
其实这个跟斐波那契数列很像。只不过多了两个条件：

- `s.charAt(i + 1) == '0' && (s.charAt(i) == '1' || s.charAt(i) == '2'`，那么`dp[i + 1] = dp[i - 1]`
- 否则，`dp[i + 1] = dp[i]`，但是，在最后两个字符位于`11 - 26`之间时，`dp[i + 1] += dp[i - 1]`。

``` java
    public int numDecodings(String s) {
        if(s == null || s.length() == 0)
            return 0;
        int len = s.length();
        int[] dp = new int[len+1];
        dp[0] = 1;
        char cur = '#', prev = '#';
        for(int i = 0; i < len; i++) {
            cur = s.charAt(i);
            if(cur == '0') {
                if(!(prev == '1' || prev == '2'))
                    return 0;
                dp[i+1] = dp[i-1];
            } else {
                dp[i+1] = dp[i];
                if(prev == '1' || prev == '2' && cur >= '1' && cur <= '6')
                    dp[i+1] += dp[i-1];
            }
            prev = cur;
        }
        return dp[len];
    }
```

