---
id: 240
title: "Search a 2D Matrix II"
slug: search-a-2d-matrix-ii
difficulty: Medium
tags: [Array, Binary Search, Divide and Conquer, Matrix]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Search a 2D Matrix II

Write an efficient algorithm that searches for a value `target` in an `m x n` integer matrix `matrix`. This matrix has the following properties:

*   Integers in each row are sorted in ascending from left to right.
*   Integers in each column are sorted in ascending from top to bottom.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/11/24/searchgrid2.jpg)

**Input:** matrix = \[\[1,4,7,11,15\],\[2,5,8,12,19\],\[3,6,9,16,22\],\[10,13,14,17,24\],\[18,21,23,26,30\]\], target = 5
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/11/24/searchgrid.jpg)

**Input:** matrix = \[\[1,4,7,11,15\],\[2,5,8,12,19\],\[3,6,9,16,22\],\[10,13,14,17,24\],\[18,21,23,26,30\]\], target = 20
**Output:** false

**Constraints:**

*   `m == matrix.length`
*   `n == matrix[i].length`
*   `1 <= n, m <= 300`
*   `-109 <= matrix[i][j] <= 109`
*   All the integers in each row are **sorted** in ascending order.
*   All the integers in each column are **sorted** in ascending order.
*   `-109 <= target <= 109`

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
