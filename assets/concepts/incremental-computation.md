---
title: "增量運算 vs DP"
category: Algorithms
slug: incremental-computation
subtitle: 都是「從上一步接著算」，但機制完全不同
date: 2026-03-14T18:03:46
---

# 增量運算 vs DP

Prefix sum、sliding window、monotonic stack、DP，四個 pattern 有一個共通點：不從頭算，從上一步的結果接著算。

聽起來很像同一件事。但機制完全不同。搞混了會在面試時選錯工具。

---

## 什麼是增量運算？

**算過的不要重算。**

暴力解的浪費，通常是每次都從頭算一遍。增量運算的做法是保留上一步的結果，只處理「變化的部分」。

[238 Product of Array Except Self](/problem/product-of-array-except-self) 就是典型。暴力解對每個位置重新乘 n-1 個數，$O(n^2)$。Prefix product 從左掃一次、從右掃一次，每個位置只做一次乘法，O(n)。

但「從上一步接著算」只是手段，不是分類。四個 pattern 用這個手段做的事完全不一樣。

---

## 四個 pattern 的本質差異

### Prefix Sum：純累積，沒有決策

```go
// prefix[i] = prefix[i-1] + nums[i]
prefix := make([]int, len(nums)+1)
for i, n := range nums {
    prefix[i+1] = prefix[i] + n // 上一步的結果加一個數，就是這一步的結果
}
```

`prefix[i]` 的值是唯一確定的。沒有「要不要加」的選擇。給定 `nums`，prefix sum 只有一種結果。

這不是 DP。DP 需要在多個子問題的結果中**做選擇**。Prefix sum 沒有選擇。

典型題：[238 Product of Array Except Self](/problem/product-of-array-except-self)、[209 Minimum Size Subarray Sum](/problem/minimum-size-subarray-sum)（prefix sum + sliding window 交叉）。

### Sliding Window：維護窗口狀態，右加左減

```go
// 固定窗口：右邊加一個、左邊減一個
sum += nums[right] - nums[right-k] // 窗口滑動，只改變兩端
```

窗口往右滑一格，狀態只需要一次加法和一次減法。不用重新算整個窗口。

跟 DP 的差異：sliding window 的「狀態」是窗口內容的快照，不是子問題的最優解。窗口移動時，舊的狀態被丟掉，不會被後面的步驟引用。DP 的 `dp[i]` 會被 `dp[i+1]` 引用。

典型題：[643 Maximum Average Subarray I](/problem/maximum-average-subarray-i)（固定窗口）、[3 Longest Substring Without Repeating Characters](/problem/longest-substring-without-repeating-characters)（可變窗口）。

### Monotonic Stack：淘汰無用候選

```go
// 遇到新元素，彈掉所有比它小的（或大的）
for len(stack) > 0 && nums[stack[len(stack)-1]] < nums[i] {
    stack = stack[:len(stack)-1] // 這些元素不可能再成為答案，淘汰
}
stack = append(stack, i)
```

Monotonic stack 的增量性質來自「每個元素最多進 stack 一次、出 stack 一次」。amortized O(n)。

跟 DP 的差異：stack 裡存的是「還有可能成為答案的候選」，不是子問題的解。彈出的元素是確定不會再有用的，這是貪心的判斷，不是狀態轉移。

典型題：[739 Daily Temperatures](/problem/daily-temperatures)、[853 Car Fleet](/problem/car-fleet)。

### DP：狀態轉移 + 決策

```go
// dp[i] = max(nums[i], dp[i-1] + nums[i])
// 兩個選項：從自己重新開始，或接在前一個後面
dp[i] = max(nums[i], dp[i-1]+nums[i]) // 這裡有選擇
```

其他三個沒有選擇，DP 有：**`dp[i]` 的值取決於一個決策。**

[53 Maximum Subarray](/problem/maximum-subarray) 的 Kadane's algorithm：到位置 i 的時候，要把 `nums[i]` 接在前一段後面，還是從 `nums[i]` 重新開始？`max` 就是那個決策。

[70 Climbing Stairs](/problem/climbing-stairs) 看起來沒有 `max`/`min`，只有 `dp[i] = dp[i-1] + dp[i-2]`。但它在數所有可能的路徑，每條路徑背後都是一個「走 1 步還是走 2 步」的選擇。加法是兩個選擇的方案數之和。

---

## 怎麼判斷一道題該用哪個？

先看這一步「變化的部分」是什麼。

**加一個數就能更新結果？** → Prefix sum。區間和、區間乘積、前綴 XOR。特徵：結果是所有元素的某種「可逆運算」的累積。

**加一個、減一個就能更新結果？** → Sliding window。連續 subarray/substring 的性質。特徵：窗口內的狀態可以 O(1) 增量維護。

**新元素進來，能確定哪些舊元素永遠不會再被需要？** → Monotonic stack。下一個更大/更小元素、直方圖最大矩形。特徵：答案跟「最近的比我大/小的元素」有關。

**需要在多個子問題的結果中做選擇？** → DP。最長、最短、最多、最少、有幾種方法。特徵：暴力 DFS 有重複子問題，加 memo 就變 DP。

---

## 容易混淆的邊界

### Kadane's algorithm 是 DP 還是 sliding window？

是 DP。

看起來像 sliding window：也在掃一遍陣列，也在維護一個「當前的 subarray」。但 Kadane 每一步都在跑 `max(nums[i], dp[i-1]+nums[i])`，做「接著還是重來」的決策。Sliding window 的左邊界收縮是被動的（不滿足條件才縮），Kadane 的重新開始是主動的（因為接著走更差）。

### Prefix sum + sliding window 算什麼？

[209 Minimum Size Subarray Sum](/problem/minimum-size-subarray-sum) 可以用 prefix sum 解，也可以用 sliding window 解。兩者不衝突。Prefix sum 是預處理手段，sliding window 是搜尋策略。一個準備資料，一個搜尋答案。

### Monotonic stack 跟 greedy 的關係？

Monotonic stack 的「淘汰」判斷本質上是 greedy：如果新元素比 stack top 更好，stack top 就不可能再是答案。這種局部最優能推出全局最優，才能用 monotonic stack。如果不行（比如需要回頭考慮被淘汰的元素），就得用 DP。

---

## 總結

| Pattern | 增量方式 | 有沒有決策 | 典型信號 |
|---|---|---|---|
| Prefix Sum | 累積：加一個數 | 沒有 | 區間和、區間乘積 |
| Sliding Window | 右加左減 | 沒有 | 連續 subarray/substring |
| Monotonic Stack | 淘汰無用候選 | 沒有（greedy 判斷） | 下一個更大/更小 |
| DP | 狀態轉移 | **有** | 最長/最短/幾種方法 |

四個都是「從上一步接著算」，但只有 DP 在每一步做選擇。判斷一道題用哪個，就回到同一個問題：這一步「變化的部分」在做什麼？純累積、加減兩端、淘汰候選，還是做選擇。上面那張表就是這四個答案。

---

## 高頻題清單

### Prefix Sum

| 題目 | 核心 |
|------|------|
| [#238 Product of Array Except Self](/problem/product-of-array-except-self) | 左右 prefix product |
| [#209 Minimum Size Subarray Sum](/problem/minimum-size-subarray-sum) | prefix sum + sliding window 交叉 |
| [#307 Range Sum Query - Mutable](/problem/range-sum-query-mutable) | prefix sum 進階，segment tree |

### Sliding Window

| 題目 | 核心 |
|------|------|
| [#643 Maximum Average Subarray I](/problem/maximum-average-subarray-i) | 固定窗口，右加左減 |
| [#3 Longest Substring Without Repeating Characters](/problem/longest-substring-without-repeating-characters) | 可變窗口，hashmap 記重複 |
| [#424 Longest Repeating Character Replacement](/problem/longest-repeating-character-replacement) | 可變窗口，maxFreq 技巧 |
| [#567 Permutation in String](/problem/permutation-in-string) | 固定窗口 + hashmap 比對 |
| [#76 Minimum Window Substring](/problem/minimum-window-substring) | 可變窗口，覆蓋所有字元 |
| [#239 Sliding Window Maximum](/problem/sliding-window-maximum) | 固定窗口 + monotonic deque |

### Monotonic Stack

| 題目 | 核心 |
|------|------|
| [#739 Daily Temperatures](/problem/daily-temperatures) | 下一個更大的元素 |
| [#853 Car Fleet](/problem/car-fleet) | 排序 + stack 模擬追趕 |
| [#84 Largest Rectangle in Histogram](/problem/largest-rectangle-in-histogram) | 左右邊界用 monotonic stack |
| [#42 Trapping Rain Water](/problem/trapping-rain-water) | monotonic stack 或雙指標 |

### DP（增量決策型）

| 題目 | 核心 |
|------|------|
| [#70 Climbing Stairs](/problem/climbing-stairs) | dp[i-1] + dp[i-2] |
| [#53 Maximum Subarray](/problem/maximum-subarray) | Kadane's，接著或重來 |
| [#198 House Robber](/problem/house-robber) | 搶或不搶 |
| [#152 Maximum Product Subarray](/problem/maximum-product-subarray) | 同時追蹤最大最小 |
| [#300 Longest Increasing Subsequence](/problem/longest-increasing-subsequence) | 以 i 結尾的 LIS |
