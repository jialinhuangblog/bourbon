---
id: 368
title: "Largest Divisible Subset"
slug: largest-divisible-subset
difficulty: Medium
tags: [Array, Math, Dynamic Programming, Sorting]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Largest Divisible Subset

Given a set of **distinct** positive integers `nums`, return the largest subset `answer` such that every pair `(answer[i], answer[j])` of elements in this subset satisfies:

*   `answer[i] % answer[j] == 0`, or
*   `answer[j] % answer[i] == 0`

If there are multiple solutions, return any of them.

**Example 1:**

**Input:** nums = \[1,2,3\]
**Output:** \[1,2\]
**Explanation:** \[1,3\] is also accepted.

**Example 2:**

**Input:** nums = \[1,2,4,8\]
**Output:** \[1,2,4,8\]

**Constraints:**

*   `1 <= nums.length <= 1000`
*   `1 <= nums[i] <= 2 * 109`
*   All the integers in `nums` are **unique**.

## Code Template

### Go
```go
func largestDivisibleSubset(nums []int) []int {
    
}
```

### TypeScript
```typescript
function largestDivisibleSubset(nums: number[]): number[] {
    
};
```
