---
id: 125
title: "Valid Palindrome"
slug: valid-palindrome
difficulty: Easy
tags: [Two Pointers, String]
neetcode150_category: Two Pointers
blind75_category: String
date_solved: 2026-03-22
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "two pointers; c|32 lowercase; skip non-alphanumeric"
---

# Valid Palindrome

A phrase is a **palindrome** if, after converting all uppercase letters into lowercase letters and removing all non-alphanumeric characters, it reads the same forward and backward. Alphanumeric characters include letters and numbers.

Given a string `s`, return `true` _if it is a **palindrome**, or_ `false` _otherwise_.

**Example 1:**

**Input:** s = "A man, a plan, a canal: Panama"
**Output:** true
**Explanation:** "amanaplanacanalpanama" is a palindrome.

**Example 2:**

**Input:** s = "race a car"
**Output:** false
**Explanation:** "raceacar" is not a palindrome.

**Example 3:**

**Input:** s = " "
**Output:** true
**Explanation:** s is an empty string "" after removing non-alphanumeric characters.
Since an empty string reads the same forward and backward, it is a palindrome.

**Constraints:**

*   `1 <= s.length <= 2 * 105`
*   `s` consists only of printable ASCII characters.

## Code Template

### Go
```go
func isPalindrome(s string) bool {
    
}
```

### TypeScript
```typescript
function isPalindrome(s: string): boolean {
    
};
```
