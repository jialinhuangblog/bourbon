---
id: 1584
title: "Min Cost to Connect All Points"
slug: min-cost-to-connect-all-points
difficulty: Medium
tags: [Array, Union-Find, Graph Theory, Minimum Spanning Tree]
neetcode150_category: Advanced Graphs
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Min Cost to Connect All Points

You are given an array `points` representing integer coordinates of some points on a 2D-plane, where `points[i] = [xi, yi]`.

The cost of connecting two points `[xi, yi]` and `[xj, yj]` is the **manhattan distance** between them: `|xi - xj| + |yi - yj|`, where `|val|` denotes the absolute value of `val`.

Return _the minimum cost to make all points connected._ All points are connected if there is **exactly one** simple path between any two points.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/08/26/d.png)

**Input:** points = \[\[0,0\],\[2,2\],\[3,10\],\[5,2\],\[7,0\]\]
**Output:** 20
**Explanation:** 
![](https://assets.leetcode.com/uploads/2020/08/26/c.png)
We can connect the points as shown above to get the minimum cost of 20.
Notice that there is a unique path between every pair of points.

**Example 2:**

**Input:** points = \[\[3,12\],\[-2,5\],\[-4,1\]\]
**Output:** 18

**Constraints:**

*   `1 <= points.length <= 1000`
*   `-106 <= xi, yi <= 106`
*   All pairs `(xi, yi)` are distinct.

## Code Template

### Go
```go
func minCostConnectPoints(points [][]int) int {
    
}
```

### TypeScript
```typescript
function minCostConnectPoints(points: number[][]): number {
    
};
```
