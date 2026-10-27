---
id: 371
title: "Sum of Two Integers"
slug: sum-of-two-integers
difficulty: Medium
tags: [Math, Bit Manipulation]
"@local": [Operator]
neetcode150_category: Bit Manipulation
blind75_category: Binary
date_solved: 2026-03-29
languages: [golang, typescript]
time_complexity: O(1)
space_complexity: O(1)
insight: "a^b = 不帶進位的和；(a&b)<<1 = 進位；重複直到進位為 0"
---

# Sum of Two Integers

Given two integers `a` and `b`, return _the sum of the two integers without using the operators_ `+` _and_ `-`.

**Example 1:**

**Input:** a = 1, b = 2
**Output:** 3

**Example 2:**

**Input:** a = 2, b = 3
**Output:** 5

**Constraints:**

*   `-1000 <= a, b <= 1000`

## Code Template

### Go
```go
func getSum(a int, b int) int {
    
}
```

### TypeScript
```typescript
function getSum(a: number, b: number): number {
    
};
```
