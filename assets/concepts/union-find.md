---
title: "Union-Find"
category: Data Structures
slug: union-find
subtitle: 誰跟誰同一組？O(α(n)) 回答
date: 2026-04-21T00:44:00
---

# Union-Find

班上 30 個同學，課後互相加朋友。

小明加了小華。小華加了小芳。後來有人問：小明跟小芳是朋友嗎？

暴力解是從小明開始 DFS，看能不能走到小芳。因為每次 query 都要重走一次，query 越多總時間就越長。

Union-Find：動態維護「哪些人在同一組」。加朋友 O(1)，查朋友 O(1)。合起來叫 **DSU（Disjoint Set Union）**。

---

## 每組選一個代表

把「朋友圈」想成一棵樹。每個人指向自己的 **parent**。樹根那個人就是「代表」。

```
小明 → 小華 → 小芳（root）
小傑 → 小美（root）
```

判斷兩個人同組：一路往上找 root，看 root 是不是同一個。

```go
parent := make([]int, n)
for i := range parent { parent[i] = i }   // 初始：每個人自己一組，自己是 root
```

---

## 兩個操作

### Find：找到這組的代表

```go
func find(x int) int {
    if parent[x] == x { return x }    // 自己就是 root
    return find(parent[x])            // 往上找
}
```

### Union：合併兩組

```go
func union(x, y int) {
    rx, ry := find(x), find(y)
    if rx == ry { return }            // 已經同組
    parent[rx] = ry                   // 把 x 的 root 指向 y 的 root
}
```

加朋友 = union。問朋友 = `find(x) == find(y)`。

---

## 問題：會退化成 linked list

一直 union 下去，樹可能長得很歪：

```
0 → 1 → 2 → 3 → 4 → ... → n-1
```

`find(0)` 要走 n-1 步，也就是 O(n)，這樣就沒比暴力快了。

兩個優化把它救回來。

---

## 優化一：路徑壓縮（Path Compression）

每次 `find` 走過的節點，直接掛到 root 下。

```go
func find(x int) int {
    if parent[x] != x {
        parent[x] = find(parent[x])   // 回來的時候順便更新
    }
    return parent[x]
}
```

第一次 `find(0)` 還是要走 n 步，但走完之後樹變成這樣：

```
1, 2, 3, ..., n-1 全部 → root
```

之後再 find 任何一個，一步到位。

---

## 優化二：按秩合併（Union by Rank）

合併兩棵樹時，把**矮的掛到高的下面**。這樣樹不會越長越高。

```go
rank := make([]int, n)                // 每棵樹的高度

func union(x, y int) {
    rx, ry := find(x), find(y)
    if rx == ry { return }
    if rank[rx] < rank[ry] {
        parent[rx] = ry
    } else if rank[rx] > rank[ry] {
        parent[ry] = rx
    } else {
        parent[ry] = rx
        rank[rx]++                    // 高度相同時才會加 1
    }
}
```

也可以用 **size**（節點數）代替 rank，效果差不多。

---

## 複雜度

路徑壓縮 + 按秩合併，每個操作的**均攤**複雜度是 $O(α(n))$。

α 是 inverse Ackermann 函數，增長極慢：n 就算大到宇宙的原子數，α(n) 還是不超過 4，所以實務上直接當成 O(1)。

只用其中一個優化：O(log n)。兩個都用才是 O(α(n))。

---

## 完整模板

```go
type UnionFind struct {
    parent, rank []int
    count        int                  // 剩幾組
}

func NewUnionFind(n int) *UnionFind {
    uf := &UnionFind{
        parent: make([]int, n),
        rank:   make([]int, n),
        count:  n,
    }
    for i := range uf.parent { uf.parent[i] = i }
    return uf
}

func (uf *UnionFind) Find(x int) int {
    if uf.parent[x] != x {
        uf.parent[x] = uf.Find(uf.parent[x])
    }
    return uf.parent[x]
}

func (uf *UnionFind) Union(x, y int) bool {
    rx, ry := uf.Find(x), uf.Find(y)
    if rx == ry { return false }      // 已經同組

    if uf.rank[rx] < uf.rank[ry] {
        uf.parent[rx] = ry
    } else if uf.rank[rx] > uf.rank[ry] {
        uf.parent[ry] = rx
    } else {
        uf.parent[ry] = rx
        uf.rank[rx]++
    }
    uf.count--
    return true
}

func (uf *UnionFind) Connected(x, y int) bool {
    return uf.Find(x) == uf.Find(y)
}
```

`count` 拿來回答「有幾個連通分量」。每次 union 成功就減 1。

---

## 什麼時候用 Union-Find

三個訊號：

1. **動態連通性**：邊一條一條加進來，中間要回答「這兩點連了嗎？」
2. **不需要路徑**：只關心「是不是同一組」，不關心「怎麼走過去」。需要路徑用 BFS/DFS。
3. **離線題變線上題**：DFS 適合靜態圖一次跑完。Union-Find 適合邊和 query 交錯進來。

反過來，這些情況**不要用** Union-Find：

- 要找**最短路徑** → BFS 或 Dijkstra
- 要**拆邊**（刪除邊） → Union-Find 不支援拆，只支援合。刪邊題要倒著做、離線處理
- 圖的結構**完全知道**且只問一次連通性 → DFS 就夠，不用動用 DSU

---

## 經典題一：Number of Provinces

[#547 Number of Provinces](/problem/number-of-provinces)

`isConnected[i][j] = 1` 代表 i 和 j 是朋友。朋友的朋友也算同一個省。問有幾個省。

```go
func findCircleNum(isConnected [][]int) int {
    n := len(isConnected)
    uf := NewUnionFind(n)
    for i := 0; i < n; i++ {
        for j := i + 1; j < n; j++ {
            if isConnected[i][j] == 1 {
                uf.Union(i, j)
            }
        }
    }
    return uf.count
}
```

DFS 也能寫，但 DSU 的寫法更短、更直觀。

---

## 經典題二：Redundant Connection

[#684 Redundant Connection](/problem/redundant-connection)

一棵 tree 被多加了一條邊變成有環的 graph。找出多的那條。

從頭開始 union，碰到要 union 兩個**已經同組**的節點，那條邊就是多的。

```go
func findRedundantConnection(edges [][]int) []int {
    uf := NewUnionFind(len(edges) + 1)
    for _, e := range edges {
        if !uf.Union(e[0], e[1]) {    // union 回 false = 已同組 = 成環
            return e
        }
    }
    return nil
}
```

DSU 一個 pass 就結束。DFS 則是每加一條邊都要重新檢查一次環，所以慢得多。

---

## 經典題三：Graph Valid Tree

[#261 Graph Valid Tree](/problem/graph-valid-tree)

n 個節點 + 一組邊，問是不是一棵 tree。

tree 的兩個條件：
1. **無環**
2. **連通**（剩一個分量）

```go
func validTree(n int, edges [][]int) bool {
    if len(edges) != n-1 { return false }  // tree 一定 n-1 條邊
    uf := NewUnionFind(n)
    for _, e := range edges {
        if !uf.Union(e[0], e[1]) { return false }  // 成環
    }
    return uf.count == 1
}
```

Union-Find 直接吃下兩個條件：union 失敗就是有環，`count == 1` 就是連通。

---

## 進階：帶權 Union-Find

parent 指標額外帶一個「到 parent 的權重」。可以回答「x 和 y 的關係是什麼」不只是「x 和 y 同組嗎」。

例：[#399 Evaluate Division](/problem/evaluate-division)，給 `a/b=2, b/c=3`，問 `a/c=?`。

每條邊存比例，find 的時候沿路乘起來；路徑壓縮把節點搬到 root 底下的同時，權重也要跟著更新。

這是進階用法，一般面試不會考。知道有這東西就好。

---

## Union-Find vs DFS

| | Union-Find | DFS |
|---|---|---|
| 靜態圖找連通分量 | 可以，但不是最快 | 更直觀 |
| 動態加邊 + 回答連通性 | **強項** | 每次都要重跑 |
| 找最短路徑 | 做不到 | BFS/Dijkstra |
| 偵測環 | 一個 pass，簡潔 | 要維護 visited |
| 支援刪邊 | 不支援 | 支援（每次重算） |

看到「邊一條條加」「問同組嗎」「連通分量計數」→ 先想 Union-Find。

---

## 總結

Union-Find 就是**每組選一個代表，用樹狀結構記代表是誰。**

兩個操作：find 找代表、union 合併兩組。

兩個優化：路徑壓縮讓樹變扁、按秩合併讓樹不變高。合起來得到 O(α(n)) 均攤，實務上當 O(1)。

適合動態連通性，不適合最短路徑，而且不支援刪邊。

---

## 高頻題清單

| 題目 | 核心 |
|------|------|
| [#547 Number of Provinces](/problem/number-of-provinces) | 連通分量計數 |
| [#684 Redundant Connection](/problem/redundant-connection) | union 失敗就是多餘邊 |
| [#128 Longest Consecutive Sequence](/problem/longest-consecutive-sequence) | 相鄰數字 union，找最大組 |
| [#261 Graph Valid Tree](/problem/graph-valid-tree) | 無環 + 連通 = tree |
| [#323 Number of Connected Components](/problem/number-of-connected-components-in-an-undirected-graph) | 經典 DSU |
| [#1319 Number of Operations to Make Network Connected](/problem/number-of-operations-to-make-network-connected) | 多餘邊數 vs 缺的邊數 |
| [#990 Satisfiability of Equality Equations](/problem/satisfiability-of-equality-equations) | `==` 做 union，`!=` 檢查 |
| [#721 Accounts Merge](/problem/accounts-merge) | 用 email 當 key 合併帳號 |
| [#399 Evaluate Division](/problem/evaluate-division) | 帶權 Union-Find |
| [#200 Number of Islands](/problem/number-of-islands) | DFS 更直觀，但 DSU 也能寫 |
