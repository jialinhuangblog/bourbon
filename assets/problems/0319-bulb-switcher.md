---
id: 319
title: "Bulb Switcher"
slug: bulb-switcher
difficulty: Medium
tags: [Math, Brainteaser]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Bulb Switcher

There are `n` bulbs that are initially off. You first turn on all the bulbs, then you turn off every second bulb.

On the third round, you toggle every third bulb (turning on if it's off or turning off if it's on). For the `ith` round, you toggle every `i` bulb. For the `nth` round, you only toggle the last bulb.

Return _the number of bulbs that are on after `n` rounds_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/11/05/bulb.jpg)

**Input:** n = 3
**Output:** 1
**Explanation:** At first, the three bulbs are \[off, off, off\].
After the first round, the three bulbs are \[on, on, on\].
After the second round, the three bulbs are \[on, off, on\].
After the third round, the three bulbs are \[on, off, off\]. 
So you should return 1 because there is only one bulb is on.

**Example 2:**

**Input:** n = 0
**Output:** 0

**Example 3:**

**Input:** n = 1
**Output:** 1

**Constraints:**

*   `0 <= n <= 109`

## Code Template

### Go
```go
func bulbSwitch(n int) int {
    
}
```

### TypeScript
```typescript
function bulbSwitch(n: number): number {
    
};
```
