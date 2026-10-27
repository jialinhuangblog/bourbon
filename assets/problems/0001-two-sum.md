---
id: 1
title: "Two Sum"
slug: two-sum
difficulty: Easy
tags: [Array, Hash Table]
neetcode150_category: Arrays & Hashing
blind75_category: Array
date_solved: 2026-02-24
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "seen[target-num] exists? return indices"
---

# Two Sum

Given an array of integers `nums` and an integer `target`, return _indices of the two numbers such that they add up to `target`_.

You may assume that each input would have **_exactly_ one solution**, and you may not use the _same_ element twice.

You can return the answer in any order.

**Example 1:**

**Input:** nums = \[2,7,11,15\], target = 9
**Output:** \[0,1\]
**Explanation:** Because nums\[0\] + nums\[1\] == 9, we return \[0, 1\].

**Example 2:**

**Input:** nums = \[3,2,4\], target = 6
**Output:** \[1,2\]

**Example 3:**

**Input:** nums = \[3,3\], target = 6
**Output:** \[0,1\]

**Constraints:**

*   `2 <= nums.length <= 104`
*   `-109 <= nums[i] <= 109`
*   `-109 <= target <= 109`
*   **Only one valid answer exists.**

**Follow-up:** Can you come up with an algorithm that is less than `O(n2)` time complexity?

## Code Template

### Go
```go
func twoSum(nums []int, target int) []int {
    
}
```

### TypeScript
```typescript
function twoSum(nums: number[], target: number): number[] {
    
};
```
