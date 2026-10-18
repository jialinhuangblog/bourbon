給一個二維格子，`1` 是陸地，`0` 是水。數有幾坨連在一起的 `1`。只看上下左右（4 方向），斜的不算相連。

```
1 0        1 1
0 1        1 0

→ 2 座島    → 1 座島
（斜角不連） （上下左右有連）
```

```
Example 1:                    Example 2:

1 1 1 1 0                    1 1 0 0 0
1 1 0 1 0                    1 1 0 0 0
1 1 0 0 0                    0 0 1 0 0
0 0 0 0 0                    0 0 0 1 1

→ 1 座島（全部連在一起）       → 3 座島（左上、中間、右下各一座）
```

---

**解題引導**

**Step 1：掃到一個 `1`，怎麼知道整座島有多大？**

<span class="spoiler">從這個 1 出發，往上下左右擴散，碰到 1 就繼續走，碰到 0 或邊界就停。走完就是一整座島</span>

**Step 2：走過的 1 怎麼避免重複計算？**

<span class="spoiler">走過就把 1 改成 0（沉島）。下次掃到這裡看到 0，直接跳過</span>

**Step 3：怎麼數有幾座島？**

<span class="spoiler">從左上到右下掃整個格子。每碰到一個還沒沉的 1，就是一座新島，計數 +1，然後把整座島沉掉</span>

想完再往下看 code。

---

## 解法一：DFS

用 Example 2 走一遍：

```
原始：
1 1 0 0 0
1 1 0 0 0
0 0 1 0 0
0 0 0 1 1

掃到 (0,0) 是 1 → DFS，把相連的全部改成 0
0 0 0 0 0
0 0 0 0 0        islands = 1
0 0 1 0 0
0 0 0 1 1

繼續掃... (2,2) 是 1 → DFS
0 0 0 0 0
0 0 0 0 0        islands = 2
0 0 0 0 0
0 0 0 1 1

繼續掃... (3,3) 是 1 → DFS，(3,4) 也連著
0 0 0 0 0
0 0 0 0 0        islands = 3
0 0 0 0 0
0 0 0 0 0

掃完，答案 = 3
```

---

完整程式碼（DFS）：

```typescript
function numIslands(grid: string[][]): number {
    const m = grid.length, n = grid[0].length;
    let islands = 0;

    function dfs(i: number, j: number): void {
        if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] === '0') {
            return;
        }
        grid[i][j] = '0'; // 沉島
        dfs(i + 1, j);
        dfs(i - 1, j);
        dfs(i, j + 1);
        dfs(i, j - 1);
    }

    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            if (grid[i][j] === '1') {
                islands++;
                dfs(i, j);
            }
        }
    }
    return islands;
}
```

- Time: O(m × n) — 每個格子最多被走一次
- Space: O(m × n) — 最壞情況遞迴深度（全部都是 `1` 時）

<details>
<summary>Go 版本</summary>

```go
func numIslands(grid [][]byte) int {
    m, n := len(grid), len(grid[0])
    islands := 0

    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if grid[i][j] == '1' {
                islands++      // 發現新島
                dfs(grid, i, j) // 把整座島沉掉
            }
        }
    }
    return islands
}

func dfs(grid [][]byte, i, j int) {
    m, n := len(grid), len(grid[0])
    // 超出邊界，或是水，就停
    if i < 0 || i >= m || j < 0 || j >= n || grid[i][j] == '0' {
        return
    }
    grid[i][j] = '0' // 標記為已走過（沉島）
    dfs(grid, i+1, j) // 下
    dfs(grid, i-1, j) // 上
    dfs(grid, i, j+1) // 右
    dfs(grid, i, j-1) // 左
}
```

</details>

DFS 的空間花在哪？每呼叫一次 `dfs()`，系統自動在 call stack 上疊一層，記住「這次的 i, j 是多少、做到哪了、等等回來要繼續做什麼」。你沒寫 `make` 或 `new`，但空間照樣用掉了。

```
DFS 遞迴：
dfs(0,0)       ← call stack 第 1 層
  dfs(1,0)     ← call stack 第 2 層
    dfs(2,0)   ← call stack 第 3 層
    return     ← 第 3 層釋放
  return       ← 第 2 層釋放
return         ← 第 1 層釋放

每一層大約佔幾十 bytes（存 i, j, 返回地址等）
90,000 層 × 幾十 bytes ≈ 幾 MB → call stack 上限大約幾 MB → 可能炸

BFS 版本：
queue = [(0,0), (1,0), (0,1), (1,1)]

只有一層函式呼叫，queue 的資料存在一般記憶體（heap）
90,000 個座標 × 幾 bytes ≈ 幾百 KB → 一般記憶體上限幾 GB → 完全不會炸
```

**DFS 不會重複走嗎？**

會嘗試，但不會真的走進去。用小例子看 call stack：

```
格子：
1 1
1 0

dfs(0,0)  沉掉(0,0)，往四個方向走
│ dfs(1,0)  沉掉(1,0)，往四個方向走
│ │ dfs(2,0)  超出邊界 → return
│ │ dfs(0,0)  已經是 0 → return  ← 走回來了，但已經沉了，直接擋掉
│ │ dfs(1,1)  是 0 → return
│ │ dfs(1,-1) 超出邊界 → return
│ 回到 dfs(1,0)，四個方向都試完了 → return
│ dfs(-1,0) 超出邊界 → return
│ dfs(0,1)  沉掉(0,1)，往四個方向走
│ │ ...全部碰到 0 或邊界 → return
│ dfs(0,-1) 超出邊界 → return
dfs(0,0) 做完了
```

關鍵：`dfs(1,0)` 往上看到 `(0,0)`，但 `(0,0)` 早就被沉成 `0` 了，第一行 `if grid[i][j] == '0' { return }` 直接擋掉。所以 DFS 會「嘗試」走回已走過的格子，但碰到 `0` 就立刻 return，不會真的走進去。

如果不沉島會怎樣？`(0,0)` 叫 `dfs(1,0)`，`(1,0)` 又叫 `dfs(0,0)`，`(0,0)` 又叫 `dfs(1,0)`... 無限遞迴，stack overflow。沉島不只是「避免重複計算」，是「避免無限遞迴」。

標記也可以另外開一個 `visited` 陣列記錄，但直接改原陣列省空間。面試時可以問面試官「可以修改 input 嗎？」，通常可以。

---

## 解法二：BFS

DFS 是一條路走到底再回頭。BFS 是從起點一圈一圈往外擴，像丟石頭到水裡的漣漪。結果一樣，都是把整座島沉掉。

用第一座島 (0,0) 展示 BFS 怎麼走：

```
起點 (0,0)，放進 queue，沉掉

queue: [(0,0)]
0 1 0 0 0
1 1 0 0 0
0 0 1 0 0
0 0 0 1 1

取出 (0,0)，看四個方向：
  上 (-1,0) 超出邊界
  下 (1,0) 是 1 → 沉掉，放進 queue
  左 (0,-1) 超出邊界
  右 (0,1) 是 1 → 沉掉，放進 queue

queue: [(1,0), (0,1)]
0 0 0 0 0
0 1 0 0 0
0 0 1 0 0
0 0 0 1 1

取出 (1,0)，看四個方向：
  上 (0,0) 是 0，跳過
  下 (2,0) 是 0，跳過
  左 超出邊界
  右 (1,1) 是 1 → 沉掉，放進 queue

取出 (0,1)，看四個方向：
  全部是 0 或已沉，沒有新的

queue: [(1,1)]
0 0 0 0 0
0 0 0 0 0
0 0 1 0 0
0 0 0 1 1

取出 (1,1)，四個方向全是 0

queue: []  ← 空了，第一座島走完了，islands = 1
              外層迴圈繼續掃，碰到下一個 1 再做一次 BFS
```

程式碼：

```typescript
function bfs(grid: string[][], i: number, j: number): void {
    const m = grid.length, n = grid[0].length;
    const queue: [number, number][] = [[i, j]];
    grid[i][j] = '0'; // 先沉起點

    //                                下       上       右       左
    const dirs: [number, number][] = [[1, 0], [-1, 0], [0, 1], [0, -1]];

    while (queue.length > 0) {
        const cur = queue.shift()!;      // 拿出第一個（先進先出）

        for (const d of dirs) {
            const ni = cur[0] + d[0], nj = cur[1] + d[1];  // 鄰居的座標
            // 沒超出邊界，而且是陸地
            if (ni >= 0 && ni < m && nj >= 0 && nj < n && grid[ni][nj] === '1') {
                grid[ni][nj] = '0';    // 沉掉
                queue.push([ni, nj]);  // 放進 queue，等等處理
            }
        }
    }
}
```

<details>
<summary>Go 版本</summary>

```go
func bfs(grid [][]byte, i, j int) {
    m, n := len(grid), len(grid[0])
    queue := [][2]int{{i, j}}
    grid[i][j] = '0' // 先沉起點

    //        下      上       右      左
    dirs := [][2]int{{1,0}, {-1,0}, {0,1}, {0,-1}}

    for len(queue) > 0 {
        cur := queue[0]          // 拿出第一個（先進先出）
        queue = queue[1:]        // 從 queue 移除它

        for _, d := range dirs {
            ni, nj := cur[0]+d[0], cur[1]+d[1]  // 鄰居的座標
            // 沒超出邊界，而且是陸地
            if ni >= 0 && ni < m && nj >= 0 && nj < n && grid[ni][nj] == '1' {
                grid[ni][nj] = '0'                    // 沉掉
                queue = append(queue, [2]int{ni, nj})  // 放進 queue，等等處理
            }
        }
    }
}
```

</details>


外層邏輯不變，把 `dfs(grid, i, j)` 換成 `bfs(grid, i, j)` 就好。

queue 配置在一般記憶體，不吃 call stack。Time 和 Space 都一樣是 O(m × n)。

---

**這題跟 tree 的 DFS 有什麼不同？**

Tree 的 DFS 往左右走（兩個方向）。這題往上下左右走（四個方向）。

Tree 不需要防止回頭走，因為 tree 的邊是單向的（parent → child），走下去不會走回來。格子不一樣，每個 cell 跟上下左右雙向相連，從 A 走到 B，B 也能走回 A。不標記的話就會無限繞圈。

這就是 graph DFS 跟 tree DFS 的核心差別：**格子是雙向連通的，所以需要標記已走過的節點，tree 不需要。**

---

**DFS 遞迴會不會 stack overflow？**

會。最壞情況整個格子全是 `1`，遞迴深度 = m × n，300 × 300 就是 90,000 層。LeetCode 上過得了，但面試官可能追問。面試策略：先寫 DFS（簡單好懂），被問到 stack overflow 再改成上面的 BFS 版本。

---

## 解法三：Union-Find

DFS / BFS 的想法：從一塊陸地把整座島走完（top-down）。

Union-Find 把想法倒過來：每塊陸地先各自算一座，相鄰就合併，最後剩幾組就是答案（bottom-up）。

概念文章 [Union-Find](/concept/union-find) 已經鋪好了基礎，這題是實戰。

每個格子給一個編號 `i * n + j`（把二維攤平成一維）。初始每個陸地是一個獨立的組，總島數 = 陸地格子數。每合併成功一次，島數 −1。

```typescript
class UnionFind {
    parent: number[];
    rank: number[];
    count: number;

    constructor(grid: string[][]) {
        const m = grid.length, n = grid[0].length;
        this.parent = [];
        this.rank = new Array(m * n).fill(0);
        this.count = 0;
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                this.parent.push(i * n + j);  // 每格自己是 root
                if (grid[i][j] === '1') this.count++;
            }
        }
    }

    find(x: number): number {
        if (this.parent[x] !== x) {
            this.parent[x] = this.find(this.parent[x]);  // path compression
        }
        return this.parent[x];
    }

    union(x: number, y: number): void {
        const rx = this.find(x), ry = this.find(y);
        if (rx === ry) return;
        if (this.rank[rx] < this.rank[ry]) this.parent[rx] = ry;
        else if (this.rank[rx] > this.rank[ry]) this.parent[ry] = rx;
        else { this.parent[ry] = rx; this.rank[rx]++; }
        this.count--;
    }
}

function numIslands(grid: string[][]): number {
    const m = grid.length, n = grid[0].length;
    const uf = new UnionFind(grid);
    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            if (grid[i][j] !== '1') continue;
            // 只看右和下
            if (i + 1 < m && grid[i+1][j] === '1') uf.union(i*n+j, (i+1)*n+j);
            if (j + 1 < n && grid[i][j+1] === '1') uf.union(i*n+j, i*n+j+1);
        }
    }
    return uf.count;
}
```

<details>
<summary>Go 版本</summary>

```go
type UnionFind struct {
    parent []int
    rank   []int
    count  int  // 島的數量
}

func NewUnionFind(grid [][]byte) *UnionFind {
    m, n := len(grid), len(grid[0])
    parent := make([]int, m*n)
    rank := make([]int, m*n)
    count := 0
    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            idx := i*n + j
            parent[idx] = idx  // 每格自己是 root
            if grid[i][j] == '1' {
                count++  // 每塊陸地初始都算獨立一座島
            }
        }
    }
    return &UnionFind{parent, rank, count}
}

func (uf *UnionFind) Find(x int) int {
    if uf.parent[x] != x {
        uf.parent[x] = uf.Find(uf.parent[x])  // path compression：壓平
    }
    return uf.parent[x]
}

func (uf *UnionFind) Union(x, y int) {
    rx, ry := uf.Find(x), uf.Find(y)
    if rx == ry { return }  // 早就同組，不用合
    // union by rank：矮樹掛到高樹底下，避免退化成 linked list
    if uf.rank[rx] < uf.rank[ry] {
        uf.parent[rx] = ry
    } else if uf.rank[rx] > uf.rank[ry] {
        uf.parent[ry] = rx
    } else {
        uf.parent[ry] = rx
        uf.rank[rx]++
    }
    uf.count--  // 兩座島合成一座，總數 -1
}

func numIslands(grid [][]byte) int {
    m, n := len(grid), len(grid[0])
    uf := NewUnionFind(grid)
    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if grid[i][j] != '1' { continue }
            // 只看右和下就夠，左和上之前已經合過了
            if i+1 < m && grid[i+1][j] == '1' {
                uf.Union(i*n+j, (i+1)*n+j)
            }
            if j+1 < n && grid[i][j+1] == '1' {
                uf.Union(i*n+j, i*n+j+1)
            }
        }
    }
    return uf.count
}
```

</details>


**為什麼只看右和下？** 掃描順序是從左上到右下。走到 (i, j) 時，(i-1, j) 和 (i, j-1) 早就處理過了——它們輪到自己的時候，會去檢查「右」和「下」，也就是 (i, j) 本身。所以 (i, j) 的上鄰居和左鄰居早就跟它 union 過了，不用重做。

走 Example 2（每格編號 = i * 5 + j）：

```
grid（編號）：               初始：每塊陸地一個 group，7 塊陸地 → count = 7
 0  1  2  3  4              
 5  6  7  8  9              (0,0)=0 (0,1)=1 (1,0)=5 (1,1)=6 (2,2)=12 (3,3)=18 (3,4)=19
10 11 12 13 14
15 16 17 18 19

處理 (0,0)=0：右 (0,1)=1 是陸地 → union(0,1)，count=6
             下 (1,0)=5 是陸地 → union(0,5)，count=5
處理 (0,1)=1：右 水，下 (1,1)=6 是陸地 → union(1,6)，count=4
             （find(1) 得到 root=0，find(6) 還是 6 自己；合併後 0-1-5-6 同一組）
處理 (1,0)=5：右 (1,1)=6 → union(5,6)，兩者早就同組，不動
處理 (1,1)=6：右下都是水
處理 (2,2)=12：右下都是水或越界
處理 (3,3)=18：右 (3,4)=19 → union(18,19)，count=3
處理 (3,4)=19：越界

四次有效 union（0-1、0-5、1-6、18-19），7 - 4 = 3。最後 count = 3 ✓
```

- Time: O(m × n × α(m × n))，α 是反 Ackermann 函數，實際上 ≤ 4，可視為常數
- Space: O(m × n) — parent 和 rank 陣列

---

**三種解法怎麼選？**

| | DFS | BFS | Union-Find |
|---|---|---|---|
| 思路 | 探索整座島 | 探索整座島 | 合併相鄰陸地 |
| 需要改原陣列 | 要 | 要 | 不用 |
| 容易 stack overflow | 會（大島） | 不會 | 不會 |
| 程式碼長度 | 短 | 中 | 長 |
| 面試首選 | ✓ | 問到 overflow 才用 | 動態合併題才發揮 |

這題 DFS 最短最好寫。Union-Find 寫得複雜，但它的**真本事在動態題**——如果題目改成「陸地一塊塊加進來，每加一塊要回答當前有幾座島」（LeetCode 305 Number of Islands II），DFS 每次都要重新掃整張圖，UF 每次加陸地只要做兩三次 union，O(α) 回答。

把 UF 當成 DFS 的替代品不划算。把它當成「之後題目會變形時我還撐得住」的工具。

---

## 結論

掃整個格子，碰到 `1` 就 DFS 把整座島沉掉，計數 +1。Graph 的 DFS 跟 Tree 的 DFS 一樣，只是多了四個方向和防止重複走的標記。BFS 解決 stack overflow，Union-Find 換個角度——不探索、只合併——為動態題鋪路。

---

**延伸**：[289. Game of Life](/problem/game-of-life) — 同樣是二維格子，但不是找連通區塊，而是數每個格子的 8 方向鄰居決定下一輪生死。
