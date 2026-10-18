---
id: 1353
title: "Maximum Number of Events That Can Be Attended"
slug: maximum-number-of-events-that-can-be-attended
difficulty: Medium
tags: [Array, Greedy, Sorting, Heap (Priority Queue)]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Maximum Number of Events That Can Be Attended

You are given an array of `events` where `events[i] = [startDayi, endDayi]`. Every event `i` starts at `startDayi` and ends at `endDayi`.

You can attend an event `i` at any day `d` where `startDayi <= d <= endDayi`. You can only attend one event at any time `d`.

Return _the maximum number of events you can attend_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/02/05/e1.png)

**Input:** events = \[\[1,2\],\[2,3\],\[3,4\]\]
**Output:** 3
**Explanation:** You can attend all the three events.
One way to attend them all is as shown.
Attend the first event on day 1.
Attend the second event on day 2.
Attend the third event on day 3.

**Example 2:**

**Input:** events= \[\[1,2\],\[2,3\],\[3,4\],\[1,2\]\]
**Output:** 4

**Constraints:**

*   `1 <= events.length <= 105`
*   `events[i].length == 2`
*   `1 <= startDayi <= endDayi <= 105`

## Code Template

### Go
```go
func maxEvents(events [][]int) int {
    
}
```

### TypeScript
```typescript
function maxEvents(events: number[][]): number {
    
};
```
