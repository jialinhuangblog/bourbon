---
id: 1143
title: "Longest Common Subsequence"
slug: longest-common-subsequence
difficulty: Medium
tags: [String, Dynamic Programming]
neetcode150_category: 2-D Dynamic Programming
blind75_category: Dynamic Programming
date_solved: 2026-08-09
languages: [golang, typescript]
time_complexity: O(m * n)
space_complexity: O(n)
insight: "字元相同就左上角加一，不同就上面跟左邊挑大的；滾動要留兩列，因為左上角住在上一列"
---

# Longest Common Subsequence

Given two strings `text1` and `text2`, return _the length of their longest **common subsequence**._ If there is no **common subsequence**, return `0`.

A **subsequence** of a string is a new string generated from the original string with some characters (can be none) deleted without changing the relative order of the remaining characters.

*   For example, `"ace"` is a subsequence of `"abcde"`.

A **common subsequence** of two strings is a subsequence that is common to both strings.

**Example 1:**

**Input:** text1 = "abcde", text2 = "ace" 
**Output:** 3  
**Explanation:** The longest common subsequence is "ace" and its length is 3.

**Example 2:**

**Input:** text1 = "abc", text2 = "abc"
**Output:** 3
**Explanation:** The longest common subsequence is "abc" and its length is 3.

**Example 3:**

**Input:** text1 = "abc", text2 = "def"
**Output:** 0
**Explanation:** There is no such common subsequence, so the result is 0.

**Constraints:**

*   `1 <= text1.length, text2.length <= 1000`
*   `text1` and `text2` consist of only lowercase English characters.

## Code Template

### Go
```go
func longestCommonSubsequence(text1 string, text2 string) int {
    
}
```

### TypeScript
```typescript
function longestCommonSubsequence(text1: string, text2: string): number {
    
};
```
