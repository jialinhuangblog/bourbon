---
id: 1993
title: "Operations on Tree"
slug: operations-on-tree
difficulty: Medium
tags: [Tree, Hash Table, Depth-First Search, Breadth-First Search, Design]
neetcode150_category: null
blind75_category: null
date_solved: 2026-05-05
languages: [typescript, golang]
time_complexity: O(n)
space_complexity: O(n)
insight: "lock/unlock O(1) 看 lockedBy[i]；upgrade 要走祖先 + DFS 後代解鎖，n≤2000 撐得住 O(n)"
---

# Operations on the Tree

You are given a tree with `n` nodes numbered from `0` to `n - 1` in the form of a parent array `parent` where `parent[i]` is the parent of the i-th node. The root of the tree is node `0`, so `parent[0] = -1` since it has no parent. You want to design a data structure that allows users to lock, unlock, and **upgrade** nodes in the tree.

The data structure should support the following functions:

* **Lock:** Locks the given node for the given user and prevents other users from locking the same node. You may only lock a node using this function if the node is unlocked.
* **Unlock:** Unlocks the given node for the given user. You may only unlock a node using this function if it is currently locked by the same user.
* **Upgrade:** Locks the given node for the given user and unlocks all of its descendants regardless of who locked it. You may only upgrade a node if **all** 3 conditions are true:
  * The node is unlocked,
  * It has at least one locked descendant (by any user), and
  * It does not have any locked ancestors.

Implement the `LockingTree` class:

* `LockingTree(int[] parent)` initializes the data structure with the parent array.
* `lock(int num, int user)` returns `true` if it is possible for the user with id `user` to lock the node `num`, or `false` otherwise. If it is possible, the node `num` will become locked by the user with id `user`.
* `unlock(int num, int user)` returns `true` if it is possible for the user with id `user` to unlock the node `num`, or `false` otherwise. If it is possible, the node `num` will become unlocked.
* `upgrade(int num, int user)` returns `true` if it is possible for the user with id `user` to upgrade the node `num`, or `false` otherwise. If it is possible, the node `num` will be upgraded.

**Example:**

```
Input
["LockingTree", "lock", "unlock", "unlock", "lock", "upgrade", "lock"]
[[[-1, 0, 0, 1, 1, 2, 2]], [2, 2], [2, 3], [2, 2], [4, 5], [0, 1], [0, 1]]
Output
[null, true, false, true, true, true, false]
```

樹結構：
```
         0
        / \
       1   2
      / \ / \
     3  4 5  6
```

| call | result | 為什麼 |
|---|---|---|
| `lock(2, 2)` | `true` | 節點 2 沒鎖，user=2 鎖成功 |
| `unlock(2, 3)` | `false` | 節點 2 是 user=2 鎖的，user=3 不能解 |
| `unlock(2, 2)` | `true` | user=2 自己解鎖 |
| `lock(4, 5)` | `true` | 節點 4 沒鎖，user=5 鎖成功 |
| `upgrade(0, 1)` | `true` | 節點 0 沒鎖、無鎖住祖先（它是 root）、有鎖住後代（4 被 5 鎖了）→ 升級成功，4 被解鎖、0 被 user=1 鎖 |
| `lock(0, 1)` | `false` | 節點 0 已被 user=1 鎖（剛才 upgrade 帶鎖） |

**Constraints:**

* `n == parent.length`
* `2 <= n <= 2000`
* `0 <= parent[i] <= n - 1` for `i != 0`
* `parent[0] == -1`
* `0 <= num <= n - 1`
* `1 <= user <= 10^4`
* `parent` represents a valid tree.
* At most `2000` calls **in total** will be made to `lock`, `unlock`, and `upgrade`.

## Code Template

### TypeScript

```typescript
class LockingTree {
    constructor(parent: number[]) {
        
    }

    lock(num: number, user: number): boolean {
        
    }

    unlock(num: number, user: number): boolean {
        
    }

    upgrade(num: number, user: number): boolean {
        
    }
}
```

### Go

```go
type LockingTree struct {
    
}

func Constructor(parent []int) LockingTree {
    
}

func (this *LockingTree) Lock(num int, user int) bool {
    
}

func (this *LockingTree) Unlock(num int, user int) bool {
    
}

func (this *LockingTree) Upgrade(num int, user int) bool {
    
}
```

## My Solution
<!-- 自己寫解題筆記的地方 -->

## Top Discussions
<!-- fetch-discussions.ts 填入 -->
