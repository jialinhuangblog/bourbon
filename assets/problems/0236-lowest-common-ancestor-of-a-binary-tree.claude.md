給一棵 binary tree 和兩個節點 p、q，找它們最低的共同祖先（LCA）。

---

**解題引導**

用這棵樹思考，p=5, q=1：

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

答案是 3。

**Step 1：LCA 的定義是什麼？**

*The lowest node that has both p and q as descendants — and a node counts as its own descendant.*

<span class="spoiler">最低的那個節點，它的子樹裡同時包含 p 和 q。節點本身也算自己的後代，所以如果 p 是 q 的祖先，LCA 就是 p。 / The deepest node whose subtree contains both p and q. A node counts as its own descendant — so if p is an ancestor of q, LCA is p.</span>

**Step 2：站在某個節點，你怎麼知道 LCA 在哪？**

*You need information from both subtrees before you can decide.*

<span class="spoiler">看左子樹有沒有 p 或 q，右子樹有沒有 p 或 q。如果左右各有一個，當前節點就是 LCA。如果都在同一側，LCA 在那一側更深的地方。 / Check if left subtree contains p or q, right subtree contains p or q. If they split across left and right, current node is the LCA. If both are on one side, recurse deeper.</span>

**Step 3：DFS 回傳什麼？**

*Think about what each recursive call should tell its parent.*

<span class="spoiler">找到 p 或 q 就回傳那個節點，找不到回傳 null。如果左右各回傳一個非 null，當前節點就是 LCA，回傳 current。如果只有一側有，回傳那一側。 / Return the node if it's p or q, else null. If both left and right return non-null, current node is the LCA. Otherwise return whichever side is non-null.</span>

**Step 4：base case 是什麼？**

*What stops the recursion?*

<span class="spoiler">node 是 null 回傳 null。node 是 p 或 q 就直接回傳 node（不用繼續往下，因為 LCA 不可能在 p 或 q 的子樹裡）。 / Return null if node is nil. Return node itself if it equals p or q — no need to go deeper.</span>

想完再往下看 code。

---

## 解法：後序 DFS

先用一個深層的例子建立直覺，再看 code。

p=7, q=4，答案是 node 2：

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

DFS 從 3 出發，一路往左走到底再回來：

```
dfs(3)
  left = dfs(5)
    left = dfs(6) → null（沒有 7 或 4）
    right = dfs(2)
      left = dfs(7) → 回傳 7（找到 p）
      right = dfs(4) → 回傳 4（找到 q）
      左右都非 null → 回傳 node 2  ← LCA 在這找到
    left=null, right=2 → 回傳 2（往上傳）
  right = dfs(1)
    left = dfs(0) → null
    right = dfs(8) → null
    → 回傳 null（1 那側沒有目標）
  left=2, right=null → 回傳 2
```

答案 2。node 2 是第一個「兩側各有一個目標」的節點。node 3 只收到左側有東西，繼續往上傳。

---

想像公司 org chart，是一棵樹。要找員工 p 和 q 的最低共同主管。

每個主管問自己管的兩個部門：「你們底下有 p 或 q 嗎？」

- 左部門回「有 p」，右部門回「有 q」→ 這個主管就是答案，他是第一個「兩邊都管到」的人
- 只有一邊說有→ 把那邊的訊息往上傳，答案在更上面

碰到 p 本人：直接回傳「我在這」，不再往下問。如果 q 剛好是 p 的下屬，LCA 就是 p（p 既管自己也管 q）。

code 完全照這個邏輯：左右遞迴 = 問兩邊部門，兩個都非 null = 當前就是 LCA，只有一邊非 null = 往上傳。

```typescript
class TreeNode {
    val: number;
    left: TreeNode | null;
    right: TreeNode | null;
    constructor(val: number) { this.val = val; this.left = null; this.right = null; }
}

function lowestCommonAncestor(root: TreeNode | null, p: TreeNode, q: TreeNode): TreeNode | null {
    if (!root) return null;
    if (root === p || root === q) return root;  // 找到目標，不往下

    const left = lowestCommonAncestor(root.left, p, q);
    const right = lowestCommonAncestor(root.right, p, q);

    if (left && right) return root;   // 左右各有一個，當前就是 LCA
    return left ?? right;
}
```

<details>
<summary>Go 版本</summary>

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil { return nil }
    if root == p || root == q { return root }   // 找到目標，不往下

    left := lowestCommonAncestor(root.Left, p, q)
    right := lowestCommonAncestor(root.Right, p, q)

    if left != nil && right != nil {
        return root     // 左右各有一個，當前節點就是 LCA
    }
    if left != nil { return left }
    return right
}
```

</details>

走一遍，p=5, q=1：

```
dfs(3):
  left = dfs(5):
    root==p → 回傳 5（不往下）
  right = dfs(1):
    root==q → 回傳 1（不往下）
  left=5, right=1，兩側都非 null → 回傳 3
```

答案 3。

再走一遍，p=5, q=4：

```
dfs(3):
  left = dfs(5):
    root==p → 回傳 5（不往下，4 在 5 的子樹裡也不管）
  right = dfs(1):
    left = dfs(0) = null
    right = dfs(8) = null
    → 回傳 null
  left=5, right=null → 回傳 5
```

答案 5。

這裡有個關鍵：dfs(5) 直接回傳 5，不繼續往下找 4。為什麼還對？

因為 `p` 和 `q` 題目保證都存在於樹中。5 是 p，直接回傳。當 3 收到 left=5, right=null，知道兩個目標都在左側（因為 right 沒找到），LCA 就是 left 回傳的 5。這也符合「節點可以是自己的後代」的定義：5 是 4 的祖先，所以 LCA(5, 4) = 5。

- Time: O(n) — 每個節點最多訪問一次
- Space: O(h) — 遞迴深度，h 是樹高

---

**為什麼找到 p 就不往下繼續？**

因為即使 q 在 p 的子樹裡，LCA 也是 p（p 是 q 的祖先）。往下找 q 對答案沒有幫助，反而多走路。

找到 p 就回傳 p，讓上層來決定：如果另一側也找到了什麼，LCA 是它們的分叉點；如果另一側是 null，說明 q 在 p 下面，LCA 就是 p。

**直覺擔心：「萬一 p、q 沒關係怎麼辦」**

樹裡任兩個節點**永遠有共同祖先**，最壞情況也會是 root。所以「沒關係」這個 case 不存在 — 兩條 DFS 路徑一定會在某個祖先匯流。

題目寫 `p != q` 而且 `p、q 都存在於樹中`，這兩個保證疊起來就是：演算法一定能找到一個「左右都非 null」的節點，那就是 LCA。

`return root` 不是宣告答案，是傳「我這側找到了」的信號燈。真正決定 LCA 的是**第一個收到兩盞燈同時亮的祖先**。

---

## 結論

後序 DFS。左右各回傳找到的節點，兩側都非 null 就是分叉點（LCA），否則往有值的那側傳回去。
