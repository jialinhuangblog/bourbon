---
id: 1411
title: "Number of Ways to Paint N × 3 Grid"
slug: number-of-ways-to-paint-n-3-grid
difficulty: Hard
tags: [Dynamic Programming]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Number of Ways to Paint N × 3 Grid

You have a `grid` of size `n x 3` and you want to paint each cell of the grid with exactly one of the three colors: **Red**, **Yellow,** or **Green** while making sure that no two adjacent cells have the same color (i.e., no two cells that share vertical or horizontal sides have the same color).

Given `n` the number of rows of the grid, return _the number of ways_ you can paint this `grid`. As the answer may grow large, the answer **must be** computed modulo `109 + 7`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/03/26/e1.png)

**Input:** n = 1
**Output:** 12
**Explanation:** There are 12 possible way to paint the grid as shown.

**Example 2:**

**Input:** n = 5000
**Output:** 30228214

**Constraints:**

*   `n == grid.length`
*   `1 <= n <= 5000`

## Code Template

### Go
```go
func numOfWays(n int) int {
    
}
```

### TypeScript
```typescript
function numOfWays(n: number): number {
    
};
```
