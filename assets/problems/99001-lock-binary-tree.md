---
id: 99001
title: "Lock Binary Tree"
slug: lock-binary-tree
difficulty: Medium
tags: [Tree, Binary Tree, Design]
neetcode150_category: null
blind75_category: null
source: "Google Interview / Daily Coding Problem #24 / EPI Ch.10"
date_solved: 2026-05-05
languages: [typescript, golang]
time_complexity: O(h)
space_complexity: O(n)
insight: "Counter 記錄子樹鎖數，把後代檢查從 O(n) 壓到 O(1)，維護成本分攤到 O(h) 的 parent chain 上"
---

# Lock Binary Tree

> Google interview question, 不在 LeetCode 上。最接近的題目是 [LeetCode 1993 Operations on Tree](https://leetcode.com/problems/operations-on-tree/)（N-ary 變體 + `upgrade` 操作 + user 權限）。

Implement locking in a binary tree. A binary tree node can be locked or unlocked **only if all of its descendants and ancestors are not locked**.

Design a binary tree node class with the following methods:

- `is_locked()` — returns whether the node is locked.
- `lock()` — attempts to lock the node. Returns `false` if it cannot be locked. Otherwise locks it and returns `true`.
- `unlock()` — unlocks the node. Returns `false` if it cannot be unlocked. Otherwise unlocks it and returns `true`.

You may augment the node with a parent pointer or any other property. Single-threaded — no actual mutex needed. Tree structure does not change.

**Each method must run in O(h) time, where h is the height of the tree.**

**Example:**

```
         A
        / \
       B   C
      / \
     D   E
```

| Operation | Returns | Reason |
|---|---|---|
| `A.lock()` | `true` | nothing locked |
| `B.lock()` | `false` | ancestor A is locked |
| `A.unlock()` | `true` | — |
| `B.lock()` | `true` | ancestors clear |
| `D.lock()` | `false` | ancestor B is locked |
| `A.lock()` | `false` | descendant B is locked |

## Code Template

### TypeScript

```typescript
class Node {
  val: number;
  left: Node | null = null;
  right: Node | null = null;
  parent: Node | null = null;
  // 你可以加任何欄位

  constructor(val: number) { this.val = val; }

  is_locked(): boolean {
    // TODO
  }

  lock(): boolean {
    // TODO
  }

  unlock(): boolean {
    // TODO
  }
}
```

### Go

```go
type Node struct {
    Val    int
    Left   *Node
    Right  *Node
    Parent *Node
    // 你可以加任何欄位
}

func (n *Node) IsLocked() bool {
    // TODO
}

func (n *Node) Lock() bool {
    // TODO
}

func (n *Node) Unlock() bool {
    // TODO
}
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
