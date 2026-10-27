---
id: 137
title: "Single Number II"
slug: single-number-ii
difficulty: Medium
tags: [Array, Bit Manipulation]
"@local": [Operator]
neetcode150_category: null
blind75_category: null
date_solved: 2026-02-25
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "count each bit mod 3; remainder bit belongs to single"
---

# Single Number II

Given an integer array `nums` where every element appears **three times** except for one, which appears **exactly once**. _Find the single element and return it_.

You must implement a solution with a linear runtime complexity and use only constant extra space.

**Example 1:**

**Input:** nums = \[2,2,3,2\]
**Output:** 3

**Example 2:**

**Input:** nums = \[0,1,0,1,0,1,99\]
**Output:** 99

**Constraints:**

*   `1 <= nums.length <= 3 * 104`
*   `-231 <= nums[i] <= 231 - 1`
*   Each element in `nums` appears exactly **three times** except for one element which appears **once**.

## Code Template

### Go
```go
func singleNumber(nums []int) int {
    
}
```

### TypeScript
```typescript
function singleNumber(nums: number[]): number {
    
};
```
