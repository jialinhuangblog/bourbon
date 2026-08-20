公司組織圖，每個位子可以被某個員工「占用」。三個動作：占一個空位子（別人就不能占）、放掉自己占的位子、還有一個霸道的「收編」——某個主管一次占住自己這格、把底下整條線所有人全部踢掉，不管那些位子原本是誰占的。

```
         0
        / \
       1   2
      / \ / \
     3  4 5  6
```

翻回 code 術語：一棵樹（只給 parent 陣列），實作 `lock` / `unlock` / `upgrade` 三個操作。upgrade 是重點，它要同時滿足三個條件才能動。

---

**解題引導**

用上面那棵樹想。

**Step 1：lock 跟 unlock 需要走整棵樹嗎？**

*a lock is just a stamp on one node*

<span class="spoiler">不用。每個節點記一個「被誰鎖」的值，lock 看它是不是空的、unlock 看是不是你鎖的，都是 O(1)。The lock state is one value per node, both ops are O(1).</span>

**Step 2：題目只給 parent 陣列，但 upgrade 要處理「後代」。缺什麼？**

*parent points up, upgrade needs to go down*

<span class="spoiler">缺 children。parent 只能往上走，upgrade 要往下掃後代，所以 constructor 先從 parent 反建一份 children 鄰接表。Build a children adjacency list from the parent array up front.</span>

**Step 3：upgrade 的三個條件，各自要看樹的哪個方向？**

*self, up, down*

<span class="spoiler">自己沒鎖（看一格）、沒有被鎖的祖先（往上走到 root）、至少一個被鎖的後代（往下 DFS 整棵子樹）。三個方向：本身、往上、往下。Self O(1), ancestors walk up O(h), descendants DFS the subtree.</span>

**Step 4：確認有被鎖的後代之後，還要做什麼？**

*upgrade also unlocks everyone below*

<span class="spoiler">把所有後代解鎖。所以那趟 DFS 一邊檢查有沒有鎖、一邊把被鎖的收集起來，條件都過就全部清掉。The same DFS that checks also collects the locked descendants to clear them.</span>

想完再往下看 code。

---

## 解法：lockedBy + parent + children 模擬

一個陣列 `lockedBy` 記每個節點被哪個 user 鎖住，`0` 代表沒鎖（題目保證 user ≥ 1，所以 0 當空值是安全的）。constructor 順便從 parent 反建 children，因為 upgrade 要往下走。

lock 跟 unlock 只碰一格，直接 O(1)。upgrade 是這題的肉：先確認自己沒鎖，往上走看有沒有被鎖的祖先，往下 DFS 看有沒有被鎖的後代、順便記下來準備解鎖。三條件都過才動手。

```typescript
class LockingTree {
    private lockedBy: number[];        // lockedBy[i] = 鎖住 i 的 user，0 代表沒鎖
    private parent: number[];
    private children: number[][];

    constructor(parent: number[]) {
        this.parent = parent;
        this.lockedBy = new Array(parent.length).fill(0);
        this.children = Array.from({ length: parent.length }, () => []);
        for (let i = 1; i < parent.length; i++) {
            this.children[parent[i]].push(i);   // 從 parent 反建 children
        }
    }

    lock(num: number, user: number): boolean {
        if (this.lockedBy[num] !== 0) return false;   // 已經被鎖
        this.lockedBy[num] = user;
        return true;
    }

    unlock(num: number, user: number): boolean {
        if (this.lockedBy[num] !== user) return false; // 不是你鎖的，不能解
        this.lockedBy[num] = 0;
        return true;
    }

    upgrade(num: number, user: number): boolean {
        if (this.lockedBy[num] !== 0) return false;    // 條件一：自己得是沒鎖的

        // 條件三：往上走到 root，任何祖先被鎖就失敗
        for (let a = this.parent[num]; a !== -1; a = this.parent[a]) {
            if (this.lockedBy[a] !== 0) return false;
        }

        // 條件二：DFS 整棵子樹，看有沒有被鎖的後代，順便收集起來
        let hasLockedDescendant = false;
        const toUnlock: number[] = [];
        const stack = [...this.children[num]];
        while (stack.length) {
            const node = stack.pop()!;
            if (this.lockedBy[node] !== 0) {
                hasLockedDescendant = true;
                toUnlock.push(node);
            }
            for (const c of this.children[node]) stack.push(c);
        }
        if (!hasLockedDescendant) return false;        // 沒有被鎖的後代，不能升級

        // 三條件都過：鎖住自己、解鎖所有後代
        this.lockedBy[num] = user;
        for (const node of toUnlock) this.lockedBy[node] = 0;
        return true;
    }
}
```

- Time: lock / unlock O(1)，upgrade O(n)（往上 O(h) + 往下掃子樹）
- Space: O(n)（lockedBy + children）

<details>
<summary>Go 版本</summary>

```go
type LockingTree struct {
    lockedBy []int      // lockedBy[i] = 鎖住 i 的 user，0 代表沒鎖
    parent   []int
    children [][]int
}

func Constructor(parent []int) LockingTree {
    n := len(parent)
    children := make([][]int, n)
    for i := 1; i < n; i++ {
        children[parent[i]] = append(children[parent[i]], i)  // 從 parent 反建 children
    }
    return LockingTree{
        lockedBy: make([]int, n),
        parent:   parent,
        children: children,
    }
}

func (t *LockingTree) Lock(num int, user int) bool {
    if t.lockedBy[num] != 0 {
        return false
    }
    t.lockedBy[num] = user
    return true
}

func (t *LockingTree) Unlock(num int, user int) bool {
    if t.lockedBy[num] != user {   // 不是你鎖的，不能解
        return false
    }
    t.lockedBy[num] = 0
    return true
}

func (t *LockingTree) Upgrade(num int, user int) bool {
    if t.lockedBy[num] != 0 {      // 條件一：自己得沒鎖
        return false
    }
    for a := t.parent[num]; a != -1; a = t.parent[a] {  // 條件三：祖先有鎖就失敗
        if t.lockedBy[a] != 0 {
            return false
        }
    }
    hasLocked := false
    toUnlock := []int{}
    stack := append([]int{}, t.children[num]...)
    for len(stack) > 0 {
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        if t.lockedBy[node] != 0 {   // 條件二：收集被鎖的後代
            hasLocked = true
            toUnlock = append(toUnlock, node)
        }
        stack = append(stack, t.children[node]...)
    }
    if !hasLocked {
        return false
    }
    t.lockedBy[num] = user           // 鎖住自己
    for _, node := range toUnlock {  // 解鎖所有後代
        t.lockedBy[node] = 0
    }
    return true
}
```

</details>

**走一遍題目的呼叫序列**

parent = `[-1,0,0,1,1,2,2]`，樹長這樣：

```
         0
        / \
       1   2
      / \ / \
     3  4 5  6
```

```
lock(2, 2)    → 2 沒鎖 → lockedBy[2]=2 → true
unlock(2, 3)  → 2 是 user 2 鎖的，user 3 想解 → false
unlock(2, 2)  → 是 user 2 自己 → lockedBy[2]=0 → true
lock(4, 5)    → 4 沒鎖 → lockedBy[4]=5 → true

upgrade(0, 1) → 0 沒鎖 ✓
              → 往上：0 是 root，沒有祖先 ✓
              → 往下 DFS：走到 4，lockedBy[4]=5 → 有被鎖的後代 ✓
              → 三條件都過 → lockedBy[0]=1、把 4 解鎖 → true

lock(0, 1)    → 0 已經被 user 1 鎖住（剛才 upgrade 帶鎖）→ false
```

跟題目給的 `[null, true, false, true, true, true, false]` 一致。

**為什麼一趟 DFS 就夠？**

upgrade 的「有沒有被鎖的後代」跟「要解鎖哪些後代」問的是同一批節點。分兩趟掃是浪費：第一趟找到就設 `hasLockedDescendant`，同一趟把被鎖的丟進 `toUnlock`，確認條件成立後直接清掉。祖先檢查放在 DFS 之前，是因為它更便宜（O(h)）又可能提早 return，不必先掃完整棵子樹才發現祖先被鎖。

**跟 [99001 Lock Binary Tree](/problem/lock-binary-tree) 的關聯**

兩題是同一個機制的兩種包裝。99001 是 Google 面試的「設計」版，自己定義 Node 類別、可以在節點上直接掛 `parent` 欄位；1993 是 LeetCode 版，只給你一個 parent 陣列，要自己反建 children。核心動作完全一樣：lock/unlock 看一格 O(1)，牽涉祖先或後代的操作就往上走 O(h)、往下 DFS O(子樹)。

---

## 結論

lock/unlock 是一格的事，O(1)。upgrade 把樹的三個方向各查一次：自己、祖先、後代，一趟 DFS 同時完成「檢查後代」跟「解鎖後代」。n ≤ 2000，O(n) 的 upgrade 綽綽有餘。
