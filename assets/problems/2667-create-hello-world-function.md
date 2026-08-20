---
id: 2667
title: "Create Hello World Function"
slug: create-hello-world-function
difficulty: Easy
tags: []
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [typescript]
time_complexity: null
space_complexity: null
---

# Create Hello World Function

Write a function `createHelloWorld`. It should return a new function that always returns `"Hello World"`.

**Example 1:**

**Input:** args = \[\]
**Output:** "Hello World"
**Explanation:**
const f = createHelloWorld();
f(); // "Hello World"

The function returned by createHelloWorld should always return "Hello World".

**Example 2:**

**Input:** args = \[{},null,42\]
**Output:** "Hello World"
**Explanation:**
const f = createHelloWorld();
f({}, null, 42); // "Hello World"

Any arguments could be passed to the function but it should still always return "Hello World".

**Constraints:**

*   `0 <= args.length <= 10`

## Code Template

### TypeScript
```typescript
function createHelloWorld() {
    
    return function(...args): string {
        
    };
};

/**
 * const f = createHelloWorld();
 * f(); // "Hello World"
 */
```
