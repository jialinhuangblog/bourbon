爛橘子會傳染，問幾分鐘後全爛；如果有橘子永遠爛不到就回傳 -1。

「感染從所有爛橘子同時出發，一圈一圈往外擴 — 這不就是 BFS 的形狀？」

---

**解題引導**

用這個 grid 思考：

```
2 1 1
1 1 0
0 1 1
```

答案是 4。

**Step 1：每分鐘發生什麼？**

*Each minute, the rot spreads one step outward from every rotten orange simultaneously.*

<span class="spoiler">每個爛橘子同時感染四個鄰居。不是一顆一顆傳，是所有爛橘子同時行動。 / Every rotten orange spreads to all 4 neighbors at the same time, every minute.</span>

**Step 2：「同時從多個起點往外擴一層」這是什麼演算法？**

*Multiple sources, expanding level by level...*

<span class="spoiler">BFS。而且是多源 BFS — 把所有爛橘子一次全放進 queue，然後一層層展開。每一層 = 一分鐘。 / Multi-source BFS. Enqueue all rotten oranges at once, process level by level. Each level = one minute.</span>

**Step 3：BFS 跑完後，答案是什麼？**

*Not just how many levels — you also need to check if anyone was left out.*

<span class="spoiler">答案是跑了幾層（幾分鐘）。但跑完後還要掃一遍 grid，如果還有 1 就代表有橘子永遠爛不到，回傳 -1。 / The number of BFS levels is the answer. But after BFS, scan the grid — if any 1 remains, return -1.</span>

**Step 4：怎麼數「幾層」？**

*The classic BFS level-tracking pattern.*

<span class="spoiler">每次處理完當前 queue 的所有元素（一層），minutes 加一。或者用 queue 長度記錄每層邊界。注意：如果最後一層沒有新感染，minutes 不要加。 / Process all nodes of the current level, then increment minutes. Be careful not to count a level that produced no new infections.</span>

**Step 5：起點怎麼初始化？也要數 fresh 橘子嗎？**

*Before BFS even starts, do a full scan.*

<span class="spoiler">先掃一遍 grid：把所有 2 加進 queue，同時數 fresh 橘子總數 freshCount。BFS 每感染一顆就 freshCount--。最後 freshCount == 0 才算成功。 / First pass: enqueue all 2s, count all 1s into freshCount. Each time a fresh orange rots, freshCount--. If freshCount == 0 at end, return minutes. Else -1.</span>

想完再往下看 code。

---

## 解法：多源 BFS

BFS 解。先掃 grid 收集起點 + 數 fresh，然後一層層展開。

```typescript
function orangesRotting(grid: number[][]): number {
    const rows = grid.length, cols = grid[0].length;
    const queue: [number, number][] = [];
    let fresh = 0;

    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            if (grid[r][c] === 2) queue.push([r, c]);
            else if (grid[r][c] === 1) fresh++;
        }
    }

    if (fresh === 0) return 0;

    const dirs: [number, number][] = [[0,1],[0,-1],[1,0],[-1,0]];
    let minutes = 0;

    while (queue.length > 0 && fresh > 0) {
        minutes++;
        const size = queue.length;      // 這一層有幾個
        for (let i = 0; i < size; i++) {
            const [r, c] = queue[i];
            for (const [dr, dc] of dirs) {
                const nr = r + dr, nc = c + dc;
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                if (grid[nr][nc] !== 1) continue;
                grid[nr][nc] = 2;
                fresh--;
                queue.push([nr, nc]);
            }
        }
        queue.splice(0, size);          // 切掉這一層
    }

    return fresh > 0 ? -1 : minutes;
}
```

<details>
<summary>Go 版本</summary>

```go
func orangesRotting(grid [][]int) int {
    rows, cols := len(grid), len(grid[0])
    queue := [][2]int{}
    fresh := 0

    // 先掃一遍：爛的入 queue，fresh 計數
    for r := 0; r < rows; r++ {
        for c := 0; c < cols; c++ {
            if grid[r][c] == 2 {
                queue = append(queue, [2]int{r, c})
            } else if grid[r][c] == 1 {
                fresh++
            }
        }
    }

    if fresh == 0 {
        return 0 // 本來就沒有 fresh，直接回傳
    }

    dirs := [][2]int{{0, 1}, {0, -1}, {1, 0}, {-1, 0}}
    minutes := 0

    for len(queue) > 0 && fresh > 0 {
        minutes++
        size := len(queue)          // 這一層有幾個爛橘子
        for i := 0; i < size; i++ {
            curr := queue[i]
            for _, d := range dirs {
                nr, nc := curr[0]+d[0], curr[1]+d[1]
                if nr < 0 || nr >= rows || nc < 0 || nc >= cols {
                    continue
                }
                if grid[nr][nc] != 1 {
                    continue            // 不是 fresh 就跳過
                }
                grid[nr][nc] = 2        // 感染
                fresh--
                queue = append(queue, [2]int{nr, nc})
            }
        }
        queue = queue[size:]            // 切掉這一層，留下下一層
    }

    if fresh > 0 {
        return -1
    }
    return minutes
}
```

</details>

走一遍 `[[2,1,1],[1,1,0],[0,1,1]]`：

```
初始 queue: [(0,0)]   fresh=6

Minute 1：處理 (0,0)
  感染 (0,1)、(1,0)
  queue: [(0,1),(1,0)]   fresh=4

Minute 2：處理 (0,1)、(1,0)
  (0,1) 感染 (0,2)、(1,1)
  (1,0) 感染 → (0,0) 已爛、(2,0) 是 0、(1,1) 剛被感染
  queue: [(0,2),(1,1)]   fresh=2

Minute 3：處理 (0,2)、(1,1)
  (0,2) → 無新鮮鄰居
  (1,1) 感染 (2,1)
  queue: [(2,1)]   fresh=1

Minute 4：處理 (2,1)
  感染 (2,2)
  queue: []   fresh=0
```

fresh=0，回傳 4。

- Time: $O(m \times n)$ — 每格最多進 queue 一次
- Space: $O(m \times n)$ — queue 最壞裝滿整個 grid

每格至少要看一次，$O(m \times n)$ 是下限。

---

**為什麼要記錄每層邊界？**

BFS 一般只問「能不能到、最短路幾步」。這題要問「幾分鐘」，所以要知道「現在處理的是第幾層」。

方法：進 loop 前先存 `size = len(queue)`，只處理前 `size` 個，處理完 minutes 加一。這樣 queue 裡新加進去的下一層不會被當成這層處理。

---

**-1 的判斷不是看 queue 空了嗎？**

不是。queue 空了只代表「所有能爛的都爛了」，但如果有 fresh 橘子被 0 包圍、跟任何爛橘子都不連通，它永遠進不了 queue。所以要靠 `freshCount` 判斷。

### 小優化：splice → head pointer

上面 TypeScript 用 `queue.splice(0, size)` 切掉每層。`splice` 會 shift 整個陣列，O(size)，每層都做一次，總複雜度悄悄變 $O(n^2)$。

改用 head pointer，只動 index 不動陣列：

```typescript
let head = 0;
while (head < queue.length && fresh > 0) {
    minutes++;
    const size = queue.length - head;   // 這一層的長度
    for (let i = 0; i < size; i++) {
        const [x, y] = queue[head++];   // head 往前推，不 splice
        for (const [dx, dy] of dirs) {
            const nx = x + dx, ny = y + dy;
            if (nx < 0 || nx >= rows || ny < 0 || ny >= cols) continue;
            if (grid[nx][ny] !== 1) continue;
            grid[nx][ny] = 2;
            fresh--;
            queue.push([nx, ny]);
        }
    }
}
```

整體還是 $O(m \times n)$，只是把隱藏的 O(n) splice 拿掉。面試能說出「splice 會 shift 整個陣列，我用 head index 繞過」就是加分。

---

## 結論

多源 BFS：所有起點同時入 queue，一層 = 一分鐘，freshCount 歸零才成功。
