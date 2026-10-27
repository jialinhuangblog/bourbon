---
id: 166
title: "Fraction to Recurring Decimal"
slug: fraction-to-recurring-decimal
difficulty: Medium
tags: [Hash Table, Math, String]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Fraction to Recurring Decimal

Given two integers representing the `numerator` and `denominator` of a fraction, return _the fraction in string format_.

If the fractional part is repeating, enclose the repeating part in parentheses

If multiple answers are possible, return **any of them**.

It is **guaranteed** that the length of the answer string is less than `104` for all the given inputs.

**Note** that if the fraction can be represented as a _finite length string_, you **must** return it.

**Example 1:**

**Input:** numerator = 1, denominator = 2
**Output:** "0.5"

**Example 2:**

**Input:** numerator = 2, denominator = 1
**Output:** "2"

**Example 3:**

**Input:** numerator = 4, denominator = 333
**Output:** "0.(012)"

**Constraints:**

*   `-231 <= numerator, denominator <= 231 - 1`
*   `denominator != 0`

## Code Template

### Go
```go
func fractionToDecimal(numerator int, denominator int) string {
    
}
```

### TypeScript
```typescript
function fractionToDecimal(numerator: number, denominator: number): string {
    
};
```
