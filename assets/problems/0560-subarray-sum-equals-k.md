---
id: 560
title: "Subarray Sum Equals K"
slug: subarray-sum-equals-k
difficulty: Medium
tags: [Array, Hash Table, Prefix Sum]
neetcode150_category: null
blind75_category: null
date_solved: 2026-04-14
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(n)
"@local": [Prefix Sum]
insight: "prefix - k 找配對 → hashmap 存 count，解有負數的連續子陣列求和"
---

# Subarray Sum Equals K

Given an array of integers `nums` and an integer `k`, return _the total number of subarrays whose sum equals to_ `k`.

A subarray is a contiguous **non-empty** sequence of elements within an array.

**Example 1:**

**Input:** nums = \[1,1,1\], k = 2
**Output:** 2

**Example 2:**

**Input:** nums = \[1,2,3\], k = 3
**Output:** 2

**Constraints:**

*   `1 <= nums.length <= 2 * 104`
*   `-1000 <= nums[i] <= 1000`
*   `-107 <= k <= 107`

## Code Template

### Go
```go
func subarraySum(nums []int, k int) int {
    
}
```

### TypeScript
```typescript
function subarraySum(nums: number[], k: number): number {
    
};
```
