---
id: 62
title: "Unique Paths"
slug: unique-paths
difficulty: Medium
tags: [Math, Dynamic Programming, Combinatorics]
neetcode150_category: 2-D Dynamic Programming
blind75_category: Dynamic Programming
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(m * n)
space_complexity: O(n)
insight: "走到一格的路徑數 = 上面加左邊，第 0 列與第 0 行填 1；就是 Climbing Stairs 攤到平面上"
---

# Unique Paths

There is a robot on an `m x n` grid. The robot is initially located at the **top-left corner** (i.e., `grid[0][0]`). The robot tries to move to the **bottom-right corner** (i.e., `grid[m - 1][n - 1]`). The robot can only move either down or right at any point in time.

Given the two integers `m` and `n`, return _the number of possible unique paths that the robot can take to reach the bottom-right corner_.

The test cases are generated so that the answer will be less than or equal to `2 * 109`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2018/10/22/robot_maze.png)

**Input:** m = 3, n = 7
**Output:** 28

**Example 2:**

**Input:** m = 3, n = 2
**Output:** 3
**Explanation:** From the top-left corner, there are a total of 3 ways to reach the bottom-right corner:
1. Right -> Down -> Down
2. Down -> Down -> Right
3. Down -> Right -> Down

**Constraints:**

*   `1 <= m, n <= 100`

## Code Template

### Go
```go
func uniquePaths(m int, n int) int {
    
}
```

### TypeScript
```typescript
function uniquePaths(m: number, n: number): number {
    
};
```
