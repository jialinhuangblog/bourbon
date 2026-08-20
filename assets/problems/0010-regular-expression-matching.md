---
id: 10
title: "Regular Expression Matching"
slug: regular-expression-matching
difficulty: Hard
tags: [String, Dynamic Programming, Recursion]
neetcode150_category: 2-D Dynamic Programming
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Regular Expression Matching

Given an input string `s` and a pattern `p`, implement regular expression matching with support for `'.'` and `'*'` where:

*   `'.'` Matches any single character.​​​​
*   `'*'` Matches zero or more of the preceding element.

Return a boolean indicating whether the matching covers the entire input string (not partial).

**Example 1:**

**Input:** s = "aa", p = "a"
**Output:** false
**Explanation:** "a" does not match the entire string "aa".

**Example 2:**

**Input:** s = "aa", p = "a\*"
**Output:** true
**Explanation:** '\*' means zero or more of the preceding element, 'a'. Therefore, by repeating 'a' once, it becomes "aa".

**Example 3:**

**Input:** s = "ab", p = ".\*"
**Output:** true
**Explanation:** ".\*" means "zero or more (\*) of any character (.)".

**Constraints:**

*   `1 <= s.length <= 20`
*   `1 <= p.length <= 20`
*   `s` contains only lowercase English letters.
*   `p` contains only lowercase English letters, `'.'`, and `'*'`.
*   It is guaranteed for each appearance of the character `'*'`, there will be a previous valid character to match.

## Code Template

### Go
```go
func isMatch(s string, p string) bool {
    
}
```

### TypeScript
```typescript
function isMatch(s: string, p: string): boolean {
    
};
```
