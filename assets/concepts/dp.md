---
title: "Dynamic Programming"
category: Algorithms
slug: dp
subtitle: 記住算過的，不要重複算
date: 2026-02-28T11:51:31
---

# Dynamic Programming

DP 跟遞迴有什麼不同？

答案：沒有不同。DP 就是遞迴。只是加了一步：**把算過的結果記起來，不要重複算。**

聽起來簡單。做起來是另一回事。因為難的不是「記起來」，是「怎麼定義子問題」。


---

## 從費波那契開始

最常見的遞迴練習：

```go
func fib(n int) int {
    if n <= 1 { return n }
    return fib(n-1) + fib(n-2)
}
```

`fib(5)` 的遞迴樹：

```
                    fib(5)
                   /      \
              fib(4)       fib(3)
             /     \       /    \
          fib(3)  fib(2) fib(2) fib(1)
          /   \    / \    / \
       fib(2) fib(1) fib(0) fib(1) fib(0)
       / \
    fib(1) fib(0)
```

數一下。`fib(3)` 被算了 2 次。`fib(2)` 被算了 3 次。`fib(1)` 被算了 5 次。

n = 5 就重複這麼多。n = 50？遞迴樹有 $2^{50}$ 個節點。跑到天荒地老都算不完。

問題很明顯：**同一個子問題被重複算了無數次。**

---

## 解法一：Top-Down（記憶化）

把算過的結果存起來。下次碰到同一個問題，直接查表。

```go
func fib(n int) int {
    memo := make([]int, n+1)
    for i := range memo { memo[i] = -1 }
    return helper(n, memo)
}

func helper(n int, memo []int) int {
    if n <= 1 { return n }
    if memo[n] != -1 { return memo[n] }     // 算過了，直接回傳

    memo[n] = helper(n-1, memo) + helper(n-2, memo)
    return memo[n]
}
```

加了兩行。一行查表，一行存表。遞迴樹從 $2^n$ 剪成 n 個節點。$O(n)$。

這叫 **memoization**。也叫 top-down DP。因為從大問題往下拆。

---

## 解法二：Bottom-Up（填表）

不用遞迴。從最小的子問題開始，一格一格往上填。

```go
func fib(n int) int {
    if n <= 1 { return n }
    dp := make([]int, n+1)
    dp[0] = 0
    dp[1] = 1
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    return dp[n]
}
```

沒有遞迴。一個 for 迴圈。`dp[i]` 代表「第 i 個費波那契數」。每一格只依賴前兩格。

這叫 bottom-up DP。也叫 tabulation。因為你在填一張表。

### 哪個好？

| | Top-Down | Bottom-Up |
|---|---|---|
| 寫法 | 遞迴 + memo | for 迴圈 + dp 陣列 |
| 思考方式 | 從大問題拆小 | 從小問題堆大 |
| 空間優化 | 難 | 容易（只留前幾格） |
| 不需要的子問題 | 不會算（lazy） | 全部算 |
| Stack Overflow | 遞迴太深會爆 | 不會 |
| 常數時間 | 較慢（遞迴呼叫 overhead） | 較快 |

### Top-Down 唯一的優勢：只算用到的

Bottom-up 不管用不用得到，整張表都填。Top-down 只算遞迴碰到的狀態。

費波那契感受不到差異，因為 f(0) 到 f(n) 每個都會算。但有些題的狀態空間很大，實際走到的只有一小部分。

具體例子：[787 Cheapest Flights Within K Stops](/problem/cheapest-flights-within-k-stops)。狀態是 `(城市, 剩餘轉機次數)`，假設 100 個城市、K=50，狀態空間 5000。但從起點出發，很多城市根本到不了。Top-down 只算可達的狀態，bottom-up 填滿 5000 格。

另一個場景：狀態的某一維度很大。背包容量 $10^9$，物品只有 5 個。Bottom-up 開不出 $10^9$ 的陣列。Top-down 用 hashmap 當 memo，只存實際遞迴到的狀態。

### 怎麼選？

費波那契、House Robber、Climbing Stairs 這類「每個狀態都會用到」的題 → bottom-up 全贏。沒有遞迴 overhead，可以壓縮空間。

狀態空間稀疏、或狀態維度太大開不出陣列 → top-down。

面試時兩種都行。但 bottom-up 比較容易做空間優化，也不怕 stack overflow。

---

## 空間優化

費波那契的 `dp[i]` 只看 `dp[i-1]` 和 `dp[i-2]`。不需要整個陣列。

```go
func fib(n int) int {
    if n <= 1 { return n }
    prev2, prev1 := 0, 1
    for i := 2; i <= n; i++ {
        curr := prev1 + prev2
        prev2 = prev1
        prev1 = curr
    }
    return prev1
}
```

$O(1)$ 空間。兩個變數就夠。

很多 DP 題都能這樣優化。如果 `dp[i]` 只依賴 `dp[i-1]`，只留一個變數。依賴前兩個，留兩個。依賴上一整行（2D DP），留兩行。

---

## DP 的四步公式

每一道 DP 題都在做同一件事：

### 1. 定義狀態

`dp[i]` 代表什麼？這是最難的一步。定義對了，剩下的自然就出來。

### 2. 找轉移方程

`dp[i]` 怎麼從更小的子問題算出來？

### 3. 定義 base case

最小的子問題，答案是什麼？

### 4. 決定計算順序

Bottom-up 從小到大填。Top-down 從大到小遞迴。

用三道經典題走一遍。

---

## 經典一：爬樓梯

[#70 Climbing Stairs](/problem/climbing-stairs)

你在樓梯底部。每次可以走 1 階或 2 階。到第 n 階有幾種走法？

### 1. 定義狀態

`dp[i]` = 到第 i 階的走法數

### 2. 轉移方程

到第 i 階，你從哪來？兩種可能：
- 從第 i-1 階走 1 步上來
- 從第 i-2 階走 2 步上來

```
dp[i] = dp[i-1] + dp[i-2]
```

等等，這不就是費波那契？

對。爬樓梯 = 費波那契。很多 DP 題的轉移方程長得一模一樣，只是包裝不同。

### 3. Base case

```
dp[0] = 1    // 站在地上，一種方式（不動）
dp[1] = 1    // 走一步
```

### 4. 填表

```
n = 5

dp[0] = 1
dp[1] = 1
dp[2] = dp[1] + dp[0] = 2
dp[3] = dp[2] + dp[1] = 3
dp[4] = dp[3] + dp[2] = 5
dp[5] = dp[4] + dp[3] = 8

答案：8
```

```go
func climbStairs(n int) int {
    if n <= 1 { return 1 }
    prev2, prev1 := 1, 1
    for i := 2; i <= n; i++ {
        curr := prev1 + prev2
        prev2 = prev1
        prev1 = curr
    }
    return prev1
}
```

---

## 經典二：零錢問題

[#322 Coin Change](/problem/coin-change)

硬幣 `[1, 3, 5]`，湊出 11 元，最少要幾個硬幣？

### 1. 定義狀態

`dp[i]` = 湊出 i 元的最少硬幣數

### 2. 轉移方程

湊出 i 元的最後一步，你丟了哪個硬幣？

- 丟了 1 元 → 之前要湊 i-1 元 → `dp[i-1] + 1`
- 丟了 3 元 → 之前要湊 i-3 元 → `dp[i-3] + 1`
- 丟了 5 元 → 之前要湊 i-5 元 → `dp[i-5] + 1`

取最小：

```
dp[i] = min(dp[i-1], dp[i-3], dp[i-5]) + 1
```

通用版：

```
dp[i] = min(dp[i - coin] for coin in coins) + 1
```

### 3. Base case

```
dp[0] = 0    // 湊 0 元，不需要硬幣
```

### 4. 填表

```
coins = [1, 3, 5], amount = 11

dp[0]  = 0
dp[1]  = dp[0] + 1 = 1                      （用 1 元）
dp[2]  = dp[1] + 1 = 2                      （1+1）
dp[3]  = min(dp[2], dp[0]) + 1 = 1          （用 3 元！）
dp[4]  = min(dp[3], dp[1]) + 1 = 2          （3+1）
dp[5]  = min(dp[4], dp[2], dp[0]) + 1 = 1   （用 5 元！）
dp[6]  = min(dp[5], dp[3], dp[1]) + 1 = 2   （5+1 或 3+3）
dp[7]  = min(dp[6], dp[4], dp[2]) + 1 = 3   （5+1+1）
dp[8]  = min(dp[7], dp[5], dp[3]) + 1 = 2   （5+3）
dp[9]  = min(dp[8], dp[6], dp[4]) + 1 = 3   （5+3+1）
dp[10] = min(dp[9], dp[7], dp[5]) + 1 = 2   （5+5）
dp[11] = min(dp[10], dp[8], dp[6]) + 1 = 3  （5+5+1）

答案：3（5 + 5 + 1）
```

```go
func coinChange(coins []int, amount int) int {
    dp := make([]int, amount+1)
    for i := range dp { dp[i] = amount + 1 }  // 初始化為不可能的大數
    dp[0] = 0

    for i := 1; i <= amount; i++ {
        for _, coin := range coins {
            if coin <= i && dp[i-coin]+1 < dp[i] {
                dp[i] = dp[i-coin] + 1
            }
        }
    }

    if dp[amount] > amount { return -1 }
    return dp[amount]
}
```

### 為什麼 Greedy 不行？

你可能想：先用最大的硬幣，不夠再用小的。11 = 5 + 5 + 1 = 3 個。剛好。

但如果硬幣是 `[1, 3, 4]`，湊 6？

Greedy：4 + 1 + 1 = 3 個。
DP：3 + 3 = 2 個。

Greedy 找不到 3 + 3 這條路。它只看當下最大的，不會回頭。DP 把所有可能都試過，保證找到最少。

---

## 經典三：最長遞增子序列

[#300 Longest Increasing Subsequence](/problem/longest-increasing-subsequence)

給 `[10, 9, 2, 5, 3, 7, 101, 18]`，找最長嚴格遞增的子序列長度。

答案是 `[2, 3, 7, 101]` 或 `[2, 3, 7, 18]`，長度 4。

### 1. 定義狀態

`dp[i]` = 以 `nums[i]` 結尾的 LIS 長度

是**以 nums[i] 結尾**，不是「前 i 個的 LIS」。這個定義讓轉移方程好寫。

### 2. 轉移方程

以 `nums[i]` 結尾的 LIS，前面一個元素是誰？

掃 j = 0 到 i-1。如果 `nums[j] < nums[i]`，`nums[i]` 可以接在 `nums[j]` 後面。

```
dp[i] = max(dp[j] + 1) for all j < i where nums[j] < nums[i]
```

### 3. Base case

```
每個元素自己就是長度 1 的遞增子序列。dp[i] = 1。
```

### 4. 填表

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]
         0  1  2  3  4  5   6    7

dp[0] = 1  (10)
dp[1] = 1  (9，前面沒有比 9 小的)
dp[2] = 1  (2，前面沒有比 2 小的)
dp[3] = 2  (5，nums[2]=2 < 5 → dp[2]+1 = 2)
dp[4] = 2  (3，nums[2]=2 < 3 → dp[2]+1 = 2)
dp[5] = 3  (7，nums[3]=5 < 7 → dp[3]+1 = 3，也可以接 nums[4]=3)
dp[6] = 4  (101，nums[5]=7 < 101 → dp[5]+1 = 4)
dp[7] = 4  (18，nums[5]=7 < 18 → dp[5]+1 = 4)

答案：max(dp) = 4
```

```go
func lengthOfLIS(nums []int) int {
    n := len(nums)
    dp := make([]int, n)
    for i := range dp { dp[i] = 1 }

    result := 1
    for i := 1; i < n; i++ {
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] && dp[j]+1 > dp[i] {
                dp[i] = dp[j] + 1
            }
        }
        if dp[i] > result { result = dp[i] }
    }
    return result
}
```

$O(n^2)$。有 $O(n \log n)$ 的做法（binary search + patience sorting），但 $O(n^2)$ 的 DP 版本更好理解。

---

## 2D DP：背包問題

一維不夠用的時候，加一維。

[#416 Partition Equal Subset Sum](/problem/partition-equal-subset-sum) — 能不能把陣列分成兩半，兩半的和相等？

等價於：能不能從陣列裡選一些數字，湊出 `sum / 2`？

這是 0/1 背包。每個數字選或不選。

### 1. 定義狀態

`dp[i][j]` = 用前 i 個數字，能不能湊出 j

### 2. 轉移方程

第 i 個數字（`nums[i-1]`）選或不選：

```
dp[i][j] = dp[i-1][j]                      // 不選
         || dp[i-1][j - nums[i-1]]         // 選（如果 j >= nums[i-1]）
```

### 3. Base case

```
dp[0][0] = true    // 0 個數字湊出 0，可以
dp[0][j] = false   // 0 個數字湊出 j>0，不行
```

### 空間優化：2D → 1D

`dp[i][j]` 只看 `dp[i-1][...]`，也就是上一行，所以只留一行就夠。

但要**從右往左填**。不然 `dp[j - nums[i]]` 會用到這一輪剛更新過的值，等於同一個數字被選了兩次。0/1 背包每個數字只能選一次，從右往左才保證讀到的都是「上一輪」的結果。

```go
func canPartition(nums []int) bool {
    sum := 0
    for _, n := range nums { sum += n }
    if sum%2 != 0 { return false }
    target := sum / 2

    dp := make([]bool, target+1)
    dp[0] = true

    for _, n := range nums {
        for j := target; j >= n; j-- {    // 從右往左！
            dp[j] = dp[j] || dp[j-n]
        }
    }
    return dp[target]
}
```

---

## 回溯路徑：不只要答案，還要過程

到目前為止，`dp[i]` 都只存「最優值」。但很多題要的不是數字，是**選了什麼**。

- Coin Change：最少 3 個硬幣。好，哪 3 個？
- LIS：最長 4。好，是哪 4 個元素？
- Edit Distance：最少 5 步。好，哪 5 步？

算出 `dp[n]` 只回答「多少」。要回答「哪些」，得多記一樣東西：**這個最優解從哪來。**

### 例一：Coin Change 還原硬幣

原本 `dp[i]` 只存最少硬幣數。再加一個 `from[i]`，存「湊到 i 元時，最後丟的是哪個硬幣」。

```go
func coinChangeWithPath(coins []int, amount int) (int, []int) {
    dp := make([]int, amount+1)
    from := make([]int, amount+1)          // 新增：紀錄來源
    for i := range dp { dp[i] = amount + 1 }
    dp[0] = 0

    for i := 1; i <= amount; i++ {
        for _, coin := range coins {
            if coin <= i && dp[i-coin]+1 < dp[i] {
                dp[i] = dp[i-coin] + 1
                from[i] = coin              // 記下：這一步用了 coin
            }
        }
    }

    if dp[amount] > amount { return -1, nil }

    // 回溯：從 amount 往回走
    path := []int{}
    for i := amount; i > 0; i -= from[i] {
        path = append(path, from[i])
    }
    return dp[amount], path
}
```

`coins=[1,3,5], amount=11` → `dp[11]=3, path=[1,5,5]`。

從 `i=11` 看 `from[11]=1`，跳到 `i=10`，看 `from[10]=5`，跳到 `i=5`，看 `from[5]=5`，跳到 `i=0`，結束。一路記下的硬幣就是解。

### 例二：LIS 還原子序列

`dp[i]` 存以 `nums[i]` 結尾的 LIS 長度。再加一個 `prev[i]`，存「前一個接誰」。

```go
func lengthOfLISWithPath(nums []int) []int {
    n := len(nums)
    dp := make([]int, n)
    prev := make([]int, n)
    for i := range dp { dp[i] = 1; prev[i] = -1 }

    bestEnd := 0
    for i := 1; i < n; i++ {
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] && dp[j]+1 > dp[i] {
                dp[i] = dp[j] + 1
                prev[i] = j                  // i 接在 j 後面
            }
        }
        if dp[i] > dp[bestEnd] { bestEnd = i }
    }

    // 從最長的結尾往前串
    path := []int{}
    for i := bestEnd; i != -1; i = prev[i] {
        path = append([]int{nums[i]}, path...)
    }
    return path
}
```

`nums=[10,9,2,5,3,7,101,18]` → `path=[2,5,7,101]`。

### 通用手法

每次更新 `dp[i]` 時，同時記下「這次更新來自哪個決策」。跑完之後，從答案格子開始，沿著來源指標一路往回跳，直到 base case。

記什麼看題目：
- 可能是「前一個 index」（LIS、最短路徑）
- 可能是「選了什麼」（Coin Change 的硬幣、背包的物品）
- 可能是「哪個操作」（Edit Distance 的插入/刪除/替換）

空間代價：再開一個跟 dp 一樣大的陣列。通常可以接受。

### 小陷阱

如果有做空間壓縮（滾動陣列、只留前兩格），**不能直接回溯**。壓縮後中間狀態已經被覆蓋，沒東西可以回追。要回溯就得留完整的 dp 表。

這也是為什麼面試時常問「只要答案」還是「要過程」。只要答案可以壓空間，要過程就得保留全表。

---

## DP 的思考順序

碰到一道新題，按這個順序想：

1. **暴力怎麼做？** 通常是 DFS 窮舉所有可能。
2. **有重複子問題嗎？** 畫遞迴樹，看有沒有重複的節點。
3. **定義 dp[i] 代表什麼。** 這是最難的。通常是「以 i 結尾的最優解」或「前 i 個元素的最優解」。
4. **寫轉移方程。** dp[i] 怎麼從更小的 dp 算出來。
5. **Base case。** 最小的問題答案是什麼。
6. **能不能優化空間？** dp[i] 只看前幾個？只留那幾個。

---

## 常見 DP 模式

### 一維 DP

| 題目 | dp[i] 定義 | 轉移 |
|------|-----------|------|
| [#70 Climbing Stairs](/problem/climbing-stairs) | 到第 i 階的走法數 | dp[i-1] + dp[i-2] |
| [#198 House Robber](/problem/house-robber) | 前 i 間的最大收益 | max(dp[i-1], dp[i-2]+nums[i]) |
| [#300 LIS](/problem/longest-increasing-subsequence) | 以 i 結尾的 LIS 長度 | max(dp[j]+1) |
| [#322 Coin Change](/problem/coin-change) | 湊出 i 的最少硬幣 | min(dp[i-coin]) + 1 |
| [#139 Word Break](/problem/word-break) | 前 i 個字元能不能拆成字典裡的字 | dp[j] && s[j:i] in dict |

### 二維 DP

| 題目 | dp[i][j] 定義 | 模式 |
|------|-------------|------|
| [#62 Unique Paths](/problem/unique-paths) | 到 (i,j) 的路徑數 | dp[i-1][j] + dp[i][j-1] |
| [#1143 Longest Common Subsequence](/problem/longest-common-subsequence) | s1 前 i 個和 s2 前 j 個的 LCS | 字元相同 +1，不同取 max |
| [#416 Partition Equal Subset Sum](/problem/partition-equal-subset-sum) | 用前 i 個能不能湊出 j | 0/1 背包 |
| [#72 Edit Distance](/problem/edit-distance) | s1 前 i 個變成 s2 前 j 個的最少操作 | 插入/刪除/替換 取 min |
| [#5 Longest Palindromic Substring](/problem/longest-palindromic-substring) | s[i..j] 是不是回文 | 區間 DP |

---

## 面試常問

**Q：DP 和 Greedy 的差別？**

Greedy 每步都選當下最好的，不回頭。DP 考慮所有可能，取全域最優。

Greedy 能用的時候更快（通常 $O(n)$）。但很多題 greedy 選不到最優解。Coin change `[1,3,4]` 湊 6 就是例子。

**Q：DP 和 DFS 的關係？**

Top-down DP = DFS + memoization。

DFS 窮舉所有路徑。如果有重複子問題，加 memo。就變成 DP。

沒有重複子問題？那就只是 DFS/backtracking，不是 DP。

**Q：怎麼判斷一道題是不是 DP？**

兩個特徵：
1. **最優子結構**：大問題的最優解包含小問題的最優解
2. **重疊子問題**：同一個小問題被算很多次

看到「最少」「最多」「有幾種方法」「能不能」，先往 DP 想。

---

## 總結

DP 就是遞迴加記憶。

暴力遞迴把所有可能走一遍，指數級時間。加 memo，每個子問題只算一次，多項式時間。

難的不是 memo。難的是定義 `dp[i]`。

碰到新題，先暴力 DFS。畫遞迴樹。看哪些節點重複。把重複的記起來。這就是 DP。

不用背轉移方程。理解「dp[i] 代表什麼」就夠了。轉移方程是自然的推論。

---

## 高頻題清單

### 一維 DP

| 題目 | 核心 |
|------|------|
| [#70 Climbing Stairs](/problem/climbing-stairs) | dp[i-1] + dp[i-2]，費波那契變形 |
| [#198 House Robber](/problem/house-robber) | 搶或不搶，max(dp[i-1], dp[i-2]+nums[i]) |
| [#213 House Robber II](/problem/house-robber-ii) | 環形，拆成兩次 House Robber |
| [#300 Longest Increasing Subsequence](/problem/longest-increasing-subsequence) | 以 i 結尾的 LIS，$O(n^2)$ 或 O(n log n) |
| [#322 Coin Change](/problem/coin-change) | 湊金額的最少硬幣數 |
| [#139 Word Break](/problem/word-break) | 前 i 個字元能不能拆成字典裡的字 |
| [#152 Maximum Product Subarray](/problem/maximum-product-subarray) | 同時追蹤最大和最小（負負得正） |
| [#53 Maximum Subarray](/problem/maximum-subarray) | Kadane's algorithm |
| [#91 Decode Ways](/problem/decode-ways) | 一位或兩位解碼，類似爬樓梯 |
| [#377 Combination Sum IV](/problem/combination-sum-iv) | 排列數版的零錢問題 |

### 二維 DP

| 題目 | 核心 |
|------|------|
| [#62 Unique Paths](/problem/unique-paths) | 格子路徑數，dp[i-1][j] + dp[i][j-1] |
| [#1143 Longest Common Subsequence](/problem/longest-common-subsequence) | 兩字串比對，相同 +1 不同取 max |
| [#72 Edit Distance](/problem/edit-distance) | 插入/刪除/替換取 min |
| [#416 Partition Equal Subset Sum](/problem/partition-equal-subset-sum) | 0/1 背包 |
| [#494 Target Sum](/problem/target-sum) | 背包變形，+/- 兩種選擇 |
| [#5 Longest Palindromic Substring](/problem/longest-palindromic-substring) | 區間 DP，s[i..j] 是否回文 |
| [#97 Interleaving String](/problem/interleaving-string) | 兩字串交錯組成第三串 |
| [#115 Distinct Subsequences](/problem/distinct-subsequences) | 子序列匹配計數 |
| [#309 Best Time to Buy and Sell Stock with Cooldown](/problem/best-time-to-buy-and-sell-stock-with-cooldown) | 狀態機 DP |
| [#312 Burst Balloons](/problem/burst-balloons) | 區間 DP，最後戳哪個 |
