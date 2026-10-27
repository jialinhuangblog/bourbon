---
id: 3714
title: "Longest Balanced Substring II"
slug: longest-balanced-substring-ii
difficulty: Medium
tags: [Hash Table, String, Prefix Sum]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Longest Balanced Substring II

You are given a string `s` consisting only of the characters `'a'`, `'b'`, and `'c'`.

A **substring** of `s` is called **balanced** if all **distinct** characters in the **substring** appear the **same** number of times.

Return the **length of the longest balanced substring** of `s`.

**Example 1:**

**Input:** s = "abbac"

**Output:** 4

**Explanation:**

The longest balanced substring is `"abba"` because both distinct characters `'a'` and `'b'` each appear exactly 2 times.

**Example 2:**

**Input:** s = "aabcc"

**Output:** 3

**Explanation:**

The longest balanced substring is `"abc"` because all distinct characters `'a'`, `'b'` and `'c'` each appear exactly 1 time.

**Example 3:**

**Input:** s = "aba"

**Output:** 2

**Explanation:**

One of the longest balanced substrings is `"ab"` because both distinct characters `'a'` and `'b'` each appear exactly 1 time. Another longest balanced substring is `"ba"`.

**Constraints:**

*   `1 <= s.length <= 105`
*   `s` contains only the characters `'a'`, `'b'`, and `'c'`.

## Code Template

### Go
```go
func longestBalanced(s string) int {
    
}
```

### TypeScript
```typescript
function longestBalanced(s: string): number {
    
};
```
