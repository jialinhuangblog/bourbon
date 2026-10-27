---
id: 704
title: "Binary Search"
slug: binary-search
difficulty: Easy
tags: [Array, Binary Search]
"@local": [Operator]
neetcode150_category: Binary Search
blind75_category: null
date_solved: 2026-03-23
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "l+(r-l)/2; advance mid+1 or mid-1 to avoid infinite loop"
---

# Binary Search

Given an array of integers `nums` which is sorted in ascending order, and an integer `target`, write a function to search `target` in `nums`. If `target` exists, then return its index. Otherwise, return `-1`.

You must write an algorithm with `O(log n)` runtime complexity.

**Example 1:**

**Input:** nums = \[-1,0,3,5,9,12\], target = 9
**Output:** 4
**Explanation:** 9 exists in nums and its index is 4

**Example 2:**

**Input:** nums = \[-1,0,3,5,9,12\], target = 2
**Output:** -1
**Explanation:** 2 does not exist in nums so return -1

**Constraints:**

*   `1 <= nums.length <= 104`
*   `-104 < nums[i], target < 104`
*   All the integers in `nums` are **unique**.
*   `nums` is sorted in ascending order.

## Code Template

### Go
```go
func search(nums []int, target int) int {
    
}
```

### TypeScript
```typescript
function search(nums: number[], target: number): number {
    
};
```
