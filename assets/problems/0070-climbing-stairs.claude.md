樓梯有 n 階。腳只有兩種踏法：踏一階，或者跨兩階。

問從地面爬到頂端，有幾種不同的走法。

```
n = 3

1 + 1 + 1
1 + 2
2 + 1

三種
```

`1 + 2` 跟 `2 + 1` 算兩種，因為腳步的順序不一樣。

翻成 code 的講法：每一步能走 1 或 2，求走到第 n 階的方法數。

---

**解題引導**

拿 n = 5 想。

**Step 1：現在人站在第 5 階，上一步是從哪裡踏上來的？**

*only two places it could have come from.*

<span class="spoiler">只有兩個可能：從第 4 階踏一階上來，或從第 3 階跨兩階上來。沒有第三種。Either from step 4 or from step 3.</span>

**Step 2：所以走到第 5 階的方法數，怎麼算？**

<span class="spoiler">走到第 4 階的方法數，加上走到第 3 階的方法數。兩群走法的最後一步不同，不會重複。f(5) = f(4) + f(3)。Add the two, they never overlap.</span>

**Step 3：這兩群為什麼一定不會重複？**

*the last footstep tells them apart.*

<span class="spoiler">因為最後一步不一樣。從第 4 階上來的那群，最後一步都是 1；從第 3 階上來的那群，最後一步都是 2。同一條走法不可能同時屬於兩群。The final step differs, so the sets are disjoint.</span>

**Step 4：直接照這個式子寫遞迴，會發生什麼事？**

<span class="spoiler">f(5) 要 f(4) 跟 f(3)，f(4) 又要 f(3)。f(3) 被算了兩次，而且愈往下重複愈多。n=45 的時候要呼叫 22 億次。The same subproblem gets recomputed exponentially many times.</span>

想完再往下看 code。

---

## 解法一：照著式子寫遞迴

```typescript
function climbStairs(n: number): number {
    if (n <= 2) return n;   // 1 階一種走法，2 階兩種
    return climbStairs(n - 1) + climbStairs(n - 2);
}
```

- Time: $O(2^n)$ — 每一層分岔成兩個呼叫
- Space: $O(n)$ — call stack 最深 n 層

**走一遍。** n = 5 的呼叫樹：

```
climbStairs(5)
├── climbStairs(4)
│   ├── climbStairs(3)
│   │   ├── climbStairs(2) → 2
│   │   └── climbStairs(1) → 1
│   │   → 3
│   └── climbStairs(2) → 2
│   → 5
└── climbStairs(3)          ← 又算了一次
    ├── climbStairs(2) → 2
    └── climbStairs(1) → 1
    → 3
→ 8
```

`climbStairs(3)` 出現兩次，`climbStairs(2)` 出現三次。n 才 5 就這樣，總共 9 次呼叫。

**這棵樹長得多快。** 呼叫次數的遞推式是 $T(n) = T(n-1) + T(n-2) + 1$。n=10 是 109 次，看起來還好。n=45 是 2,269,806,339 次。題目的上限剛好就是 45，所以這一版寫出來一定 TLE。

秒數這裡要講清楚。照這個站上一貫的 $10^7$ 次除法算是 227 秒，但實際拿 Node v22 跑 `climbStairs(45)` 只花 6.9 秒。差 33 倍，因為這個函式的迴圈體只有一次加法跟兩次比較，JIT 編完之後每秒能跑到 $3 \times 10^8$ 次。$10^7$ 那個換算是給「一次迭代做幾件事」的一般情況用的，遇到這種極簡的遞迴會高估很多。兩個數字都超過 LeetCode 的時限，結論不變。

<details>
<summary>Go 版本</summary>

```go
func climbStairs(n int) int {
    if n <= 2 {
        return n
    }
    return climbStairs(n-1) + climbStairs(n-2)
}
```

</details>

---

## 解法二：算過的記起來

`climbStairs(3)` 不管被問幾次，答案都是 3。第一次算完就存起來，之後直接拿。

```typescript
function climbStairs(n: number): number {
    const memo = new Map<number, number>();

    function ways(k: number): number {
        if (k <= 2) return k;
        if (memo.has(k)) return memo.get(k)!;   // 算過了，直接給

        const result = ways(k - 1) + ways(k - 2);
        memo.set(k, result);
        return result;
    }

    return ways(n);
}
```

- Time: $O(n)$ — 每個 k 只有第一次會真的往下算
- Space: $O(n)$ — memo 加上 call stack

**走一遍。** 同樣 n = 5，這次記下每一格什麼時候被寫進 memo：

| 呼叫 | memo 有嗎 | 動作 | memo 內容 |
|---|---|---|---|
| ways(5) | 沒有 | 往下問 4 跟 3 | 空 |
| ways(4) | 沒有 | 往下問 3 跟 2 | 空 |
| ways(3) | 沒有 | 往下問 2 跟 1，得到 3 | `{3: 3}` |
| ways(2) | 基底 | 回傳 2 | `{3: 3}` |
| ways(4) 收齊 | | 3 + 2 = 5，寫進去 | `{3: 3, 4: 5}` |
| ways(3) | **有** | 直接回傳 3 | `{3: 3, 4: 5}` |
| ways(5) 收齊 | | 5 + 3 = 8，寫進去 | `{3: 3, 4: 5, 5: 8}` |

倒數第二列就是 memo 發揮作用的地方。解法一在那裡會重新展開整棵子樹，這裡直接回傳。

<details>
<summary>Go 版本</summary>

```go
func climbStairs(n int) int {
    memo := map[int]int{}

    var ways func(int) int
    ways = func(k int) int {
        if k <= 2 {
            return k
        }
        if v, ok := memo[k]; ok {
            return v // 算過了，直接給
        }

        result := ways(k-1) + ways(k-2)
        memo[k] = result
        return result
    }

    return ways(n)
}
```

</details>

---

## 解法三：由下往上，只留兩個變數

memo 版是從 n 往下問。反過來從小的開始算，答案自然會浮上來，而且不用遞迴。

再看一次 `f(i) = f(i-1) + f(i-2)`：算第 i 格的時候只用得到前兩格，更早的那些放著也是佔位子。

```typescript
function climbStairs(n: number): number {
    let prev = 1;      // f(0)：站在地面，一種走法（什麼都不做）
    let cur = 1;       // f(1)：踏一階

    for (let i = 2; i <= n; i++) {
        const next = prev + cur;
        prev = cur;    // 整組往右移一格
        cur = next;
    }

    return cur;
}
```

- Time: $O(n)$ — 迴圈跑 n-1 次
- Space: $O(1)$ — 兩個變數

**走一遍。** n = 5：

| i | prev（f(i-2)） | cur（f(i-1)） | next = prev + cur | 移完之後 prev, cur |
|---|---|---|---|---|
| 起始 | 1（f(0)） | 1（f(1)） | | 1, 1 |
| 2 | 1 | 1 | 2 | 1, 2 |
| 3 | 1 | 2 | 3 | 2, 3 |
| 4 | 2 | 3 | 5 | 3, 5 |
| 5 | 3 | 5 | 8 | 5, 8 |

回傳 `cur` = 8。

`f(0) = 1` 這一格看起來很怪，站在地面哪有走法可言。但它是被 `f(2) = f(1) + f(0) = 2` 逼出來的：2 階確實有兩種走法，所以 f(0) 只能是 1。把「什麼都不做」也算成一種走法，式子才會通。

<details>
<summary>Go 版本</summary>

```go
func climbStairs(n int) int {
    prev, cur := 1, 1 // f(0), f(1)

    for i := 2; i <= n; i++ {
        next := prev + cur
        prev = cur // 整組往右移一格
        cur = next
    }

    return cur
}
```

</details>

---

**Overthinking**

**這就是 Fibonacci 數列。** 從 n=1 開始算：1, 2, 3, 5, 8, 13, 21, 34。跟 Fibonacci 對齊之後 f(n) = Fib(n+1)。題目把 n 的上限訂在 45 不是隨便訂的，`climbStairs(45)` 等於 1,836,311,903，剛好還塞得進 32 bits 有號整數的上限 2,147,483,647。再多一階就會溢位。

**每一步能走 1、2 或 3 呢。** 遞推式變成 `f(i) = f(i-1) + f(i-2) + f(i-3)`，滾動變數從兩個變三個。走法的集合按「最後一步走幾階」分群，這個想法沒變，只是分成三群。

**跟 1-D DP 那幾題的關係。** 這一族的骨架都是「第 i 格只看前面一兩格」，差別在中間那個運算子：

| 題目 | 第 i 格在問什麼 | 遞推式 | 邊界 |
|---|---|---|---|
| 70 Climbing Stairs | 走到這裡有幾種走法 | `f[i] = f[i-1] + f[i-2]` | `f[0]=1, f[1]=1` |
| [746 Min Cost](/problem/min-cost-climbing-stairs) | 走到這裡最少花多少 | `f[i] = min(f[i-1]+c[i-1], f[i-2]+c[i-2])` | `f[0]=0, f[1]=0` |
| [198 House Robber](/problem/house-robber) | 搶到這裡最多拿多少 | `f[i] = max(f[i-1], f[i-2]+nums[i])` | `f[0]=nums[0]` |
| [213 House Robber II](/problem/house-robber-ii) | 同上，但頭尾算相鄰 | 跑兩次 198 | 去頭一次、去尾一次 |

70 用加法，因為它在數方法數，兩群走法要合起來。其他三題用 min 或 max，因為它們在挑最好的一條。認出這一格是「加總」還是「挑最好」，遞推式就寫得出來了。

---

## 解法比較表

| 解法 | Time | Space | n=45 操作次數 | 備註 |
|---|---|---|---|---|
| 樸素遞迴 | $O(2^n)$ | $O(n)$ | 2,269,806,339 次呼叫 | 實測 6.9 秒，TLE |
| memo | $O(n)$ | $O(n)$ | 43 次真正的計算 | 從遞迴改過來只多兩行 |
| 兩個變數 | $O(n)$ | $O(1)$ | 44 次迴圈 | 面試要的答案 |

樸素遞迴的次數是跑遞推式 $T(n) = T(n-1) + T(n-2) + 1$ 算出來的，6.9 秒是 Node v22 實際計時。後兩個都在 0.00001 秒等級，看不出差別，選滾動變數是因為它連 memo 那個 Map 都省了。

---

## 結論

站在第 i 階，上一步只可能來自 i-1 或 i-2，兩群走法的最後一步不同所以不會重複，加起來就是答案。寫成遞迴會爬 22 億次，加個 memo 或改成兩個變數往上滾，就是 45 次的事。
