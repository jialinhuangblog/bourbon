---
title: "Sliding Window"
category: Algorithms
slug: sliding-window
subtitle: 視窗滑過去，O(n) 解區間問題
date: 2026-02-28T11:51:31
---

# Sliding Window

Two pointers 是兩個指標在 array 上走。

Sliding window 是兩個指標圍出一個窗口，窗口在 array 上滑動。

```
[a, b, c, d, e, f, g]
    [-----]
    L     R
```

窗口裡的元素就是當下在看的那段 subarray 或 substring。右邊擴大，左邊收縮。

每次只加一個、減一個，不用重新算整個窗口，所以整趟是 $O(n)$。

---

## 兩種窗口

### 固定大小

窗口大小固定 = k。每次右移一格，左邊也右移一格。

```
k = 3
[1, 3, 2, 6, -1, 4, 1, 8, 2]
[1, 3, 2]                      sum = 6
   [3, 2, 6]                   sum = 6 - 1 + 6 = 11
      [2, 6, -1]               sum = 11 - 3 + (-1) = 7
```

每次只做一次加法一次減法，不用重新把 k 個數加一遍。

### 可變大小

窗口大小不固定。右邊一直擴大，直到不滿足條件。然後左邊收縮，直到重新滿足條件。

```
找最短的 subarray 使得 sum >= 7

[2, 3, 1, 2, 4, 3]
[2, 3, 1, 2]          sum=8 >= 7 → 記錄長度 4，收縮左邊
   [3, 1, 2]          sum=6 < 7 → 擴大右邊
   [3, 1, 2, 4]       sum=10 >= 7 → 記錄長度 4，收縮
      [1, 2, 4]       sum=7 >= 7 → 記錄長度 3，收縮
         [2, 4]       sum=6 < 7 → 擴大
         [2, 4, 3]    sum=9 >= 7 → 記錄長度 3，收縮
            [4, 3]    sum=7 >= 7 → 記錄長度 2 → 最短！
```

---

## 可變窗口的模板

可變窗口的題目大多是這個結構：

```go
func slidingWindow(s string) int {
    window := map[byte]int{}    // 窗口裡的狀態
    left := 0
    result := 0

    for right := 0; right < len(s); right++ {
        // 1. 右邊的元素進入窗口
        window[s[right]]++

        // 2. 不滿足條件時，收縮左邊
        for /* 窗口不合法 */ {
            window[s[left]]--
            left++
        }

        // 3. 更新結果
        result = max(result, right-left+1)
    }
    return result
}
```

三步：擴大、收縮、更新。

---

## 經典一：最長不重複子串

[#3 Longest Substring Without Repeating Characters](/problem/longest-substring-without-repeating-characters)

`"abcabcbb"` → 最長不重複子串是 `"abc"`，長度 3。

```
s = "abcabcbb"

right=0: 'a' 進入    window: {a:1}       left=0  len=1
right=1: 'b' 進入    window: {a:1,b:1}   left=0  len=2
right=2: 'c' 進入    window: {a:1,b:1,c:1} left=0 len=3
right=3: 'a' 進入    window: {a:2,b:1,c:1} ← a 重複了！
  收縮：移除 s[0]='a'  window: {a:1,b:1,c:1} left=1 len=3
right=4: 'b' 進入    window: {a:1,b:2,c:1} ← b 重複了！
  收縮：移除 s[1]='b'  window: {a:1,b:1,c:1} left=2 len=3
right=5: 'c' 進入    window: {a:1,b:1,c:2} ← c 重複了！
  收縮：移除 s[2]='c'  window: {a:1,b:1,c:1} left=3 len=3
right=6: 'b' 進入    window: {a:1,b:2,c:1} ← b 重複了！
  收縮：移除 s[3]='a'  window: {b:2,c:1} left=4 ← 還重複
  收縮：移除 s[4]='b'  window: {b:1,c:1} left=5  len=2
right=7: 'b' 進入    window: {b:2,c:1} ← 重複
  收縮：移除 s[5]='c'  window: {b:2} left=6 ← 還重複
  收縮：移除 s[6]='b'  window: {b:1} left=7  len=1

最長 = 3
```

```go
func lengthOfLongestSubstring(s string) int {
    window := map[byte]int{}
    left, result := 0, 0

    for right := 0; right < len(s); right++ {
        window[s[right]]++

        for window[s[right]] > 1 {    // 有重複
            window[s[left]]--
            left++
        }

        if right-left+1 > result {
            result = right - left + 1
        }
    }
    return result
}
```

---

## 經典二：最小覆蓋子串

[#76 Minimum Window Substring](/problem/minimum-window-substring)

`s = "ADOBECODEBANC"`, `t = "ABC"`。找 s 中包含 t 所有字元的最短子串。

```
need: {A:1, B:1, C:1}   ← t 需要的
window: {}                ← 當前窗口有的
matched = 0               ← 滿足了幾個字元

right=0: 'A' 進入  window: {A:1}  matched=1
right=1: 'D'       window: {A:1,D:1}
right=2: 'O'       ...
right=3: 'B'       window: {A:1,B:1,...}  matched=2
right=4: 'E'       ...
right=5: 'C'       window: {A:1,B:1,C:1,...}  matched=3 → 全部滿足！

窗口 "ADOBEC" 包含 ABC。記錄長度 6。

收縮左邊：
  移除 'A' → matched=2 → 不滿足了，停止收縮。

right 繼續擴大...

right=10: 'A'      matched=3 → 又滿足了！
窗口 "CODEBA" 長度 6。

收縮：移除 'C' → matched=2 → 停。
...

right=12: 'C'      matched=3
窗口 "BANC" 長度 4 → 最短！
```

```go
func minWindow(s string, t string) string {
    need := map[byte]int{}
    for i := range t { need[t[i]]++ }

    window := map[byte]int{}
    left, matched := 0, 0
    start, minLen := 0, len(s)+1

    for right := 0; right < len(s); right++ {
        c := s[right]
        window[c]++
        if window[c] == need[c] { matched++ }

        for matched == len(need) {
            if right-left+1 < minLen {
                minLen = right - left + 1
                start = left
            }
            d := s[left]
            if window[d] == need[d] { matched-- }
            window[d]--
            left++
        }
    }

    if minLen > len(s) { return "" }
    return s[start : start+minLen]
}
```

這是 sliding window 的 hard 題。但結構跟模板一模一樣：擴大右邊、收縮左邊、更新結果。

---

## 經典三：固定窗口 — 最大平均值

[#643 Maximum Average Subarray I](/problem/maximum-average-subarray-i)

```
nums = [1, 12, -5, -6, 50, 3], k = 4
```

```go
func findMaxAverage(nums []int, k int) float64 {
    sum := 0
    for i := 0; i < k; i++ { sum += nums[i] }
    maxSum := sum

    for i := k; i < len(nums); i++ {
        sum += nums[i] - nums[i-k]    // 加右邊，減左邊
        if sum > maxSum { maxSum = sum }
    }

    return float64(maxSum) / float64(k)
}
```

跟前面固定窗口那節同一個作法，一次加法一次減法，$O(n)$。

---

## Sliding Window vs Two Pointers

Sliding window 是 two pointers 的特化版。

| | Two Pointers | Sliding Window |
|---|---|---|
| 指標方向 | 相向或同向 | 同向（left <= right） |
| 關注的 | 兩個端點的值 | 窗口裡的全部內容 |
| 狀態 | 通常不需要 | 需要維護窗口狀態（count、sum） |
| 典型題 | two sum、container | substring、subarray |

要是題目問的是 **subarray** 或 **substring** 的某種性質（最長、最短、包含什麼），就往 sliding window 想。

要是問的是**兩個端點**的關係（和等於多少、面積多大），就往 two pointers 想。

---

## 怎麼判斷用 sliding window？

問自己兩個問題：

1. 題目要找的是 **連續** subarray/substring 嗎？
2. 當窗口擴大時，可以 **增量更新** 狀態嗎？（加一個元素、減一個元素）

兩個都是 → sliding window。

如果不是連續的（比如 subsequence），那不是 sliding window。可能是 DP 或 two pointers。

---

## 高頻題清單

### 可變窗口

| 題目 | 核心 |
|------|------|
| [#3 Longest Substring Without Repeating Characters](/problem/longest-substring-without-repeating-characters) | 窗口內不能有重複字元 |
| [#76 Minimum Window Substring](/problem/minimum-window-substring) | 窗口必須包含 t 的所有字元 |
| [#209 Minimum Size Subarray Sum](/problem/minimum-size-subarray-sum) | 窗口 sum >= target |
| [#424 Longest Repeating Character Replacement](/problem/longest-repeating-character-replacement) | 窗口內最多替換 k 個 |
| [#567 Permutation in String](/problem/permutation-in-string) | 窗口是 s1 的排列 |
| [#438 Find All Anagrams in a String](/problem/find-all-anagrams-in-a-string) | 窗口是 p 的 anagram |

### 固定窗口

| 題目 | 核心 |
|------|------|
| [#643 Maximum Average Subarray I](/problem/maximum-average-subarray-i) | 固定 k，找最大 sum |
| [#239 Sliding Window Maximum](/problem/sliding-window-maximum) | 固定 k，用 deque 找 max |

---

## 總結

Sliding window 做一件事：**用 $O(1)$ 的增量更新取代 $O(k)$ 的重新計算。**

看到 subarray/substring + 最長/最短/包含 → sliding window。

模板是三步：擴大右邊、收縮左邊、更新結果。差別只在「不合法的條件」是什麼。
