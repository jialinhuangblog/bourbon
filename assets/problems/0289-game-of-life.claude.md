一個二維格子，`1` 是活細胞，`0` 是死細胞。根據每個格子周圍 8 個鄰居（上下左右 + 四個斜角）的活細胞數量，決定下一輪的生死。原地更新整個格子。

規則只有四條：

```
活細胞：
  鄰居 < 2 個活的 → 死（太孤獨）
  鄰居 2-3 個活的 → 活（剛剛好）
  鄰居 > 3 個活的 → 死（太擠了）

死細胞：
  鄰居剛好 3 個活的 → 活（繁殖）
```

簡化一下：活細胞活下來只有一種情況（2-3 個鄰居），死細胞復活只有一種情況（剛好 3 個鄰居）。

```
Example 1:

原始：          下一輪：
0 1 0           0 0 0
0 0 1     →     1 0 1
1 1 1           0 1 1
0 0 0           0 1 0
```

---

**難點：同時更新**

所有格子的生死要「同時」發生。不能先更新 (0,0) 再拿新的值去算 (0,1)，因為 (0,1) 應該看到的是 (0,0) 的「舊狀態」。

**解題引導**

想像自己是格子裡的一個 cell，要決定下一輪活不活。

**Step 1：怎麼知道周圍有幾個活鄰居？**

<span class="spoiler">看八個方向（上下左右 + 四個斜角），活著的就報數，數完就是 live</span>

**Step 2：知道 live 之後，怎麼決定生死？**

<span class="spoiler">自己是活的：鄰居不夠力（< 2）或太擠（> 3）就死。自己是死的：鄰居剛好 3 個就復活。其他情況不變</span>

**Step 3：但所有 cell 要同時更新，先改的 cell 會影響後面的計算，怎麼辦？**

<span class="spoiler">最簡單：複製一份格子，對著舊的算，寫到新的。進階：用 2 和 3 編碼「原本是什麼 → 要變成什麼」，最後再還原</span>

想完再往下看 code。

---

## 解法一：複製格子

最直覺的做法：複製一份格子，對著舊格子數鄰居，寫到新格子。

```typescript
function gameOfLife(board: number[][]): void {
    const m = board.length, n = board[0].length;

    // 複製一份原始狀態
    const old = board.map(row => [...row]);

    const dirs = [[-1,-1],[-1,0],[-1,1],[0,-1],[0,1],[1,-1],[1,0],[1,1]];

    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            let live = 0;
            for (const [di, dj] of dirs) {
                const ni = i + di, nj = j + dj;
                if (ni >= 0 && ni < m && nj >= 0 && nj < n && old[ni][nj] === 1) {
                    live++;
                }
            }
            if (old[i][j] === 1) {
                if (live < 2 || live > 3) board[i][j] = 0;
            } else {
                if (live === 3) board[i][j] = 1;
            }
        }
    }
}
```

- Time: $O(m \times n)$ — m = n 的時候就是 $O(n^2)$，聽起來很糟，但每個格子至少要看一次才知道生死，所以 $O(m \times n)$ 已經是下限，不可能更快。$O(n^2)$ 可惡的情況是明明有更快的做法卻暴力跑，像 Two Sum 暴力解 $O(n^2)$，用 HashMap 每次查找 O(1)、n 個元素各查一次，總共 O(n) 就解決了。這題沒有那種捷徑
- Space: $O(m \times n)$（複製了一份格子）

<details>
<summary>Go 版本</summary>

```go
func gameOfLife(board [][]int) {
    m, n := len(board), len(board[0])

    // 複製一份原始狀態
    old := make([][]int, m)
    for i := range board {
        old[i] = make([]int, n)
        copy(old[i], board[i])
    }

    // 8 個方向的偏移量：
    // (-1,-1) (-1,0) (-1,1)     左上  上  右上
    //  (0,-1)  自己  (0,1)      左   我   右
    //  (1,-1)  (1,0) (1,1)      左下  下  右下
    dirs := [][2]int{{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}}

    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            live := 0
            for _, d := range dirs {
                ni, nj := i+d[0], j+d[1] // 從我的位置加偏移量 = 鄰居座標
                if ni >= 0 && ni < m && nj >= 0 && nj < n && old[ni][nj] == 1 {
                    live++ // 活著就報數
                }
            }
            // 套規則
            if old[i][j] == 1 {
                if live < 2 || live > 3 {
                    board[i][j] = 0 // 死
                }
            } else {
                if live == 3 {
                    board[i][j] = 1 // 活
                }
            }
        }
    }
}
```

</details>

---

## 解法二：狀態編碼 in-place

題目 follow-up 問的。不複製格子，直接在原陣列上改。**但改了之後，鄰居看到的就是新狀態，不是舊狀態，會算錯。**

破題的關鍵：用額外的數字同時記住「原本是什麼」和「要變成什麼」：

```
2 = 原本是活的，下一輪死了（活 → 死）
3 = 原本是死的，下一輪活了（死 → 活）
```

數鄰居的時候，看到 `1` 或 `2` 都算「原本是活的」。最後再把 2 改回 0，3 改回 1。

```typescript
function gameOfLife(board: number[][]): void {
    const m = board.length, n = board[0].length;
    const dirs = [[-1,-1],[-1,0],[-1,1],[0,-1],[0,1],[1,-1],[1,0],[1,1]];

    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            let live = 0;
            for (const [di, dj] of dirs) {
                const ni = i + di, nj = j + dj;
                if (ni >= 0 && ni < m && nj >= 0 && nj < n && (board[ni][nj] === 1 || board[ni][nj] === 2)) {
                    live++; // 1 或 2 都代表「原本是活的」
                }
            }
            if (board[i][j] === 1) {
                if (live < 2 || live > 3) board[i][j] = 2; // 活 → 死，暫時標記為 2
            } else {
                if (live === 3) board[i][j] = 3; // 死 → 活，暫時標記為 3
            }
        }
    }

    // 最後把 2 → 0，3 → 1
    for (let i = 0; i < m; i++) {
        for (let j = 0; j < n; j++) {
            if (board[i][j] === 2) board[i][j] = 0;
            else if (board[i][j] === 3) board[i][j] = 1;
        }
    }
}
```

- Time: $O(m \times n)$
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func gameOfLife(board [][]int) {
    m, n := len(board), len(board[0])
    dirs := [][2]int{{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}}

    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            live := 0
            for _, d := range dirs {
                ni, nj := i+d[0], j+d[1]
                if ni >= 0 && ni < m && nj >= 0 && nj < n && (board[ni][nj] == 1 || board[ni][nj] == 2) {
                    live++ // 1 或 2 都代表「原本是活的」
                }
            }
            if board[i][j] == 1 {
                if live < 2 || live > 3 {
                    board[i][j] = 2 // 活 → 死，暫時標記為 2
                }
            } else {
                if live == 3 {
                    board[i][j] = 3 // 死 → 活，暫時標記為 3
                }
            }
        }
    }

    // 最後把 2 → 0，3 → 1
    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if board[i][j] == 2 {
                board[i][j] = 0
            } else if board[i][j] == 3 {
                board[i][j] = 1
            }
        }
    }
}
```

</details>

---

**跟 Number of Islands 的差別**

| | Number of Islands | Game of Life |
|---|---|---|
| 在做什麼 | 數有幾坨連在一起的 | 數每格的鄰居，套規則 |
| 方向 | 4 方向（上下左右） | 8 方向（含斜角） |
| 核心技巧 | DFS/BFS 找連通區塊 | 數鄰居 + 狀態編碼 |
| 需要 DFS 嗎 | 需要 | 不需要 |

---

## 結論

對每個格子數 8 方向的活鄰居，套四條規則。難點是「同時更新」：用額外的數字（2 = 活→死，3 = 死→活）編碼舊狀態，最後再還原。
