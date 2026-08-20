---
id: 746
title: "Min Cost Climbing Stairs"
slug: min-cost-climbing-stairs
difficulty: Easy
tags: [Array, Dynamic Programming]
neetcode150_category: 1-D Dynamic Programming
blind75_category: null
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(1)
insight: "終點在最後一階外面，表要開 n+1 格、答案是 f[n]；cost[i] 是離開那一階的錢"
---

# Min Cost Climbing Stairs

You are given an integer array `cost` where `cost[i]` is the cost of `ith` step on a staircase. Once you pay the cost, you can either climb one or two steps.

You can either start from the step with index `0`, or the step with index `1`.

Return _the minimum cost to reach the top of the floor_.

**Example 1:**

**Input:** cost = \[10,15,20\]
**Output:** 15
**Explanation:** You will start at index 1.
- Pay 15 and climb two steps to reach the top.
The total cost is 15.

**Example 2:**

**Input:** cost = \[1,100,1,1,1,100,1,1,100,1\]
**Output:** 6
**Explanation:** You will start at index 0.
- Pay 1 and climb two steps to reach index 2.
- Pay 1 and climb two steps to reach index 4.
- Pay 1 and climb two steps to reach index 6.
- Pay 1 and climb one step to reach index 7.
- Pay 1 and climb two steps to reach index 9.
- Pay 1 and climb one step to reach the top.
The total cost is 6.

**Constraints:**

*   `2 <= cost.length <= 1000`
*   `0 <= cost[i] <= 999`

## Code Template

### Go
```go
func minCostClimbingStairs(cost []int) int {
    
}
```

### TypeScript
```typescript
function minCostClimbingStairs(cost: number[]): number {
    
};
```
