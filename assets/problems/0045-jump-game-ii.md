---
id: 45
title: "Jump Game II"
slug: jump-game-ii
difficulty: Medium
tags: [Array, Dynamic Programming, Greedy]
neetcode150_category: Greedy
blind75_category: null
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(1)
"@local": [Greedy]
insight: "跳 k 次能到的是連續一段，整層只記右界；掃到右界才結算一跳，踩哪一格不用選"
---

# Jump Game II

You are given a **0-indexed** array of integers `nums` of length `n`. You are initially positioned at index 0.

Each element `nums[i]` represents the maximum length of a forward jump from index `i`. In other words, if you are at index `i`, you can jump to any index `(i + j)` where:

*   `0 <= j <= nums[i]` and
*   `i + j < n`

Return _the minimum number of jumps to reach index_ `n - 1`. The test cases are generated such that you can reach index `n - 1`.

**Example 1:**

**Input:** nums = \[2,3,1,1,4\]
**Output:** 2
**Explanation:** The minimum number of jumps to reach the last index is 2. Jump 1 step from index 0 to 1, then 3 steps to the last index.

**Example 2:**

**Input:** nums = \[2,3,0,1,4\]
**Output:** 2

**Constraints:**

*   `1 <= nums.length <= 104`
*   `0 <= nums[i] <= 1000`
*   It's guaranteed that you can reach `nums[n - 1]`.

## Code Template

### Go
```go
func jump(nums []int) int {
    
}
```

### TypeScript
```typescript
function jump(nums: number[]): number {
    
};
```
