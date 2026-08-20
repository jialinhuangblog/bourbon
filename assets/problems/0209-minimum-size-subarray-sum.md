---
id: 209
title: "Minimum Size Subarray Sum"
slug: minimum-size-subarray-sum
difficulty: Medium
tags: [Array, Binary Search, Sliding Window, Prefix Sum]
neetcode150_category: null
blind75_category: null
date_solved: 2026-03-14
languages: [golang, typescript]
time_complexity: null
space_complexity: null
"@local": [Prefix Sum]
insight: "shrink left while sum >= target; track min length"
---

# Minimum Size Subarray Sum

Given an array of positive integers `nums` and a positive integer `target`, return _the **minimal length** of a_ _subarray_ _whose sum is greater than or equal to_ `target`. If there is no such subarray, return `0` instead.

**Example 1:**

**Input:** target = 7, nums = \[2,3,1,2,4,3\]
**Output:** 2
**Explanation:** The subarray \[4,3\] has the minimal length under the problem constraint.

**Example 2:**

**Input:** target = 4, nums = \[1,4,4\]
**Output:** 1

**Example 3:**

**Input:** target = 11, nums = \[1,1,1,1,1,1,1,1\]
**Output:** 0

**Constraints:**

*   `1 <= target <= 109`
*   `1 <= nums.length <= 105`
*   `1 <= nums[i] <= 104`

**Follow up:** If you have figured out the `O(n)` solution, try coding another solution of which the time complexity is `O(n log(n))`.

## Code Template

### Go
```go
func minSubArrayLen(target int, nums []int) int {
    
}
```

### TypeScript
```typescript
function minSubArrayLen(target: number, nums: number[]): number {
    
};
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
