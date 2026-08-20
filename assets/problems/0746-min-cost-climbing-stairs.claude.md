樓梯的每一階都裝了投幣口。站在第 i 階要繼續往上走，得先投 `cost[i]` 塊，投完可以踏一階或跨兩階。

起點自己選，第 0 階或第 1 階都行，站上去不用錢。目標是走出樓梯，總共花最少。

```
cost = [10, 15, 20]

  ↑ 頂端（第 3 格，樓梯外面）
  |
 20   第 2 階
 15   第 1 階
 10   第 0 階

從第 1 階開始，投 15，跨兩階直接出去
花 15
```

從第 0 階開始也可以：投 10 走到第 2 階，再投 20 出去，總共 30。比較貴。

翻成 code 的講法：`cost[i]` 是離開第 i 階的代價，起點是 index 0 或 1，求走到陣列外面那一格的最小總花費。

---

**解題引導**

拿 `cost = [10, 15, 20]` 想。

**Step 1：「走到頂端」是走到哪一格？**

*one past the last step. that is the whole trick.*

<span class="spoiler">第 n 格，也就是最後一階再上面一格，陣列裡沒有它。這一格不用投幣，因為人已經離開樓梯了。答案是 f[3] 不是 f[2]。The goal is index n, which sits outside the array.</span>

**Step 2：要站上第 i 格，最後一步從哪裡來？**

<span class="spoiler">從第 i-1 格踏一階，或從第 i-2 格跨兩階。跟 70 Climbing Stairs 一模一樣。Same two predecessors as Climbing Stairs.</span>

**Step 3：從第 i-1 格上來，總共花了多少？**

*getting there, plus the toll to leave there.*

<span class="spoiler">走到第 i-1 格花的錢，加上離開第 i-1 格要投的 cost[i-1]。兩條路各算一次，取比較便宜的。That is f[i-1] + cost[i-1].</span>

**Step 4：起點免費要怎麼表達？**

<span class="spoiler">f[0] = 0、f[1] = 0。站到這兩格身上都還沒花過錢，錢是離開的時候才付的。Both starting positions cost nothing to stand on.</span>

想完再往下看 code。

---

## 解法一：填一張表

`f[i]` 定義成「站上第 i 格，到目前為止花了多少」。注意是站上去，還沒投那一格的幣。

```typescript
function minCostClimbingStairs(cost: number[]): number {
    const n = cost.length;
    const f = new Array(n + 1).fill(0);   // 多一格放頂端

    for (let i = 2; i <= n; i++) {
        const fromOneBelow = f[i - 1] + cost[i - 1];   // 從下面一格踏上來
        const fromTwoBelow = f[i - 2] + cost[i - 2];   // 從下面兩格跨上來
        f[i] = Math.min(fromOneBelow, fromTwoBelow);
    }

    return f[n];
}
```

- Time: $O(n)$ — 一個迴圈
- Space: $O(n)$ — 表格

`f[0]` 跟 `f[1]` 都留 0，因為題目說這兩格都能當起點。迴圈從 2 開始，第一個要算的是 `f[2]`。

**走一遍。** `cost = [1, 100, 1, 1, 1, 100]`，表格有 7 格：

| i | f[i-1] + cost[i-1] | f[i-2] + cost[i-2] | 便宜的那個 | f[i] |
|---|---|---|---|---|
| 0 | | | 起點 | 0 |
| 1 | | | 起點 | 0 |
| 2 | 0 + 100 = 100 | 0 + 1 = 1 | 跨兩階 | 1 |
| 3 | 1 + 1 = 2 | 0 + 100 = 100 | 踏一階 | 2 |
| 4 | 2 + 1 = 3 | 1 + 1 = 2 | 跨兩階 | 2 |
| 5 | 2 + 1 = 3 | 2 + 1 = 3 | 平手，都是 3 | 3 |
| 6 | 3 + 100 = 103 | 2 + 1 = 3 | 跨兩階 | 3 |

答案 `f[6] = 3`。走法是 0 → 2 → 4 → 6，三次各投 1 塊，兩個 100 都跳過去了。

最後一列就是 Step 1 講的那件事。`f[5] = 3` 跟 `f[6] = 3` 剛好一樣，換一組數字就會不同。回傳 `f[n-1]` 是這題最常見的錯，測資 `[10,15,20]` 會回傳 10 而不是 15。

<details>
<summary>Go 版本</summary>

```go
func minCostClimbingStairs(cost []int) int {
    n := len(cost)
    f := make([]int, n+1) // 多一格放頂端

    for i := 2; i <= n; i++ {
        fromOneBelow := f[i-1] + cost[i-1] // 從下面一格踏上來
        fromTwoBelow := f[i-2] + cost[i-2] // 從下面兩格跨上來
        if fromOneBelow < fromTwoBelow {
            f[i] = fromOneBelow
        } else {
            f[i] = fromTwoBelow
        }
    }

    return f[n]
}
```

</details>

---

## 解法二：只留兩個變數

`f[i]` 只用得到 `f[i-1]` 跟 `f[i-2]`，整張表沒有留下來的必要。

```typescript
function minCostClimbingStairs(cost: number[]): number {
    let prev = 0;   // f[i-2]
    let cur = 0;    // f[i-1]

    for (let i = 2; i <= cost.length; i++) {
        const next = Math.min(cur + cost[i - 1], prev + cost[i - 2]);
        prev = cur;   // 整組往右移一格
        cur = next;
    }

    return cur;
}
```

- Time: $O(n)$
- Space: $O(1)$ — 兩個變數

**走一遍。** 同一組 `cost = [1, 100, 1, 1, 1, 100]`：

| i | prev | cur | cur + cost[i-1] | prev + cost[i-2] | next | 移完之後 prev, cur |
|---|---|---|---|---|---|---|
| 起始 | 0 | 0 | | | | 0, 0 |
| 2 | 0 | 0 | 0 + 100 = 100 | 0 + 1 = 1 | 1 | 0, 1 |
| 3 | 0 | 1 | 1 + 1 = 2 | 0 + 100 = 100 | 2 | 1, 2 |
| 4 | 1 | 2 | 2 + 1 = 3 | 1 + 1 = 2 | 2 | 2, 2 |
| 5 | 2 | 2 | 2 + 1 = 3 | 2 + 1 = 3 | 3 | 2, 3 |
| 6 | 2 | 3 | 3 + 100 = 103 | 2 + 1 = 3 | 3 | 3, 3 |

回傳 `cur` = 3，跟解法一的表格每一格都對得上。

迴圈結束的條件寫 `i <= cost.length`，最後一圈算的就是頂端那一格。寫成 `i < cost.length` 會少跑一圈，答案變成 `f[n-1]`。

<details>
<summary>Go 版本</summary>

```go
func minCostClimbingStairs(cost []int) int {
    prev, cur := 0, 0 // f[i-2], f[i-1]

    for i := 2; i <= len(cost); i++ {
        a := cur + cost[i-1]
        b := prev + cost[i-2]
        next := a
        if b < a {
            next = b
        }
        prev = cur // 整組往右移一格
        cur = next
    }

    return cur
}
```

</details>

---

**Overthinking**

**為什麼不用寫樸素遞迴那一版。** 寫得出來，而且會跟 [70 Climbing Stairs](/problem/climbing-stairs) 的解法一長得一樣：每一格分岔成兩個呼叫，同一格被重算很多次。那段推導在 70 有完整的呼叫樹跟 22 億次的實測，這題直接跳到有記憶的版本。

**cost[i] 是離開的錢，不是站上去的錢。** 題目原文寫 "Once you pay the cost, you can either climb one or two steps"，順序是先付再爬。把它讀成「踏上第 i 階要付 cost[i]」也算得出答案，只是邊界會變成 `f[0] = cost[0]`、`f[1] = cost[1]`，然後答案是 `min(f[n-1], f[n-2])`。兩種寫法都對，但混在一起就會錯。上面那版把付錢的時機統一在「離開」，只需要一個出口 `f[n]`。

**跟 1-D DP 那幾題的對照表**放在 [70](/problem/climbing-stairs) 的 Overthinking。這題跟 70 的遞推式只差一個運算子：70 把兩條路加起來（在數方法數），這題取比較便宜的（在挑最好的一條）。

---

## 解法比較表

| 解法 | Time | Space | n=1000 操作次數 | 記憶體 | 備註 |
|---|---|---|---|---|---|
| 填表 | $O(n)$ | $O(n)$ | 999 次迴圈 | 1001 個數字 | 表格留著，好 debug、好畫走查 |
| 兩個變數 | $O(n)$ | $O(1)$ | 999 次迴圈 | 3 個數字 | 面試要的答案 |

題目上限只有 1000，兩個都是 0.0001 秒等級，時間分不出差別。填表版的價值在寫錯的時候可以印出來看，滾動版一旦算錯只剩兩個數字可以看。

---

## 結論

終點在最後一階的外面，所以表格要開 n+1 格、答案是 `f[n]`。`f[i]` 記的是站上第 i 格花了多少，那一格的幣要離開時才投，這個定義一鎖定，遞推式就只有一行。
