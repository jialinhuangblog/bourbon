---
id: 23
title: "Merge k Sorted Lists"
slug: merge-k-sorted-lists
difficulty: Hard
tags: [Linked List, Divide and Conquer, Heap (Priority Queue), Merge Sort]
neetcode150_category: Linked List
blind75_category: Linked List
date_solved: 2026-02-27
date_updated: 2026-05-03
languages: [golang, typescript]
time_complexity: null
space_complexity: null
"@local": [Heap]
insight: "min-heap of k heads; pop min, push its next"
---

# Merge k Sorted Lists

You are given an array of `k` linked-lists `lists`, each linked-list is sorted in ascending order.

_Merge all the linked-lists into one sorted linked-list and return it._

**Example 1:**

**Input:** lists = \[\[1,4,5\],\[1,3,4\],\[2,6\]\]
**Output:** \[1,1,2,3,4,4,5,6\]
**Explanation:** The linked-lists are:
\[
  1->4->5,
  1->3->4,
  2->6
\]
merging them into one sorted linked list:
1->1->2->3->4->4->5->6

**Example 2:**

**Input:** lists = \[\]
**Output:** \[\]

**Example 3:**

**Input:** lists = \[\[\]\]
**Output:** \[\]

**Constraints:**

*   `k == lists.length`
*   `0 <= k <= 104`
*   `0 <= lists[i].length <= 500`
*   `-104 <= lists[i][j] <= 104`
*   `lists[i]` is sorted in **ascending order**.
*   The sum of `lists[i].length` will not exceed `104`.

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
func mergeKLists(lists []*ListNode) *ListNode {
    
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

function mergeKLists(lists: Array<ListNode | null>): ListNode | null {
    
};
```
