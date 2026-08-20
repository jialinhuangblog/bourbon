你是小偷。一排房子，每間有不同金額。不能搶相鄰的兩間，求最大金額。

---

**解題引導**

用 `[2, 7, 9, 3, 1]` 想。

**Step 1：每間房子有幾種選擇？**<br>
<span class="spoiler">兩種：搶，或不搶。</span>

**Step 2：如果搶了第 i 間，下一間能搶誰？**<br>
<span class="spoiler">不能搶 i+1（相鄰），最早搶 i+2。</span>

**Step 3：如果不搶第 i 間呢？**<br>
<span class="spoiler">直接看 i+1，所有選項都還在。</span>

**Step 4：所以每間房子的最佳結果 = ？**<br>
<span class="spoiler">max(搶這間 + 從 i+2 開始的最佳, 不搶 + 從 i+1 開始的最佳)。寫成遞迴試試看。</span>

**Step 5：遞迴會怎樣？**<br>
<span class="spoiler">畫出 recursion tree，dfs(3) 被多個節點重複呼叫。O(2^n)。</span>

**Step 6：重複計算怎麼解決？**<br>
<span class="spoiler">加 memo 陣列，算過就存起來。每個位置只算一次，O(n)。</span>

**Step 7：memo 能不能翻成迴圈？**<br>
<span class="spoiler">定義 dp[i] = 搶到第 i 間的最大金額。dp[i] = max(dp[i-1], nums[i] + dp[i-2])。從左掃到右。</span>

**Step 8：dp 陣列每一步只用到前兩格，能壓縮嗎？**<br>
<span class="spoiler">兩個變數 prev 和 curr 就夠。空間 O(1)。</span>

想完再往下看 code。

---

## 解法一：暴力遞迴

最直覺的想法：每間房子「搶」或「不搶」，試所有組合，取最大值。

```typescript
function rob(nums: number[]): number {
    function dfs(i: number): number {
        if (i >= nums.length) return 0;
        // 搶這間，跳過下一間 vs 不搶，看下一間
        return Math.max(nums[i] + dfs(i + 2), dfs(i + 1));
    }
    return dfs(0);
}
```

- **Time: O(2ⁿ)**
- Space: O(n) — call stack

<details>
<summary>Go 版本</summary>

```go
func rob(nums []int) int {
    return dfs(nums, 0)
}

func dfs(nums []int, i int) int {
    if i >= len(nums) {
        return 0
    }
    // 搶這間：拿錢，跳過下一間
    // 不搶：直接看下一間
    return max(nums[i]+dfs(nums, i+2), dfs(nums, i+1))
}
```

</details>

畫出 recursion tree 就知道問題在哪。`dfs(3)` 被 `dfs(1)` 和 `dfs(2)` 各呼叫一次。`dfs(4)` 被呼叫更多次。重複計算爆炸成長。

---

## 解法二：Memo

重複計算 → 記住算過的結果。加一個 memo 陣列。

```typescript
function rob(nums: number[]): number {
    const memo = new Array(nums.length).fill(-1);
    function dfs(i: number): number {
        if (i >= nums.length) return 0;
        if (memo[i] !== -1) return memo[i]; // 算過了
        memo[i] = Math.max(nums[i] + dfs(i + 2), dfs(i + 1));
        return memo[i];
    }
    return dfs(0);
}
```

- Time: O(n) — 每個位置只算一次
- Space: O(n) — memo + call stack

<details>
<summary>Go 版本</summary>

```go
func rob(nums []int) int {
    memo := make([]int, len(nums))
    for i := range memo {
        memo[i] = -1 // -1 代表還沒算過
    }
    return dfs(nums, 0, memo)
}

func dfs(nums []int, i int, memo []int) int {
    if i >= len(nums) {
        return 0
    }
    if memo[i] != -1 {
        return memo[i] // 算過了，直接回傳
    }
    memo[i] = max(nums[i]+dfs(nums, i+2, memo), dfs(nums, i+1, memo))
    return memo[i]
}
```

</details>

從 $2^n$ 砍到 $n$。這就是 DP。

用 `nums = [3, 1, 4, 1, 5, 9, 2, 6]` 走一遍，理解 memo 怎麼運作。

memo 是什麼？一個陣列，`memo[i]` 存「從第 i 間開始搶到最後，最多能拿多少」。初始全部填 -1，代表還沒算過。

```
nums = [3, 1, 4, 1, 5, 9, 2, 6]
index:  0  1  2  3  4  5  6  7
memo = [-1,-1,-1,-1,-1,-1,-1,-1]   ← 全部還沒算
```

dfs(0) 開始。它要算 `max(搶第0間, 不搶第0間)`：
- 搶第 0 間 → 拿 3 + dfs(2)
- 不搶第 0 間 → dfs(1)

兩邊都還沒算，先往下鑽。從最深的開始算回來：

```
dfs(7): 搶6 vs 不搶
  dfs(9)=0, dfs(8)=0
  memo[7] = max(6+0, 0) = 6

dfs(6): 搶2 vs 不搶
  dfs(8)=0, dfs(7)=memo[7]=6  ← memo 命中！
  memo[6] = max(2+0, 6) = 6   （不搶比較好，6 > 2）

dfs(5): 搶9 vs 不搶
  dfs(7)=memo[7]=6  ← memo 命中！
  dfs(6)=memo[6]=6  ← memo 命中！
  memo[5] = max(9+6, 6) = 15  （搶 9 + 後面最佳 6 = 15）

dfs(4): 搶5 vs 不搶
  dfs(6)=memo[6]=6  ← memo 命中！
  dfs(5)=memo[5]=15 ← memo 命中！
  memo[4] = max(5+6, 15) = 15 （不搶比較好，15 > 11）

dfs(3): 搶1 vs 不搶
  dfs(5)=memo[5]=15 ← memo 命中！
  dfs(4)=memo[4]=15 ← memo 命中！
  memo[3] = max(1+15, 15) = 16

dfs(2): 搶4 vs 不搶
  dfs(4)=memo[4]=15 ← memo 命中！
  dfs(3)=memo[3]=16 ← memo 命中！
  memo[2] = max(4+15, 16) = 19

dfs(1): 搶1 vs 不搶
  dfs(3)=memo[3]=16 ← memo 命中！
  dfs(2)=memo[2]=19 ← memo 命中！
  memo[1] = max(1+16, 19) = 19

dfs(0): 搶3 vs 不搶
  dfs(2)=memo[2]=19 ← memo 命中！
  dfs(1)=memo[1]=19 ← memo 命中！
  memo[0] = max(3+19, 19) = 22
```

最後 memo 長這樣：

```
index:  0   1   2   3   4   5   6   7
nums:  [3,  1,  4,  1,  5,  9,  2,  6]
memo:  [22, 19, 19, 16, 15, 15,  6,  6]
        ↑
       答案 = 22
```

最佳組合是搶 index 0, 2, 5, 7 → 3 + 4 + 9 + 6 = 22。

注意 index 2 和 5 之間跳了兩間（3 和 4 都不搶）。DP 不會只考慮「搶隔壁或跳一間」，它透過 `dfs(i+1)` 把所有跳法的最佳結果帶過來，自然找到最遠的跳法。

沒有 memo 的話，dfs(5) 會被 dfs(3) 和 dfs(4) 各呼叫一次，dfs(6) 會被呼叫更多次。n 更大時重複呈指數成長。有了 memo，每個位置只算一次，O(n)。

但還能再簡化。

---

## 解法三：Bottom-up DP

Top-down 遞迴可以翻成 bottom-up 迴圈。定義 `dp[i]` = 搶到第 i 間的最大金額。

每間房子只有兩個選擇：
- 搶：拿 `nums[i]` + 前前間的最佳 `dp[i-2]`
- 不搶：維持前一間的最佳 `dp[i-1]`

```typescript
function rob(nums: number[]): number {
    const n = nums.length;
    if (n === 1) return nums[0];
    const dp = new Array(n);
    dp[0] = nums[0];
    dp[1] = Math.max(nums[0], nums[1]);
    for (let i = 2; i < n; i++) {
        dp[i] = Math.max(dp[i - 1], nums[i] + dp[i - 2]); // 不搶 vs 搶
    }
    return dp[n - 1];
}
```

- Time: $O(n)$
- Space: $O(n)$

<details>
<summary>Go 版本</summary>

```go
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }
    dp := make([]int, n)
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    for i := 2; i < n; i++ {
        dp[i] = max(dp[i-1], nums[i]+dp[i-2]) // 不搶 vs 搶
    }
    return dp[n-1]
}
```

</details>

用 `[3, 1, 4, 1, 5, 9, 2, 6]` 走一遍：

```
dp[0] = 3
dp[1] = max(3,  1)    = 3
dp[2] = max(3,  4+3)  = 7
dp[3] = max(7,  1+3)  = 7
dp[4] = max(7,  5+7)  = 12
dp[5] = max(12, 9+7)  = 16
dp[6] = max(16, 2+12) = 16
dp[7] = max(16, 6+16) = 22

答案：22（搶 index 0, 2, 5, 7 → 3+4+9+6 = 22）✓
```

注意 dp[4]=12 選了 index 0 和 4（跳了兩間），dp[5]=16 選了 index 2 和 5（也跳了兩間）。DP 不限於「只跳一間」，`dp[i-1]` 會帶著之前所有可能的最佳結果往前走。

Memo 和 bottom-up 的結果並排看：

```
index:   0   1   2   3   4   5   6   7
nums:   [3,  1,  4,  1,  5,  9,  2,  6]
memo:   [22, 19, 19, 16, 15, 15,  6,  6]   ← 右→左算，答案在 memo[0]
dp:     [3,  3,  7,  7,  12, 16, 16, 22]   ← 左→右算，答案在 dp[7]
```

同一個問題，方向相反：

| | Memo（top-down） | Bottom-up |
|---|---|---|
| 定義 | `memo[i]` = 從第 i 間到最後的最佳 | `dp[i]` = 從第 0 間到第 i 間的最佳 |
| 方向 | 先算出 memo[7]，最後算出 memo[0] | 先算出 dp[0]，最後算出 dp[7] |
| 答案在 | `memo[0] = 22` | `dp[7] = 22` |

兩個都是 22。只是一個從終點算回起點，一個從起點算到終點。

沒有遞迴，沒有 call stack。但空間還是 $O(n)$。

---

## 解法四：壓縮變數

看一下 `dp[i]` 的公式：只用到 `dp[i-1]` 和 `dp[i-2]`。兩個變數就夠了。

```typescript
function rob(nums: number[]): number {
    let prev = 0, curr = 0;
    for (const n of nums) {
        [prev, curr] = [curr, Math.max(curr, n + prev)]; // 不搶 vs 搶
    }
    return curr;
}
```

- Time: O(n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func rob(nums []int) int {
    prev, curr := 0, 0 // prev = dp[i-2], curr = dp[i-1]
    for _, n := range nums {
        prev, curr = curr, max(curr, n+prev) // 不搶 vs 搶
    }
    return curr
}
```

</details>

走一遍 `[3, 1, 4, 1, 5, 9, 2, 6]`：

```
初始：prev=0, curr=0

n=3: prev=0,  curr=max(0, 3+0)=3
n=1: prev=3,  curr=max(3, 1+0)=3
n=4: prev=3,  curr=max(3, 4+3)=7
n=1: prev=7,  curr=max(7, 1+3)=7
n=5: prev=7,  curr=max(7, 5+7)=12
n=9: prev=12, curr=max(12, 9+7)=16
n=2: prev=16, curr=max(16, 2+12)=16
n=6: prev=16, curr=max(16, 6+16)=22

答案：22 ✓
```

---

**Overthinking：如果房子排成一圈呢？**

這就是 [#213 House Robber II](/problem/house-robber-ii)。第一間和最後一間相鄰。

不能同時搶第一間和最後一間。拆成兩個子問題：

1. 搶 `nums[0..n-2]`（不考慮最後一間）
2. 搶 `nums[1..n-1]`（不考慮第一間）

取兩者最大值。每個子問題都是原本的 House Robber。

---

## 結論

| 解法 | Time | Space |
|---|---|---|
| 暴力遞迴 | $O(2^n)$ | $O(n)$ |
| Memo | $O(n)$ | $O(n)$ |
| Bottom-up DP | $O(n)$ | $O(n)$ |
| 壓縮變數 | $O(n)$ | $O(1)$ |

DP 的三步：暴力遞迴 → memo 記住重複 → 翻成迴圈 → 壓縮變數。House Robber 是最乾淨的示範。兩個變數，一個迴圈，$O(1)$ 空間。

一個值得注意的觀察：memo（右→左）和 bottom-up（左→右）方向完全相反，但答案一樣。最佳組合是輸入決定的，不是算的方向決定的。`[3, 1, 4, 1, 5, 9, 2, 6]` 的最佳組合就是 index 0, 2, 5, 7，這在開始算之前就已經固定了。從左走到右量這條路，跟從右走到左量，結果一樣。過程中的子問題不同（`dp[3]=7` vs `memo[3]=16`，問的問題不同），但最終答案一定相同，因為量的是同一條路。
