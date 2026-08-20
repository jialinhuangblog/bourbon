---
id: 658
title: "Find K Closest Elements"
slug: find-k-closest-elements
difficulty: Medium
tags: [Array, Two Pointers, Binary Search, Sliding Window, Sorting, Heap (Priority Queue)]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Find K Closest Elements

Given a **sorted** integer array `arr`, two integers `k` and `x`, return the `k` closest integers to `x` in the array. The result should also be sorted in ascending order.

An integer `a` is closer to `x` than an integer `b` if:

*   `|a - x| < |b - x|`, or
*   `|a - x| == |b - x|` and `a < b`

**Example 1:**

**Input:** arr = \[1,2,3,4,5\], k = 4, x = 3

**Output:** \[1,2,3,4\]

**Example 2:**

**Input:** arr = \[1,1,2,3,4,5\], k = 4, x = -1

**Output:** \[1,1,2,3\]

**Constraints:**

*   `1 <= k <= arr.length`
*   `1 <= arr.length <= 104`
*   `arr` is sorted in **ascending** order.
*   `-104 <= arr[i], x <= 104`

## Code Template

### Go
```go
func findClosestElements(arr []int, k int, x int) []int {
    
}
```

### TypeScript
```typescript
function findClosestElements(arr: number[], k: number, x: number): number[] {
    
};
```
