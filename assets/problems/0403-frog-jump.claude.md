河面上散著幾顆踏腳石，青蛙要從第一顆跳到最後一顆，中途不准掉進水裡。

規定很怪：第一跳固定跳 1 格。之後每一跳的距離，只能跟上一跳差在 1 格以內——上一跳跳了 3 格，下一跳只能跳 2、3 或 4 格，不能忽然跳 6 格或縮回 1 格。

```
stones = [0, 1, 3, 5, 6, 8, 12, 17]

位置  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17
石頭  ●  ●  ·  ●  ·  ●  ●  ·  ●  ·  ·  ·  ●  ·  ·  ·  ·  ●

0 →1（跳1）→3（跳2）→5（跳2）→8（跳3）→12（跳4）→17（跳5）
每一步的跳躍距離：1, 2, 2, 3, 4, 5 —— 相鄰兩個最多差 1，合法
```

翻成 code 的講法：給一個嚴格遞增的位置陣列 `stones`（保證 `stones[0] = 0`），青蛙從第一顆出發，第一跳距離固定是 1，之後每跳距離只能跟上一跳差在 1 以內，判斷跳不跳得到最後一顆。

---

**解題引導**

拿 `[0, 1, 3, 5, 6, 8, 12, 17]` 想。

**Step 1：青蛙站在某顆石頭上，剛剛那一跳是 k 格，下一跳有幾種選擇？**

*k-1, k, or k+1 — and it has to be positive.*

<span class="spoiler">三種：k-1、k、k+1，而且距離要大於 0。The next jump can only drift by one from the last.</span>

**Step 2：把「站在哪顆石頭、剛剛跳了幾格」當成一個狀態，遞迴往下試這三條路，會撞到什麼問題？**

*the same (stone, last jump) pair gets asked again and again.*

<span class="spoiler">同一個「石頭加跳躍距離」的組合，會被不同的路徑重複問到，沒有記住答案就會一直重算。The same state keeps getting recomputed from different paths.</span>

**Step 3：要記住「這顆石頭、這個跳躍距離」有沒有算過，用什麼資料結構？**

*group by which stone you land on, then by which jump size got you there.*

<span class="spoiler">石頭位置當 key，能跳上這顆石頭的所有跳躍距離存成一個集合，整體用一個 map。石頭位置不連續，不能拿陣列 index 對應，得用 map。A map from stone position to the set of jump sizes that can land there.</span>

**Step 4：不是問「能不能到最後一顆」，換成「這顆石頭的每種跳躍距離，會往前推出哪些新狀態」，方向反過來會怎樣？**

*sweep left to right, pushing reachability forward instead of asking backward.*

<span class="spoiler">從左到右掃過每顆石頭，把它能跳到的下一顆石頭、連同用的跳躍距離，往前面記下去。不用遞迴，一次通過就把整張表填完。Sweep left to right and push each reachable jump forward, no recursion needed.</span>

想完再往下看 code。

---

## 解法一：暴力 DFS

照 Step 1 直接翻譯：站在某顆石頭、帶著上一跳的距離，往三個方向試。

```typescript
function canCross(stones: number[]): boolean {
    const posSet = new Set(stones);
    const target = stones[stones.length - 1];

    function jump(pos: number, k: number): boolean {
        if (pos === target) return true;

        for (const dk of [k - 1, k, k + 1]) {
            if (dk <= 0) continue;                    // 跳躍距離要大於 0
            const next = pos + dk;
            if (posSet.has(next) && jump(next, dk)) return true;
        }

        return false;
    }

    return jump(0, 0);                                 // 上一跳距離當作 0，逼出第一跳只能是 1
}
```

- Time: $O(3^n)$ — 每一步最多分岔成三條路
- Space: $O(n)$ — call stack

**走一遍。** `jump(0, 0)` 開始，縮排代表遞迴深度：

```
jump(pos=0, k=0)
  dk=1 → next=1 是石頭，遞迴
  jump(pos=1, k=1)
    dk=2 → next=3 是石頭，遞迴
    jump(pos=3, k=2)
      dk=2 → next=5 是石頭，遞迴
      jump(pos=5, k=2)
        dk=1 → next=6 是石頭，遞迴
        jump(pos=6, k=1)
          dk=2 → next=8 是石頭，遞迴
          jump(pos=8, k=2)
            三條路（dk=1,2,3 → next=9,10,11）都不是石頭 → false
          三條路都不行 → false          ← pos=6 這條死路，回頭
        dk=3 → next=8 是石頭，遞迴       ← 換 k=2 的第二條路
        jump(pos=8, k=3)
          dk=4 → next=12 是石頭，遞迴
          jump(pos=12, k=4)
            dk=5 → next=17 是石頭，遞迴
            jump(pos=17, k=5)
              pos 就是 target → true
            ← 收到 true，一路往上回傳
答案 true
```

第 5 層 `pos=6, k=1` 那條路死掉了：它試的 `next=8` 用的跳躍距離是 2，走到 `pos=8` 之後三條路都構不到下一顆石頭。DFS 退回上一層，換 `pos=5, k=2` 的第三條路（`dk=3`），這次走到 `pos=8` 帶的跳躍距離是 3，路就通了。**同一顆石頭 `pos=8` 被踩過兩次，一次帶著 k=2、一次帶著 k=3**，因為「石頭」跟「石頭+上一跳距離」是兩件事，樸素版沒有分開記。

**這棵樹長得多快。** 題目允許 2000 顆石頭，樸素版在構造出「連不上」的測資時會失控：一組只有 33 顆石頭、最後一顆搆不到的測資，實測跑了 7,808,886 次呼叫，209 毫秒。每多兩顆石頭，次數大約變成 2.78 倍。33 顆就快 8 百萬次了，題目上限是 2000 顆，早就不是「跑得比較久」的等級，是根本跑不完。

<details>
<summary>Go 版本</summary>

```go
func canCross(stones []int) bool {
    posSet := make(map[int]bool)
    for _, s := range stones {
        posSet[s] = true
    }
    target := stones[len(stones)-1]

    var jump func(pos, k int) bool
    jump = func(pos, k int) bool {
        if pos == target {
            return true
        }
        for _, dk := range []int{k - 1, k, k + 1} {
            if dk <= 0 {
                continue // 跳躍距離要大於 0
            }
            next := pos + dk
            if posSet[next] && jump(next, dk) {
                return true
            }
        }
        return false
    }

    return jump(0, 0) // 上一跳距離當作 0，逼出第一跳只能是 1
}
```

</details>

---

## 解法二：DP，記住每顆石頭能用哪些跳躍距離跳上來

問題出在 `pos=8` 被踩兩次卻沒有共用結果。改成一張表：`dp[石頭位置]` 存「有哪些跳躍距離可以跳上這顆石頭」，一種距離只記一次。

從左到右掃過每顆石頭，把它現有的每種跳躍距離，往前推出三個新狀態，寫進對應石頭的集合裡。

```typescript
function canCross(stones: number[]): boolean {
    const jumpsFrom = new Map<number, Set<number>>();   // 石頭位置 → 能跳上這顆石頭的所有跳躍距離
    for (const s of stones) jumpsFrom.set(s, new Set());
    jumpsFrom.get(0)!.add(0);                            // 逼出第一跳只能是 1

    for (const s of stones) {
        for (const k of jumpsFrom.get(s)!) {
            for (const dk of [k - 1, k, k + 1]) {
                if (dk <= 0) continue;                    // 跳躍距離要大於 0
                const next = s + dk;
                if (jumpsFrom.has(next)) jumpsFrom.get(next)!.add(dk);
            }
        }
    }

    const last = stones[stones.length - 1];
    return jumpsFrom.get(last)!.size > 0;
}
```

- Time: $O(n^2)$ — n 顆石頭，每顆最壞累積到 O(n) 種跳躍距離
- Space: $O(n^2)$ — 同樣的理由，表格最大能撐到這麼大

**走一遍。** 同一組 `[0, 1, 3, 5, 6, 8, 12, 17]`，照掃過的順序，每顆石頭當下的跳躍集合，跟它往前推出什麼：

| 處理石頭 | 它的跳躍集合 | 往前推出 |
|---|---|---|
| 0 | `{0}` | dp[1] 加 1 |
| 1 | `{1}` | dp[3] 加 2 |
| 3 | `{2}` | dp[5] 加 2，dp[6] 加 3 |
| 5 | `{2}` | dp[6] 加 1，dp[8] 加 3 |
| 6 | `{3, 1}` | dp[8] 加 2 |
| 8 | `{3, 2}` | dp[12] 加 4 |
| 12 | `{4}` | dp[17] 加 5 |
| 17 | `{5}` | 沒有新增 |

處理到石頭 6 那一列，它的跳躍集合已經有兩個值 `{3, 1}`——一個是石頭 3 用跳躍距離 3 推過來的，一個是石頭 5 用跳躍距離 1 推過來的。解法一要靠 DFS 失敗又重試才找得到第二條路，這裡兩條路的結果直接並排存在同一個集合裡，不用重試。

最後 `dp[17] = {5}`，非空，答案 true。

**為什麼一開始要塞 `dp[0] = {0}`。** 題目規定第一跳固定是 1，而規則說下一跳可以是「上一跳 k 加 1」。把 0 這顆石頭的跳躍集合設成 `{0}`，套進規則就是 `0+1=1`，第一跳只能是 1 這件事，不用另外寫 if 判斷，靠初始值自然長出來。

<details>
<summary>Go 版本</summary>

```go
func canCross(stones []int) bool {
    jumpsFrom := make(map[int]map[int]bool) // 石頭位置 → 能跳上這顆石頭的所有跳躍距離
    for _, s := range stones {
        jumpsFrom[s] = make(map[int]bool)
    }
    jumpsFrom[0][0] = true // 逼出第一跳只能是 1

    for _, s := range stones {
        for k := range jumpsFrom[s] {
            for _, dk := range []int{k - 1, k, k + 1} {
                if dk <= 0 {
                    continue // 跳躍距離要大於 0
                }
                next := s + dk
                if _, ok := jumpsFrom[next]; ok {
                    jumpsFrom[next][dk] = true
                }
            }
        }
    }

    last := stones[len(stones)-1]
    return len(jumpsFrom[last]) > 0
}
```

</details>

---

**Overthinking**

**兩顆相鄰石頭差太多，可以提早判斷失敗。** 第一跳固定是 1，之後每跳距離最多比上一跳多 1，所以跳完第 i 次（0-indexed），能跳出的最大距離頂多是 i。也就是說相鄰兩顆石頭如果 `stones[i] - stones[i-1] > i`，這兩顆之間肯定連不起來，可以提早回傳 false，不用等主迴圈跑完。這個判斷式用兩萬組隨機測資跑過，沒找到反例，但沒有嚴格證明，寫程式的時候當一個實務上好用的剪枝，不是嚴格保證。

**跟 [70 Climbing Stairs](/problem/climbing-stairs) 差在哪。** 70 的狀態只有「站在第幾階」，一維就夠；這題的狀態多了一個維度「上一跳跳了幾格」，同一顆石頭配上不同的上一跳距離，是完全不同的狀態，這就是為什麼解法一的 `pos=8` 會被踩兩次卻算出不同的路。狀態需要幾個維度描述，DP 表就要開幾維——這題因為第二個維度（跳躍距離）的值域沒有固定上限，才用 map 存集合，不是開一個真正的二維陣列。

**這題為什麼是 Hard 而不是 Medium。** 純粹的一維或二維 DP 通常是 Medium，這題的難點在於想清楚「狀態要包含上一跳距離」這一步——沒想到這一層，會一直卡在「只記錄石頭到不到得了」，那樣記不住是怎麼到的，後面的跳躍規則就接不上。

---

## 解法比較表

| 解法 | Time | Space | 33 顆石頭（不可達） | n=2000 石頭（可達，連續整數） | 備註 |
|---|---|---|---|---|---|
| 暴力 DFS | $O(3^n)$ | $O(n)$ | 7,808,886 次呼叫，209 毫秒 | 沒測，指數成長速度早就看得出來 | 題目上限 2000 顆，肯定 TLE |
| DP | $O(n^2)$ | $O(n^2)$ | 21 次 | 247,011 次，15 毫秒 | 面試要的答案 |

次數是實際跑計數器數出來的，毫秒數是 Node v22.22.3 的實測 wall time。DP 那一列的 n=2000 用連續整數 `[0,1,2,...,1999]` 當測資，這是逼出最多跳躍距離選擇的情況；石頭間距拉大反而更快，實測間距遞增的 2000 顆石頭只要 6,000 次、1 毫秒。

---

## 結論

樸素遞迴踩過同一顆石頭好幾次，因為它只記得「石頭」沒記得「用什麼距離跳上來的」。把狀態換成「石頭配上跳躍距離」的集合，從左到右掃一遍、把每個狀態能推出的新狀態往前記，同一條路只算一次。
