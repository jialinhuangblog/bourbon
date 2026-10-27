一張股票每天的收盤價寫在一排格子裡。買一次、賣一次，而且賣一定要在買之後。

問最多能賺多少。賺不到就不要買，答案是 0。

```
7   1   5   3   6   4
    ↑           ↑
    買          賣

第 2 天買在 1，第 5 天賣在 6，賺 5
```

倒過來不算數。第 1 天的 7 賣不掉，那時候手上還沒有股票。

翻成 code 的講法：給一個陣列 `prices`，找 `i < j` 使 `prices[j] - prices[i]` 最大，最大值是負的就回傳 0。

---

**解題引導**

拿 `[7, 1, 5, 3, 6, 4]` 想。

**Step 1：全部組合試一遍要試幾組？**

<span class="spoiler">買的日子 n 種、賣的日子在它後面，大約 n²/2 組。n=10^5 的話是 50 億次。Roughly n squared over two pairs.</span>

**Step 2：換個問法。假設今天就是賣出日，我會希望之前買在哪一天？**

*the cheapest day so far. nothing else matters.*

<span class="spoiler">今天之前最便宜的那一天。買貴的沒有任何理由，反正賣價都一樣。The single cheapest day before today.</span>

**Step 3：那「今天之前最便宜的價格」要怎麼維護？**

<span class="spoiler">從左往右掃的時候拿一個變數記著，看到更低的就換掉。掃到今天的時候，那個變數剛好就是今天以前的最低價。One variable, updated as you sweep left to right.</span>

**Step 4：所以每一天要做幾件事？**

<span class="spoiler">兩件：用今天的價格減掉目前的最低價，跟答案比一下；然後把今天的價格也拿去更新最低價。One subtraction, one comparison.</span>

想完再往下看 code。

---

## 解法一：每組買賣日都試一次

```typescript
function maxProfit(prices: number[]): number {
    let best = 0;

    for (let buy = 0; buy < prices.length; buy++) {
        for (let sell = buy + 1; sell < prices.length; sell++) {
            best = Math.max(best, prices[sell] - prices[buy]);
        }
    }

    return best;
}
```

- Time: $O(n^2)$
- Space: $O(1)$

**走一遍。** `[7, 1, 5, 3, 6, 4]`：

| buy | 買價 | 賣價試過的 | 這一輪最好 | best |
|---|---|---|---|---|
| 0 | 7 | 1, 5, 3, 6, 4 | 6 - 7 = -1 | 0 |
| 1 | 1 | 5, 3, 6, 4 | 6 - 1 = 5 | 5 |
| 2 | 5 | 3, 6, 4 | 6 - 5 = 1 | 5 |
| 3 | 3 | 6, 4 | 6 - 3 = 3 | 5 |
| 4 | 6 | 4 | 4 - 6 = -2 | 5 |
| 5 | 4 | 沒有了 | | 5 |

`best` 初始設 0，所以第一列那個 -1 不會被記進去，「賺不到就不買」自動成立。

**n=10^5 的話。** 組合數是 $n(n-1)/2$ = 4,999,950,000，換算 500 秒，明確 TLE。

<details>
<summary>Go 版本</summary>

```go
func maxProfit(prices []int) int {
    best := 0

    for buy := 0; buy < len(prices); buy++ {
        for sell := buy + 1; sell < len(prices); sell++ {
            if p := prices[sell] - prices[buy]; p > best {
                best = p
            }
        }
    }

    return best
}
```

</details>

---

## 解法二：一趟掃完，只記最低價

解法一的內圈在做一件很浪費的事：對每個買入日，重新看一遍後面所有的賣出日。反過來，把每一天都當成賣出日，需要的就只有「之前的最低價」這一個數字。

```typescript
function maxProfit(prices: number[]): number {
    let minPrice = Infinity;   // 到目前為止看過的最低價
    let best = 0;

    for (const price of prices) {
        minPrice = Math.min(minPrice, price);
        best = Math.max(best, price - minPrice);   // 今天賣的話賺多少
    }

    return best;
}
```

- Time: $O(n)$
- Space: $O(1)$

**走一遍。** `[7, 1, 5, 3, 6, 4]`：

| 天 | price | 更新後的 minPrice | price - minPrice | best |
|---|---|---|---|---|
| 1 | 7 | 7 | 0 | 0 |
| 2 | 1 | 1 | 0 | 0 |
| 3 | 5 | 1 | 4 | 4 |
| 4 | 3 | 1 | 2 | 4 |
| 5 | 6 | 1 | 5 | 5 |
| 6 | 4 | 1 | 3 | 5 |

答案 5。

`minPrice` 先更新再相減，所以同一天買又同一天賣的情況會算出 0，剛好是題目說的「賺不到就回傳 0」。順序反過來寫成先算獲利再更新最低價，答案也對，因為那樣算的是「今天賣、之前買」，一樣合法。兩種寫法只差第一天的中間值。

**minPrice 一路往下，不會回頭。** 第 2 天之後它就固定在 1 了。價格再怎麼漲，之前那個 1 都還在那裡等著。這也是為什麼一個變數就夠：買入日只可能是「目前為止的最低點」，其他日子永遠不會勝出。

<details>
<summary>Go 版本</summary>

```go
func maxProfit(prices []int) int {
    minPrice := math.MaxInt32 // 到目前為止看過的最低價
    best := 0

    for _, price := range prices {
        if price < minPrice {
            minPrice = price
        }
        if p := price - minPrice; p > best {
            best = p // 今天賣的話賺多少
        }
    }

    return best
}
```

</details>

---

## 解法三：轉成 53 Maximum Subarray

把價格改成「每天比前一天漲跌多少」，這題就變成另一題。

```
prices = [7,  1,  5,  3,  6,  4]
diff   =    [-6, +4, -2, +3, -2]      每天減前一天

買在第 2 天、賣在第 5 天賺的 5
= 第 3 天到第 5 天的漲跌相加
= 4 + (-2) + 3 = 5
```

買入到賣出中間賺的錢，就是這段期間每天漲跌的總和。所以「挑買賣日」等於「挑一段連續的漲跌，總和最大」，那就是 [53 Maximum Subarray](/problem/maximum-subarray)。

```typescript
function maxProfit(prices: number[]): number {
    let cur = 0;    // 以今天結尾的最佳連續漲跌
    let best = 0;

    for (let i = 1; i < prices.length; i++) {
        const diff = prices[i] - prices[i - 1];
        cur = Math.max(0, cur + diff);   // 累積跌到負的就重新開始
        best = Math.max(best, cur);
    }

    return best;
}
```

- Time: $O(n)$
- Space: $O(1)$

**走一遍。** 同一組價格：

| i | diff | cur + diff | cur | best |
|---|---|---|---|---|
| 1 | -6 | 0 + (-6) = -6 | 0（重新開始） | 0 |
| 2 | +4 | 0 + 4 = 4 | 4 | 4 |
| 3 | -2 | 4 + (-2) = 2 | 2 | 4 |
| 4 | +3 | 2 + 3 = 5 | 5 | 5 |
| 5 | -2 | 5 + (-2) = 3 | 3 | 5 |

答案 5，跟解法二一樣。

跟 53 的 Kadane 差一個地方：那邊寫 `Math.max(nums[i], cur + nums[i])`，這邊寫 `Math.max(0, cur + diff)`。因為這題允許不買，空手也是合法答案，53 則規定至少要挑一個元素。

<details>
<summary>Go 版本</summary>

```go
func maxProfit(prices []int) int {
    cur, best := 0, 0

    for i := 1; i < len(prices); i++ {
        cur += prices[i] - prices[i-1]
        if cur < 0 {
            cur = 0 // 累積跌到負的就重新開始
        }
        if cur > best {
            best = cur
        }
    }

    return best
}
```

</details>

---

**Overthinking**

**為什麼這題被歸在 Sliding Window。** 把買入日當窗口左界、賣出日當右界，右界每天往前一格，價格比左界低的時候就把左界搬過來。解法二的 `minPrice` 就是那個左界，只是不需要記它在哪一天，記價格就夠了。這題的窗口不會縮，也不需要記窗內的狀態，所以它跟 [3 Longest Substring](/problem/longest-substring-without-repeating-characters) 那種需要 hashmap 的滑動視窗差滿多的。

**買賣可以做很多次呢。** 那是 122 Best Time to Buy and Sell Stock II，答案變成「把所有上漲的日子加起來」，因為每一段漲勢都可以獨立吃下來。再往下 123 限制兩次、188 限制 k 次，就得開 DP 表記「第 i 天、已經交易 j 次、手上有沒有股票」。這一題是那一串的起點。

**三個解法其實是同一個東西。** 解法二追最低價，解法三追累積漲跌，兩者的關係是：`cur = price - minPrice`。價格跌破前低的那天，`cur` 剛好變負然後被歸零。寫法不同，走的路一模一樣。

---

## 解法比較表

| 解法 | Time | Space | n=10^5 操作次數 | 備註 |
|---|---|---|---|---|
| 每組買賣日都試 | $O(n^2)$ | $O(1)$ | 4,999,950,000 次，500 秒 | TLE |
| 追最低價 | $O(n)$ | $O(1)$ | 100,000 次，0.01 秒 | 面試要的答案 |
| 轉成 Kadane | $O(n)$ | $O(1)$ | 100,000 次，0.01 秒 | 一樣快，價值在看出它跟 53 是同一題 |

暴力的次數用 $n(n-1)/2$ 算（跑不動的規模），後兩個是實際跑計數器數出來的，秒數是次數除以 $10^7$。

---

## 結論

把每一天都當成賣出日，需要的只有「今天以前的最低價」。一個變數往下追，一趟掃完。想清楚它就是 53 的變形，兩題可以一起記。
