---
id: 3346
title: "Maximum Frequency of an Element After Performing Operations I"
slug: maximum-frequency-of-an-element-after-performing-operations-i
difficulty: Medium
tags: [Array, Binary Search, Sliding Window, Sorting, Prefix Sum]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Maximum Frequency of an Element After Performing Operations I

You are given an integer array `nums` and two integers `k` and `numOperations`.

You must perform an **operation** `numOperations` times on `nums`, where in each operation you:

*   Select an index `i` that was **not** selected in any previous operations.
*   Add an integer in the range `[-k, k]` to `nums[i]`.

Return the **maximum** possible frequency of any element in `nums` after performing the **operations**.

**Example 1:**

**Input:** nums = \[1,4,5\], k = 1, numOperations = 2

**Output:** 2

**Explanation:**

We can achieve a maximum frequency of two by:

*   Adding 0 to `nums[1]`. `nums` becomes `[1, 4, 5]`.
*   Adding -1 to `nums[2]`. `nums` becomes `[1, 4, 4]`.

**Example 2:**

**Input:** nums = \[5,11,20,20\], k = 5, numOperations = 1

**Output:** 2

**Explanation:**

We can achieve a maximum frequency of two by:

*   Adding 0 to `nums[1]`.

**Constraints:**

*   `1 <= nums.length <= 105`
*   `1 <= nums[i] <= 105`
*   `0 <= k <= 105`
*   `0 <= numOperations <= nums.length`

## Code Template

### Go
```go
func maxFrequency(nums []int, k int, numOperations int) int {
    
}
```

### TypeScript
```typescript
function maxFrequency(nums: number[], k: number, numOperations: number): number {
    
};
```
