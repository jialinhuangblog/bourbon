---
id: 509
title: "Fibonacci Number"
slug: fibonacci-number
difficulty: Easy
tags: [Math, Dynamic Programming, Recursion, Memoization]
neetcode150_category: null
blind75_category: null
date_solved: 2026-03-07
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "dp[i] = dp[i-1] + dp[i-2]"
---

# Fibonacci Number

The **Fibonacci numbers**, commonly denoted `F(n)` form a sequence, called the **Fibonacci sequence**, such that each number is the sum of the two preceding ones, starting from `0` and `1`. That is,

F(0) = 0, F(1) = 1
F(n) = F(n - 1) + F(n - 2), for n > 1.

Given `n`, calculate `F(n)`.

**Example 1:**

**Input:** n = 2
**Output:** 1
**Explanation:** F(2) = F(1) + F(0) = 1 + 0 = 1.

**Example 2:**

**Input:** n = 3
**Output:** 2
**Explanation:** F(3) = F(2) + F(1) = 1 + 1 = 2.

**Example 3:**

**Input:** n = 4
**Output:** 3
**Explanation:** F(4) = F(3) + F(2) = 2 + 1 = 3.

**Constraints:**

*   `0 <= n <= 30`

## Code Template

### Go
```go
func fib(n int) int {
    
}
```

### TypeScript
```typescript
function fib(n: number): number {

};
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
