---
id: 695
title: "Max Area of Island"
slug: max-area-of-island
difficulty: Medium
tags: [Array, Depth-First Search, Breadth-First Search, Union-Find, Matrix]
neetcode150_category: Graphs
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Max Area of Island

You are given an `m x n` binary matrix `grid`. An island is a group of `1`'s (representing land) connected **4-directionally** (horizontal or vertical.) You may assume all four edges of the grid are surrounded by water.

The **area** of an island is the number of cells with a value `1` in the island.

Return _the maximum **area** of an island in_ `grid`. If there is no island, return `0`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/05/01/maxarea1-grid.jpg)

**Input:** grid = \[\[0,0,1,0,0,0,0,1,0,0,0,0,0\],\[0,0,0,0,0,0,0,1,1,1,0,0,0\],\[0,1,1,0,1,0,0,0,0,0,0,0,0\],\[0,1,0,0,1,1,0,0,1,0,1,0,0\],\[0,1,0,0,1,1,0,0,1,1,1,0,0\],\[0,0,0,0,0,0,0,0,0,0,1,0,0\],\[0,0,0,0,0,0,0,1,1,1,0,0,0\],\[0,0,0,0,0,0,0,1,1,0,0,0,0\]\]
**Output:** 6
**Explanation:** The answer is not 11, because the island must be connected 4-directionally.

**Example 2:**

**Input:** grid = \[\[0,0,0,0,0,0,0,0\]\]
**Output:** 0

**Constraints:**

*   `m == grid.length`
*   `n == grid[i].length`
*   `1 <= m, n <= 50`
*   `grid[i][j]` is either `0` or `1`.

## Code Template

### Go
```go
func maxAreaOfIsland(grid [][]int) int {
    
}
```

### TypeScript
```typescript
function maxAreaOfIsland(grid: number[][]): number {
    
};
```
