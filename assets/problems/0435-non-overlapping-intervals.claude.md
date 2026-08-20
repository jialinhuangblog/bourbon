會議室只有一間，桌上一疊預約單，每張寫著開始跟結束時間。有些時段撞在一起，只能取消掉一些。**最少要取消幾張？**

```
四張預約單

[1,2]  [2,3]  [3,4]  [1,3]

時間軸
1----2----3----4
|--A--|                A = [1,2]
      |--B--|          B = [2,3]
            |--C--|    C = [3,4]
|-------D-------|      D = [1,3]   跟 A、B 都撞到

取消 D，剩下 A、B、C 互不重疊
答案 1
```

碰在同一個點不算重疊，`[1,2]` 跟 `[2,3]` 可以並存。

翻成 code 的講法：給一堆區間，求最少移除幾個，讓剩下的兩兩不重疊。

---

**解題引導**

拿 `[[1,2],[2,3],[3,4],[1,3]]` 想。

**Step 1：「移除最少」可以換個問法嗎？**

*four minus how many i keep. minimizing removals is maximizing keeps.*

<span class="spoiler">移除最少等於保留最多。總數固定，所以求「最多能留幾個互不重疊的」，答案就是總數減掉它。這一步把問題翻成活動選擇。</span>

**Step 2：想保留最多，第一個該留哪一個？**

*whichever ends soonest leaves the most room behind it.*

<span class="spoiler">留結束最早的那個。它佔用的時間軸最短，後面剩下的空間最大。這是貪心規則。</span>

**Step 3：這個規則要怎麼確定是對的？**

*swap the first pick of the best answer with mine. does anything break?*

<span class="spoiler">交換論證。假設最佳解第一個留 A，我的貪心留 B（結束最早），所以 B 的結束時間不會晚於 A。把 A 換成 B，後面那些原本接在 A 之後的，照樣接得上 B。數量沒變，所以貪心不會比較差。</span>

**Step 4：按開始時間排序也是貪心，為什麼不行？**

*one long interval sitting at the front eats everything.*

<span class="spoiler">開始得早不代表結束得早。`[[1,100],[2,3],[3,4]]` 按開始排序會先留 `[1,100]`，後面兩個都撞到，只留 1 個；按結束排序留 `[2,3]` 跟 `[3,4]`，留 2 個。同樣是貪心，規則挑錯就輸。</span>

想完再往下看 code。

---

## 解法一：DP，最長不重疊鏈

先按開始時間排序，然後問每個區間：「以我結尾的話，最多能串幾個？」

`f[i]` 記「以第 i 個區間結尾的最長不重疊鏈長度」。往前找所有結束時間不晚於我開始時間的 `j`，取最大的 `f[j] + 1`。

```typescript
function eraseOverlapIntervals(intervals: number[][]): number {
    intervals.sort((a, b) => a[0] - b[0]);
    const n = intervals.length;

    const f = new Array(n).fill(1);   // f[i] = 以 i 結尾的最長鏈
    let best = 1;

    for (let i = 1; i < n; i++) {
        for (let j = 0; j < i; j++) {
            // j 結束的時候我才開始，接得上
            if (intervals[j][1] <= intervals[i][0]) {
                f[i] = Math.max(f[i], f[j] + 1);
            }
        }
        best = Math.max(best, f[i]);
    }

    return n - best;
}
```

- Time: $O(n^2)$ — 每個 i 都往前掃一遍
- Space: $O(n)$ — `f` 陣列

**走一遍。** 按開始時間排序後是 `[[1,2],[1,3],[2,3],[3,4]]`：

| i | 區間 | 接得上的前面那些 | f[i] |
|---|---|---|---|
| 0 | `[1,2]` | 沒有 | 1 |
| 1 | `[1,3]` | 沒有（`[1,2]` 結束在 2，但我 1 就開始了） | 1 |
| 2 | `[2,3]` | `[1,2]` 的 f=1 | 2 |
| 3 | `[3,4]` | `[1,2]` f=1、`[1,3]` f=1、`[2,3]` f=2 | 3 |

最長鏈 3，總共 4 個，移除 `4 - 3 = 1`。

**問題是 $O(n^2)$。** 題目 `intervals.length <= 10^5`，內圈次數約 $n^2 / 2 = 5 \times 10^9$，換算約 500 秒，明確 TLE。

<details>
<summary>Go 版本</summary>

```go
func eraseOverlapIntervals(intervals [][]int) int {
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })

    n := len(intervals)
    f := make([]int, n)
    best := 0

    for i := 0; i < n; i++ {
        f[i] = 1
        for j := 0; j < i; j++ {
            // j 結束的時候我才開始，接得上
            if intervals[j][1] <= intervals[i][0] && f[j]+1 > f[i] {
                f[i] = f[j] + 1
            }
        }
        if f[i] > best {
            best = f[i]
        }
    }

    return n - best
}
```

</details>

---

## 解法二：Greedy，按結束時間排序

DP 每一格都往前掃，是因為它不知道哪個前輩最好，只好全部問一次。

換成按**結束時間**排序，就不用問了。從左到右掃，只要開始時間不早於上一個留下來的結束時間，就留；否則移除。

```typescript
function eraseOverlapIntervals(intervals: number[][]): number {
    intervals.sort((a, b) => a[1] - b[1]);   // 照結束時間排，早結束的先

    let keep = 0;
    let lastEnd = -Infinity;                 // 上一個留下來的結束時間

    for (const [start, end] of intervals) {
        if (start >= lastEnd) {              // 跟上一個不重疊
            keep++;
            lastEnd = end;
        }
    }

    return intervals.length - keep;
}
```

- Time: $O(n \log n)$ — 排序支配，掃描只有 $O(n)$
- Space: $O(1)$ — 只有兩個變數

**走一遍。** 按結束時間排序後是 `[[1,2],[2,3],[1,3],[3,4]]`：

| 區間 | start vs lastEnd | 動作 | lastEnd | keep |
|---|---|---|---|---|
| `[1,2]` | 1 ≥ -∞ | 留下 | 2 | 1 |
| `[2,3]` | 2 ≥ 2 | 留下 | 3 | 2 |
| `[1,3]` | 1 < 3 | 移除 | 3 | 2 |
| `[3,4]` | 3 ≥ 3 | 留下 | 4 | 3 |

留下 3 個，移除 `4 - 3 = 1`。

`[2,3]` 那一列的 `2 >= 2` 成立，因為題目說碰在同一點不算重疊。寫成 `>` 的話 `[1,2]` 跟 `[2,3]` 會被當成撞到，Example 3 就會回傳 1 而不是 0。

**為什麼是結束時間，不是開始時間。** 兩個規則都符合「挑當下最好的」，但只有一個站得住：

```
[[1,100], [2,3], [3,4]]

按結束時間排序   [[2,3], [3,4], [1,100]]
                 留 [2,3]、[3,4]        移除 1 個   ← 正確

按開始時間排序   [[1,100], [2,3], [3,4]]
                 留 [1,100]             移除 2 個   ← 錯
```

`[1,100]` 開始得最早，但它一個人佔掉整條時間軸。開始早跟結束早是兩件不同的事，而剩下的空間由結束時間決定。

證明用交換論證：假設最佳解留了 k 個，第一個是 A。貪心留的是結束最早的 B，所以 B 的結束時間不會晚於 A。把最佳解裡的 A 換成 B，後面那 k-1 個原本都在 A 結束之後才開始，B 結束得更早，所以照樣塞得下。換完還是 k 個，沒有變差。同樣的論證套到第二個、第三個，一路下去。

詳細的推導在 [Greedy](/concept/greedy)。

<details>
<summary>Go 版本</summary>

```go
func eraseOverlapIntervals(intervals [][]int) int {
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][1] < intervals[j][1] // 照結束時間排
    })

    keep := 0
    lastEnd := -1 << 62 // 比任何 start 都小

    for _, iv := range intervals {
        if iv[0] >= lastEnd { // 跟上一個不重疊
            keep++
            lastEnd = iv[1]
        }
    }

    return len(intervals) - keep
}
```

</details>

---

**Overthinking**

**跟 [56 Merge Intervals](/problem/merge-intervals) 差在哪。** 56 要把重疊的合併成一段，所以按**開始**時間排序，一路往右擴 `end`。435 要從重疊的裡面挑掉幾個，所以按**結束**時間排序。同一堆區間，問法不同，排序的 key 就不同。判準是「答案跟區間的哪一端有關」：合併看的是起點連不連得起來，挑選看的是終點留多少空間。

**`[1,2]` 跟 `[2,3]` 到底算不算重疊。** 題目明說不算。這條規則只影響一個字元：`start >= lastEnd` 還是 `start > lastEnd`。Example 3 就是專門測這個的，`[[1,2],[2,3]]` 答案是 0。

**同樣是區間題，什麼時候該用 DP？** 這題的區間沒有權重，每個都一樣重要，所以求「最多留幾個」就夠，greedy 成立。如果每個區間帶一個分數、要求總分最高（Weighted Interval Scheduling），greedy 就不成立了，得回去用 DP 加二分搜尋，$O(n \log n)$。差別在「數量」換成「加權總和」之後，結束最早的那個不一定屬於最佳解。

---

## 解法比較表

| 解法 | Time | Space | n=10^5 秒數 | 備註 |
|---|---|---|---|---|
| DP 最長鏈 | $O(n^2)$ | $O(n)$ | 約 500 秒 | TLE，但推導直觀，可以當起點講 |
| Greedy 按結束排序 | $O(n \log n)$ | $O(1)$ | 0.16 秒 | 面試要的答案 |

秒數是實際數操作次數再除以 $10^7$。Greedy 在 `n = 10^5` 實測排序比較 1,530,317 次加掃描 100,000 次，合計 1,630,317 次。DP 的內圈約 $n^2/2 = 5 \times 10^9$，這個規模跑不動，用公式估。

暴力列舉所有子集合是 $O(2^n)$，`n = 10^5` 沒有意義，所以沒有列進表。驗證的時候有寫一份跑小輸入，跟 DP、greedy 三方對拍 20000 組隨機區間，答案完全一致。

---

## 結論

移除最少等於保留最多，翻過去就是活動選擇。按結束時間排序，掃一遍，開始時間不早於上一個的結束時間就留下。排序的 key 選錯就錯，而 `[[1,100],[2,3],[3,4]]` 這組會告訴你選錯了。
