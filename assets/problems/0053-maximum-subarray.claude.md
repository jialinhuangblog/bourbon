一家店把每天的盈虧記在本子上。有的天賺，有的天賠。

老闆想知道：連續的哪幾天加起來賺最多。中間不能跳過，挑到的必須是連在一起的一段。

```
-2   1   -3   4   -1   2   1   -5   4
                 └──────────────┘
                  4 + (-1) + 2 + 1 = 6

最好的一段是第 4 到第 7 天，總和 6
```

第 8 天賠 5，接下去會變成 1，所以那一段到第 7 天就該收手。

翻成 code 的講法：給一個整數陣列，求總和最大的連續子陣列，回傳那個總和。

---

**解題引導**

拿 `[-2, 1, -3, 4, -1, 2, 1, -5, 4]` 想。

**Step 1：全部試一遍要試幾段？**

<span class="spoiler">起點 n 種、終點 n 種，大約 n²/2 段。n=10^5 的話是 50 億次，跑不完。Roughly n squared over two.</span>

**Step 2：換個角度。只問「以第 i 天結尾的最佳段」，這種段有幾種可能？**

*either the streak continues, or a new one starts here.*

<span class="spoiler">兩種。要嘛接在「以第 i-1 天結尾的最佳段」後面，要嘛從第 i 天自己重新開始。Extend the previous best, or start fresh at i.</span>

**Step 3：什麼時候該重新開始？**

<span class="spoiler">前面那段的總和是負的時候。負數帶著走只會讓總和變小，不如丟掉重來。When the running sum has gone negative it is pure drag.</span>

**Step 4：陣列全都是負數的話會怎樣？**

*there is no such thing as taking nothing here.*

<span class="spoiler">題目要求至少挑一個元素，所以答案是最大的那個負數，不是 0。best 的初始值要設成 nums[0]，設成 0 會回傳 0。With all negatives the answer is the largest single element.</span>

想完再往下看 code。

---

## 解法一：每個起點各掃一次

固定起點，往右一個一個加，邊加邊記最大值。

```typescript
function maxSubArray(nums: number[]): number {
    let best = nums[0];

    for (let start = 0; start < nums.length; start++) {
        let sum = 0;
        for (let end = start; end < nums.length; end++) {
            sum += nums[end];              // 延長到 end，不用重算前面
            best = Math.max(best, sum);
        }
    }

    return best;
}
```

- Time: $O(n^2)$ — 每個起點掃一次剩下的
- Space: $O(1)$

內圈用累加而不是每次重算，已經省掉一個 n。再省不下去了。

**走一遍。** `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`：

| start | 往右累加的 sum | 這一輪最大 | best |
|---|---|---|---|
| 0 | -2, -1, -4, 0, -1, 1, 2, -3, 1 | 2 | 2 |
| 1 | 1, -2, 2, 1, 3, 4, -1, 3 | 4 | 4 |
| 2 | -3, 1, 0, 2, 3, -2, 2 | 3 | 4 |
| 3 | 4, 3, 5, 6, 1, 5 | 6 | 6 |
| 4 | -1, 1, 2, -3, 1 | 2 | 6 |
| 5 | 2, 3, -2, 2 | 3 | 6 |
| 6 | 1, -4, 0 | 1 | 6 |
| 7 | -5, -1 | -1 | 6 |
| 8 | 4 | 4 | 6 |

答案 6，出現在 start=3 那一列的第四個累加值（`4, 3, 5, 6`），也就是 `[4,-1,2,1]`。全部試了 9+8+...+1 = 45 段。

**n=10^5 的話。** 段數是 $n(n-1)/2$ = 4,999,950,000，換算 500 秒，明確 TLE。

<details>
<summary>Go 版本</summary>

```go
func maxSubArray(nums []int) int {
    best := nums[0]

    for start := 0; start < len(nums); start++ {
        sum := 0
        for end := start; end < len(nums); end++ {
            sum += nums[end] // 延長到 end，不用重算前面
            if sum > best {
                best = sum
            }
        }
    }

    return best
}
```

</details>

---

## 解法二：Kadane，一趟掃完

解法一每換一個起點就從頭累加。改成問 Step 2 那個問題，起點就不用列舉了。

`cur` 記「以現在這一格結尾的最佳段總和」。每走一格，二選一：延長前面那段，或者從這一格重新開始。

```typescript
function maxSubArray(nums: number[]): number {
    let cur = nums[0];    // 以目前這格結尾的最佳段
    let best = nums[0];   // 全部看過的最佳段

    for (let i = 1; i < nums.length; i++) {
        cur = Math.max(nums[i], cur + nums[i]);   // 重新開始，還是接下去
        best = Math.max(best, cur);
    }

    return best;
}
```

- Time: $O(n)$ — 一個迴圈
- Space: $O(1)$ — 兩個變數

**走一遍。** `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`：

| i | nums[i] | cur + nums[i] | 選哪個 | cur | best |
|---|---|---|---|---|---|
| 0 | -2 | 起始 | | -2 | -2 |
| 1 | 1 | -2 + 1 = -1 | 重新開始（1 > -1） | 1 | 1 |
| 2 | -3 | 1 + (-3) = -2 | 接下去（-2 > -3） | -2 | 1 |
| 3 | 4 | -2 + 4 = 2 | 重新開始（4 > 2） | 4 | 4 |
| 4 | -1 | 4 + (-1) = 3 | 接下去 | 3 | 4 |
| 5 | 2 | 3 + 2 = 5 | 接下去 | 5 | 5 |
| 6 | 1 | 5 + 1 = 6 | 接下去 | 6 | 6 |
| 7 | -5 | 6 + (-5) = 1 | 接下去 | 1 | 6 |
| 8 | 4 | 1 + 4 = 5 | 接下去 | 5 | 6 |

答案 6。

i=3 那一列就是「丟掉前面重來」發生的地方。走到 i=2 的時候 cur 是 -2，帶著它走到 i=3 只會得到 2，不如從 4 自己開始。

`best` 跟 `cur` 要分開兩個變數，因為最佳段可能早就結束了。i=7 之後 cur 掉到 1，但 best 記住的 6 不受影響。

**`cur` 是 DP 的狀態，只是被壓成一個變數。** 完整寫法是 `f[i] = max(nums[i], f[i-1] + nums[i])`，`f[i]` 的意思是「以第 i 格結尾的最佳段」。因為只用得到前一格，整張表縮成 `cur`。這一步跟 [70 Climbing Stairs](/problem/climbing-stairs) 從表格縮成兩個變數是同一件事。

<details>
<summary>Go 版本</summary>

```go
func maxSubArray(nums []int) int {
    cur := nums[0]  // 以目前這格結尾的最佳段
    best := nums[0] // 全部看過的最佳段

    for i := 1; i < len(nums); i++ {
        if cur < 0 {
            cur = nums[i] // 前面那段是負的，丟掉重來
        } else {
            cur += nums[i]
        }
        if cur > best {
            best = cur
        }
    }

    return best
}
```

`cur < 0` 就重來，跟 `max(nums[i], cur + nums[i])` 是同一件事：`cur` 為負的時候 `nums[i]` 一定比 `cur + nums[i]` 大。

</details>

---

## 解法三：Divide and Conquer

一列運煤的貨運火車，每節車廂記著這節車廂賺賠多少：載滿貨賺一筆，空車廂還要付油錢，記成負的。從中間的接口把火車拆成前後兩截，獲利最高的那段車廂，是整段在前截、整段在後截，還是卡在接口上？

題目的 follow-up 指名要這個。翻成陣列：中線就是那個接口。切完之後，任何一段連續的子陣列只會是三種之一：

```
[-2, 1, -3, 4, -1 | 2, 1, -5, 4]
                  ↑ 中線切在 index 4 跟 5 中間

[4,-1]        index 3~4    整段在左半
[2,1]         index 5~6    整段在右半
[4,-1,2,1]    index 3~6    左右都碰到，橫跨中線
```

前兩種丟給遞迴自己去算，第三種沒人管，得手動處理。後面統一叫它「跨中線」的段。

跨中線的段有一個性質：它一定同時含有 index 4 跟 index 5。因為它是連續的，既然左右兩邊都碰得到，中間就不准有缺口，中線兩側那兩格必然在裡面。

所以最好的跨中線段長這樣：從 index 4 往左延伸、總和最大的那一段，接上從 index 5 往右延伸、總和最大的那一段。兩邊各掃一次就找得到。

```typescript
function maxSubArray(nums: number[]): number {
    function solve(l: number, r: number): number {
        if (l === r) return nums[l];

        const mid = (l + r) >> 1;
        const left = solve(l, mid);          // 整段在左半
        const right = solve(mid + 1, r);     // 整段在右半

        let sum = 0;
        let leftBest = -Infinity;
        for (let i = mid; i >= l; i--) {     // 從中線往左，一定要含 mid
            sum += nums[i];
            leftBest = Math.max(leftBest, sum);
        }

        sum = 0;
        let rightBest = -Infinity;
        for (let i = mid + 1; i <= r; i++) { // 從中線往右，一定要含 mid+1
            sum += nums[i];
            rightBest = Math.max(rightBest, sum);
        }

        return Math.max(left, right, leftBest + rightBest);
    }

    return solve(0, nums.length - 1);
}
```

- Time: $O(n \log n)$ — 每一層合計掃 n 格，切 log n 層
- Space: $O(\log n)$ — call stack

**走一遍。** 同一組 `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`。divide and conquer 是遞迴，17 次呼叫彼此是父子關係，照呼叫順序用縮排寫，比塞進一張平面表格看得出關係：

```
solve(index 0~8)  中線 index 4（值 -1）
  solve(index 0~4)  中線 index 2（值 -3）
    solve(index 0~2)  中線 index 1（值 1）
      solve(index 0~1)  中線 index 0（值 -2）
        solve(index 0) → -2
        solve(index 1) → 1
        往左（從 index 0 往左）：-2 → 最好 -2
        往右（從 index 1 往右）：1 → 最好 1
        跨中線 = -2+1 = -1
        回傳 max(-2, 1, -1) = 1
      solve(index 2) → -3
      往左（從 index 1 往左）：1，1-2=-1 → 最好 1
      往右（從 index 2 往右）：-3 → 最好 -3
      跨中線 = 1-3 = -2
      回傳 max(1, -3, -2) = 1
    solve(index 3~4)  中線 index 3（值 4）
      solve(index 3) → 4
      solve(index 4) → -1
      往左（從 index 3 往左）：4 → 最好 4
      往右（從 index 4 往右）：-1 → 最好 -1
      跨中線 = 4-1 = 3
      回傳 max(4, -1, 3) = 4
    往左（從 index 2 往左）：-3，-3+1=-2，-2-2=-4 → 最好 -2
    往右（從 index 3 往右）：4，4-1=3 → 最好 4
    跨中線 = -2+4 = 2
    回傳 max(1, 4, 2) = 4
  solve(index 5~8)  中線 index 6，同一套邏輯遞迴到底，回傳 4（過程跟左半對稱，這裡省略）
  往左（從 index 4 往左）：-1，-1+4=3，3-3=0，0+1=1，1-2=-1 → 最好 3
  往右（從 index 5 往右）：2，2+1=3，3-5=-2，-2+4=2 → 最好 3
  跨中線 = 3+3 = 6
  回傳 max(4, 4, 6) = 6
```

**`-3, -2, -4` 這串怎麼來的：** `solve(index 0~4)` 的往左累加，從中線 index 2（值 -3）出發，每步往左多拉一格：`-3` → `-3+1=-2` → `-2-2=-4`。中間那個 -2 最好，對應段 `[1,-3]`（index 1~2）。

`solve(index 0~1)` 是中線兩側各只有一格的極端例子：跨中線沒有選擇，直接是整個 `[-2,1]`，總和 -1。這格回傳 1，是右半自己的 1 贏過跨中線的 -1，不是跨中線贏。

**答案落在最上層的跨中線。** `solve(index 0~4)` 跟 `solve(index 5~8)` 都回傳 4，因為最佳段 `[4,-1,2,1]`（index 3~6）橫跨 index 4/5，兩個子問題各自都切不到完整的它。等最上層合併，往左最好的 3（`4,-1`）接上往右最好的 3（`2,1`），6 才湊得出來。

跟解法二的表對照：Kadane 在 i=6 的時候 `best` 就變成 6 了，一趟掃過去；divide and conquer 要走完全部 17 次呼叫，最上層合併那一步才知道。

**兩個內圈都是從中線出發，這件事不能改。** 寫成「左半最大的一段加上右半最大的一段」就錯了。`solve(index 0~8)` 的左半最好是 `[4]`（index 3），右半最好也是 `[4]`（index 8），中間隔著 index 4 到 7 那四格。兩段接不起來，硬加得到 8，比正確答案 6 還大。從中線出發才保證兩段黏得住。

<details>
<summary>Go 版本</summary>

```go
func maxSubArray(nums []int) int {
    var solve func(l, r int) int
    solve = func(l, r int) int {
        if l == r {
            return nums[l]
        }

        mid := (l + r) / 2
        left := solve(l, mid)
        right := solve(mid+1, r)

        sum, leftBest := 0, math.MinInt32
        for i := mid; i >= l; i-- { // 從中線往左，一定要含 mid
            sum += nums[i]
            if sum > leftBest {
                leftBest = sum
            }
        }

        sum = 0
        rightBest := math.MinInt32
        for i := mid + 1; i <= r; i++ { // 從中線往右
            sum += nums[i]
            if sum > rightBest {
                rightBest = sum
            }
        }

        best := left
        if right > best {
            best = right
        }
        if leftBest+rightBest > best {
            best = leftBest + rightBest
        }
        return best
    }

    return solve(0, len(nums)-1)
}
```

</details>

---

**Overthinking**

**[121 Best Time to Buy and Sell Stock](/problem/best-time-to-buy-and-sell-stock) 就是這一題換一個包裝。** 把股價陣列改成「每天相對前一天的漲跌」，最大獲利就等於漲跌陣列的最大連續和。

```
prices = [7, 1, 5, 3, 6, 4]
diff   =   [-6, 4, -2, 3, -2]      每天減前一天

diff 的最大連續和 = 4 + (-2) + 3 = 5
121 的答案也是 5
```

買在某天、賣在後面某天，賺的錢就是中間所有漲跌的總和。所以「挑一段連續的漲跌」跟「挑買賣日」是同一件事。實際寫 121 不用真的做出 diff 陣列，追一個最低價就好，但知道它是 53 的變形，兩題就只剩一題要記。

**Kadane 是 greedy 還是 DP。** 「cur 變負就丟掉」聽起來很 greedy，但它是 DP：狀態是 `f[i] = 以第 i 格結尾的最佳段`，遞推式看前一格。之所以看起來像 greedy，是因為狀態壓成了一個變數，表格消失了。判準在有沒有「用子問題的答案組出這一格的答案」，有就是 DP，跟 code 裡有沒有陣列無關。這條線的完整討論在 [Greedy](/concept/greedy) 跟 [Dynamic Programming](/concept/dp)。

**divide and conquer 還能更快。** 上面那版每一層都要重新掃 n 格，所以是 $O(n \log n)$。改成讓每次遞迴回傳四個值（總和、最佳前綴、最佳後綴、最佳段），合併就變成常數時間，總共 $O(n)$。實測 n=10^5 只要 199,999 次呼叫。寫起來比 Kadane 麻煩十倍而複雜度一樣，所以現實中沒人這樣寫，但它是線段樹解區間最大子段和的基礎。

---

## 解法比較表

| 解法 | Time | Space | n=10^5 操作次數 | 備註 |
|---|---|---|---|---|
| 每個起點掃一次 | $O(n^2)$ | $O(1)$ | 4,999,950,000 次，500 秒 | TLE，但它是推導的起點 |
| Kadane | $O(n)$ | $O(1)$ | 100,000 次，0.01 秒 | 面試要的答案，五行 |
| Divide and Conquer | $O(n \log n)$ | $O(\log n)$ | 1,768,928 次，0.18 秒 | follow-up 指名的解 |

暴力的次數用 $n(n-1)/2$ 算（跑不動的規模），另外兩個是實際跑計數器數出來的，秒數是次數除以 $10^7$。divide and conquer 比 Kadane 慢 18 倍，寫起來也長三倍，它的價值在於能推廣成線段樹回答任意區間的查詢。

---

## 結論

問「以第 i 格結尾的最佳段」，答案只有兩種：接下去，或從這格重來。前面那段是負的就丟掉。全負數的測資要記得把 best 初始成 `nums[0]`，不是 0。
