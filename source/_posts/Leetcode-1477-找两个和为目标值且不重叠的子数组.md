---
title: Leetcode-1477 找两个和为目标值且不重叠的子数组
mathjax: true
type: "tags"
date: 2020-06-20 20:54:14
tags:
  - Leetcode
  - 滑动窗口
categories:
  - 算法
---

# 思路

## 暴力解法及其复杂度分析
拿到这道题第一想法是使用滑动窗口枚举所有和为$target$的子区间，然后二重循环遍历所有的区间对，找出答案。

时间复杂度计算如下：
滑动窗口找符合条件的子区间时间复杂度为$O(n)$，滑动窗口遍历的区间至多只有$2n$个，因此二重循环遍历所有区间对的时间复杂度为$O(n^2)$，总的时间复杂度为$O(n^2)$。

题目中$n$最大为$10^5$，暴力法显然超时了。

## 优化后的解法
假设区间左端点为$l$，右端点为$r$，我们采用左闭右开的区间表示方法$[l, r)$。注意到在滑动窗口计算的过程中，$r$与$l$始终是递增的，那么当我们找到一个符合条件的区间$[l, r)$时，在该区间左边并且与该区间不相交的所有符合条件的区间都已经被求出来了。如果我们能在滑动窗口遍历的过程中维护一个数组$prev\_min[r]$，用于记录区间右端点小于等于$r$的所有符合条件的区间的最小长度，那么在求出一个符合条件的区间$[l,r)$时我们可以利用$prev\_min$数组直接找出该区间左侧最小的区间长度（即$prev\_min[l - 1]$），并更新答案。如此一来，我们可以在滑动窗口遍历的过程中直接求出答案，因此时间复杂度优化为$O(n)$。

$prev\_min$数组的更新方式如下：
1. 每次区间右端点$r$向右移动时，将之前计算的结果直接拷贝过来
$$prev\_min[r]=prev\_min[r - 1]$$
2. 每次找到一个符合条件的区间$[l, r)$时，用该区间长度更新$prev\_min$数组（区间右端点实际上为$r - 1$，根据$prev\_min$数组的定义，此时应该更新$r-1$而不是$r$处的值
$$prev\_min[r - 1]=min(prev\_min[r - 1], r-l)$$
   
代码如下：
``` cpp
class Solution {
public:
    
    int minSumOfLengths(vector<int>& arr, int target) {
        int l = 0, r = 0, sum = 0, ans = INT_MAX;
        int prev_min[arr.size()];
        prev_min[0] = INT_MAX; //用INT_MAX表示左侧不存在任何区间
        while (true) {
            while (r < arr.size() && l <= r && sum < target) {
                sum += arr[r];
                if (r >= 1) {
                    prev_min[r] = prev_min[r - 1];
                }
                r++;
            }
            if (sum == target) {
                int len = r - l;
                if (l >= 1 && prev_min[l - 1] != INT_MAX) {
                    ans = min(ans, len + prev_min[l - 1]);
                }
                prev_min[r - 1] = min(prev_min[r - 1], len);
            } else if (sum < target) {
                break;  
            }
            sum -= arr[l];
            l++;
        }
        return ans == INT_MAX ? -1 : ans;
    }
}; 
```
