---
id: 713
title: "Subarray Product Less Than K"
slug: subarray-product-less-than-k
difficulty: Medium
tags: [Array, Binary Search, Sliding Window, Prefix Sum]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Subarray Product Less Than K

Given an array of integers `nums` and an integer `k`, return _the number of contiguous subarrays where the product of all the elements in the subarray is strictly less than_ `k`.

**Example 1:**

**Input:** nums = \[10,5,2,6\], k = 100
**Output:** 8
**Explanation:** The 8 subarrays that have product less than 100 are:
\[10\], \[5\], \[2\], \[6\], \[10, 5\], \[5, 2\], \[2, 6\], \[5, 2, 6\]
Note that \[10, 5, 2\] is not included as the product of 100 is not strictly less than k.

**Example 2:**

**Input:** nums = \[1,2,3\], k = 0
**Output:** 0

**Constraints:**

*   `1 <= nums.length <= 3 * 104`
*   `1 <= nums[i] <= 1000`
*   `0 <= k <= 106`

## Code Template

### Go
```go
func numSubarrayProductLessThanK(nums []int, k int) int {
    
}
```

### TypeScript
```typescript
function numSubarrayProductLessThanK(nums: number[], k: number): number {
    
};
```
