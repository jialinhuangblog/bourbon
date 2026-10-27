---
title: "Binary Tree"
category: Data Structures
slug: binary-tree
subtitle: 結構和走法是兩回事
date: 2026-02-28T11:51:31
---

# Binary Tree

Tree 這一族看起來很多：BFS、DFS、Heap、BST、B-Tree、B+Tree。分兩層看就不亂了：

**結構（東西長什麼樣）和走法（怎麼遍歷它）是兩回事。**

```
結構（資料結構）           走法（演算法）
├── Binary Tree            ├── BFS（一層一層走）
├── BST                    └── DFS（一條路走到底）
├── Heap                        ├── 前序
├── B-Tree                      ├── 中序
├── B+Tree                      └── 後序
└── Trie
```

走法可以套在任何結構上。BFS 可以走 Binary Tree，也可以走 Graph。DFS 可以走 BST，也可以走 Heap。結構決定資料怎麼擺，走法決定怎麼看它。


---

# 第一部分：結構

## Binary Tree 是什麼

每個節點最多兩個小孩（left 和 right），就是 binary tree。

```
        1
       / \
      2   3
     / \
    4   5
```

最上面的叫 root。沒有小孩的叫 leaf（葉子）。

程式怎麼表示？三個欄位，一個值兩個指標：

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}
```

```typescript
class TreeNode {
    constructor(
        public val: number,
        public left: TreeNode | null = null,
        public right: TreeNode | null = null
    ) {}
}
```

LeetCode 上大部分 tree 題的 input 就是一個 `TreeNode`，代表 root。從 root 開始，用 `.left` 和 `.right` 走遍整棵樹。

---

## 從陣列到樹

LeetCode 用陣列表示樹。`[1, 2, 3, 4, 5]` 代表：

```
index:    0  1  2  3  4
value:   [1, 2, 3, 4, 5]

        1          ← index 0
       / \
      2   3        ← index 1, 2
     / \
    4   5          ← index 3, 4
```

規則：
- root 在 index 0
- index `i` 的左 child 在 `2i + 1`
- index `i` 的右 child 在 `2i + 2`
- index `i` 的 parent 在 `(i - 1) / 2`

`null` 代表那個位置沒有節點。`[1, 2, 3, null, null, 4]`：

```
        1
       / \
      2   3
         /
        4
```

---

## Binary Tree 家族

Binary tree 是最寬泛的定義，其他都是加了規則的特化版。

```
Binary Tree（每個節點最多兩個小孩，沒有其他規則）
│
├── BST（加了「左小右大」）
│   └── Balanced BST（加了「左右高度差 ≤ 1」）
│       ├── AVL Tree — 嚴格平衡，查找最快
│       └── Red-Black Tree — Java TreeMap、C++ std::map 底層
│
├── Heap（加了「父 ≥ 子」或「父 ≤ 子」，左右無所謂）
│   └── 用 Complete Binary Tree 的結構存在陣列裡
│
└── Trie（嚴格來說不是二元，但常歸在 tree 類考）
```

同一組數字 `[8, 3, 10, 1, 6, 14]`，放進不同結構裡：

```
Binary Tree（隨便放）      BST（左小右大）         Min Heap（父 ≤ 子）

        8                      8                      1
       / \                    / \                    / \
      10   3                 3   10                 3   6
     /    / \               / \    \               / \  /
    6    1   14            1   6   14              8 10 14
```

| 名稱 | 規則 | 管什麼 | 保證什麼 | 面試頻率 |
|---|---|---|---|---|
| Binary Tree | 無 | 什麼都不管 | 什麼都不保證 | 高（traversal） |
| BST | 左 < 父 < 右 | 左右 | 中序遍歷有序 | 高 |
| Heap | 父 ≥ 子 或 父 ≤ 子 | 上下 | 頂端是最大/最小 | 高 |
| Balanced BST | BST + 高度平衡 | 左右 + 高度 | 搜尋 O(log n) 不退化 | 知道概念就好 |
| Trie | 每個節點是一個字元 | 前綴 | 前綴搜尋 O(m) | 中 |

**BST 管左右，Heap 只管上下。** 這是最容易搞混的地方。

---

## 不是 Binary Tree 但常搞混的：B-Tree 和 B+Tree

B-Tree 的 B 不是 Binary。一個節點可以有很多 key、很多小孩。用在資料庫索引。

```
Binary Tree（1 個值，最多 2 個小孩）

       8
      / \
     3   10


B-Tree（多個值，多個小孩）

      [3 | 8 | 10]
     /   |   |    \
   [1]  [5] [9]  [14]


B+Tree（跟 B-Tree 很像，但資料只在葉子，葉子串成 linked list）

      [3 | 8]                  ← 內部節點只存索引
     /   |    \
   [1,2] → [3,5] → [8,9,14]   ← 葉子存資料，串在一起
```

B+Tree 的葉子串成 linked list，所以 range query（找 3 到 10 之間的值）特別快：找到 3，順著鏈結往右走到 10 就好。MySQL InnoDB 的索引就是 B+Tree。

面試不會叫你手寫 B-Tree / B+Tree，但 system design 可能問「資料庫索引怎麼運作」，知道 B+Tree 葉子串在一起所以 range query 快，這樣就夠了。

---

## BST (Binary Search Tree)

普通的 binary tree 沒有順序。BST 有一條規則：**左 child < 當前 < 右 child。**

```
        8
       / \
      3   10
     / \    \
    1   6   14
```

3 < 8。10 > 8。1 < 3。6 > 3 但 6 < 8。每個節點都滿足這個規則。

### BST 的超能力

中序遍歷 BST，結果是排好序的：`1 → 3 → 6 → 8 → 10 → 14`。

搜尋一個值？從 root 開始，比它小往左，比它大往右。每一步砍掉一半。$O(\log n)$。

前提是樹夠平衡。如果插入順序剛好是排好序的（1, 3, 6, 8...），樹會歪成一條線，搜尋退化成 $O(n)$。這就是 Balanced BST（AVL、Red-Black Tree）存在的原因：強制維持平衡，保住 $O(\log n)$。

```go
func searchBST(root *TreeNode, val int) *TreeNode {
    if root == nil || root.Val == val { return root }
    if val < root.Val {
        return searchBST(root.Left, val)
    }
    return searchBST(root.Right, val)
}
```

### BST 常考的題

- [#98 Validate BST](/problem/validate-binary-search-tree) — 中序遍歷是否嚴格遞增
- [#230 Kth Smallest Element in a BST](/problem/kth-smallest-element-in-a-bst) — 中序走到第 k 個
- [#235 Lowest Common Ancestor of a BST](/problem/lowest-common-ancestor-of-a-binary-search-tree) — 利用大小關係找分叉點
- [#108 Convert Sorted Array to BST](/problem/convert-sorted-array-to-binary-search-tree) — 取中間當 root，遞迴左右

Heap 的詳細用法見 [Heap](/concept/heap)。Trie 見 [Trie](/concept/trie)。

---

# 第二部分：走法

走法跟結構是獨立的。BFS 和 DFS 可以走任何一種 tree（甚至 graph）。

## DFS：一條路走到底

DFS（深度優先）先往深處走到底，再回頭。有三種變體，差別只在**什麼時候處理當前節點**。

```
        1
       / \
      2   3
     / \
    4   5
```

### 前序（Preorder）：先自己，再左，再右

```
1 → 2 → 4 → 5 → 3
```

```go
func preorder(node *TreeNode) {
    if node == nil { return }
    visit(node)            // 先處理自己
    preorder(node.Left)
    preorder(node.Right)
}
```

用途：複製一棵樹、序列化。先記住根，再記住結構。

### 中序（Inorder）：先左，再自己，再右

```
4 → 2 → 5 → 1 → 3
```

```go
func inorder(node *TreeNode) {
    if node == nil { return }
    inorder(node.Left)
    visit(node)            // 中間處理自己
    inorder(node.Right)
}
```

用途：BST 的中序遍歷結果是排好序的。

### 後序（Postorder）：先左，先右，最後自己

```
4 → 5 → 2 → 3 → 1
```

```go
func postorder(node *TreeNode) {
    if node == nil { return }
    postorder(node.Left)
    postorder(node.Right)
    visit(node)            // 最後處理自己
}
```

用途：刪除樹（先刪 children 再刪自己）、計算子樹大小。需要「先拿到 children 的結果」的場景都是後序。

### 怎麼記？

名字說的是**自己**的位置：
- **前**序：自己在前面。自 → 左 → 右
- **中**序：自己在中間。左 → 自 → 右
- **後**序：自己在後面。左 → 右 → 自

### 為什麼要分三種？

處理的時機決定你能拿到什麼資訊。

**後序**：先拿到左右小孩的結果，再決定自己。「這棵樹多高？」左邊 2，右邊 1，我是 max(2,1)+1 = 3。你需要小孩的答案才能算自己。

**前序**：先處理自己，再往下傳。「複製一棵樹？」先複製自己，再複製左右。你需要先建好自己，小孩才有地方掛。

**中序**：左邊先，自己中間，右邊後。放在 BST 上剛好是從小到大排好的順序。

共通規則：**誰依賴誰，誰就後處理。** 需要小孩的結果？後序。需要父層的資訊？前序。需要有序輸出？中序。

### 真實場景

**後序（先小孩，後自己）** — 算資料夾大小，不知道 `/src` 多大，除非先知道 `/src/app` 和 `/src/assets` 多大。遞迴到最底層的檔案，拿到大小，一路加回來。

**前序（先自己，再小孩）** — 組織架構表，先印 CEO，再印 VP，再印 Manager。上層先出現，下層才有 context。

**中序（左、自己、右）** — BST 專屬。中序走 BST 就是排序結果。資料庫的 B-Tree index range scan 就是中序遍歷。

---

## BFS：一層一層走

BFS（廣度優先）先走完同一層的所有節點，再去下一層。

```
        1          ← 第 0 層
       / \
      2   3        ← 第 1 層
     / \
    4   5          ← 第 2 層

BFS 順序：1 → 2 → 3 → 4 → 5
```

用 queue 實作。為什麼是 queue？因為 queue 是先進先出。同一層的節點先放進去，就會先被處理完，然後才輪到下一層。

走一遍 `[1, 2, 3, 4, 5]`：

```
開始：queue = [1]

第 0 層開始，size = 1（queue 裡有 1 個節點）
  拿出 1，收集值 → level = [1]
  把 1 的小孩放進 queue → queue = [2, 3]
第 0 層結束 → result = [[1]]

第 1 層開始，size = 2（queue 裡有 2 個節點）
  拿出 2，收集值 → level = [2]
  把 2 的小孩放進 queue → queue = [3, 4, 5]
  拿出 3，收集值 → level = [2, 3]
  3 沒有小孩
第 1 層結束 → result = [[1], [2, 3]]

第 2 層開始，size = 2（queue 裡有 2 個節點）
  拿出 4，收集值 → level = [4]
  拿出 5，收集值 → level = [4, 5]
  4 和 5 都沒有小孩
第 2 層結束 → result = [[1], [2, 3], [4, 5]]

queue 空了，結束。
```

兩個變數，角色不同：

- **queue** — 全域的待處理清單。等待處理的節點都在這裡，跨層存在，一直活到整棵樹走完。
- **level** — 這一層的收集桶。每層開始時是空的，處理完就整包丟進 result，下一層重新開一個。

`size := len(queue)` 在每層開始前記住 queue 裡有幾個節點，這幾個就是「這一層的」。for 迴圈只跑 size 次，所以中途放進去的小孩（下一層的）不會在這輪被處理。

```go
func levelOrder(root *TreeNode) [][]int {
    if root == nil { return nil }
    result := [][]int{}
    queue := []*TreeNode{root}

    for len(queue) > 0 {
        size := len(queue)              // 鎖住：這層有幾個
        level := []int{}
        for i := 0; i < size; i++ {     // 只處理這層的
            node := queue[0]
            queue = queue[1:]
            level = append(level, node.Val)
            if node.Left != nil { queue = append(queue, node.Left) }   // 下一層
            if node.Right != nil { queue = append(queue, node.Right) } // 下一層
        }
        result = append(result, level)
    }
    return result
}
```

### LeetCode

- [#102 Binary Tree Level Order Traversal](/problem/binary-tree-level-order-traversal) — 最基本的 BFS
- [#103 Binary Tree Zigzag Level Order Traversal](/problem/binary-tree-zigzag-level-order-traversal) — 奇數層反轉
- [#199 Binary Tree Right Side View](/problem/binary-tree-right-side-view) — 每層最後一個

---

## DFS vs BFS：面試哪個重要？

面試出現頻率：DFS 遠高於 BFS。

DFS 能考的東西多：遞迴、回溯、分治、後序合併結果，所以 tree 題大多是 DFS 的變形。

BFS 的 tree 題基本就那幾道：level order、zigzag、right side view。套路固定，變化少。

但 BFS 在 graph 上價值很高：最短路徑、拓撲排序。那些是 graph 的題，詳見 [Graph](/concept/graph)。

回到 binary tree：先把 DFS 的三種順序和遞迴套路練熟，投報率最高。

---

# 第三部分：遞迴套路

Tree 的遞迴有一個萬用結構：

```
function solve(node):
    if node is null: return base_case

    left_result = solve(node.left)
    right_result = solve(node.right)

    return combine(left_result, right_result, node.val)
```

三步：
1. **Base case**：null 回傳什麼？
2. **遞迴**：左右 children 各自回傳什麼？
3. **合併**：怎麼用左右的結果算出這個節點的答案？

### 範例：求樹的高度

```go
func maxDepth(root *TreeNode) int {
    if root == nil { return 0 }                        // base case

    left := maxDepth(root.Left)                        // 左邊多深
    right := maxDepth(root.Right)                      // 右邊多深

    return max(left, right) + 1                        // 取大的 + 自己
}
```

左邊深度 2，右邊深度 1。這個節點的深度 = max(2, 1) + 1 = 3。

### 範例：判斷是否對稱

```go
func isSymmetric(root *TreeNode) bool {
    return isMirror(root.Left, root.Right)
}

func isMirror(a, b *TreeNode) bool {
    if a == nil && b == nil { return true }
    if a == nil || b == nil { return false }
    return a.Val == b.Val &&
        isMirror(a.Left, b.Right) &&
        isMirror(a.Right, b.Left)
}
```

左子樹的左邊 = 右子樹的右邊。左子樹的右邊 = 右子樹的左邊。鏡像。

---

## 常見術語

| 術語 | 意思 |
|------|------|
| root | 最上面的節點 |
| leaf | 沒有 children 的節點 |
| depth | 從 root 到這個節點的邊數 |
| height | 從這個節點到最深 leaf 的邊數 |
| complete | 除了最後一層，每層都填滿。最後一層靠左 |
| full | 每個節點要嘛有 0 個 children，要嘛有 2 個 |
| balanced | 左右子樹高度差不超過 1 |
| BST | 左 < 中 < 右 |

---

## 高頻題清單

| 題目 | 核心技巧 |
|------|---------|
| [#104 Maximum Depth](/problem/maximum-depth-of-binary-tree) | 後序遞迴 |
| [#226 Invert Binary Tree](/problem/invert-binary-tree) | 前序遞迴，左右交換 |
| [#101 Symmetric Tree](/problem/symmetric-tree) | 雙指標遞迴 |
| [#102 Level Order Traversal](/problem/binary-tree-level-order-traversal) | BFS + queue |
| [#236 Lowest Common Ancestor](/problem/lowest-common-ancestor-of-a-binary-tree) | 後序，找到就回傳 |
| [#543 Diameter of Binary Tree](/problem/diameter-of-binary-tree) | 後序，左深+右深的最大值 |
| [#297 Serialize and Deserialize](/problem/serialize-and-deserialize-binary-tree) | 前序 + null 標記 |
| [#105 Construct from Preorder and Inorder](/problem/construct-binary-tree-from-preorder-and-inorder-traversal) | 前序找根，中序分左右 |

---

## 總結

搞清楚兩件事就不亂了：

1. **結構和走法是獨立的。** Binary Tree、BST、Heap 是結構（資料怎麼擺）。BFS、DFS 是走法（怎麼遍歷）。走法可以套在任何結構上。
2. **Binary Tree 家族都是加規則。** BST 加了左小右大。Heap 加了上下大小。B-Tree 加了多 key 多小孩。

面試先掌握：
- DFS 三種順序：前序、中序、後序
- BFS 用 queue 一層一層走
- 遞迴萬用結構：base case → 遞迴左右 → 合併結果


想學怎麼用 DFS 解更複雜的問題？看 [DFS](/concept/dfs)。想學 tree 的搜尋怎麼推廣到圖？也是 [DFS](/concept/dfs)。
