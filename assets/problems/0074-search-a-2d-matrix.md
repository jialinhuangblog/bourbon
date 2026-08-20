---
id: 74
title: "Search a 2D Matrix"
slug: search-a-2d-matrix
difficulty: Medium
tags: [Array, Binary Search, Matrix]
neetcode150_category: Binary Search
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Search a 2D Matrix

You are given an `m x n` integer matrix `matrix` with the following two properties:

*   Each row is sorted in non-decreasing order.
*   The first integer of each row is greater than the last integer of the previous row.

Given an integer `target`, return `true` _if_ `target` _is in_ `matrix` _or_ `false` _otherwise_.

You must write a solution in `O(log(m * n))` time complexity.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/05/mat.jpg)

**Input:** matrix = \[\[1,3,5,7\],\[10,11,16,20\],\[23,30,34,60\]\], target = 3
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/10/05/mat2.jpg)

**Input:** matrix = \[\[1,3,5,7\],\[10,11,16,20\],\[23,30,34,60\]\], target = 13
**Output:** false

**Constraints:**

*   `m == matrix.length`
*   `n == matrix[i].length`
*   `1 <= m, n <= 100`
*   `-104 <= matrix[i][j], target <= 104`

## Code Template

### Go
```go
func searchMatrix(matrix [][]int, target int) bool {
    
}
```

### TypeScript
```typescript
function searchMatrix(matrix: number[][], target: number): boolean {
    
};
```
