---
id: 219
title: "Contains Duplicate II"
slug: contains-duplicate-ii
difficulty: Easy
tags: [Array, Hash Table, Sliding Window]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Contains Duplicate II

Given an integer array `nums` and an integer `k`, return `true` _if there are two **distinct indices**_ `i` _and_ `j` _in the array such that_ `nums[i] == nums[j]` _and_ `abs(i - j) <= k`.

**Example 1:**

**Input:** nums = \[1,2,3,1\], k = 3
**Output:** true

**Example 2:**

**Input:** nums = \[1,0,1,1\], k = 1
**Output:** true

**Example 3:**

**Input:** nums = \[1,2,3,1,2,3\], k = 2
**Output:** false

**Constraints:**

*   `1 <= nums.length <= 105`
*   `-109 <= nums[i] <= 109`
*   `0 <= k <= 105`

## Code Template

### Go
```go
func containsNearbyDuplicate(nums []int, k int) bool {
    
}
```

### TypeScript
```typescript
function containsNearbyDuplicate(nums: number[], k: number): boolean {
    
};
```
