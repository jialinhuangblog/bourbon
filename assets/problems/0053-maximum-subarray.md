---
id: 53
title: "Maximum Subarray"
slug: maximum-subarray
difficulty: Medium
tags: [Array, Divide and Conquer, Dynamic Programming]
neetcode150_category: Greedy
blind75_category: Array
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(1)
insight: "問「以第 i 格結尾的最佳段」，前面累積是負的就丟掉重來；全負數要把 best 初始成 nums[0]"
---

# Maximum Subarray

Given an integer array `nums`, find the subarray with the largest sum, and return _its sum_.

**Example 1:**

**Input:** nums = \[-2,1,-3,4,-1,2,1,-5,4\]
**Output:** 6
**Explanation:** The subarray \[4,-1,2,1\] has the largest sum 6.

**Example 2:**

**Input:** nums = \[1\]
**Output:** 1
**Explanation:** The subarray \[1\] has the largest sum 1.

**Example 3:**

**Input:** nums = \[5,4,-1,7,8\]
**Output:** 23
**Explanation:** The subarray \[5,4,-1,7,8\] has the largest sum 23.

**Constraints:**

*   `1 <= nums.length <= 105`
*   `-104 <= nums[i] <= 104`

**Follow up:** If you have figured out the `O(n)` solution, try coding another solution using the **divide and conquer** approach, which is more subtle.

## Code Template

### Go
```go
func maxSubArray(nums []int) int {
    
}
```

### TypeScript
```typescript
function maxSubArray(nums: number[]): number {
    
};
```
