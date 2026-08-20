---
title: "DFS"
category: Algorithms
slug: dfs
subtitle: 一條路走到底，再回頭
date: 2026-02-28T11:51:31
---

# DFS — Depth-First Search

你站在一個迷宮入口。面前有三條路。

你選最左邊那條。走到底。死路。退回來。試中間那條。走到底。又死路。退回來。試右邊那條。通了。

這就是 DFS。**一條路走到底，碰壁了再回頭試別條。**

回頭這個動作叫 backtrack。backtracking 就是 DFS 加上剪枝跟撤銷狀態：在 tree 上通常直接叫 DFS，在排列組合上叫 backtracking。兩者在解題分類裡的位置，見[解題直覺](/concept/problem-solving-intuition)。

---

## 兩種實作：遞迴 vs Stack

### 遞迴（最常用）

```typescript
function dfs(node: TreeNode | null) {
    if (node === null) return;
    // 做事
    dfs(node.left);
    dfs(node.right);
}
```

遞迴天生就是 DFS。每次 function call 就是往深處走一步。return 就是退回來一步。Call stack 幫你記住退路。

### Stack（手動模擬）

```typescript
function dfs(root: TreeNode) {
    const stack: TreeNode[] = [root];
    while (stack.length > 0) {
        const node = stack.pop()!;      // 拿最後放進去的那個
        // 做事
        if (node.right) stack.push(node.right);
        if (node.left) stack.push(node.left);
    }
}
```

stack 裡裝的是「還沒處理的節點」，不是樹的某一層。一開始只有 root，所以第一輪 `pop()` 拿到的就是 root。

先 push right 再 push left，因為 stack 是 LIFO，後放的 left 會先被拿出來，走訪順序才會是先左後右。

什麼時候用 stack？樹太深，遞迴會 stack overflow。或是面試官說「不准用遞迴」。

---

## DFS 在 Tree 上

Binary tree 的前序、中序、後序遍歷全部都是 DFS。差別只在「做事」的時機。

```
        1
       / \
      2   3
     / \
    4   5
```

```
前序 DFS：1 → 2 → 4 → 5 → 3    （先做事，再走）
中序 DFS：4 → 2 → 5 → 1 → 3    （先走左，做事，再走右）
後序 DFS：4 → 5 → 2 → 3 → 1    （先走完，最後做事）
```

詳細的三種順序看 [Binary Tree](/concept/binary-tree)。這裡聚焦 DFS 的思考方式。

### 這個節點要回傳什麼？

DFS 在 tree 上的套路：

```typescript
function dfs(node: TreeNode | null) {
    if (node === null) return base;              // 走到底了，回傳什麼

    const left = dfs(node.left);
    const right = dfs(node.right);

    return combine(left, right, node.val);       // 兩邊的答案跟自己怎麼合起來
}
```

`base` 跟 `combine` 是兩個空格，每一題填的不一樣：

| 題目 | base | combine |
|------|------|---------|
| 最大深度 | 0 | max(left, right) + 1 |
| 節點總和 | 0 | left + right + node.val |
| 是否平衡 | true, 0 | abs(left - right) <= 1 |
| 路徑總和 | target === 0 | dfs(left, target-val) or dfs(right, target-val) |

### 走一遍：最大深度

把 `base` 填 0、`combine` 填 `max(left, right) + 1`，就是這份 code：

```typescript
function maxDepth(node: TreeNode | null): number {
    if (node === null) {
        return 0;                              // 空的，深度 0
    }
    const left = maxDepth(node.left);          // 左邊多深？
    const right = maxDepth(node.right);        // 右邊多深？
    return Math.max(left, right) + 1;          // 比較深的那邊，加上自己這一層
}
```

```
        1
       / \
      2   3
     /
    4
```

呼叫是先往下鑽到底，鑽不動了才開始往上回傳。縮排代表誰呼叫誰：

```
maxDepth(1)
├── maxDepth(2)
│   ├── maxDepth(4)
│   │   ├── maxDepth(null) → 0
│   │   └── maxDepth(null) → 0
│   │   max(0, 0) + 1 → return 1
│   └── maxDepth(null) → 0
│   max(1, 0) + 1 → return 2
└── maxDepth(3)
    ├── maxDepth(null) → 0
    └── maxDepth(null) → 0
    max(0, 0) + 1 → return 1
max(2, 1) + 1 → return 3
```

答案 3。

`maxDepth(1)` 停在 `const left = maxDepth(node.left)` 這一行等著，等 `maxDepth(2)` 回傳 2 才繼續。`maxDepth(2)` 又停在同一行等 `maxDepth(4)`。所以真正算出數字的順序是反過來的，從最底下的 `null` 開始，一層層往上。

每個節點只問兩個 children「你多深？」，取大的加 1。它不需要知道自己在整棵樹的哪裡。

---

## DFS 在 Graph 上

Tree 沒有環。往下走不會回到上面。

Graph 有環。如果不記住走過的路，DFS 會無限繞圈。

所以 graph 的 DFS 多一步：**visited set**。

```typescript
function dfs(node: number, graph: Map<number, number[]>, visited: Set<number>) {
    if (visited.has(node)) return;
    visited.add(node);

    for (const neighbor of graph.get(node) ?? []) {
        dfs(neighbor, graph, visited);
    }
}
```

visited 就是 [Hash Map](/concept/hashmap) 的「記住看過的」用法。

### 走一遍：數島嶼

[#200 Number of Islands](/problem/number-of-islands)

```
grid:
1 1 0 0 0
1 1 0 0 0
0 0 1 0 0
0 0 0 1 1
```

掃過每個格子。碰到 1，DFS 把整座島標記成 visited（或改成 0）。每次從新的 1 開始 DFS，島嶼數 +1。

```
碰到 (0,0)=1 → DFS，把連通的 1 全部標記
  (0,0) → (0,1) → (1,0) → (1,1)    全部標記完
  島嶼數 = 1

碰到 (2,2)=1 → DFS，只有自己
  島嶼數 = 2

碰到 (3,3)=1 → DFS
  (3,3) → (3,4)
  島嶼數 = 3

答案：3
```

```typescript
function numIslands(grid: string[][]): number {
    let count = 0;
    for (let i = 0; i < grid.length; i++) {
        for (let j = 0; j < grid[i].length; j++) {
            if (grid[i][j] === '1') {
                dfs(grid, i, j);
                count++;
            }
        }
    }
    return count;
}

function dfs(grid: string[][], i: number, j: number) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length) return;
    if (grid[i][j] !== '1') return;

    grid[i][j] = '0';        // 標記走過（直接改 grid，省掉 visited）
    dfs(grid, i + 1, j);
    dfs(grid, i - 1, j);
    dfs(grid, i, j + 1);
    dfs(grid, i, j - 1);
}
```

四個方向遞迴。碰到邊界或水（0）就停。

---

## DFS 在排列組合上（Backtracking）

DFS 不只能走 tree 和 graph。它能走「所有可能的選擇」。

每一步有幾個選擇？從那裡分叉。走到底就是一個結果。退回來試下一個選擇。

### 走一遍：子集

[#78 Subsets](/problem/subsets) — 給 `[1, 2, 3]`，列出所有子集。

每個元素有兩個選擇：選或不選。

```
[]
├── 選 1 → [1]
│   ├── 選 2 → [1,2]
│   │   ├── 選 3 → [1,2,3]
│   │   └── 不選 3 → [1,2]
│   └── 不選 2 → [1]
│       ├── 選 3 → [1,3]
│       └── 不選 3 → [1]
└── 不選 1 → []
    ├── 選 2 → [2]
    │   ├── 選 3 → [2,3]
    │   └── 不選 3 → [2]
    └── 不選 2 → []
        ├── 選 3 → [3]
        └── 不選 3 → []
```

八個葉子由上而下就是輸出順序：`[1,2,3]`、`[1,2]`、`[1,3]`、`[1]`、`[2,3]`、`[2]`、`[3]`、`[]`。

DFS 走這棵「決策樹」：

```typescript
function subsets(nums: number[]): number[][] {
    const result: number[][] = [];

    function dfs(i: number, current: number[]) {
        if (i === nums.length) {
            result.push([...current]);           // 複製一份存起來
            return;
        }
        dfs(i + 1, [...current, nums[i]]);       // 選 nums[i]
        dfs(i + 1, current);                     // 不選 nums[i]
    }

    dfs(0, []);
    return result;
}
```

### 走一遍：排列

[#46 Permutations](/problem/permutations) — 給 `[1, 2, 3]`，列出所有排列。

第一個位置有 3 個選擇。第二個位置有 2 個（扣掉用過的）。第三個位置有 1 個。

```
[]
├── 1
│   ├── 1,2
│   │   └── 1,2,3
│   └── 1,3
│       └── 1,3,2
├── 2
│   ├── 2,1
│   │   └── 2,1,3
│   └── 2,3
│       └── 2,3,1
└── 3
    ├── 3,1
    │   └── 3,1,2
    └── 3,2
        └── 3,2,1
```

第二層每個節點只剩兩個分支，第三層只剩一個，因為用掉的數字被 `used` 擋住了。六個葉子就是六種排列。

```typescript
function permute(nums: number[]): number[][] {
    const result: number[][] = [];
    const used = new Array(nums.length).fill(false);

    function dfs(current: number[]) {
        if (current.length === nums.length) {
            result.push([...current]);
            return;
        }
        for (let i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            used[i] = true;
            dfs([...current, nums[i]]);
            used[i] = false;          // backtrack：退回來，把它標成沒用過
        }
    }

    dfs([]);
    return result;
}
```

兩行 `used` 各管一件事，少哪一行都會壞。

`used[i] = true` 記的是「這條路上我已經用掉這個數字」，下一層的迴圈碰到它就 `continue` 跳過。少了這行，`used` 永遠全是 false，每一層都把三個數字試一遍，跑出 27 個長度 3 的組合，第一個是 `[1,1,1]`。那是「可以重複選」，不是排列。

<details>
<summary>拿掉 used[i] = true 之後的決策樹</summary>

```
[]
├── 1
│   ├── 1,1
│   │   ├── 1,1,1
│   │   ├── 1,1,2
│   │   └── 1,1,3
│   ├── 1,2
│   │   ├── 1,2,1
│   │   ├── 1,2,2
│   │   └── 1,2,3
│   └── 1,3
│       ├── 1,3,1
│       ├── 1,3,2
│       └── 1,3,3
├── 2
│   ├── 2,1
│   │   ├── 2,1,1
│   │   ├── 2,1,2
│   │   └── 2,1,3
│   ├── 2,2
│   │   ├── 2,2,1
│   │   ├── 2,2,2
│   │   └── 2,2,3
│   └── 2,3
│       ├── 2,3,1
│       ├── 2,3,2
│       └── 2,3,3
└── 3
    ├── 3,1
    │   ├── 3,1,1
    │   ├── 3,1,2
    │   └── 3,1,3
    ├── 3,2
    │   ├── 3,2,1
    │   ├── 3,2,2
    │   └── 3,2,3
    └── 3,3
        ├── 3,3,1
        ├── 3,3,2
        └── 3,3,3
```

每一層都是滿的三個分支，3 × 3 × 3 = 27 個葉子。跟上面那棵對照：正確版的 `1` 底下只有 `1,2` 跟 `1,3`，沒有 `1,1`，因為 1 被標成用過了。`used` 做的就是把這些分支剪掉，27 剪成 6。

</details>

`used[i] = false` 是 backtrack。走完一條路把狀態還原，這個數字才輪得到別條路徑用。少了這行只會得到一個答案 `[1,2,3]`：第一條路走完 `used` 全是 true，回頭時沒還原，其他分支全部被擋在 `continue`。

---

## DFS vs BFS：什麼時候用誰？

| | DFS | BFS |
|---|---|---|
| 資料結構 | stack（或遞迴） | queue |
| 走法 | 一條路到底 | 一層一層 |
| 空間 | $O(\text{深度})$ | $O(\text{寬度})$ |
| 找最短路 | 不行 | 可以 |
| 走遍全部 | 可以 | 可以 |
| 排列組合 | 用 DFS | 不適合 |

**需要量距離或時間？用 BFS。** BFS 一層一層往外擴，層數就是步數或時間。DFS 深入一條路走到底，拿不到「幾層」這個資訊。

同樣是 grid graph：

- **Number of Islands** — 只問「有幾塊」，不在乎順序和距離。DFS 直接把整塊標記掉，寫起來最短。BFS 也行，但沒必要。
- **Rotting Oranges** — 問「幾分鐘全爛」。時間 = 層數。只能 BFS，讓所有爛橘子同時往外擴，一層 = 一分鐘。

**要最短路徑？用 BFS。** BFS 第一次碰到目標時，走的步數就是最短。因為它一層一層展開，先碰到的一定比較近。

**要列出所有可能？用 DFS。** 排列、組合、子集、N-Queens。DFS 天生就是「一條路走完、退回來、走下一條」。

**走遍圖？都行。** 但 DFS 寫起來比較短（遞迴三行），BFS 要維護 queue。

---

## 高頻題清單

### Tree DFS

| 題目 | 核心 |
|------|------|
| [#104 Maximum Depth](/problem/maximum-depth-of-binary-tree) | 後序，max(left, right) + 1 |
| [#112 Path Sum](/problem/path-sum) | 前序，帶著 remaining target 往下走 |
| [#236 Lowest Common Ancestor](/problem/lowest-common-ancestor-of-a-binary-tree) | 後序，左右各找，找到就回傳 |
| [#124 Binary Tree Maximum Path Sum](/problem/binary-tree-maximum-path-sum) | 後序，全域變數記最大值 |

### Graph DFS

| 題目 | 核心 |
|------|------|
| [#200 Number of Islands](/problem/number-of-islands) | 碰到 1 就 DFS 標記整座島 |
| [#133 Clone Graph](/problem/clone-graph) | DFS + hash map 記已複製的 |
| [#207 Course Schedule](/problem/course-schedule) | DFS 偵測環 |
| [#695 Max Area of Island](/problem/max-area-of-island) | DFS 回傳面積 |

### Graph BFS

| 題目 | 核心 |
|------|------|
| [#200 Number of Islands](/problem/number-of-islands) | BFS 也可以標記整座島，邏輯同 DFS，只是用 queue |
| [#994 Rotting Oranges](/problem/rotting-oranges) | 多源 BFS，所有起點同時入 queue，層數 = 分鐘數 |

### Backtracking

| 題目 | 核心 |
|------|------|
| [#78 Subsets](/problem/subsets) | 每個元素選或不選 |
| [#46 Permutations](/problem/permutations) | used 陣列追蹤 |
| [#39 Combination Sum](/problem/combination-sum) | 可重複選，用 start index 避免重複 |
| [#51 N-Queens](/problem/n-queens) | 逐行放，check 列和對角線 |
| [#79 Word Search](/problem/word-search) | grid 上 DFS，visited + backtrack |

---

## 總結

DFS 做一件事：**窮舉所有路徑**。

在 tree 上，它走遍每個節點。
在 graph 上，它走遍每個連通的節點（加 visited）。
在排列組合上，它走遍每一種可能的選擇。

每一題都是這三行：

```
做選擇
遞迴
撤銷選擇
```

DFS 題目大多是這三行的變形，差別只在「選擇是什麼」跟「什麼時候收集結果」。

想學怎麼把重複的子問題記起來、不重複算？看 [DP](/concept/dp)。DFS + 記憶化 = DP 的 top-down 版本。
