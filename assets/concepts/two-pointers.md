---
title: "Two Pointers"
category: Algorithms
slug: two-pointers
subtitle: 兩根指標從兩端往中間夾
date: 2026-02-28T11:51:31
---

# Two Pointers

暴力解 array 題，通常是兩層 for 迴圈。$O(n^2)$。

兩層 for 迴圈在做什麼？嘗試所有 (i, j) 的組合。

如果 array 是排好序的，很多組合不需要試。你可以用兩個指標，一左一右，根據條件決定移動哪一個。

一遍就掃完。$O(n)$。

這就是 two pointers。

---

## 三種模式

### 模式一：左右指標（相向而行）

一個從左邊開始，一個從右邊開始。往中間靠攏。

```
[1, 2, 3, 4, 6, 8, 11]
 L                   R
```

用途：sorted array 上找目標和、反轉陣列、container with most water。

### 模式二：同向指標（快慢指標）

兩個都從左邊開始。一個快一個慢。

```
[1, 1, 2, 2, 3]
 s
 f
```

用途：去重、移除元素、linked list 找中點。

### 模式三：合併指標

兩個指標分別在兩個陣列上。

```
[1, 3, 5]    [2, 4, 6]
 i            j
```

用途：merge sorted arrays、intersection。

---

## 模式一：左右指標

### Two Sum II

[#167 Two Sum II - Input Array Is Sorted](/problem/two-sum-ii-input-array-is-sorted)

sorted array `[2, 7, 11, 15]`，找兩數之和 = 9。

暴力：兩層 for。$O(n^2)$。

Two pointers：左右各一個。

```
[2, 7, 11, 15]   target = 9
 L            R

2 + 15 = 17 > 9 → 太大了，R 左移（讓 sum 變小）

[2, 7, 11, 15]
 L       R

2 + 11 = 13 > 9 → 還是太大，R 左移

[2, 7, 11, 15]
 L   R

2 + 7 = 9 → 找到了！
```

**為什麼這樣做是對的？**

`L + R > target` 時，R 左移。這等於跳過了所有 `(L, R)` 的組合，因為 L 只會更大，加起來只會更大。不需要試。

`L + R < target` 時，L 右移。同理，R 只會更小，加起來只會更小。

每一步至少淘汰一個指標的位置。最多走 n 步。$O(n)$。

```go
func twoSum(numbers []int, target int) []int {
    l, r := 0, len(numbers)-1
    for l < r {
        sum := numbers[l] + numbers[r]
        if sum == target {
            return []int{l + 1, r + 1}
        } else if sum < target {
            l++
        } else {
            r--
        }
    }
    return nil
}
```

### 3Sum

[#15 3Sum](/problem/3sum)

找三個數之和 = 0。

先排序。固定一個數 `nums[i]`。剩下的兩個用 two pointers 找。

```
nums = [-1, 0, 1, 2, -1, -4]
排序 → [-4, -1, -1, 0, 1, 2]

i=0: nums[0]=-4, 找 two sum = 4 in [-1,-1,0,1,2]
     L=-1, R=2 → sum=1 < 4 → L++
     L=-1, R=2 → sum=1 < 4 → L++
     L=0, R=2 → sum=2 < 4 → L++
     L=1, R=2 → sum=3 < 4 → L++
     L >= R → 沒找到

i=1: nums[1]=-1, 找 two sum = 1 in [-1,0,1,2]
     L=-1, R=2 → sum=1 = 1 → 找到 [-1,-1,2]
     L++, R--
     L=0, R=1 → sum=1 = 1 → 找到 [-1,0,1]

i=2: nums[2]=-1, 跟 i=1 一樣，跳過（避免重複）
...
```

$O(n^2)$。比暴力的 $O(n^3)$ 好一個量級。

### Container With Most Water

[#11 Container With Most Water](/problem/container-with-most-water)

```
heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]
           L                          R

面積 = min(1, 7) * 8 = 8
L 比較矮 → L 右移（矮的那邊移動，才有機會找到更高的）
```

為什麼移動矮的那邊？因為面積受限於矮的。移動高的那邊，寬度減少，高度最多不變（受矮的限制），面積一定變小。只有移動矮的，高度才可能變大。

---

## 模式二：同向指標

### 移除重複

[#26 Remove Duplicates from Sorted Array](/problem/remove-duplicates-from-sorted-array)

sorted array `[1, 1, 2, 2, 3]`。in-place 去重。

```
[1, 1, 2, 2, 3]
 s  f

nums[f]=1 == nums[s]=1 → f++

[1, 1, 2, 2, 3]
 s     f

nums[f]=2 != nums[s]=1 → s++, nums[s]=nums[f]

[1, 2, 2, 2, 3]
    s     f

nums[f]=2 == nums[s]=2 → f++

[1, 2, 2, 2, 3]
    s        f

nums[f]=3 != nums[s]=2 → s++, nums[s]=nums[f]

[1, 2, 3, 2, 3]
       s        f

f 到底了。s+1 = 3 = 去重後的長度。
```

slow 指標標記「下一個不重複的應該放哪」。fast 指標掃過全部。碰到新的值就寫進 slow 的位置。

```go
func removeDuplicates(nums []int) int {
    if len(nums) == 0 { return 0 }
    s := 0
    for f := 1; f < len(nums); f++ {
        if nums[f] != nums[s] {
            s++
            nums[s] = nums[f]
        }
    }
    return s + 1
}
```

### 移除指定值

[#27 Remove Element](/problem/remove-element)

同樣的套路。碰到不是目標值的就寫進 slow 位置。

```go
func removeElement(nums []int, val int) int {
    s := 0
    for f := 0; f < len(nums); f++ {
        if nums[f] != val {
            nums[s] = nums[f]
            s++
        }
    }
    return s
}
```

---

## 模式三：合併指標

### Merge Sorted Array

[#88 Merge Sorted Array](/problem/merge-sorted-array)

```
nums1 = [1, 2, 3, 0, 0, 0], m = 3
nums2 = [2, 5, 6],           n = 3
```

從後面開始填。比較兩個陣列最後面的，大的先放。

```
i=2, j=2, k=5

nums1[2]=3 vs nums2[2]=6 → 6 大, nums1[5]=6, j--, k--
nums1[2]=3 vs nums2[1]=5 → 5 大, nums1[4]=5, j--, k--
nums1[2]=3 vs nums2[0]=2 → 3 大, nums1[3]=3, i--, k--
nums1[1]=2 vs nums2[0]=2 → 相等, nums1[2]=2, j--, k--
j < 0 → 結束

nums1 = [1, 2, 2, 3, 5, 6]
```

為什麼從後面填？因為 nums1 後面是空的。從前面填會覆蓋掉還沒比較的值。

```go
func merge(nums1 []int, m int, nums2 []int, n int) {
    i, j, k := m-1, n-1, m+n-1
    for i >= 0 && j >= 0 {
        if nums1[i] > nums2[j] {
            nums1[k] = nums1[i]; i--
        } else {
            nums1[k] = nums2[j]; j--
        }
        k--
    }
    for j >= 0 {
        nums1[k] = nums2[j]; j--; k--
    }
}
```

這跟 [Merge Sort](/concept/merge-sort) 的 merge 操作本質相同。

---

## 迴圈條件：`<=` 還是 `<`

兩根指標從兩端往中間走，最後會在同一格交會。**那一格還要不要做最後一輪？** 決定條件寫法。

- `left <= right`：交會那格還算數，要再做一輪
- `left < right`：交會那格不重要，跳過

直覺記法：「**指標指到同一格時，那格的值還要不要處理？要 → `<=`，不要 → `<`**」。

例題對照：

| 題目 | 條件 | 為什麼 |
|---|---|---|
| [#977 Squares of a Sorted Array](/problem/squares-of-a-sorted-array) | `<=` | 奇數長度時中間那格平方還沒填進結果 |
| [#75 Sort Colors](/problem/sort-colors) | `<=` | mid 那格也要分類，不能跳 |
| [#167 Two Sum II](/problem/two-sum-ii-input-array-is-sorted) | `<` | 「兩數」要不同位置，指到同一格沒意義 |
| [#11 Container With Most Water](/problem/container-with-most-water) | `<` | 寬度 = 0 不算容器 |
| [#125 Valid Palindrome](/problem/valid-palindrome) | `<` | 一個字元自己等於自己，不用比 |
| [#15 3Sum](/problem/3sum) | `<` | two-sum 內層也是「兩數不同位置」 |
| [#42 Trapping Rain Water](/problem/trapping-rain-water) | `<` | 一根柱子接不到水 |

奇偶長度的影響：

- **偶數**長度：兩指標自然錯開（最後 left > right），`<` 跟 `<=` 行為一樣
- **奇數**長度：兩指標在中間交會（最後 left == right），這時 `<` 會少做一輪，`<=` 會多做一輪

寫的時候不用判斷奇偶。**先想「同一格那筆要不要算」，答案直接決定條件**。

---

## Two Pointers vs Hash Map

很多題兩種都能解。怎麼選？

| | Two Pointers | Hash Map |
|---|---|---|
| 前提 | 通常需要排好序 | 不需要排序 |
| 空間 | $O(1)$ | $O(n)$ |
| 時間 | $O(n)$ 或 $O(n \log n)$(含排序) | $O(n)$ |
| 適合 | sorted array、in-place | 任意 array、需要查找 |

Two Sum 原版（#1）：array 沒排好。用 hash map。$O(n)$ 時間 $O(n)$ 空間。

Two Sum II（#167）：array 排好了。用 two pointers。$O(n)$ 時間 $O(1)$ 空間。

如果面試官追問「能不能不用額外空間？」→ 排序 + two pointers。

---

## 高頻題清單

| 題目 | 模式 | 核心 |
|------|------|------|
| [#167 Two Sum II](/problem/two-sum-ii-input-array-is-sorted) | 左右 | sum 大了 R 左移，小了 L 右移 |
| [#15 3Sum](/problem/3sum) | 左右 | 固定一個，剩下 two sum |
| [#11 Container With Most Water](/problem/container-with-most-water) | 左右 | 移動矮的那邊 |
| [#42 Trapping Rain Water](/problem/trapping-rain-water) | 左右 | 維護左右最高 |
| [#26 Remove Duplicates](/problem/remove-duplicates-from-sorted-array) | 同向 | slow/fast 去重 |
| [#27 Remove Element](/problem/remove-element) | 同向 | slow/fast 過濾 |
| [#283 Move Zeroes](/problem/move-zeroes) | 同向 | 非零的寫到前面 |
| [#88 Merge Sorted Array](/problem/merge-sorted-array) | 合併 | 從後往前填 |
| [#125 Valid Palindrome](/problem/valid-palindrome) | 左右 | 左右比，跳過非字母 |
| [#75 Sort Colors](/problem/sort-colors) | 三指標 | Dutch National Flag |

---

## 總結

Two pointers 做一件事：**用兩個指標的移動取代兩層 for 迴圈。**

左右指標：sorted array 上找配對。$O(n^2)$ → $O(n)$。
同向指標：in-place 去重、過濾。$O(n)$ 時間 $O(1)$ 空間。
合併指標：merge 兩個 sorted array。

看到 sorted array + 找配對 → 左右指標。
看到 in-place + 去重/過濾 → 同向指標。
看到 merge → 合併指標。

想學 two pointers 的進階版（兩個指標圍出窗口在 array 上滑動）？看 [Sliding Window](/concept/sliding-window)。
