要從台北飛到某個城市，直飛太貴，轉機比較便宜。但轉太多次會累，所以先講好：**最多只轉 k 次**。在這個限制底下，最便宜要多少錢？

限制不是「不能繞路」，是「中間停的次數有上限」。有時候繞遠一點反而便宜，只要停的次數還在額度內就可以。

```
四個城市，五條航線

0 --100--> 1 --100--> 2 --200--> 3
           |          ^
           |600       |100
           v          |
           3          0

src = 0，dst = 3，k = 1（最多停 1 次）

走 0 → 1 → 3        停 1 次，700 元   ← 額度內
走 0 → 1 → 2 → 3    停 2 次，400 元   ← 更便宜，但超過額度
答案 700
```

翻成 code 的講法：有向圖找最短路徑，但路徑的邊數不能超過 `k + 1`。

---

**解題引導**

拿上面那組資料（`n = 4`、`src = 0`、`dst = 3`、`k = 1`）想。

**Step 1：一般的最短路徑演算法為什麼不能直接用？**

*dijkstra just wants cheap. it does not count how many times you landed.*

<span class="spoiler">因為 Dijkstra 只認總價，不管走了幾條邊。它會挑 400 那條，而那條停了 2 次，超過額度。限制是邊數，最短路徑演算法沒有這個概念。</span>

**Step 2：那把「已經停幾次」也記進去呢？**

*a city is no longer just a city. it is a city plus how tired you are.*

<span class="spoiler">狀態從「城市」變成「城市 + 已用停靠數」。同一個城市，用 1 次停靠到達跟用 3 次到達，是兩個不同的狀態，後面能做的事也不同。</span>

**Step 3：如果反過來，按「走了幾條邊」分層來算呢？**

*round one: everything reachable in one flight. round two: two flights. stop after k plus one.*

<span class="spoiler">第 1 輪只算走一條邊到得了哪裡，第 2 輪算走兩條邊，跑滿 k+1 輪就停。這樣邊數上限自動被輪數控制住，不用另外檢查。這就是 Bellman-Ford。</span>

**Step 4：第 2 輪在更新的時候，讀到的資料應該是誰？**

*if round two reads a value round two just wrote, that path used two edges in one round.*

<span class="spoiler">必須讀第 1 輪結束時的快照。要是直接讀當下的陣列，同一輪裡前面剛更新的值會被後面用掉，一輪就走了兩條邊，輪數再也管不住邊數。</span>

想完再往下看 code。

---

## 解法一：暴力 DFS

從起點開始，每條航線都試，累加價格，停靠數超過 k 就回頭。

```typescript
function findCheapestPrice(n: number, flights: number[][], src: number, dst: number, k: number): number {
    const adj: [number, number][][] = Array.from({ length: n }, () => []);
    for (const [from, to, price] of flights) {
        adj[from].push([to, price]);
    }

    let best = Infinity;

    function dfs(city: number, stops: number, cost: number): void {
        if (cost >= best) return;      // 已經比目前最好的貴，不用往下
        if (city === dst) {
            best = cost;
            return;
        }
        if (stops > k) return;         // 停靠額度用完

        for (const [next, price] of adj[city]) {
            dfs(next, stops + 1, cost + price);
        }
    }

    dfs(src, 0, 0);
    return best === Infinity ? -1 : best;
}
```

- Time: $O(n^{k})$ — 每一層最多 n 個分支，最深走 k+1 層
- Space: $O(k)$ — call stack 深度

**走一遍。** 同一組資料，鄰接表是 `adj[0] = [(1,100)]`、`adj[1] = [(2,100), (3,600)]`、`adj[2] = [(0,100), (3,200)]`。每次呼叫進來依序做三個檢查：`cost >= best`、`city === dst`、`stops > k`。

```
dfs(city=0, stops=0, cost=0)             best=∞
│    cost 0 沒超過 ∞、0 不是終點、stops 0 ≤ k 1 → 展開唯一的出邊
└─ dfs(city=1, stops=1, cost=100)        best=∞
   │    cost 100 沒超過 ∞、1 不是終點、stops 1 ≤ k 1 → 展開兩條出邊
   ├─ dfs(city=2, stops=2, cost=200)     best=∞
   │      cost 沒超、2 不是終點、stops 2 > k 1 → return
   │      2 → 3 那條 200 元的邊連看都沒看到
   └─ dfs(city=3, stops=2, cost=700)     best=∞
          cost 沒超、city === dst → best = 700，return

回傳 700
```

`best` 從頭到尾都是 ∞，`cost >= best` 這個檢查在這組資料上一次都沒生效。要它發揮作用，得先有一條路徑真的抵達終點，後面比較貴的分支才有得比。

`dfs(2, ...)` 那一支在檢查 `stops > k` 之前先檢查了 `city === dst`，所以順序有差：先判斷有沒有到終點，再判斷額度。倒過來寫，三個 example 分別回傳 `-1`、`500`、`-1`，全錯。

**為什麼順序有差。** `stops` 數的是已經飛了幾段，所以 `stops > k` 的意思是「不能再飛下一段了」，不是「這個狀態不合法」。`k = 1` 代表最多飛兩段，用兩段抵達 dst 完全合法，而 `dfs(3, stops=2, cost=700)` 進來的時候 `stops` 已經是 2。檢查放前面，它在 `best` 被寫進去之前就 return 了。

兩種改法讓順序不再有影響。第一種是把額度檢查搬到展開的位置，兩個檢查就不在同一條路上：

```typescript
if (city === dst) {
    best = cost;
    return;
}
if (stops <= k) {                  // 還飛得動才展開
    for (const [next, price] of adj[city]) {
        dfs(next, stops + 1, cost + price);
    }
}
```

第二種是留在原位，但門檻改成 `stops > k + 1`，因為合法狀態的上限是 `k + 1` 段。這樣也對，代價是多下一層才被判定超額，Example 1 的呼叫次數從 4 次變成 6 次。

**問題是分支太多，而且擋不擋得住要看運氣。** 兩種 `n = 10` 的輸入實際數節點數 [SRC: 本機 node]：

```
完全有向圖、價格隨機、dst 到得了       523 個節點
0..8 完全有向圖、dst 沒有任何入邊      1,227,133,513 個節點
```

差六個數量級，因為第二種讓 `best` 永遠是 ∞，`cost >= best` 一次都不生效，整棵樹得走完。同一種輸入 `n = 6` 是 5,461 個節點、`n = 8` 是 2,015,539 個，每多一個城市乘上將近 8 倍。題目上限是 100。

剪枝救不了最壞情況，這是暴力解不能交的理由：它的執行時間取決於能不能早點找到一條便宜的路，而那不是輸入保證的。

---

## 解法二：Bellman-Ford

暴力解重複走了同一段路很多次。0 → 1 這條邊，在每一條經過 1 的路徑裡都被重算一遍。

先問一個比原題小的問題：從 0 出發，只准飛一段，到得了哪些城市、各多少錢？

看 0 的出邊就答得出來。把答案寫成一張表：

```
用 1 段     0 → 0 元    1 → 100 元    2 → 到不了    3 → 到不了
```

再問下一題：只准飛兩段呢？這次不必從 0 重新出發。飛兩段就是先飛一段到某個城市，再從那裡飛一段，而「飛一段到得了哪、多少錢」上面那張表已經算完了。所以拿上一張表，每個有值的城市各飛一條出邊：

```
從 1（100 元）飛 1→2 的 100 元   →   2 要 200 元
從 1（100 元）飛 1→3 的 600 元   →   3 要 700 元

用 2 段     0 → 0 元    1 → 100 元    2 → 200 元    3 → 700 元
```

`k = 1` 代表最多停 1 次，也就是最多飛兩段，所以算到「兩段的表」就停，答案是這張表的 `dst` 格，700。邊數的上限由輪數決定，不必像 DFS 那樣每次呼叫檢查一次。

**為什麼可以丟掉「路徑」只留一個數字。** 同一個城市可能有好幾條路走到，但從這個城市接下去要花多少，跟怎麼走過來的無關，只跟目前累積了多少錢有關。所以每個城市只留最便宜的那個數字就夠。DFS 記的是一整條路徑，這裡記的只有 n 個數字。

翻成 code，`cheapest` 就是那張表。跑 `k + 1` 輪，第 i 輪產出「用不超過 i 段抵達各城市最便宜多少」。`prev` 是上一輪的表，內圈掃過每一條邊，看拿上一輪的值再飛這一段，會不會比現在的數字便宜。

```typescript
function findCheapestPrice(n: number, flights: number[][], src: number, dst: number, k: number): number {
    const cheapest = new Array(n).fill(Infinity);
    cheapest[src] = 0;

    for (let round = 0; round <= k; round++) {
        const prev = [...cheapest];        // 這一輪只准讀上一輪那張表
        for (const [from, to, price] of flights) {
            if (prev[from] === Infinity) continue;   // 上一張表裡到不了，飛不出去
            if (prev[from] + price < cheapest[to]) {
                cheapest[to] = prev[from] + price;
            }
        }
    }

    return cheapest[dst] === Infinity ? -1 : cheapest[dst];
}
```

- Time: $O(k \times E)$ — k+1 輪，每輪掃過所有邊
- Space: $O(n)$ — 兩個長度 n 的陣列

**走一遍** [SRC: 本機 node]。內圈照 `flights` 原本的順序掃，也就是 `0→1`、`1→2`、`2→0`、`1→3`、`2→3`，起始 `cheapest = [0, ∞, ∞, ∞]`。

第 1 輪，`prev = [0, ∞, ∞, ∞]`：

| 邊 | prev[from] | prev[from] + price | 當下的 cheapest[to] | 動作 | cheapest |
|---|---|---|---|---|---|
| `0→1` 100 | 0 | 100 | ∞ | 更新 | `[0, 100, ∞, ∞]` |
| `1→2` 100 | ∞ | — | ∞ | 跳過 | `[0, 100, ∞, ∞]` |
| `2→0` 100 | ∞ | — | 0 | 跳過 | `[0, 100, ∞, ∞]` |
| `1→3` 600 | ∞ | — | ∞ | 跳過 | `[0, 100, ∞, ∞]` |
| `2→3` 200 | ∞ | — | ∞ | 跳過 | `[0, 100, ∞, ∞]` |

第二列就看得到快照在做什麼。`cheapest[1]` 上一列才被寫成 100，可是 `prev[1]` 仍然是 ∞，所以 `1→2` 這一輪不能走。

第 2 輪，`prev = [0, 100, ∞, ∞]`：

| 邊 | prev[from] | prev[from] + price | 當下的 cheapest[to] | 動作 | cheapest |
|---|---|---|---|---|---|
| `0→1` 100 | 0 | 100 | 100 | 不比 100 便宜，不動 | `[0, 100, ∞, ∞]` |
| `1→2` 100 | 100 | 200 | ∞ | 更新 | `[0, 100, 200, ∞]` |
| `2→0` 100 | ∞ | — | 0 | 跳過 | `[0, 100, 200, ∞]` |
| `1→3` 600 | 100 | 700 | ∞ | 更新 | `[0, 100, 200, 700]` |
| `2→3` 200 | ∞ | — | 700 | 跳過 | `[0, 100, 200, 700]` |

`k = 1`，兩輪跑完就結束，答案 `cheapest[3] = 700`。

最後一列的 `2→3` 就是 400 那條路徑的最後一段。`prev[2]` 是 ∞ 所以它進不來，因為走到 2 已經用掉兩條邊，再接一段就是第三條。

**`prev` 這個快照不能省。** 「拿上一張表」是字面上的意思：直接讀當下的 `cheapest`，讀到的可能是這一輪剛寫進去的值，那個值已經用掉 i 段，再接一條邊就變 i+1 段。同一組資料會答錯，第 1 輪直接讀 `cheapest` 的結果是：

| 邊 | cheapest[from] | 算出來 | 動作 | cheapest |
|---|---|---|---|---|
| `0→1` 100 | 0 | 100 | 更新 | `[0, 100, ∞, ∞]` |
| `1→2` 100 | 100（上一列剛寫的） | 200 | 更新 | `[0, 100, 200, ∞]` |
| `2→0` 100 | 200（上一列剛寫的） | 300 | 不比 0 便宜，不動 | `[0, 100, 200, ∞]` |
| `1→3` 600 | 100 | 700 | 更新 | `[0, 100, 200, 700]` |
| `2→3` 200 | 200 | 400 | 更新 | `[0, 100, 200, 400]` |

第 2 輪五條邊掃完，沒有任何一條更新得動，回傳 400。

一輪之內從 0 連走了 `0→1`、`1→2`、`2→3` 三條邊，輪數就限制不了邊數了。Example 3 也會錯，`k = 0` 卻回傳 200 而不是 500。

<details>
<summary>Go 版本</summary>

```go
func findCheapestPrice(n int, flights [][]int, src int, dst int, k int) int {
    const INF = math.MaxInt32

    cheapest := make([]int, n)
    for i := range cheapest {
        cheapest[i] = INF
    }
    cheapest[src] = 0

    for round := 0; round <= k; round++ {
        prev := make([]int, n)
        copy(prev, cheapest) // 這一輪只准讀上一輪那張表

        for _, f := range flights {
            from, to, price := f[0], f[1], f[2]
            if prev[from] == INF { // 上一張表裡到不了，飛不出去
                continue
            }
            if prev[from]+price < cheapest[to] {
                cheapest[to] = prev[from] + price
            }
        }
    }

    if cheapest[dst] == INF {
        return -1
    }
    return cheapest[dst]
}
```

</details>

<details>
<summary>暴力 DFS 的 Go 版本</summary>

```go
func findCheapestPrice(n int, flights [][]int, src int, dst int, k int) int {
    type edge struct{ to, price int }

    adj := make([][]edge, n)
    for _, f := range flights {
        adj[f[0]] = append(adj[f[0]], edge{f[1], f[2]})
    }

    best := math.MaxInt32

    var dfs func(city, stops, cost int)
    dfs = func(city, stops, cost int) {
        if cost >= best {
            return
        }
        if city == dst {
            best = cost
            return
        }
        if stops > k {
            return
        }
        for _, e := range adj[city] {
            dfs(e.to, stops+1, cost+e.price)
        }
    }

    dfs(src, 0, 0)
    if best == math.MaxInt32 {
        return -1
    }
    return best
}
```

</details>

---

**Overthinking**

**負價格的航線呢？** 題目保證 `1 <= price`，所以不會有。真要有負權重，Dijkstra 就不能用，因為它靠「先出 queue 的一定最便宜」這個前提，負邊會推翻它。Bellman-Ford 可以處理負權重，那正是它存在的理由。

**這題有沒有更快的解法？** 有，把狀態設成「城市 + 已用停靠數」丟進 priority queue，用 Dijkstra 的方式只展開有機會的狀態，同一張圖只 pop 1,164 次。代價是一般的 Dijkstra「每個城市只處理一次」在這題不成立，得另外記「用最少幾次停靠到過這裡」來剪枝，狀態設計比 Bellman-Ford 容易寫錯。面試先寫 Bellman-Ford，被追問再講這個。

**k 跟停靠數的關係。** 題目說的是「停靠次數」，不是「航段數」。`k = 1` 代表最多停 1 次中間城市，也就是最多飛 2 段。code 裡 `stops` 從 0 開始、每飛一段加 1，所以判斷寫 `stops > k`。Example 3 的 `k = 0` 就是直飛。

---

## 解法比較表

| 解法 | Time | Space | n=100 秒數 | 備註 |
|---|---|---|---|---|
| 暴力 DFS | $O(n^k)$ | $O(k)$ | 跑不完 | 最壞情況 n=10 就要 123 秒 |
| Bellman-Ford | $O(k \times E)$ | $O(n)$ | 0.05 秒 | 面試要的答案，好寫好解釋 |

秒數是實際數操作次數再除以 $10^7$。Bellman-Ford 那組是 `n = 100`、4950 條邊（題目給的邊數上限 `n(n-1)/2`）、`k = 99`，數出 505,000 次：100 輪，每輪掃 4950 條邊再複製一次長度 100 的陣列。換一張同樣 4950 條邊的隨機圖，數字一模一樣，因為它不管圖長什麼樣都掃滿 k+1 輪。暴力 DFS 量到 `n = 10` 的最壞情況就是 12.3 億個節點，`n = 100` 沒有意義。

兩個解法跑 20000 組隨機圖對拍，答案完全一致。

---

## 結論

限制在邊數上，就按邊數分輪。Bellman-Ford 跑 `k + 1` 輪，每輪讀上一輪的快照，邊數上限自動被輪數管住。快照不能省，省掉就變成一輪走好幾條邊，Example 1 直接從 700 錯成 400。
