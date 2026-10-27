---
id: 347
title: "Top K Frequent Elements"
slug: top-k-frequent-elements
difficulty: Medium
tags: [Array, Hash Table, Divide and Conquer, Sorting, Heap (Priority Queue), Bucket Sort, Counting, Quickselect]
neetcode150_category: Arrays & Hashing
blind75_category: Heap
date_solved: 2026-07-13
languages: [golang, typescript]
time_complexity: null
space_complexity: null
"@local": [Heap]
---

# Top K Frequent Elements

Given an integer array `nums` and an integer `k`, return _the_ `k` _most frequent elements_. You may return the answer in **any order**.

**Example 1:**

**Input:** nums = \[1,1,1,2,2,3\], k = 2

**Output:** \[1,2\]

**Example 2:**

**Input:** nums = \[1\], k = 1

**Output:** \[1\]

**Example 3:**

**Input:** nums = \[1,2,1,2,1,2,3,1,3,2\], k = 2

**Output:** \[1,2\]

**Constraints:**

*   `1 <= nums.length <= 105`
*   `-104 <= nums[i] <= 104`
*   `k` is in the range `[1, the number of unique elements in the array]`.
*   It is **guaranteed** that the answer is **unique**.

**Follow up:** Your algorithm's time complexity must be better than `O(n log n)`, where n is the array's size.

## Code Template

### Go
```go
func topKFrequent(nums []int, k int) []int {
    
}
```

### TypeScript
```typescript
function topKFrequent(nums: number[], k: number): number[] {
    
};
```
