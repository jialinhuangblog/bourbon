---
id: 4
title: "Median of Two Sorted Arrays"
slug: median-of-two-sorted-arrays
difficulty: Hard
tags: [Array, Binary Search, Divide and Conquer]
neetcode150_category: Binary Search
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Median of Two Sorted Arrays

Given two sorted arrays `nums1` and `nums2` of size `m` and `n` respectively, return **the median** of the two sorted arrays.

The overall run time complexity should be `O(log (m+n))`.

**Example 1:**

**Input:** nums1 = \[1,3\], nums2 = \[2\]
**Output:** 2.00000
**Explanation:** merged array = \[1,2,3\] and median is 2.

**Example 2:**

**Input:** nums1 = \[1,2\], nums2 = \[3,4\]
**Output:** 2.50000
**Explanation:** merged array = \[1,2,3,4\] and median is (2 + 3) / 2 = 2.5.

**Constraints:**

*   `nums1.length == m`
*   `nums2.length == n`
*   `0 <= m <= 1000`
*   `0 <= n <= 1000`
*   `1 <= m + n <= 2000`
*   `-106 <= nums1[i], nums2[i] <= 106`

## Code Template

### Go
```go
func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    
}
```

### TypeScript
```typescript
function findMedianSortedArrays(nums1: number[], nums2: number[]): number {
    
};
```
