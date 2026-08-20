---
id: 341
title: "Flatten Nested List Iterator"
slug: flatten-nested-list-iterator
difficulty: Medium
tags: [Stack, Tree, Depth-First Search, Design, Queue, Iterator]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Flatten Nested List Iterator

You are given a nested list of integers `nestedList`. Each element is either an integer or a list whose elements may also be integers or other lists. Implement an iterator to flatten it.

Implement the `NestedIterator` class:

*   `NestedIterator(List<NestedInteger> nestedList)` Initializes the iterator with the nested list `nestedList`.
*   `int next()` Returns the next integer in the nested list.
*   `boolean hasNext()` Returns `true` if there are still some integers in the nested list and `false` otherwise.

Your code will be tested with the following pseudocode:

initialize iterator with nestedList
res = \[\]
while iterator.hasNext()
    append iterator.next() to the end of res
return res

If `res` matches the expected flattened list, then your code will be judged as correct.

**Example 1:**

**Input:** nestedList = \[\[1,1\],2,\[1,1\]\]
**Output:** \[1,1,2,1,1\]
**Explanation:** By calling next repeatedly until hasNext returns false, the order of elements returned by next should be: \[1,1,2,1,1\].

**Example 2:**

**Input:** nestedList = \[1,\[4,\[6\]\]\]
**Output:** \[1,4,6\]
**Explanation:** By calling next repeatedly until hasNext returns false, the order of elements returned by next should be: \[1,4,6\].

**Constraints:**

*   `1 <= nestedList.length <= 500`
*   The values of the integers in the nested list is in the range `[-106, 106]`.

## Code Template

### Go
```go
/**
 * // This is the interface that allows for creating nested lists.
 * // You should not implement it, or speculate about its implementation
 * type NestedInteger struct {
 * }
 *
 * // Return true if this NestedInteger holds a single integer, rather than a nested list.
 * func (this NestedInteger) IsInteger() bool {}
 *
 * // Return the single integer that this NestedInteger holds, if it holds a single integer
 * // The result is undefined if this NestedInteger holds a nested list
 * // So before calling this method, you should have a check
 * func (this NestedInteger) GetInteger() int {}
 *
 * // Set this NestedInteger to hold a single integer.
 * func (n *NestedInteger) SetInteger(value int) {}
 *
 * // Set this NestedInteger to hold a nested list and adds a nested integer to it.
 * func (this *NestedInteger) Add(elem NestedInteger) {}
 *
 * // Return the nested list that this NestedInteger holds, if it holds a nested list
 * // The list length is zero if this NestedInteger holds a single integer
 * // You can access NestedInteger's List element directly if you want to modify it
 * func (this NestedInteger) GetList() []*NestedInteger {}
 */

type NestedIterator struct {
    
}

func Constructor(nestedList []*NestedInteger) *NestedIterator {
    
}

func (this *NestedIterator) Next() int {
    
}

func (this *NestedIterator) HasNext() bool {
    
}
```

### TypeScript
```typescript
/**
 * // This is the interface that allows for creating nested lists.
 * // You should not implement it, or speculate about its implementation
 * class NestedInteger {
 *     If value is provided, then it holds a single integer
 *     Otherwise it holds an empty nested list
 *     constructor(value?: number) {
 *         ...
 *     };
 *
 *     Return true if this NestedInteger holds a single integer, rather than a nested list.
 *     isInteger(): boolean {
 *         ...
 *     };
 *
 *     Return the single integer that this NestedInteger holds, if it holds a single integer
 *     Return null if this NestedInteger holds a nested list
 *     getInteger(): number | null {
 *         ...
 *     };
 *
 *     Set this NestedInteger to hold a single integer equal to value.
 *     setInteger(value: number) {
 *         ...
 *     };
 *
 *     Set this NestedInteger to hold a nested list and adds a nested integer elem to it.
 *     add(elem: NestedInteger) {
 *         ...
 *     };
 *
 *     Return the nested list that this NestedInteger holds,
 *     or an empty list if this NestedInteger holds a single integer
 *     getList(): NestedInteger[] {
 *         ...
 *     };
 * };
 */

class NestedIterator {
    constructor(nestedList: NestedInteger[]) {
		
    }

    hasNext(): boolean {
		
    }

	next(): number {
		
    }
}

/**
 * Your ParkingSystem object will be instantiated and called as such:
 * var obj = new NestedIterator(nestedList)
 * var a: number[] = []
 * while (obj.hasNext()) a.push(obj.next());
 */
```
