---
id: 155
title: "Min Stack"
slug: min-stack
difficulty: Medium
tags: [Stack, Design]
neetcode150_category: Stack
blind75_category: null
date_solved: 2026-07-13
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Min Stack

Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

Implement the `MinStack` class:

*   `MinStack()` initializes the stack object.
*   `void push(int val)` pushes the element `val` onto the stack.
*   `void pop()` removes the element on the top of the stack.
*   `int top()` gets the top element of the stack.
*   `int getMin()` retrieves the minimum element in the stack.

You must implement a solution with `O(1)` time complexity for each function.

**Example 1:**

**Input**
\["MinStack","push","push","push","getMin","pop","top","getMin"\]
\[\[\],\[-2\],\[0\],\[-3\],\[\],\[\],\[\],\[\]\]

**Output**
\[null,null,null,null,-3,null,0,-2\]

**Explanation**
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin(); // return -3
minStack.pop();
minStack.top();    // return 0
minStack.getMin(); // return -2

**Constraints:**

*   `-231 <= val <= 231 - 1`
*   Methods `pop`, `top` and `getMin` operations will always be called on **non-empty** stacks.
*   At most `3 * 104` calls will be made to `push`, `pop`, `top`, and `getMin`.

## Code Template

### Go
```go
type MinStack struct {
    
}


func Constructor() MinStack {
    
}


func (this *MinStack) Push(val int)  {
    
}


func (this *MinStack) Pop()  {
    
}


func (this *MinStack) Top() int {
    
}


func (this *MinStack) GetMin() int {
    
}


/**
 * Your MinStack object will be instantiated and called as such:
 * obj := Constructor();
 * obj.Push(val);
 * obj.Pop();
 * param_3 := obj.Top();
 * param_4 := obj.GetMin();
 */
```

### TypeScript
```typescript
class MinStack {
    constructor() {
        
    }

    push(val: number): void {
        
    }

    pop(): void {
        
    }

    top(): number {
        
    }

    getMin(): number {
        
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * var obj = new MinStack()
 * obj.push(val)
 * obj.pop()
 * var param_3 = obj.top()
 * var param_4 = obj.getMin()
 */
```
