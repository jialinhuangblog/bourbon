---
id: 698
title: "Partition to K Equal Sum Subsets"
slug: partition-to-k-equal-sum-subsets
difficulty: Medium
tags: [Array, Dynamic Programming, Backtracking, Bit Manipulation, Memoization, Bitmask]
"@local": [Operator]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Partition to K Equal Sum Subsets

Given an integer array `nums` and an integer `k`, return `true` if it is possible to divide this array into `k` non-empty subsets whose sums are all equal.

**Example 1:**

**Input:** nums = \[4,3,2,3,5,2,1\], k = 4
**Output:** true
**Explanation:** It is possible to divide it into 4 subsets (5), (1, 4), (2,3), (2,3) with equal sums.

**Example 2:**

**Input:** nums = \[1,2,3,4\], k = 3
**Output:** false

**Constraints:**

*   `1 <= k <= nums.length <= 16`
*   `1 <= nums[i] <= 104`
*   The frequency of each element is in the range `[1, 4]`.

## Code Template

### Go
```go
func canPartitionKSubsets(nums []int, k int) bool {
    
}
```

### TypeScript
```typescript
function canPartitionKSubsets(nums: number[], k: number): boolean {
    
};
```
