---
id: 435
title: "Non-overlapping Intervals"
slug: non-overlapping-intervals
difficulty: Medium
tags: [Array, Dynamic Programming, Greedy, Sorting]
neetcode150_category: Intervals
blind75_category: Interval
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(n log n)
space_complexity: O(1)
"@local": [Greedy]
insight: "移除最少 = 保留最多；按結束時間排序才成立，按開始時間會被長區間佔住"
---

# Non-overlapping Intervals

Given an array of intervals `intervals` where `intervals[i] = [starti, endi]`, return _the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping_.

**Note** that intervals which only touch at a point are **non-overlapping**. For example, `[1, 2]` and `[2, 3]` are non-overlapping.

**Example 1:**

**Input:** intervals = \[\[1,2\],\[2,3\],\[3,4\],\[1,3\]\]
**Output:** 1
**Explanation:** \[1,3\] can be removed and the rest of the intervals are non-overlapping.

**Example 2:**

**Input:** intervals = \[\[1,2\],\[1,2\],\[1,2\]\]
**Output:** 2
**Explanation:** You need to remove two \[1,2\] to make the rest of the intervals non-overlapping.

**Example 3:**

**Input:** intervals = \[\[1,2\],\[2,3\]\]
**Output:** 0
**Explanation:** You don't need to remove any of the intervals since they're already non-overlapping.

**Constraints:**

*   `1 <= intervals.length <= 105`
*   `intervals[i].length == 2`
*   `-5 * 104 <= starti < endi <= 5 * 104`

## Code Template

### Go
```go
func eraseOverlapIntervals(intervals [][]int) int {
    
}
```

### TypeScript
```typescript
function eraseOverlapIntervals(intervals: number[][]): number {
    
};
```
