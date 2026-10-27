---
id: 2
title: "Add Two Numbers"
slug: add-two-numbers
difficulty: Medium
tags: [Linked List, Math, Recursion]
neetcode150_category: Linked List
blind75_category: null
date_solved: 2026-02-24
languages: [golang, typescript]
time_complexity: null
space_complexity: null
insight: "carry = sum/10; digit = sum%10; advance both lists"
---

# Add Two Numbers

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/02/addtwonumber1.jpg)

**Input:** l1 = \[2,4,3\], l2 = \[5,6,4\]
**Output:** \[7,0,8\]
**Explanation:** 342 + 465 = 807.

**Example 2:**

**Input:** l1 = \[0\], l2 = \[0\]
**Output:** \[0\]

**Example 3:**

**Input:** l1 = \[9,9,9,9,9,9,9\], l2 = \[9,9,9,9\]
**Output:** \[8,9,9,9,0,0,0,1\]

**Constraints:**

*   The number of nodes in each linked list is in the range `[1, 100]`.
*   `0 <= Node.val <= 9`
*   It is guaranteed that the list represents a number that does not have leading zeros.

## Code Template

### Go
```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    
}
```

### TypeScript
```typescript
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     val: number
 *     next: ListNode | null
 *     constructor(val?: number, next?: ListNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.next = (next===undefined ? null : next)
 *     }
 * }
 */

function addTwoNumbers(l1: ListNode | null, l2: ListNode | null): ListNode | null {
    
};
```
