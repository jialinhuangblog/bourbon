---
id: 707
title: "Design Linked List"
slug: design-linked-list
difficulty: Medium
tags: [Linked List, Design]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Design Linked List

Design your implementation of the linked list. You can choose to use a singly or doubly linked list.  
A node in a singly linked list should have two attributes: `val` and `next`. `val` is the value of the current node, and `next` is a pointer/reference to the next node.  
If you want to use the doubly linked list, you will need one more attribute `prev` to indicate the previous node in the linked list. Assume all nodes in the linked list are **0-indexed**.

Implement the `MyLinkedList` class:

*   `MyLinkedList()` Initializes the `MyLinkedList` object.
*   `int get(int index)` Get the value of the `indexth` node in the linked list. If the index is invalid, return `-1`.
*   `void addAtHead(int val)` Add a node of value `val` before the first element of the linked list. After the insertion, the new node will be the first node of the linked list.
*   `void addAtTail(int val)` Append a node of value `val` as the last element of the linked list.
*   `void addAtIndex(int index, int val)` Add a node of value `val` before the `indexth` node in the linked list. If `index` equals the length of the linked list, the node will be appended to the end of the linked list. If `index` is greater than the length, the node **will not be inserted**.
*   `void deleteAtIndex(int index)` Delete the `indexth` node in the linked list, if the index is valid.

**Example 1:**

**Input**
\["MyLinkedList", "addAtHead", "addAtTail", "addAtIndex", "get", "deleteAtIndex", "get"\]
\[\[\], \[1\], \[3\], \[1, 2\], \[1\], \[1\], \[1\]\]
**Output**
\[null, null, null, null, 2, null, 3\]

**Explanation**
MyLinkedList myLinkedList = new MyLinkedList();
myLinkedList.addAtHead(1);
myLinkedList.addAtTail(3);
myLinkedList.addAtIndex(1, 2);    // linked list becomes 1->2->3
myLinkedList.get(1);              // return 2
myLinkedList.deleteAtIndex(1);    // now the linked list is 1->3
myLinkedList.get(1);              // return 3

**Constraints:**

*   `0 <= index, val <= 1000`
*   Please do not use the built-in LinkedList library.
*   At most `2000` calls will be made to `get`, `addAtHead`, `addAtTail`, `addAtIndex` and `deleteAtIndex`.

## Code Template

### Go
```go
type MyLinkedList struct {
    
}


func Constructor() MyLinkedList {
    
}


func (this *MyLinkedList) Get(index int) int {
    
}


func (this *MyLinkedList) AddAtHead(val int)  {
    
}


func (this *MyLinkedList) AddAtTail(val int)  {
    
}


func (this *MyLinkedList) AddAtIndex(index int, val int)  {
    
}


func (this *MyLinkedList) DeleteAtIndex(index int)  {
    
}


/**
 * Your MyLinkedList object will be instantiated and called as such:
 * obj := Constructor();
 * param_1 := obj.Get(index);
 * obj.AddAtHead(val);
 * obj.AddAtTail(val);
 * obj.AddAtIndex(index,val);
 * obj.DeleteAtIndex(index);
 */
```

### TypeScript
```typescript
class MyLinkedList {
    constructor() {
        
    }

    get(index: number): number {
        
    }

    addAtHead(val: number): void {
        
    }

    addAtTail(val: number): void {
        
    }

    addAtIndex(index: number, val: number): void {
        
    }

    deleteAtIndex(index: number): void {
        
    }
}

/**
 * Your MyLinkedList object will be instantiated and called as such:
 * var obj = new MyLinkedList()
 * var param_1 = obj.get(index)
 * obj.addAtHead(val)
 * obj.addAtTail(val)
 * obj.addAtIndex(index,val)
 * obj.deleteAtIndex(index)
 */
```
