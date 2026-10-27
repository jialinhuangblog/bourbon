城市的街道排成棋盤格。從左上角的家走到右下角的公司，每一步只能往東或往南，不能往回走。

問總共有幾條不同的走法。

```
m = 3, n = 2      三列兩行

  家 ┌───┬───┐
     │   │   │
     ├───┼───┤
     │   │   │
     ├───┼───┤
     │   │ 公司
     └───┴───┘

右下下、下下右、下右下
三種
```

翻成 code 的講法：`m × n` 的格子，從 `grid[0][0]` 走到 `grid[m-1][n-1]`，每步只能往右或往下，求路徑數。

---

**解題引導**

拿 `m = 3, n = 7` 想，答案是 28。

**Step 1：人現在站在右下角那一格，上一步是從哪裡來的？**

*from the left, or from above. that is all.*

<span class="spoiler">從左邊那格往右走過來，或從上面那格往下走過來。只有兩個來源。Either from the left cell or the cell above.</span>

**Step 2：所以走到 (i, j) 的路徑數怎麼算？**

<span class="spoiler">走到 (i-1, j) 的路徑數，加上走到 (i, j-1) 的路徑數。兩群路徑的最後一步不同，不會重複算到同一條。f[i][j] = f[i-1][j] + f[i][j-1]。Add the two, the final move tells them apart.</span>

**Step 3：這個式子看起來很眼熟，跟哪一題一樣？**

<span class="spoiler">跟 70 Climbing Stairs 一樣，都是「前面兩個來源加起來」。差別只在 70 的兩個來源排在一條線上，這題排在平面上。Same recurrence, one dimension higher.</span>

**Step 4：第一列跟第一行要填什麼？**

*there is only one way to walk in a straight line.*

<span class="spoiler">全部填 1。第一列只能一路往右，第一行只能一路往下，各自只有一條路。The edges have exactly one path each.</span>

想完再往下看 code。

---

## 解法一：照著式子寫遞迴

```typescript
function uniquePaths(m: number, n: number): number {
    function walk(i: number, j: number): number {
        if (i === m - 1 || j === n - 1) return 1;   // 貼著邊了，只剩一條直線
        return walk(i + 1, j) + walk(i, j + 1);
    }

    return walk(0, 0);
}
```

- Time: $O(2^{m+n})$ — 每一格分岔成兩個呼叫
- Space: $O(m + n)$ — call stack 最深走到右下角

**走一遍。** `m = 3, n = 2`：

```
walk(0,0)
├── walk(1,0)
│   ├── walk(2,0)  貼著最後一列 → 1
│   └── walk(1,1)  貼著最後一行 → 1
│   → 2
└── walk(0,1)  貼著最後一行 → 1
→ 3
```

答案 3，總共 5 次呼叫。

**格子一大就跑不動。** 中間的格子會被很多條路徑重複問到，跟 [70 Climbing Stairs](/problem/climbing-stairs) 的樸素遞迴是同一個問題。實測 `m = n = 15` 要 80,233,199 次呼叫，換算 8 秒，而題目允許 100。

<details>
<summary>Go 版本</summary>

```go
func uniquePaths(m int, n int) int {
    var walk func(i, j int) int
    walk = func(i, j int) int {
        if i == m-1 || j == n-1 {
            return 1 // 貼著邊了，只剩一條直線
        }
        return walk(i+1, j) + walk(i, j+1)
    }

    return walk(0, 0)
}
```

</details>

---

## 解法二：填一張二維表

從左上往右下填。每一格只看上面跟左邊那兩格，填的時候它們已經算好了。

```typescript
function uniquePaths(m: number, n: number): number {
    const f: number[][] = Array.from({ length: m }, () => new Array(n).fill(1));
    // 第一列跟第一行都是 1，fill(1) 已經處理掉了

    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            f[i][j] = f[i - 1][j] + f[i][j - 1];   // 上面來的加左邊來的
        }
    }

    return f[m - 1][n - 1];
}
```

- Time: $O(m \times n)$ — 每格填一次
- Space: $O(m \times n)$ — 表格

**走一遍。** `m = 3, n = 7`，填完的表：

| | j=0 | j=1 | j=2 | j=3 | j=4 | j=5 | j=6 |
|---|---|---|---|---|---|---|---|
| **i=0** | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| **i=1** | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| **i=2** | 1 | 3 | 6 | 10 | 15 | 21 | 28 |

答案 `f[2][6] = 28`。

拿 `f[2][3] = 10` 對一下：上面是 `f[1][3] = 4`，左邊是 `f[2][2] = 6`，4 + 6 = 10。

第 0 列全是 1，因為只能一路往右；第 0 行全是 1，因為只能一路往下。這兩排不用算，`fill(1)` 就填好了，迴圈才敢從 `i = 1, j = 1` 開始。

### 壓成一維

填 `f[i][j]` 只用得到正上方跟正左邊。掃到第 i 列的時候，第 i-2 列以上的都用不到了。

留一列就夠：`row[j]` 在被覆寫之前存的是上一列的值（正上方），覆寫之後存的是這一列的值（正左邊）。

```typescript
function uniquePaths(m: number, n: number): number {
    const row = new Array(n).fill(1);   // 第 0 列

    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            row[j] = row[j] + row[j - 1];
            //       ↑上一列    ↑這一列已經更新過的左邊
        }
    }

    return row[n - 1];
}
```

- Time: $O(m \times n)$
- Space: $O(n)$

**走一遍。** 同樣 `m = 3, n = 7`：

| 跑完第幾列 | row 的內容 |
|---|---|
| 起始（第 0 列） | `[1, 1, 1, 1, 1, 1, 1]` |
| i=1 | `[1, 2, 3, 4, 5, 6, 7]` |
| i=2 | `[1, 3, 6, 10, 15, 21, 28]` |

每一列都跟二維表的那一列一模一樣。

`row[j] = row[j] + row[j-1]` 這一行要 j 由小到大跑。輪到 `row[j]` 的時候，`row[j-1]` 剛剛才被這一列更新過（正左邊），而 `row[j]` 還沒被碰（還是上一列的值，正上方）。反過來由大到小跑，`row[j-1]` 會拿到上一列的值，就錯了。

<details>
<summary>Go 版本</summary>

```go
func uniquePaths(m int, n int) int {
    row := make([]int, n)
    for j := range row {
        row[j] = 1 // 第 0 列
    }

    for i := 1; i < m; i++ {
        for j := 1; j < n; j++ {
            row[j] = row[j] + row[j-1] // 上一列的值 加 這一列左邊的值
        }
    }

    return row[n-1]
}
```

</details>

---

## 解法三：直接算組合數

換個角度看這件事。從左上到右下，不管怎麼走，往下一定走 `m-1` 步、往右一定走 `n-1` 步，總共 `m+n-2` 步。

一條路徑就是這 `m+n-2` 步的一種排列。決定哪幾步往下，剩下的自動就是往右。所以答案是「從 `m+n-2` 步裡挑 `m-1` 步當往下」的組合數：

$$C(m+n-2,\ m-1)$$

`m = 3, n = 7` 代進去是 $C(8, 2) = 28$，跟表格算出來的一樣。

```typescript
function uniquePaths(m: number, n: number): number {
    let result = 1;
    const k = Math.min(m - 1, n - 1);   // 挑小的那個，乘的次數比較少

    for (let i = 1; i <= k; i++) {
        result = result * (m + n - 2 - k + i) / i;   // 乘一個、除一個，交錯著算
    }

    return Math.round(result);
}
```

- Time: $O(\min(m, n))$
- Space: $O(1)$

**走一遍。** `m = 3, n = 7`，`k = min(2, 6) = 2`，要算 $C(8, 2)$：

| i | 乘的項 `m+n-2-k+i` | 算式 | result |
|---|---|---|---|
| 起始 | | | 1 |
| 1 | 8 - 2 + 1 = 7 | 1 × 7 ÷ 1 | 7 |
| 2 | 8 - 2 + 2 = 8 | 7 × 8 ÷ 2 | 28 |

答案 28。

**為什麼要乘一個就除一個。** 先把分子 $8 \times 7$ 全部乘完再除，中間值會比答案大很多，`m` 跟 `n` 一大就超出安全整數範圍。交錯著算的話，每一步的 result 都剛好是一個組合數（$C(7,1) = 7$、$C(8,2) = 28$），一定是整數，也不會膨脹。最後的 `Math.round` 是為了收掉浮點除法的誤差。

<details>
<summary>Go 版本</summary>

```go
func uniquePaths(m int, n int) int {
    result := 1
    k := m - 1
    if n-1 < k {
        k = n - 1 // 挑小的那個，乘的次數比較少
    }

    for i := 1; i <= k; i++ {
        result = result * (m + n - 2 - k + i) / i // 乘一個、除一個，交錯著算
    }

    return result
}
```

Go 用整數除法就好，因為每一步的中間值都保證是整數。

</details>

---

**Overthinking**

**`m, n <= 100` 這個上限有誤導性。** 題目另外寫了「答案不超過 $2 \times 10^9$」，這一句才是真正的限制。100 × 100 的格子答案是 $2.3 \times 10^{58}$，早就超過了，所以那組測資根本不會出現。實際能出現的最大正方形是 17 × 17，答案 601,080,390；18 × 18 就是 2,333,606,220，超標了。長條形不受影響，3 × 100 的答案只有 5,050。

意思是這題的 DP 表最多也就填幾千格，三個解法在真實測資上都是瞬間。選哪個是講給面試官聽的，不是效能問題。

**跟 [70 Climbing Stairs](/problem/climbing-stairs) 是同一個遞推式。** 70 是 `f[i] = f[i-1] + f[i-2]`，這題是 `f[i][j] = f[i-1][j] + f[i][j-1]`。兩題都在數方法數，所以中間是加法；兩題的「來源」都只有兩個，所以式子只有兩項。差別在 70 的兩個來源住在同一條線上，這題住在平面的上方跟左方。

從 1-D DP 跨到 2-D DP，變的是狀態要幾個 index 才描述得完，遞推的想法沒變：**這一格的最後一步是從哪裡來的**。

**格子裡有障礙物呢。** 那是 63 Unique Paths II。DP 版只要多一行「這格是障礙就填 0」，其他不動。組合數那版就完全套不上去了，因為路徑不再是自由排列。DP 比較笨但比較耐改，這是它值得先學會的理由。

**跟 [1143 Longest Common Subsequence](/problem/longest-common-subsequence) 的對照。** 兩題都是二維表，但填表的規則不同：

| | 62 Unique Paths | 1143 LCS |
|---|---|---|
| 表格的兩個維度 | 格子的列與行 | 兩個字串各自的位置 |
| 每格看幾個來源 | 2 個（上、左） | 3 個（左上、上、左） |
| 中間的運算 | 加法（數方法數） | max（挑最好的） |
| 邊界 | 第 0 列與第 0 行填 1 | 第 0 列與第 0 行填 0 |

---

## 解法比較表

| 解法 | Time | Space | m=n=15 操作次數 | 備註 |
|---|---|---|---|---|
| 樸素遞迴 | $O(2^{m+n})$ | $O(m+n)$ | 80,233,199 次呼叫，8 秒 | TLE，但它是想出遞推式的起點 |
| 二維表 | $O(m \times n)$ | $O(m \times n)$ | 196 次 | 好畫、好 debug |
| 一維滾動 | $O(m \times n)$ | $O(n)$ | 196 次 | 面試要的答案 |
| 組合數 | $O(\min(m,n))$ | $O(1)$ | 14 次 | 最快，但題目一改就報廢 |

次數是實際跑計數器數出來的，秒數是次數除以 $10^7$。二維表跟一維滾動的填格次數一樣，差別只在記憶體。

---

## 結論

走到一格的路徑數等於上面加左邊，第 0 列跟第 0 行都是 1。這就是 Climbing Stairs 攤到平面上。組合數那版更快，但格子裡放一個障礙物它就沒用了。
