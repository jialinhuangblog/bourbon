---
id: 493
title: "Reverse Pairs"
slug: reverse-pairs
difficulty: Hard
tags: [Array, Binary Search, Divide and Conquer, Binary Indexed Tree, Segment Tree, Merge Sort, Ordered Set]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Reverse Pairs

Given an integer array `nums`, return _the number of **reverse pairs** in the array_.

A **reverse pair** is a pair `(i, j)` where:

*   `0 <= i < j < nums.length` and
*   `nums[i] > 2 * nums[j]`.

**Example 1:**

**Input:** nums = \[1,3,2,3,1\]
**Output:** 2
**Explanation:** The reverse pairs are:
(1, 4) --> nums\[1\] = 3, nums\[4\] = 1, 3 > 2 \* 1
(3, 4) --> nums\[3\] = 3, nums\[4\] = 1, 3 > 2 \* 1

**Example 2:**

**Input:** nums = \[2,4,3,5,1\]
**Output:** 3
**Explanation:** The reverse pairs are:
(1, 4) --> nums\[1\] = 4, nums\[4\] = 1, 4 > 2 \* 1
(2, 4) --> nums\[2\] = 3, nums\[4\] = 1, 3 > 2 \* 1
(3, 4) --> nums\[3\] = 5, nums\[4\] = 1, 5 > 2 \* 1

**Constraints:**

*   `1 <= nums.length <= 5 * 104`
*   `-231 <= nums[i] <= 231 - 1`

## Code Template

### Go
```go
func reversePairs(nums []int) int {
    
}
```

### TypeScript
```typescript
function reversePairs(nums: number[]): number {
    
};
```
