---
id: 974
title: "Subarray Sums Divisible by K"
slug: subarray-sums-divisible-by-k
difficulty: Medium
tags: [Array, Hash Table, Prefix Sum]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Subarray Sums Divisible by K

Given an integer array `nums` and an integer `k`, return _the number of non-empty **subarrays** that have a sum divisible by_ `k`.

A **subarray** is a **contiguous** part of an array.

**Example 1:**

**Input:** nums = \[4,5,0,-2,-3,1\], k = 5
**Output:** 7
**Explanation:** There are 7 subarrays with a sum divisible by k = 5:
\[4, 5, 0, -2, -3, 1\], \[5\], \[5, 0\], \[5, 0, -2, -3\], \[0\], \[0, -2, -3\], \[-2, -3\]

**Example 2:**

**Input:** nums = \[5\], k = 9
**Output:** 0

**Constraints:**

*   `1 <= nums.length <= 3 * 104`
*   `-104 <= nums[i] <= 104`
*   `2 <= k <= 104`

## Code Template

### Go
```go
func subarraysDivByK(nums []int, k int) int {
    
}
```

### TypeScript
```typescript
function subarraysDivByK(nums: number[], k: number): number {
    
};
```
