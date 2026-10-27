---
id: 1539
title: "Kth Missing Positive Number"
slug: kth-missing-positive-number
difficulty: Easy
tags: [Array, Binary Search]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Kth Missing Positive Number

Given an array `arr` of positive integers sorted in a **strictly increasing order**, and an integer `k`.

Return _the_ `kth` _**positive** integer that is **missing** from this array._

**Example 1:**

**Input:** arr = \[2,3,4,7,11\], k = 5
**Output:** 9
**Explanation:** The missing positive integers are \[1,5,6,8,9,10,12,13,...\]. The 5th missing positive integer is 9.

**Example 2:**

**Input:** arr = \[1,2,3,4\], k = 2
**Output:** 6
**Explanation:** The missing positive integers are \[5,6,7,...\]. The 2nd missing positive integer is 6.

**Constraints:**

*   `1 <= arr.length <= 1000`
*   `1 <= arr[i] <= 1000`
*   `1 <= k <= 1000`
*   `arr[i] < arr[j]` for `1 <= i < j <= arr.length`

**Follow up:**

Could you solve this problem in less than O(n) complexity?

## Code Template

### Go
```go
func findKthPositive(arr []int, k int) int {
    
}
```

### TypeScript
```typescript
function findKthPositive(arr: number[], k: number): number {
    
};
```
