---
id: 200
title: "Number of Islands"
slug: number-of-islands
difficulty: Medium
tags: [Array, Depth-First Search, Breadth-First Search, Union-Find, Matrix]
neetcode150_category: Graphs
blind75_category: Graph
date_solved: 2026-03-24
date_updated: 2026-04-22
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "DFS sink visited cells to 0; count DFS calls"
---

# Number of Islands

Given an `m x n` 2D binary grid `grid` which represents a map of `'1'`s (land) and `'0'`s (water), return _the number of islands_.

An **island** is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

**Example 1:**

**Input:** grid = \[
  \["1","1","1","1","0"\],
  \["1","1","0","1","0"\],
  \["1","1","0","0","0"\],
  \["0","0","0","0","0"\]
\]
**Output:** 1

**Example 2:**

**Input:** grid = \[
  \["1","1","0","0","0"\],
  \["1","1","0","0","0"\],
  \["0","0","1","0","0"\],
  \["0","0","0","1","1"\]
\]
**Output:** 3

**Constraints:**

*   `m == grid.length`
*   `n == grid[i].length`
*   `1 <= m, n <= 300`
*   `grid[i][j]` is `'0'` or `'1'`.

## Code Template

### Go
```go
func numIslands(grid [][]byte) int {
    
}
```

### TypeScript
```typescript
function numIslands(grid: string[][]): number {
    
};
```
