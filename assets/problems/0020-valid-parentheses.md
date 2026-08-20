---
id: 20
title: "Valid Parentheses"
slug: valid-parentheses
difficulty: Easy
tags: [String, Stack]
neetcode150_category: Stack
blind75_category: String
date_solved: 2026-03-23
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "open → push; close → pop and match"
---

# Valid Parentheses

Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1.  Open brackets must be closed by the same type of brackets.
2.  Open brackets must be closed in the correct order.
3.  Every close bracket has a corresponding open bracket of the same type.

**Example 1:**

**Input:** s = "()"

**Output:** true

**Example 2:**

**Input:** s = "()\[\]{}"

**Output:** true

**Example 3:**

**Input:** s = "(\]"

**Output:** false

**Example 4:**

**Input:** s = "(\[\])"

**Output:** true

**Example 5:**

**Input:** s = "(\[)\]"

**Output:** false

**Constraints:**

*   `1 <= s.length <= 104`
*   `s` consists of parentheses only `'()[]{}'`.

## Code Template

### Go
```go
func isValid(s string) bool {
    
}
```

### TypeScript
```typescript
function isValid(s: string): boolean {
    
};
```
