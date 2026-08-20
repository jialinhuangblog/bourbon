---
id: 678
title: "Valid Parenthesis String"
slug: valid-parenthesis-string
difficulty: Medium
tags: [String, Dynamic Programming, Stack, Greedy]
neetcode150_category: Greedy
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Valid Parenthesis String

Given a string `s` containing only three types of characters: `'('`, `')'` and `'*'`, return `true` _if_ `s` _is **valid**_.

The following rules define a **valid** string:

*   Any left parenthesis `'('` must have a corresponding right parenthesis `')'`.
*   Any right parenthesis `')'` must have a corresponding left parenthesis `'('`.
*   Left parenthesis `'('` must go before the corresponding right parenthesis `')'`.
*   `'*'` could be treated as a single right parenthesis `')'` or a single left parenthesis `'('` or an empty string `""`.

**Example 1:**

**Input:** s = "()"
**Output:** true

**Example 2:**

**Input:** s = "(\*)"
**Output:** true

**Example 3:**

**Input:** s = "(\*))"
**Output:** true

**Constraints:**

*   `1 <= s.length <= 100`
*   `s[i]` is `'('`, `')'` or `'*'`.

## Code Template

### Go
```go
func checkValidString(s string) bool {
    
}
```

### TypeScript
```typescript
function checkValidString(s: string): boolean {
    
};
```
