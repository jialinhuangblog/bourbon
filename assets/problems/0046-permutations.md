---
id: 46
title: "Permutations"
slug: permutations
difficulty: Medium
tags: [Array, Backtracking]
neetcode150_category: Backtracking
blind75_category: null
date_solved: 2026-08-08
languages: [golang, typescript]
time_complexity: O(n! * n)
space_complexity: O(n)
"@local": [Backtracking]
---

# Permutations

Given an array `nums` of distinct integers, return all the possible permutations. You can return the answer in **any order**.

**Example 1:**

**Input:** nums = \[1,2,3\]
**Output:** \[\[1,2,3\],\[1,3,2\],\[2,1,3\],\[2,3,1\],\[3,1,2\],\[3,2,1\]\]

**Example 2:**

**Input:** nums = \[0,1\]
**Output:** \[\[0,1\],\[1,0\]\]

**Example 3:**

**Input:** nums = \[1\]
**Output:** \[\[1\]\]

**Constraints:**

*   `1 <= nums.length <= 6`
*   `-10 <= nums[i] <= 10`
*   All the integers of `nums` are **unique**.

## Code Template

### Go
```go
func permute(nums []int) [][]int {
    
}
```

### TypeScript
```typescript
function permute(nums: number[]): number[][] {
    
};
```
