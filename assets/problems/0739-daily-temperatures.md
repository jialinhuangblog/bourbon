---
id: 739
title: "Daily Temperatures"
slug: daily-temperatures
difficulty: Medium
tags: [Array, Stack, Monotonic Stack]
neetcode150_category: Stack
blind75_category: null
date_solved: 2026-07-13
languages: [golang, typescript]
time_complexity: null
space_complexity: null
"@local": [Monotonic Stack]
---

# Daily Temperatures

Given an array of integers `temperatures` represents the daily temperatures, return _an array_ `answer` _such that_ `answer[i]` _is the number of days you have to wait after the_ `ith` _day to get a warmer temperature_. If there is no future day for which this is possible, keep `answer[i] == 0` instead.

**Example 1:**

**Input:** temperatures = \[73,74,75,71,69,72,76,73\]
**Output:** \[1,1,4,2,1,1,0,0\]

**Example 2:**

**Input:** temperatures = \[30,40,50,60\]
**Output:** \[1,1,1,0\]

**Example 3:**

**Input:** temperatures = \[30,60,90\]
**Output:** \[1,1,0\]

**Constraints:**

*   `1 <= temperatures.length <= 105`
*   `30 <= temperatures[i] <= 100`

## Code Template

### Go
```go
func dailyTemperatures(temperatures []int) []int {
    
}
```

### TypeScript
```typescript
function dailyTemperatures(temperatures: number[]): number[] {
    
};
```
