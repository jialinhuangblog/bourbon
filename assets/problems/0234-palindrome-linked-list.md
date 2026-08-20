---
id: 234
title: "Palindrome Linked List"
slug: palindrome-linked-list
difficulty: Easy
tags: [Linked List, Two Pointers, Stack, Recursion]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Palindrome Linked List

Given the `head` of a singly linked list, return `true` _if it is a_ _palindrome_ _or_ `false` _otherwise_.

**Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/03/pal1linked-list.jpg)

**Input:** head = \[1,2,2,1\]
**Output:** true

**Example 2:**

![](https://assets.leetcode.com/uploads/2021/03/03/pal2linked-list.jpg)

**Input:** head = \[1,2\]
**Output:** false

**Constraints:**

*   The number of nodes in the list is in the range `[1, 105]`.
*   `0 <= Node.val <= 9`

**Follow up:** Could you do it in `O(n)` time and `O(1)` space?

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
func isPalindrome(head *ListNode) bool {
    
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

function isPalindrome(head: ListNode | null): boolean {
    
};
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
