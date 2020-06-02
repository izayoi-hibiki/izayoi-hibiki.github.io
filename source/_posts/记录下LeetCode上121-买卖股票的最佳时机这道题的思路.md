---
title: 记录下LeetCode上121.买卖股票的最佳时机这道题的思路
categories:
- 随笔
date: 2020-05-21 15:29:48
tags:
- 算法
- 动态规划
---

DP方程：opt(i)=price[i]-min{price[i-1]-opt(i-1),price[i-1]}
方程有点绕，记录下思路。

<!--more-->

> [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

# 原题目

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。

注意：你不能在买入股票前卖出股票。


示例 1:
```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

示例 2:
```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

## 思路

下面记录下用动态规划方法的解题思路，方程有点绕。

设在第 i 天卖出的情况下有一种方案(在之前的某天买入)使得利润 opt(i) 最大，
设第 j 到第 k 天的最低价为 minPrice(j,k),
设第 i 天的价格为price[i]

则 ```opt(i)=price[i]-minPrice(0,i-1)```    (1)
同理 ```opt(i-1)=price[i-1]-minPrice(0,i-2)``` (2)

minPrice(0,i-1)有两种情况：
1. ```minPrice(0,i-1)=minPrice(0,i-2)``` ,即 price[i-1]并非过往的最低价，
2. ```minPrice(0,i-1)=price[i-1)``` ,即 price[i-1]是过往最低价

故 ```minPrice(0,i-1)=min{minPrice(0,i-2),price[i-1]}```  (3)

将式 3 代入式 1，得：```opt(i)=price[i]-min{minPrice(0,i-2),price[i-1]}```   (4)

又由式 2 变换得 ```minPrice(0,i-2)=price[i-1]-opt(i-1)``` (5)
将式 5 代入式 4，得：```opt(i)=price[i]-min{price[i-1]-opt(i-1),price[i-1]}``` (6)

则式 6 为最终的 DP 状态转移方程。

# 代码

有了状态转移方程基本就能写出代码了，这里附上一份 js 代码。


```JavaScript
/**
 * @param {number[]} prices
 * @return {number}
 */
const maxProfit = function (prices) {
    let n = prices.length;
    let opt = [];
    opt[0] = 0;
    for (let i = 1; i < n; i++) {
        opt[i] = prices[i] - Math.min(prices[i - 1] - opt[i - 1], prices[i - 1]);
    }

    return Math.max(...opt)
};
```

