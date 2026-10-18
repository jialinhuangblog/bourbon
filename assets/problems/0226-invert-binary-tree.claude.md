把每個 node 的左右子樹對調。

```
原本：          反轉後：
    4               4
   / \             / \
  2   7           7   2
 / \ / \         / \ / \
1  3 6  9       9  6 3  1
```

<details>
<summary>提示</summary>

對每個 node，把它的 left 和 right 交換。然後遞迴處理左右子樹。

</details>

---

## 解法：DFS

**思路**

這題是 DFS 的直接應用。每個 node 做同一件事：交換左右。

順序不重要——先交換再遞迴，或先遞迴再交換，結果一樣。

**遞迴解**

```typescript
function invertTree(root: TreeNode | null): TreeNode | null {
    if (!root) return null;
    [root.left, root.right] = [root.right, root.left]; // 交換左右
    invertTree(root.left);                              // 遞迴左子樹
    invertTree(root.right);                             // 遞迴右子樹
    return root;
}
```

<details>
<summary>Go 版本</summary>

```go
func invertTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }
    root.Left, root.Right = root.Right, root.Left // 交換左右
    invertTree(root.Left)                          // 遞迴左子樹
    invertTree(root.Right)                         // 遞迴右子樹
    return root
}
```

</details>

**逐步圖解**

以 `[4,2,7,1,3,6,9]` 為例，call stack 展開：

```
invertTree(4)
  交換 4 的左右 → left=7, right=2
  invertTree(7)
    交換 7 的左右 → left=9, right=6
    invertTree(9) → leaf，回傳
    invertTree(6) → leaf，回傳
  invertTree(2)
    交換 2 的左右 → left=3, right=1
    invertTree(3) → leaf，回傳
    invertTree(1) → leaf，回傳
回傳 4
```

結果：
```
    4
   / \
  7   2
 / \ / \
9  6 3  1
```

---

**核心**

```typescript
[root.left, root.right] = [root.right, root.left];
```

<details>
<summary>Go 版本</summary>

```go
root.Left, root.Right = root.Right, root.Left
```

</details>

這一行是全題的靈魂。遞迴只是把它套到每個 node。

---

**複雜度**

- Time: O(n)，每個 node 走一次
- Space: O(h)，h = 樹高，call stack 深度
