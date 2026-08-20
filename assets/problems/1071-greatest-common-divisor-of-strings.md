---
id: 1071
title: "Greatest Common Divisor of Strings"
slug: greatest-common-divisor-of-strings
difficulty: Easy
tags: [Math, String]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Greatest Common Divisor of Strings

For two strings `s` and `t`, we say "`t` divides `s`" if and only if `s = t + t + t + ... + t + t` (i.e., `t` is concatenated with itself one or more times).

Given two strings `str1` and `str2`, return _the largest string_ `x` _such that_ `x` _divides both_ `str1` _and_ `str2`.

**Example 1:**

**Input:** str1 = "ABCABC", str2 = "ABC"

**Output:** "ABC"

**Example 2:**

**Input:** str1 = "ABABAB", str2 = "ABAB"

**Output:** "AB"

**Example 3:**

**Input:** str1 = "LEET", str2 = "CODE"

**Output:** ""

**Example 4:**

**Input:** str1 = "AAAAAB", str2 = "AAA"

**Output:** ""​​​​​​​

**Constraints:**

*   `1 <= str1.length, str2.length <= 1000`
*   `str1` and `str2` consist of English uppercase letters.

## Code Template

### Go
```go
func gcdOfStrings(str1 string, str2 string) string {
    
}
```

### TypeScript
```typescript
function gcdOfStrings(str1: string, str2: string): string {
    
};
```
