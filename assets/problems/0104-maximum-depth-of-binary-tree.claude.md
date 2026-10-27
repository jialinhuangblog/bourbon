公司的組織圖貼在牆上。最上面是 CEO，往下接主管、組長、組員。有的部門兩層就到底，有的一路拉到六層。

問的是最長的那條線上面站了幾個人。

```
        3          第 1 層
       / \
      9  20        第 2 層
        /  \
       15   7      第 3 層

3 → 9        只有 2 層
3 → 20 → 15  有 3 層
答案 3
```

翻成 code 的講法：給一棵二元樹的 root，回傳最大深度。深度數的是節點個數，不是連線的條數。

---

**解題引導**

拿上面那棵 `[3,9,20,null,null,15,7]` 想。

**Step 1：整棵樹的深度，跟左右兩邊的深度有什麼關係？**

*if i already knew both sides, the answer would be one line.*

<span class="spoiler">左邊 2 層、右邊 1 層的話，整棵就是比較深的那一邊，再加上自己站的這一層。深度等於 max(左, 右) + 1。Take the deeper side and add one for yourself.</span>

**Step 2：左右兩邊的深度誰來算？**

*same question, one size down.*

<span class="spoiler">同一個函式。左子樹自己也是一棵樹，問它一模一樣的問題。這就是遞迴。The subtree is the same problem, just smaller.</span>

**Step 3：什麼時候停下來？**

<span class="spoiler">走到 null 就回傳 0。空樹沒有節點，往上加一層變成 1，剛好是葉節點的深度。An empty tree contributes zero.</span>

**Step 4：要是這棵樹是一條 10000 個節點的直線，遞迴會怎樣？**

*ten thousand frames all waiting for the one below.*

<span class="spoiler">call stack 會疊到 10000 層。Node 預設疊不到那麼深，會丟 RangeError。這種樹要改成 BFS，或自己開一個 stack 來走。Ten thousand nested frames exceed the default stack.</span>

想完再往下看 code。

---

## 解法一：遞迴 DFS

問左邊多深、問右邊多深、取深的那邊加一。三行。

```typescript
function maxDepth(root: TreeNode | null): number {
    if (!root) return 0;                 // 空樹沒有節點
    const left = maxDepth(root.left);    // 左邊有多深
    const right = maxDepth(root.right);  // 右邊有多深
    return Math.max(left, right) + 1;    // 深的那邊，加上自己這一層
}
```

- Time: $O(n)$ — 每個節點進去一次
- Space: $O(h)$ — h 是樹高，call stack 最深就疊這麼多層

**走一遍。** call stack 展開的順序，縮排代表誰呼叫誰：

```
maxDepth(3)
  maxDepth(9)
    maxDepth(null) → 0
    maxDepth(null) → 0
    max(0, 0) + 1  → 1
  maxDepth(20)
    maxDepth(15)
      maxDepth(null) → 0
      maxDepth(null) → 0
      max(0, 0) + 1  → 1
    maxDepth(7) → 1        （跟 15 一樣，展開省略）
    max(1, 1) + 1 → 2
  max(1, 2) + 1 → 3
```

答案 3 是在最後一步才拼出來的。9 那邊回傳的 1 一路等著，直到 20 那邊也回傳了 2，root 才知道要選哪一個。

**呼叫次數。** 節點進去一次，null 也要進去一次才知道要回傳 0。n 個節點的二元樹有 n+1 個 null 連結，所以總共 2n+1 次呼叫。實測 n=10000 的完整二元樹，計數器數到 20,001 次，換算 0.002 秒。

**遞迴疊多深。** 完整二元樹的高度只有 14，疊 14 層當然沒問題。但題目允許一條 10^4 個節點的直線，高度就是 10000。本機 Node v22.22.3 實測，spine 長度 6,936 以內遞迴走得完，再長一個節點就丟 RangeError。同樣一條 100,000 節點的直線在 Go 跑得完，因為 goroutine 的 stack 會自己長大。

所以這一版在 LeetCode 上過得了，是因為測資沒有給到那麼斜的樹，不是因為它沒有這個限制。

<details>
<summary>Go 版本</summary>

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    left := maxDepth(root.Left)   // 左邊有多深
    right := maxDepth(root.Right) // 右邊有多深
    if left > right {
        return left + 1
    }
    return right + 1
}
```

</details>

---

## 解法二：BFS 層序，數層數

既然怕的是遞迴太深，那就不要遞迴。一層一層往下展開，展開幾次就是幾層。

```typescript
function maxDepth(root: TreeNode | null): number {
    if (!root) return 0;

    let level: TreeNode[] = [root];   // 目前這一層的所有節點
    let depth = 0;

    while (level.length > 0) {
        const next: TreeNode[] = [];
        for (const node of level) {   // 把這一層的小孩全部收集起來
            if (node.left) next.push(node.left);
            if (node.right) next.push(node.right);
        }
        level = next;                 // 換到下一層
        depth++;                      // 剛剛那一層數過了
    }

    return depth;
}
```

- Time: $O(n)$ — 每個節點被 push 一次
- Space: $O(w)$ — w 是最寬那一層的節點數

**走一遍。** 同一棵 `[3,9,20,null,null,15,7]`：

| 回合 | level | 收集到的 next | depth |
|---|---|---|---|
| 1 | `[3]` | `[9, 20]` | 1 |
| 2 | `[9, 20]` | `[15, 7]` | 2 |
| 3 | `[15, 7]` | `[]` | 3 |
| 4 | `[]` | 迴圈條件不成立 | 3 |

`depth++` 放在展開之後，所以第 3 回合處理 `[15, 7]` 的時候 depth 才變成 3。收集到空陣列不會多數一次，因為第 4 回合根本進不去迴圈。

一層一層展開的寫法跟 [102 Binary Tree Level Order Traversal](/problem/binary-tree-level-order-traversal) 是同一個，只是那題要把每一層的值收起來，這題只要數 while 跑了幾圈。

<details>
<summary>Go 版本</summary>

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }

    level := []*TreeNode{root}
    depth := 0

    for len(level) > 0 {
        next := []*TreeNode{}
        for _, node := range level { // 把這一層的小孩全部收集起來
            if node.Left != nil {
                next = append(next, node.Left)
            }
            if node.Right != nil {
                next = append(next, node.Right)
            }
        }
        level = next
        depth++
    }

    return depth
}
```

</details>

---

**Overthinking**

**深度跟高度差一個常數。** 這題問的是節點數，單節點的樹 `[1]` 答案是 1。[543 Diameter](/problem/diameter-of-binary-tree) 問的是路徑上有幾條線，同一棵單節點樹在那邊算 0。兩題的遞迴長得幾乎一樣，只有回傳值的起點不同，寫之前要先看清楚題目數的是節點還是連線。

**這幾題共用同一個骨架。** 遞迴往下走到底，然後帶一個值回來，差別只在帶什麼、以及答案從哪裡拿：

| 題目 | 遞迴回傳什麼 | 答案在哪裡 |
|---|---|---|
| 104 Maximum Depth | 這棵子樹的深度 | root 的回傳值 |
| [100 Same Tree](/problem/same-tree) | 兩棵子樹是不是一樣 | root 的回傳值 |
| [110 Balanced](/problem/balanced-binary-tree) | 高度，不平衡就改回傳 -1 | 回傳值是不是 -1 |
| [543 Diameter](/problem/diameter-of-binary-tree) | 這棵子樹的高度 | 回傳的路上更新外面那個變數 |
| [572 Subtree](/problem/subtree-of-another-tree) | 借 100 當子程序 | 任何一個節點成立就成立 |

前兩題答案就是回傳值，最直接。110 把「壞掉了」塞進回傳值當特殊記號。543 的答案根本不是回傳值，要另外拿一個變數在回程收集。認出自己在寫哪一種，比記住五份 code 有用。

---

## 解法比較表

| 解法 | Time | Space | n=10^4 操作次數 | 備註 |
|---|---|---|---|---|
| 遞迴 DFS | $O(n)$ | $O(h)$ | 20,001 次呼叫 | 四行寫完，樹太斜會 RangeError |
| BFS 層序 | $O(n)$ | $O(w)$ | 10,000 次 push | 深度不設限，某一層太寬就吃記憶體 |

兩個都在 0.002 秒以內，秒數分不出勝負。差別在壞掉的條件不同：遞迴撞的是樹高，BFS 撞的是單層寬度。完整二元樹的最後一層有 n/2 個節點，BFS 的 queue 就得裝 5000 個。

---

## 結論

深度等於 max(左, 右) + 1，走到 null 回傳 0。四行的遞迴是標準答案，被問到「樹很斜怎麼辦」再把 BFS 那版拿出來。
