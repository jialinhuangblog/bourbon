---
id: 128
title: "Longest Consecutive Sequence"
slug: longest-consecutive-sequence
difficulty: Medium
tags: [Array, Hash Table, Union-Find]
neetcode150_category: Arrays & Hashing
blind75_category: Graph
date_solved: 2026-04-16
date_updated: 2026-04-22
languages: [golang, typescript]
time_complexity: O(n)
space_complexity: O(n)
insight: "只從 num-1 不在 set 的起點往右數，每個數最多碰一次 → O(n)"
---

# Longest Consecutive Sequence

Given an unsorted array of integers `nums`, return _the length of the longest consecutive elements sequence._

You must write an algorithm that runs in `O(n)` time.

**Example 1:**

**Input:** nums = \[100,4,200,1,3,2\]
**Output:** 4
**Explanation:** The longest consecutive elements sequence is `[1, 2, 3, 4]`. Therefore its length is 4.

**Example 2:**

**Input:** nums = \[0,3,7,2,5,8,4,6,0,1\]
**Output:** 9

**Example 3:**

**Input:** nums = \[1,0,1,2\]
**Output:** 3

**Constraints:**

*   `0 <= nums.length <= 105`
*   `-109 <= nums[i] <= 109`

## Code Template

### Go
```go
func longestConsecutive(nums []int) int {
    
}
```

### TypeScript
```typescript
function longestConsecutive(nums: number[]): number {
    
};
```
