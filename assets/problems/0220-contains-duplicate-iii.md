---
id: 220
title: "Contains Duplicate III"
slug: contains-duplicate-iii
difficulty: Hard
tags: [Array, Sliding Window, Sorting, Bucket Sort, Ordered Set]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Contains Duplicate III

You are given an integer array `nums` and two integers `indexDiff` and `valueDiff`.

Find a pair of indices `(i, j)` such that:

*   `i != j`,
*   `abs(i - j) <= indexDiff`.
*   `abs(nums[i] - nums[j]) <= valueDiff`, and

Return `true` _if such pair exists or_ `false` _otherwise_.

**Example 1:**

**Input:** nums = \[1,2,3,1\], indexDiff = 3, valueDiff = 0
**Output:** true
**Explanation:** We can choose (i, j) = (0, 3).
We satisfy the three conditions:
i != j --> 0 != 3
abs(i - j) <= indexDiff --> abs(0 - 3) <= 3
abs(nums\[i\] - nums\[j\]) <= valueDiff --> abs(1 - 1) <= 0

**Example 2:**

**Input:** nums = \[1,5,9,1,5,9\], indexDiff = 2, valueDiff = 3
**Output:** false
**Explanation:** After trying all the possible pairs (i, j), we cannot satisfy the three conditions, so we return false.

**Constraints:**

*   `2 <= nums.length <= 105`
*   `-109 <= nums[i] <= 109`
*   `1 <= indexDiff <= nums.length`
*   `0 <= valueDiff <= 109`

## Code Template

### Go
```go
func containsNearbyAlmostDuplicate(nums []int, indexDiff int, valueDiff int) bool {
    
}
```

### TypeScript
```typescript
function containsNearbyAlmostDuplicate(nums: number[], indexDiff: number, valueDiff: number): boolean {
    
};
```
