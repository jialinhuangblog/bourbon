---
id: 225
title: "Implement Stack using Queues"
slug: implement-stack-using-queues
difficulty: Easy
tags: [Stack, Design, Queue]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Implement Stack using Queues

Implement a last-in-first-out (LIFO) stack using only two queues. The implemented stack should support all the functions of a normal stack (`push`, `top`, `pop`, and `empty`).

Implement the `MyStack` class:

*   `void push(int x)` Pushes element x to the top of the stack.
*   `int pop()` Removes the element on the top of the stack and returns it.
*   `int top()` Returns the element on the top of the stack.
*   `boolean empty()` Returns `true` if the stack is empty, `false` otherwise.

**Notes:**

*   You must use **only** standard operations of a queue, which means that only `push to back`, `peek/pop from front`, `size` and `is empty` operations are valid.
*   Depending on your language, the queue may not be supported natively. You may simulate a queue using a list or deque (double-ended queue) as long as you use only a queue's standard operations.

**Example 1:**

**Input**
\["MyStack", "push", "push", "top", "pop", "empty"\]
\[\[\], \[1\], \[2\], \[\], \[\], \[\]\]
**Output**
\[null, null, null, 2, 2, false\]

**Explanation**
MyStack myStack = new MyStack();
myStack.push(1);
myStack.push(2);
myStack.top(); // return 2
myStack.pop(); // return 2
myStack.empty(); // return False

**Constraints:**

*   `1 <= x <= 9`
*   At most `100` calls will be made to `push`, `pop`, `top`, and `empty`.
*   All the calls to `pop` and `top` are valid.

**Follow-up:** Can you implement the stack using only one queue?

## Code Template

### Go
```go
type MyStack struct {
    
}


func Constructor() MyStack {
    
}


func (this *MyStack) Push(x int)  {
    
}


func (this *MyStack) Pop() int {
    
}


func (this *MyStack) Top() int {
    
}


func (this *MyStack) Empty() bool {
    
}


/**
 * Your MyStack object will be instantiated and called as such:
 * obj := Constructor();
 * obj.Push(x);
 * param_2 := obj.Pop();
 * param_3 := obj.Top();
 * param_4 := obj.Empty();
 */
```

### TypeScript
```typescript
class MyStack {
    constructor() {
        
    }

    push(x: number): void {
        
    }

    pop(): number {
        
    }

    top(): number {
        
    }

    empty(): boolean {
        
    }
}

/**
 * Your MyStack object will be instantiated and called as such:
 * var obj = new MyStack()
 * obj.push(x)
 * var param_2 = obj.pop()
 * var param_3 = obj.top()
 * var param_4 = obj.empty()
 */
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
