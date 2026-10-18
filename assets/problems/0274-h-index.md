---
id: 274
title: "H-Index"
slug: h-index
difficulty: Medium
tags: [Array, Sorting, Counting Sort]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# H-Index

Given an array of integers `citations` where `citations[i]` is the number of citations a researcher received for their `ith` paper, return _the researcher's h-index_.

According to the [definition of h-index on Wikipedia](https://en.wikipedia.org/wiki/H-index): The h-index is defined as the maximum value of `h` such that the given researcher has published at least `h` papers that have each been cited at least `h` times.

**Example 1:**

**Input:** citations = \[3,0,6,1,5\]
**Output:** 3
**Explanation:** \[3,0,6,1,5\] means the researcher has 5 papers in total and each of them had received 3, 0, 6, 1, 5 citations respectively.
Since the researcher has 3 papers with at least 3 citations each and the remaining two with no more than 3 citations each, their h-index is 3.

**Example 2:**

**Input:** citations = \[1,3,1\]
**Output:** 1

**Constraints:**

*   `n == citations.length`
*   `1 <= n <= 5000`
*   `0 <= citations[i] <= 1000`

## Code Template

### Go
```go
func hIndex(citations []int) int {
    
}
```

### TypeScript
```typescript
function hIndex(citations: number[]): number {
    
};
```
