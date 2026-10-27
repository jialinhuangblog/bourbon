---
id: 392
title: "Is Subsequence"
slug: is-subsequence
difficulty: Easy
tags: [Two Pointers, String, Dynamic Programming]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Is Subsequence

Given two strings `s` and `t`, return `true` _if_ `s` _is a **subsequence** of_ `t`_, or_ `false` _otherwise_.

A **subsequence** of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e., `"ace"` is a subsequence of `"abcde"` while `"aec"` is not).

**Example 1:**

**Input:** s = "abc", t = "ahbgdc"
**Output:** true

**Example 2:**

**Input:** s = "axc", t = "ahbgdc"
**Output:** false

**Constraints:**

*   `0 <= s.length <= 100`
*   `0 <= t.length <= 104`
*   `s` and `t` consist only of lowercase English letters.

**Follow up:** Suppose there are lots of incoming `s`, say `s1, s2, ..., sk` where `k >= 109`, and you want to check one by one to see if `t` has its subsequence. In this scenario, how would you change your code?

## Code Template

### Go
```go
func isSubsequence(s string, t string) bool {
    
}
```

### TypeScript
```typescript
function isSubsequence(s: string, t: string): boolean {
    
};
```
