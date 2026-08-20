---
id: 2523
title: "Closest Prime Numbers in Range"
slug: closest-prime-numbers-in-range
difficulty: Medium
tags: [Math, Number Theory, Primality Test, Sieve Theory, Prime Number Sieve]
neetcode150_category: null
blind75_category: null
date_solved: 2026-08-04
languages: [golang, typescript]
time_complexity: O(n log log n)
space_complexity: O(n)
insight: "篩到 right；排好序的質數裡最近的一對必相鄰，掃一遍；gap ≤ 2 提前停"
---

# Closest Prime Numbers in Range

Given two positive integers `left` and `right`, find the two integers `num1` and `num2` such that:

*   `left <= num1 < num2 <= right` .
*   Both `num1` and `num2` are prime numbers.
*   `num2 - num1` is the **minimum** amongst all other pairs satisfying the above conditions.

Return the positive integer array `ans = [num1, num2]`. If there are multiple pairs satisfying these conditions, return the one with the **smallest** `num1` value. If no such numbers exist, return `[-1, -1]`_._

**Example 1:**

**Input:** left = 10, right = 19
**Output:** \[11,13\]
**Explanation:** The prime numbers between 10 and 19 are 11, 13, 17, and 19.
The closest gap between any pair is 2, which can be achieved by \[11,13\] or \[17,19\].
Since 11 is smaller than 17, we return the first pair.

**Example 2:**

**Input:** left = 4, right = 6
**Output:** \[-1,-1\]
**Explanation:** There exists only one prime number in the given range, so the conditions cannot be satisfied.

**Constraints:**

*   `1 <= left <= right <= 10^6`

## Code Template

### Go
```go
func closestPrimes(left int, right int) []int {
    
}
```

### TypeScript
```typescript
function closestPrimes(left: number, right: number): number[] {
    
};
```
