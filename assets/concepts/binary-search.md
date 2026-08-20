---
title: "Binary Search"
category: Algorithms
slug: binary-search
subtitle: 砍一半再砍一半，O(log n)
date: 2026-02-28T11:51:31
---

# Binary Search

你要在字典裡找「zebra」。你不會從 A 翻起。

你翻到中間。M 開頭。Zebra 在後半。翻後半的中間。T 開頭。還是太前。再翻一半。Z 開頭。找到了。

每次砍掉一半。1000 頁的字典只要翻 10 次。一百萬頁也只要 20 次。

$O(\log n)$。

---

## 前提：排好序

Binary search 只能用在已排好的資料上。沒排好，你翻到中間看到 M，不知道 Z 在左邊還是右邊。

LeetCode 的題目描述裡出現 "sorted array"，通常就是在提示 binary search。

---

## 基本模板

```go
func binarySearch(nums []int, target int) int {
    lo, hi := 0, len(nums)-1

    for lo <= hi {
        mid := lo + (hi-lo)/2       // 防溢位

        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            lo = mid + 1
        } else {
            hi = mid - 1
        }
    }
    return -1
}
```

### 三個細節

**1. `lo + (hi-lo)/2` 不是 `(lo+hi)/2`**

`lo + hi` 可能溢位（在 Java/C++ 裡）。`lo + (hi-lo)/2` 不會。

JavaScript 的 Number 夠大，不會溢位。但養成好習慣。

**2. `lo <= hi` 不是 `lo < hi`**

`<=` 代表搜尋區間 `[lo, hi]` 是閉區間。`lo == hi` 時還有一個元素沒看。

**3. `lo = mid + 1` 和 `hi = mid - 1`**

mid 已經看過了，不是答案。下一輪不包含 mid。

---

## 走一遍 `[1, 3, 5, 7, 9, 11]`，找 7

```
lo=0, hi=5
  mid = 2, nums[2]=5 < 7 → lo = 3

lo=3, hi=5
  mid = 4, nums[4]=9 > 7 → hi = 3

lo=3, hi=3
  mid = 3, nums[3]=7 = 7 → 找到！return 3
```

三步。6 個元素只比較了 3 次。

暴力從頭掃要比較 4 次。差距不大。但如果是 100 萬個元素：暴力平均 50 萬次，binary search 最多 20 次。

---

## 找邊界：左邊界和右邊界

基本 binary search 找到一個就回傳。但如果有重複的呢？

`[1, 2, 2, 2, 3]`，找 2。左邊界是 index 1，右邊界是 index 3。

### 找左邊界

找到 target 不停。繼續往左找。

```go
func leftBound(nums []int, target int) int {
    lo, hi := 0, len(nums)-1
    result := -1

    for lo <= hi {
        mid := lo + (hi-lo)/2
        if nums[mid] == target {
            result = mid       // 記下來，但繼續往左找
            hi = mid - 1
        } else if nums[mid] < target {
            lo = mid + 1
        } else {
            hi = mid - 1
        }
    }
    return result
}
```

找到 target 時不 return。改 `hi = mid - 1`，繼續往左邊找更小的 index。

### 找右邊界

找到 target 不停。繼續往右找。

```go
func rightBound(nums []int, target int) int {
    lo, hi := 0, len(nums)-1
    result := -1

    for lo <= hi {
        mid := lo + (hi-lo)/2
        if nums[mid] == target {
            result = mid       // 記下來，但繼續往右找
            lo = mid + 1
        } else if nums[mid] < target {
            lo = mid + 1
        } else {
            hi = mid - 1
        }
    }
    return result
}
```

### 走一遍：`[1, 2, 2, 2, 3]` 找 2 的左邊界

```
lo=0, hi=4
  mid=2, nums[2]=2 → result=2, hi=1

lo=0, hi=1
  mid=0, nums[0]=1 < 2 → lo=1

lo=1, hi=1
  mid=1, nums[1]=2 → result=1, hi=0

lo=1, hi=0 → 結束

左邊界 = 1
```

[#34 Find First and Last Position of Element in Sorted Array](/problem/find-first-and-last-position-of-element-in-sorted-array) 就是找左邊界 + 右邊界。

---

## 進階：搜尋空間不是陣列

Binary search 不只能搜尋陣列。任何有單調性的序列或答案空間都能二分。

### 搜尋答案

[#875 Koko Eating Bananas](/problem/koko-eating-bananas)

Koko 每小時吃 k 根香蕉。有 h 小時。最少要多快（最小的 k）才能吃完？

k 越大，吃越快，越容易吃完。k 越小，越可能吃不完。單調的。

搜尋空間：k 從 1 到 max(piles)。

```go
func minEatingSpeed(piles []int, h int) int {
    lo, hi := 1, max(piles)

    for lo < hi {
        mid := lo + (hi-lo)/2
        if canFinish(piles, mid, h) {
            hi = mid           // 吃得完，試更慢的
        } else {
            lo = mid + 1       // 吃不完，要更快
        }
    }
    return lo
}

func canFinish(piles []int, speed, h int) bool {
    hours := 0
    for _, p := range piles {
        hours += (p + speed - 1) / speed    // 向上取整
    }
    return hours <= h
}
```

不是在陣列裡找值。是在「所有可能的速度」裡找最小的那個。

### 搜尋平方根

[#69 Sqrt(x)](/problem/sqrtx)

找最大的 n 使得 n*n <= x。

```go
func mySqrt(x int) int {
    lo, hi := 0, x
    for lo <= hi {
        mid := lo + (hi-lo)/2
        if mid*mid <= x {
            lo = mid + 1
        } else {
            hi = mid - 1
        }
    }
    return hi
}
```

搜尋空間是 0 到 x。每次砍一半。$O(\log x)$。

---

## 常見的坑

### `lo < hi` vs `lo <= hi`

| | `lo <= hi` | `lo < hi` |
|---|---|---|
| 搜尋區間 | `[lo, hi]` 閉區間 | `[lo, hi)` 左閉右開 |
| 結束時 | lo = hi + 1 | lo = hi |
| mid 更新 | `lo=mid+1, hi=mid-1` | `lo=mid+1, hi=mid` |

兩種都能解題。選一種，一直用。我推薦 `lo <= hi`（閉區間），比較直覺。

### 無窮迴圈

`lo < hi` + `lo = mid`（沒有 +1）可能變無窮迴圈。因為 `lo = hi - 1` 時 `mid = lo`，lo 不動，永遠跳不出去。

**mid 看過了就要排除掉**，`lo = mid + 1` 或 `hi = mid - 1` 少寫一邊就會卡在原地。

---

## Binary Search 的威力

| 暴力 | Binary Search | 差距 |
|------|--------------|------|
| $O(n)$ | $O(\log n)$ | $n=10^6$ → 1,000,000 vs 20 |
| $O(n^2)$ | $O(n \log n)$ | 排序 + binary search |
| $O(n \cdot k)$ | $O(n \cdot \log k)$ | 搜尋答案空間 |

log n 成長極慢。n 從一百萬增加到十億，log n 只從 20 增加到 30。

---

## 高頻題清單

### 基本搜尋

| 題目 | 核心 |
|------|------|
| [#704 Binary Search](/problem/binary-search) | 標準模板 |
| [#34 Find First and Last Position](/problem/find-first-and-last-position-of-element-in-sorted-array) | 左邊界 + 右邊界 |
| [#35 Search Insert Position](/problem/search-insert-position) | 找第一個 >= target 的 |

### 變形

| 題目 | 核心 |
|------|------|
| [#33 Search in Rotated Sorted Array](/problem/search-in-rotated-sorted-array) | 一半有序，判斷在哪一半 |
| [#153 Find Minimum in Rotated Sorted Array](/problem/find-minimum-in-rotated-sorted-array) | 找旋轉點 |
| [#162 Find Peak Element](/problem/find-peak-element) | 比較 mid 和 mid+1 |
| [#74 Search a 2D Matrix](/problem/search-a-2d-matrix) | 把 2D 當 1D |

### 搜尋答案

| 題目 | 核心 |
|------|------|
| [#69 Sqrt(x)](/problem/sqrtx) | 找最大的 n 使 $n^2$ <= x |
| [#875 Koko Eating Bananas](/problem/koko-eating-bananas) | 最小速度 |
| [#1011 Capacity To Ship Packages](/problem/capacity-to-ship-packages-within-d-days) | 最小載重 |
| [#410 Split Array Largest Sum](/problem/split-array-largest-sum) | 最小化最大子陣列和 |

---

## 總結

Binary search 做一件事：**每次砍掉一半。**

在 sorted array 裡找值。在答案空間裡找最佳解。在旋轉陣列裡找斷點。

找一個條件，讓左邊全不滿足、右邊全滿足（或反過來），再二分找那個分界點。

看到 sorted → binary search。
看到「最小的最大值」「最大的最小值」→ 二分搜尋答案。
