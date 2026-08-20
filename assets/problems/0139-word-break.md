---
id: 139
title: "Word Break"
slug: word-break
difficulty: Medium
tags: [Array, Hash Table, String, Dynamic Programming, Trie, Memoization]
neetcode150_category: 1-D Dynamic Programming
blind75_category: Dynamic Programming
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Word Break

Given a string `s` and a dictionary of strings `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of one or more dictionary words.

**Note** that the same word in the dictionary may be reused multiple times in the segmentation.

**Example 1:**

**Input:** s = "leetcode", wordDict = \["leet","code"\]
**Output:** true
**Explanation:** Return true because "leetcode" can be segmented as "leet code".

**Example 2:**

**Input:** s = "applepenapple", wordDict = \["apple","pen"\]
**Output:** true
**Explanation:** Return true because "applepenapple" can be segmented as "apple pen apple".
Note that you are allowed to reuse a dictionary word.

**Example 3:**

**Input:** s = "catsandog", wordDict = \["cats","dog","sand","and","cat"\]
**Output:** false

**Constraints:**

*   `1 <= s.length <= 300`
*   `1 <= wordDict.length <= 1000`
*   `1 <= wordDict[i].length <= 20`
*   `s` and `wordDict[i]` consist of only lowercase English letters.
*   All the strings of `wordDict` are **unique**.

## Code Template

### Go
```go
func wordBreak(s string, wordDict []string) bool {
    
}
```

### TypeScript
```typescript
function wordBreak(s: string, wordDict: string[]): boolean {
    
};
```
