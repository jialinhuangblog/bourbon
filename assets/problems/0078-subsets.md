---
id: 78
title: "Subsets"
slug: subsets
difficulty: Medium
tags: [Array, Backtracking, Bit Manipulation]
"@local": [Backtracking, Operator]
neetcode150_category: Backtracking
blind75_category: null
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n * 2^n)
space_complexity: O(n)
insight: "每個元素加或不加；遞迴傳 i+1 保證只往右挑，中途狀態本身就是合法答案，進函式就收"
---

# Subsets

Given an integer array `nums` of **unique** elements, return _all possible_ _subsets_ _(the power set)_.

The solution set **must not** contain duplicate subsets. Return the solution in **any order**.

**Example 1:**

**Input:** nums = \[1,2,3\]
**Output:** \[\[\],\[1\],\[2\],\[1,2\],\[3\],\[1,3\],\[2,3\],\[1,2,3\]\]

**Example 2:**

**Input:** nums = \[0\]
**Output:** \[\[\],\[0\]\]

**Constraints:**

*   `1 <= nums.length <= 10`
*   `-10 <= nums[i] <= 10`
*   All the numbers of `nums` are **unique**.

## Code Template

### Go
```go
func subsets(nums []int) [][]int {
    
}
```

### TypeScript
```typescript
function subsets(nums: number[]): number[][] {
    
};
```
