---
id: 605
title: "Can Place Flowers"
slug: can-place-flowers
difficulty: Easy
tags: [Array, Greedy]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Can Place Flowers

You have a long flowerbed in which some of the plots are planted, and some are not. However, flowers cannot be planted in **adjacent** plots.

Given an integer array `flowerbed` containing `0`'s and `1`'s, where `0` means empty and `1` means not empty, and an integer `n`, return `true` _if_ `n` _new flowers can be planted in the_ `flowerbed` _without violating the no-adjacent-flowers rule and_ `false` _otherwise_.

**Example 1:**

**Input:** flowerbed = \[1,0,0,0,1\], n = 1
**Output:** true

**Example 2:**

**Input:** flowerbed = \[1,0,0,0,1\], n = 2
**Output:** false

**Constraints:**

*   `1 <= flowerbed.length <= 2 * 104`
*   `flowerbed[i]` is `0` or `1`.
*   There are no two adjacent flowers in `flowerbed`.
*   `0 <= n <= flowerbed.length`

## Code Template

### Go
```go
func canPlaceFlowers(flowerbed []int, n int) bool {
    
}
```

### TypeScript
```typescript
function canPlaceFlowers(flowerbed: number[], n: number): boolean {
    
};
```
