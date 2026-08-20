---
title: "Graph"
category: Data Structures
slug: graph
subtitle: 點和邊，關係的資料結構
date: 2026-02-28T11:51:31
---

# Graph

Tree 是一對多。每個 parent 有多個 children，但每個 child 只有一個 parent。

Graph 是多對多。任何節點都可以跟任何節點連。可以有環。可以有方向。可以有權重。

社群網路是 graph。路線圖是 graph。課程的先修關係是 graph。網頁之間的超連結是 graph。

Tree 是 graph 的特例：沒有環的連通 graph。

---

## 怎麼表示？

### Adjacency List（鄰接表）

每個節點存一個 list，記它連到誰。

```
0 → [1, 2]
1 → [0, 3]
2 → [0]
3 → [1]
```

```go
graph := map[int][]int{
    0: {1, 2},
    1: {0, 3},
    2: {0},
    3: {1},
}
```

空間 $O(V + E)$。V = 節點數，E = 邊數。大部分 LeetCode 題用這個。

### Adjacency Matrix（鄰接矩陣）

$V \times V$ 的 2D 陣列。`matrix[i][j] = 1` 代表 i 到 j 有邊。

```
    0  1  2  3
0 [ 0, 1, 1, 0 ]
1 [ 1, 0, 0, 1 ]
2 [ 1, 0, 0, 0 ]
3 [ 0, 1, 0, 0 ]
```

空間 $O(V^2)$。節點多但邊少（sparse graph）就很浪費。

### 什麼時候用哪個？

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| 空間 | $O(V + E)$ | $O(V^2)$ |
| 查「i 和 j 有沒有邊」 | $O(\text{degree})$ | $O(1)$ |
| 找「i 的所有鄰居」 | $O(\text{degree})$ | $O(V)$ |
| 適合 | sparse graph | dense graph |

LeetCode 的 graph 題大多用 adjacency list，因為那些 graph 通常是 sparse 的。

---

## 有向 vs 無向

### 無向圖

邊沒有方向。A 連到 B = B 連到 A。

```
0 — 1
|   |
2   3
```

adjacency list 兩邊都要加：`graph[0].add(1)` 且 `graph[1].add(0)`。

### 有向圖

邊有方向。A → B 不代表 B → A。

```
0 → 1
↑   ↓
2   3
```

課程先修關係就是有向圖。修完 A 才能修 B，不代表修完 B 能修 A。

---

## DFS 走圖

跟 tree DFS 一樣，多一個 visited。

```go
func dfs(node int, graph map[int][]int, visited map[int]bool) {
    if visited[node] { return }
    visited[node] = true

    for _, neighbor := range graph[node] {
        dfs(neighbor, graph, visited)
    }
}
```

Graph 可能有環，所以沒有 visited 就會無限繞圈。

### 數連通分量

[#200 Number of Islands](/problem/number-of-islands) 本質就是數連通分量。

每次從一個沒走過的節點開始 DFS，一次會走完一整塊連通的，所以 DFS 起跑了幾次，就有幾個連通分量。

```go
func countComponents(n int, edges [][]int) int {
    graph := make(map[int][]int)
    for _, e := range edges {
        graph[e[0]] = append(graph[e[0]], e[1])
        graph[e[1]] = append(graph[e[1]], e[0])
    }

    visited := make(map[int]bool)
    count := 0
    for i := 0; i < n; i++ {
        if !visited[i] {
            dfs(i, graph, visited)
            count++
        }
    }
    return count
}
```

---

## BFS 找最短路

BFS 一層一層展開。第一次碰到目標時走的步數 = 最短路徑。

前提：每條邊的權重相同（都是 1）。

### 走一遍：Word Ladder

[#127 Word Ladder](/problem/word-ladder)

```
beginWord = "hit", endWord = "cog"
wordList = ["hot","dot","dog","lot","log","cog"]

hit → hot → dot → dog → cog   (4 步)
hit → hot → lot → log → cog   (4 步)
```

每個 word 是一個節點，只差一個字母的兩個 word 之間有邊。這樣一來就變成無權重的最短路問題，用 BFS 走。

```
queue: ["hit"]                     step=1
queue: ["hot"]                     step=2
queue: ["dot", "lot"]              step=3
queue: ["dog", "log"]              step=4
queue: ["cog"]                     step=5 → 找到！
```

5 步（含起點）。

---

## 偵測環

### 無向圖：DFS 碰到 visited 但不是 parent

```go
func hasCycle(node, parent int, graph map[int][]int, visited map[int]bool) bool {
    visited[node] = true
    for _, neighbor := range graph[node] {
        if !visited[neighbor] {
            if hasCycle(neighbor, node, graph, visited) { return true }
        } else if neighbor != parent {
            return true    // 碰到已走過的，而且不是來的方向 → 有環
        }
    }
    return false
}
```

### 有向圖：三色標記

- 白色：還沒走
- 灰色：正在走（在 call stack 裡）
- 黑色：走完了

碰到灰色 = 走回了正在走的路 = 有環。

```go
func hasCycleDirected(node int, graph map[int][]int, color map[int]int) bool {
    color[node] = 1    // 灰色

    for _, neighbor := range graph[node] {
        if color[neighbor] == 1 { return true }     // 碰到灰色 → 環
        if color[neighbor] == 0 {
            if hasCycleDirected(neighbor, graph, color) { return true }
        }
    }

    color[node] = 2    // 黑色
    return false
}
```

[#207 Course Schedule](/problem/course-schedule) 就是有向圖偵測環。有環 = 有循環依賴 = 修不完。

---

## 拓撲排序

拓撲排序的意思是把所有節點排成一列，讓每條邊的起點都在終點前面。

只有有向無環圖（DAG）排得出來，因為要是有環，環上的節點會互相要求對方排在自己前面。

```
課程依賴：
0 → 1 → 3
0 → 2 → 3

拓撲排序：0, 1, 2, 3  或  0, 2, 1, 3
```

修課順序：先修 0，再修 1 和 2（順序不限），最後修 3。

### BFS 拓撲排序（Kahn's Algorithm）

1. 計算每個節點的 in-degree（被幾條邊指向）。
2. in-degree = 0 的丟進 queue。
3. 從 queue 拿出一個，放進結果。把它指向的節點 in-degree - 1。如果變 0，丟進 queue。
4. 重複直到 queue 空。

```go
func topologicalSort(n int, edges [][]int) []int {
    graph := make(map[int][]int)
    inDegree := make([]int, n)

    for _, e := range edges {
        graph[e[0]] = append(graph[e[0]], e[1])
        inDegree[e[1]]++
    }

    queue := []int{}
    for i := 0; i < n; i++ {
        if inDegree[i] == 0 { queue = append(queue, i) }
    }

    result := []int{}
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        result = append(result, node)

        for _, neighbor := range graph[node] {
            inDegree[neighbor]--
            if inDegree[neighbor] == 0 {
                queue = append(queue, neighbor)
            }
        }
    }

    if len(result) != n { return nil }   // 有環，拓撲排序不存在
    return result
}
```

如果最後 result 的長度不等於 n，代表有些節點 in-degree 永遠不會變 0 = 有環。

[#210 Course Schedule II](/problem/course-schedule-ii) 就是拓撲排序。

---

## 帶權重的最短路：Dijkstra

BFS 只能處理每條邊權重相同的情況。如果邊有不同的權重呢？

Dijkstra 演算法：用 priority queue（min heap），每次取出距離最短的節點展開。

```go
func dijkstra(n int, graph map[int][][2]int, start int) []int {
    dist := make([]int, n)
    for i := range dist { dist[i] = math.MaxInt }
    dist[start] = 0

    // min heap: [距離, 節點]
    pq := &MinHeap{{0, start}}

    for pq.Len() > 0 {
        curr := heap.Pop(pq).([2]int)
        d, u := curr[0], curr[1]

        if d > dist[u] { continue }    // 已有更短的路

        for _, edge := range graph[u] {
            v, w := edge[0], edge[1]
            if dist[u]+w < dist[v] {
                dist[v] = dist[u] + w
                heap.Push(pq, [2]int{dist[v], v})
            }
        }
    }
    return dist
}
```

$O((V + E) \log V)$。

[#743 Network Delay Time](/problem/network-delay-time) 就是 Dijkstra。

---

## Graph 題的判斷流程

```
graph 題
├── 要走遍全部？
│   ├── 是 → DFS 或 BFS
│   └── 數幾塊？ → DFS 數連通分量
├── 要找最短路？
│   ├── 無權重 → BFS
│   └── 有權重 → Dijkstra
├── 有依賴關係？
│   ├── 能不能完成？ → 偵測環
│   └── 什麼順序？ → 拓撲排序
└── 要偵測環？
    ├── 無向 → DFS + parent check
    └── 有向 → 三色標記 或 拓撲排序
```

---

## 高頻題清單

| 題目 | 核心 |
|------|------|
| [#200 Number of Islands](/problem/number-of-islands) | DFS/BFS 數連通分量 |
| [#133 Clone Graph](/problem/clone-graph) | DFS + hash map |
| [#207 Course Schedule](/problem/course-schedule) | 有向圖偵測環 |
| [#210 Course Schedule II](/problem/course-schedule-ii) | 拓撲排序 |
| [#127 Word Ladder](/problem/word-ladder) | BFS 最短路 |
| [#743 Network Delay Time](/problem/network-delay-time) | Dijkstra |
| [#785 Is Graph Bipartite?](/problem/is-graph-bipartite) | BFS/DFS 二分圖染色 |
| [#994 Rotting Oranges](/problem/rotting-oranges) | 多起點 BFS |
| [#261 Graph Valid Tree](/problem/graph-valid-tree) | 無環 + 連通 = tree |
| [#323 Number of Connected Components](/problem/number-of-connected-components-in-an-undirected-graph) | Union Find 或 DFS |

---

## 總結

Graph 是最通用的資料結構，所以 tree 跟 linked list 都可以看成它的特例。

graph 題大多在做這三件事：
1. **走遍**：DFS 或 BFS + visited
2. **最短路**：無權重 BFS，有權重 Dijkstra
3. **排序/偵測環**：拓撲排序

看到「連通」「島嶼」「朋友圈」→ DFS/BFS。
看到「先修課程」「依賴」「順序」→ 拓撲排序。
看到「最短路」「最少步數」→ BFS 或 Dijkstra。
