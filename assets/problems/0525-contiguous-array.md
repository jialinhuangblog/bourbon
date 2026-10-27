---
id: 525
title: "Contiguous Array"
slug: contiguous-array
difficulty: Medium
tags: [Array, Hash Table, Prefix Sum]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Contiguous Array

Given a binary array `nums`, return _the maximum length of a contiguous subarray with an equal number of_ `0` _and_ `1`.

**Example 1:**

**Input:** nums = \[0,1\]
**Output:** 2
**Explanation:** \[0, 1\] is the longest contiguous subarray with an equal number of 0 and 1.

**Example 2:**

**Input:** nums = \[0,1,0\]
**Output:** 2
**Explanation:** \[0, 1\] (or \[1, 0\]) is a longest contiguous subarray with equal number of 0 and 1.

**Example 3:**

**Input:** nums = \[0,1,1,1,1,1,0,0,0\]
**Output:** 6
**Explanation:** \[1,1,1,0,0,0\] is the longest contiguous subarray with equal number of 0 and 1.

**Constraints:**

*   `1 <= nums.length <= 105`
*   `nums[i]` is either `0` or `1`.

## Code Template

### Go
```go
func findMaxLength(nums []int) int {
    
}
```

### TypeScript
```typescript
function findMaxLength(nums: number[]): number {
    
};
```
