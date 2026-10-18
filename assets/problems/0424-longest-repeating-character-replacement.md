---
id: 424
title: "Longest Repeating Character Replacement"
slug: longest-repeating-character-replacement
difficulty: Medium
tags: [Hash Table, String, Sliding Window]
neetcode150_category: Sliding Window
blind75_category: String
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Longest Repeating Character Replacement

You are given a string `s` and an integer `k`. You can choose any character of the string and change it to any other uppercase English character. You can perform this operation at most `k` times.

Return _the length of the longest substring containing the same letter you can get after performing the above operations_.

**Example 1:**

**Input:** s = "ABAB", k = 2
**Output:** 4
**Explanation:** Replace the two 'A's with two 'B's or vice versa.

**Example 2:**

**Input:** s = "AABABBA", k = 1
**Output:** 4
**Explanation:** Replace the one 'A' in the middle with 'B' and form "AABBBBA".
The substring "BBBB" has the longest repeating letters, which is 4.
There may exists other ways to achieve this answer too.

**Constraints:**

*   `1 <= s.length <= 105`
*   `s` consists of only uppercase English letters.
*   `0 <= k <= s.length`

## Code Template

### Go
```go
func characterReplacement(s string, k int) int {
    
}
```

### TypeScript
```typescript
function characterReplacement(s: string, k: number): number {
    
};
```
