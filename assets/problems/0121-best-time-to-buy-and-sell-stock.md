---
id: 121
title: "Best Time to Buy and Sell Stock"
slug: best-time-to-buy-and-sell-stock
difficulty: Easy
tags: [Array, Dynamic Programming]
neetcode150_category: Sliding Window
blind75_category: Array
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(1)
insight: "每天都當賣出日，只需要「今天以前的最低價」；換成漲跌陣列就是 53 的 Kadane"
---

# Best Time to Buy and Sell Stock

You are given an array `prices` where `prices[i]` is the price of a given stock on the `ith` day.

You want to maximize your profit by choosing a **single day** to buy one stock and choosing a **different day in the future** to sell that stock.

Return _the maximum profit you can achieve from this transaction_. If you cannot achieve any profit, return `0`.

**Example 1:**

**Input:** prices = \[7,1,5,3,6,4\]
**Output:** 5
**Explanation:** Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.

**Example 2:**

**Input:** prices = \[7,6,4,3,1\]
**Output:** 0
**Explanation:** In this case, no transactions are done and the max profit = 0.

**Constraints:**

*   `1 <= prices.length <= 105`
*   `0 <= prices[i] <= 104`

## Code Template

### Go
```go
func maxProfit(prices []int) int {
    
}
```

### TypeScript
```typescript
function maxProfit(prices: number[]): number {
    
};
```
