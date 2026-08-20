課程有先修。先修連成一條路。問能不能全部修完 — 也就是：會不會出現「甲要乙，乙要丙，丙又要甲」這種死結？

```
numCourses = 4
prerequisites = [[1,0], [2,0], [3,1], [3,2]]

[a, b] 的意思：修 a 前要先修 b。畫成箭頭 b → a：

    0
   ↙ ↘
  1   2
   ↘ ↙
    3

沒有環，修得完 → true

換一組：
prerequisites = [[1,0], [0,1]]

    0 ⇄ 1

互相等對方，死鎖 → false
```

---

**解題引導**

先弄清楚箭頭方向。`[a, b]` = 「修 a 前要先修 b」→ 畫成 `b → a`，先修指向後修。

**Step 1：題目本質是什麼？**

*finish all courses = no cycle in the directed graph.*

<span class="spoiler">有向圖找環。沒有環就修得完，有環就死鎖。</span>

**Step 2：DFS 走每條路，怎麼判斷遇到環？**

*if i come back to a node i'm currently walking through, that's a cycle.*

<span class="spoiler">走到一個「目前正在 DFS 路徑上」的節點，就是環。走到一個「之前走過但已經結束了」的節點，不是環。</span>

**Step 3：走過的節點只用一個 visited 陣列夠嗎？**

*visited alone loses the distinction between on-path and done.*

<span class="spoiler">不夠。visited 分不出「正在走」還是「走完了」。鑽石形狀的圖 A→B→D、A→C→D，從 B 走到 D 再回來，C 再碰到 D 時會誤判成環。需要三種狀態。</span>

**Step 4：有沒有反過來的思路 — 不找環，找「沒有先修的課」？**

*start from nodes with no prerequisites. peel them off layer by layer.*

<span class="spoiler">拓撲排序。找 in-degree 為 0 的課先修。修完後把它指向的下游節點 in-degree 都減 1，新出現的 0 再修。全部都修到 = 無環。中途沒有 0 可選 = 有環。</span>

想完再往下看 code。

---

## 解法一：DFS 三色

第一個嘗試：把每一對 `[a, b]` 建成邊 `b → a`，然後對每個節點跑 DFS，如果 DFS 走回「這次還沒結束」的節點就是環。

但只用一個 visited 陣列會錯。看這個例子：

```
    0
   ↙ ↘
  1   2
   ↘ ↙
    3

從 0 出發，DFS：
  走 0 → 1 → 3，把 0, 1, 3 標成 visited
  回到 0，走 0 → 2
  2 → 3，3 已經 visited → 誤判：有環！

實際上沒環，只是鑽石形狀。
```

問題在於 visited 分不出「**正在走的路徑上**」和「**走完了，但之前走過**」。需要三種狀態：

- `0` 未訪問（白）
- `1` 正在 DFS 路徑上（灰）— 如果碰到它，就是環
- `2` 走完了，確認底下沒環（黑）— 碰到它可以直接略過

```typescript
function canFinish(numCourses: number, prerequisites: number[][]): boolean {
    const graph: number[][] = Array.from({ length: numCourses }, () => []);
    for (const [a, b] of prerequisites) {
        graph[b].push(a); // b → a
    }
    const state = new Array(numCourses).fill(0); // 0 白 / 1 灰 / 2 黑

    const dfs = (node: number): boolean => {
        if (state[node] === 1) return false; // 灰 = 環
        if (state[node] === 2) return true;  // 黑 = 已確認沒環
        state[node] = 1;
        for (const next of graph[node]) {
            if (!dfs(next)) return false;
        }
        state[node] = 2;
        return true;
    };

    for (let i = 0; i < numCourses; i++) {
        if (!dfs(i)) return false;
    }
    return true;
}
```

<details>
<summary>Go 版本</summary>

```go
func canFinish(numCourses int, prerequisites [][]int) bool {
    graph := make([][]int, numCourses)
    for _, p := range prerequisites {
        // [a, b] = 先修 b 才能修 a → 邊 b → a
        graph[p[1]] = append(graph[p[1]], p[0])
    }

    state := make([]int, numCourses) // 0 白 / 1 灰 / 2 黑

    var dfs func(node int) bool
    dfs = func(node int) bool {
        if state[node] == 1 {
            return false // 灰：這次正在走的路徑碰到自己 → 環
        }
        if state[node] == 2 {
            return true // 黑：之前確認過沒環，不用再走
        }
        state[node] = 1 // 進入路徑
        for _, next := range graph[node] {
            if !dfs(next) {
                return false
            }
        }
        state[node] = 2 // 退出路徑，結案
        return true
    }

    for i := 0; i < numCourses; i++ {
        if !dfs(i) {
            return false
        }
    }
    return true
}
```

</details>


- Time: O(V + E) — 每個節點、每條邊都只處理一次（黑色節點直接跳過）
- Space: O(V + E) — 鄰接表 + state + 遞迴 call stack

用鑽石圖 `[[1,0],[2,0],[3,1],[3,2]]` 走一遍，驗證三色不會誤判：

```
graph: 0→[1,2], 1→[3], 2→[3], 3→[]
state: [白, 白, 白, 白]

從 0 開始 DFS：
  dfs(0): state[0]=灰
    dfs(1): state[1]=灰
      dfs(3): state[3]=灰
        (3 沒下游)
      state[3]=黑
    state[1]=黑
    dfs(2): state[2]=灰
      dfs(3): state[3]=黑 → 直接回 true，不再走 ✓
    state[2]=黑
  state[0]=黑

沒有誤判。每條邊最多走一次。
```

換有環的 `[[1,0],[0,1]]`：

```
graph: 0→[1], 1→[0]
state: [白, 白]

dfs(0): state[0]=灰
  dfs(1): state[1]=灰
    dfs(0): state[0]=灰 → 撞到灰 → 回 false ✓
```

---

## 解法二：Kahn BFS

複雜度已經是 O(V+E)，砍不下去 — 每個節點、每條邊至少要看一次。但還有另一個角度。

不找環，改找「可以最先修的課」：in-degree 為 0 的節點（沒有任何先修要求）。修完一門，把它指向的下游節點 in-degree 都減 1。新的 0 加進來繼續修。全部修到 = 無環。有節點永遠 in-degree > 0 = 被困在環裡。

這就是 **Kahn's algorithm**，拓撲排序的 BFS 版。

```typescript
function canFinish(numCourses: number, prerequisites: number[][]): boolean {
    const graph: number[][] = Array.from({ length: numCourses }, () => []);
    const inDegree = new Array(numCourses).fill(0);
    for (const [a, b] of prerequisites) {
        graph[b].push(a);
        inDegree[a]++;
    }

    const queue: number[] = [];
    for (let i = 0; i < numCourses; i++) {
        if (inDegree[i] === 0) queue.push(i);
    }

    let taken = 0;
    while (queue.length > 0) {
        const node = queue.shift()!;
        taken++;
        for (const next of graph[node]) {
            inDegree[next]--;
            if (inDegree[next] === 0) queue.push(next);
        }
    }
    return taken === numCourses;
}
```

<details>
<summary>Go 版本</summary>

```go
func canFinish(numCourses int, prerequisites [][]int) bool {
    graph := make([][]int, numCourses)
    inDegree := make([]int, numCourses)
    for _, p := range prerequisites {
        graph[p[1]] = append(graph[p[1]], p[0])
        inDegree[p[0]]++ // 每有一個先修，自己的 in-degree +1
    }

    queue := []int{}
    for i := 0; i < numCourses; i++ {
        if inDegree[i] == 0 {
            queue = append(queue, i) // 沒有先修的課，可以直接修
        }
    }

    taken := 0
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        taken++
        for _, next := range graph[node] {
            inDegree[next]--
            if inDegree[next] == 0 {
                queue = append(queue, next) // 先修修完了，它可以排進來
            }
        }
    }
    return taken == numCourses // 全修到才算成功
}
```

</details>


- Time: O(V + E)
- Space: O(V + E)

一樣的複雜度，但邏輯直觀到不太像 graph 題。用鑽石圖 `[[1,0],[2,0],[3,1],[3,2]]` 走：

```
graph: 0→[1,2], 1→[3], 2→[3], 3→[]
inDegree: [0, 1, 1, 2]

queue = [0]   （只有 0 沒有先修）
taken = 0

修 0：taken=1
  0 指向 1、2，各自 in-degree --
  inDegree: [0, 0, 0, 2]
  1, 2 變成 0，進 queue
  queue = [1, 2]

修 1：taken=2
  1 指向 3，inDegree[3]--
  inDegree: [0, 0, 0, 1]
  queue = [2]

修 2：taken=3
  2 指向 3，inDegree[3]--
  inDegree: [0, 0, 0, 0]
  3 變 0，進 queue
  queue = [3]

修 3：taken=4

taken (4) == numCourses (4) → true ✓
```

換有環的 `[[1,0],[0,1]]`：

```
graph: 0→[1], 1→[0]
inDegree: [1, 1]

queue = []   （沒人是 0！）
taken = 0

迴圈沒跑，taken=0 != 2 → false ✓
```

環把 0 和 1 互相卡住，in-degree 永遠下不到 0，queue 從頭到尾都空。

---

**兩種寫法的差別**

| | DFS 三色 | Kahn BFS |
|---|---|---|
| 思路 | 找環（負面列表） | 找可修的課（正面列表） |
| 狀態 | 白 / 灰 / 黑 三色 | in-degree 歸零 |
| 判斷 | 走到灰 = 環 | 修到的數 != 總數 = 環 |
| 順便得到 | 拓撲序（退出 DFS 的逆序） | 拓撲序（queue 出來的順序） |

兩種寫法都是 O(V + E)。面試看哪個說得出來就寫哪個。要產生實際的拓撲順序（[210 Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)）的話，Kahn 更直覺：出 queue 的順序就是答案。DFS 也行，但要把黑化的節點**倒著放**才是拓撲序。

---

**Overthinking**

> 只跑一次 DFS(0) 夠嗎？

不夠。如果圖不是從 0 連通的（有兩個獨立子圖），只從 0 出發會漏掉另一半。所以外層要 for 遍歷每個節點，已經黑化的會被跳過，不會重複走。

> Kahn 跑到一半 queue 空了，可不可能「其實還能修，只是先後順序不對」？

不會。只要某個節點能修，它的所有先修一定在它之前就 in-degree 歸零進到 queue。一旦 queue 空，代表剩下的節點互相鎖住，誰也等不到。

> DFS 會不會 stack overflow？

V 到 2000、E 到 5000，最壞情況遞迴深度 2000。Go 的 goroutine 預設 stack 會長到 1GB，沒問題。真實面試怕爆 stack 的話就換 Kahn。

---

## 結論

| 解法 | Time | Space | 會不會額外給你拓撲序 |
|---|---|---|---|
| DFS + visited only | — | — | 錯的，會誤判鑽石為環 |
| DFS 三色 | O(V+E) | O(V+E) | 會（退出順序倒著） |
| Kahn BFS | O(V+E) | O(V+E) | 會（queue 出隊順序） |

有向圖找環 = 三色 DFS 或 Kahn。visited 一個值不夠，「走過」跟「正在走」不同件事。這是拓撲排序家族的入口題。學會這兩招，[210](https://leetcode.com/problems/course-schedule-ii/)、[269 Alien Dictionary](https://leetcode.com/problems/alien-dictionary/)、[444 Sequence Reconstruction](https://leetcode.com/problems/sequence-reconstruction/) 全部同一套模板。
