---
id: 70
title: "Climbing Stairs"
slug: climbing-stairs
difficulty: Easy
tags: [Math, Dynamic Programming, Memoization]
neetcode150_category: 1-D Dynamic Programming
blind75_category: Dynamic Programming
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(1)
insight: "最後一步只能從 i-1 或 i-2 來，兩群走法的最後一步不同所以不重複，加起來就是答案"
---

# Climbing Stairs

You are climbing a staircase. It takes `n` steps to reach the top.

Each time you can either climb `1` or `2` steps. In how many distinct ways can you climb to the top?

**Example 1:**

**Input:** n = 2
**Output:** 2
**Explanation:** There are two ways to climb to the top.
1. 1 step + 1 step
2. 2 steps

**Example 2:**

**Input:** n = 3
**Output:** 3
**Explanation:** There are three ways to climb to the top.
1. 1 step + 1 step + 1 step
2. 1 step + 2 steps
3. 2 steps + 1 step

**Constraints:**

*   `1 <= n <= 45`

## Code Template

### Go
```go
func climbStairs(n int) int {
    
}
```

### TypeScript
```typescript
function climbStairs(n: number): number {
    
};
```
