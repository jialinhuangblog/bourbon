---
id: 1047
title: "Remove All Adjacent Duplicates In String"
slug: remove-all-adjacent-duplicates-in-string
difficulty: Easy
tags: [String, Stack]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Remove All Adjacent Duplicates In String

You are given a string `s` consisting of lowercase English letters. A **duplicate removal** consists of choosing two **adjacent** and **equal** letters and removing them.

We repeatedly make **duplicate removals** on `s` until we no longer can.

Return _the final string after all such duplicate removals have been made_. It can be proven that the answer is **unique**.

**Example 1:**

**Input:** s = "abbaca"
**Output:** "ca"
**Explanation:** 
For example, in "abbaca" we could remove "bb" since the letters are adjacent and equal, and this is the only possible move.  The result of this move is that the string is "aaca", of which only "aa" is possible, so the final string is "ca".

**Example 2:**

**Input:** s = "azxxzy"
**Output:** "ay"

**Constraints:**

*   `1 <= s.length <= 105`
*   `s` consists of lowercase English letters.

## Code Template

### Go
```go
func removeDuplicates(s string) string {
    
}
```

### TypeScript
```typescript
function removeDuplicates(s: string): string {
    
};
```
