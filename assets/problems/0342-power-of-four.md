---
id: 342
title: "Power of Four"
slug: power-of-four
difficulty: Easy
tags: [Math, Bit Manipulation, Recursion]
"@local": [Operator]
neetcode150_category: null
blind75_category: null
date_solved: 2026-02-24
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "n>0 && n&(n-1)==0 && n%3==1"
---

# Power of Four

Given an integer `n`, return _`true` if it is a power of four. Otherwise, return `false`_.

An integer `n` is a power of four, if there exists an integer `x` such that `n == 4x`.

**Example 1:**

**Input:** n = 16
**Output:** true

**Example 2:**

**Input:** n = 5
**Output:** false

**Example 3:**

**Input:** n = 1
**Output:** true

**Constraints:**

*   `-231 <= n <= 231 - 1`

**Follow up:** Could you solve it without loops/recursion?

## Code Template

### Go
```go
func isPowerOfFour(n int) bool {
    
}
```

### TypeScript
```typescript
function isPowerOfFour(n: number): boolean {
    
};
```
