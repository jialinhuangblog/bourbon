---
id: 215
title: "Kth Largest Element in an Array"
slug: kth-largest-element-in-an-array
difficulty: Medium
tags: [Array, Divide and Conquer, Sorting, Heap (Priority Queue), Quickselect]
neetcode150_category: Heap / Priority Queue
blind75_category: null
date_solved: 2026-03-23
languages: [golang, typescript]
time_complexity: null
space_complexity: null
"@local": [Heap]
insight: "min-heap size k; root = kth largest"
---

# Kth Largest Element in an Array

Given an integer array `nums` and an integer `k`, return _the_ `kth` _largest element in the array_.

Note that it is the `kth` largest element in the sorted order, not the `kth` distinct element.

Can you solve it without sorting?

**Example 1:**

**Input:** nums = \[3,2,1,5,6,4\], k = 2
**Output:** 5

**Example 2:**

**Input:** nums = \[3,2,3,1,2,4,5,5,6\], k = 4
**Output:** 4

**Constraints:**

*   `1 <= k <= nums.length <= 105`
*   `-104 <= nums[i] <= 104`

## Code Template

### Go
```go
func findKthLargest(nums []int, k int) int {
    
}
```

### TypeScript
```typescript
function findKthLargest(nums: number[], k: number): number {
    
};
```
