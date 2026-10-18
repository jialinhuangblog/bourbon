---
title: "Sorting"
category: Sorting
slug: sorting
subtitle: 排序演算法總覽
date: 2026-02-24T17:01:43
updated: 2026-08-04
---

# 排序

面試官問你排序，不是要你手寫 `sort()`。是要看你能不能解釋**為什麼**有這麼多種排序，各自解決了什麼問題。

每一種排序演算法的誕生，都是因為上一種不夠好。這篇是地圖。每個演算法的詳細教學，點進去看。

---

## 最直覺的想法

### [Bubble Sort](/concept/bubble-sort)

兩兩比。大的往右推。跑一輪，最大的到底。再跑一輪，第二大到底。

<details>
<summary>展開逐步播放器</summary>

<div data-algo-viz="bubble-sort" data-input="7,3,12,5,14,2,9,6,11,4,13,8"></div>

</details>


沒人在正式環境用。它教的是排序的兩個基本動作：比較跟交換。後面所有演算法都在優化這兩步。

### [Selection Sort](/concept/selection-sort)

每次掃一遍找最小的，丟到前面。

<details>
<summary>展開逐步播放器</summary>

<div data-algo-viz="selection-sort" data-input="11,4,14,2,9,6,13,3,10,7,12,5"></div>

</details>


給它一個已經排好的陣列，它照樣跑 $n(n-1)/2$ 次比較。唯一的優點是 swap 次數固定 $n-1$ 次。

### [Insertion Sort](/concept/insertion-sort)

像整理撲克牌。拿一張新牌，從右往左找到它該在的位置，插進去。

<details>
<summary>展開逐步播放器</summary>

<div data-algo-viz="insertion-sort" data-input="6,13,2,9,4,14,7,3,11,5,12,8,10,1"></div>

</details>


對幾乎排好的資料極快，$O(n)$。標準排序碰到夠短的區間都會切回它：Go 的 `sort` 寫死 12，CPython 的 TimSort 依 list 長度算，上限 64。

### 這三個的差別在哪

三個都是每輪處理一格、複雜度都 $O(n^2)$，差別在**先固定什麼**：

| | 先固定 | 再找 | 怎麼搬 | 穩定 |
|---|---|---|---|---|
| Bubble Sort | 都不固定 | 相鄰兩格誰大 | 相鄰交換 | 是 |
| Selection Sort | 位置，這輪要填第 i 格 | 剩下的最小值在哪個 index | 一次長距離交換 | 否 |
| Insertion Sort | 值，手上這張牌 | 它該插在左邊哪個位置 | 一路相鄰交換 | 是 |

Selection sort 跟 insertion sort 的方向剛好相反：一個位置固定去找值，一個值固定去找位置。Bubble sort 兩個都不固定，只做「相鄰逆序就換」，掃完一輪最大的落到最右是這個規則的副產品，不是它去找來的。

穩定不穩定，看最後一欄就知道：只做相鄰交換的兩個，相等時不換就保住原順序；會一次跳過中間好幾個元素的那個，怎麼寫比較符號都救不回來。

---

## $O(n^2)$ 太慢。能不能更好？

能。方法是分治：把大問題拆成小問題，各自解決，再合起來。複雜度從 $O(n^2)$ 降到 $O(n \log n)$。

### [Merge Sort](/concept/merge-sort)

最純粹的分治。切一半，左邊排好，右邊排好，合併。

<details>
<summary>展開逐步播放器</summary>

<div data-algo-viz="merge-sort" data-input="9,4,13,6,2,11,7,15,3,10,5,14,8,12,1,16"></div>

</details>


不管資料長什麼樣都保證 $O(n \log n)$，代價是永遠要額外 $O(n)$ 空間。merge 這個函式本身是高頻考題，合併兩個 sorted array、合併 k 個 sorted list 都是它。

### [Quick Sort](/concept/quick-sort)

選一個 pivot。比它小的放左邊，大的放右邊。左右各自遞迴。

<details>
<summary>展開逐步播放器</summary>

<div data-algo-viz="quick-sort" data-input="8,14,3,11,6,15,2,9,13,4,12,7,10,5"></div>

</details>


比 merge sort 省空間，實務上也更快，因為 in-place 掃描對 CPU cache 友善。弱點是最糟 $O(n^2)$。

### [Heap Sort](/concept/heap-sort)

把陣列建成 max heap。最大的一定在 `[0]`。拿出來放最後，再 heapify。重複。

<details>
<summary>展開逐步播放器</summary>

<div data-algo-viz="heap-sort" data-input="6,13,2,10,4,15,7,11,3,14,8,5,12,9,1"></div>

</details>


紙面上 $O(n \log n)$ 保證加 $O(1)$ 空間，但 cache 命中率差。它的角色是保底，不是主力。

---

## 穩定性

穩定排序 = 相等的元素排完之後，**相對順序不變**。

```
排序前: [(A,3), (B,1), (C,3)]  按數字排
穩定:   [(B,1), (A,3), (C,3)]  A 還在 C 前面
不穩定: [(B,1), (C,3), (A,3)]  A 跑到 C 後面了
```

什麼時候重要？**多欄排序**。先按部門排，再按薪水排。如果第二次排序不穩定，第一次的結果就被打亂了。

穩定：Bubble、Insertion、Merge、TimSort。
不穩定：Selection、Quick、Heap、Introsort。

---

## 那 `sort()` 底層到底用什麼？

沒有一個演算法是完美的。所以現代語言混著用。

### Introsort — C++、.NET

三個演算法組在一起：

1. **開局 Quick Sort**。大部分時候最快。
2. **遞迴太深？切 Heap Sort。** 深度超過 $2 \cdot \log(n)$，代表 Quick Sort 踩到爛 case。切 Heap Sort，保底 $O(n \log n)$。
3. **區間很小？切 Insertion Sort。** n 小於某個閾值，Insertion Sort 的低開銷反而贏。Go 的 `sort` 寫死 12（`src/sort/zsortfunc.go` 的 `const maxInsertion = 12`），各家不一樣。

- Time: $O(n \log n)$，永遠
- Space: $O(\log n)$

所以 introsort 平常跟 quick sort 一樣快，碰到爛 case 也不會掉到 $O(n^2)$，小區間還省掉遞迴的開銷。

### TimSort — Python、Java、JavaScript V8

Python 的 `list.sort()`、Java 的 `Arrays.sort(Object[])`、JS 的 `Array.sort()` 都用 TimSort。

設計哲學不同：**現實世界的資料不是隨機的**。

你拿到的資料通常有片段是已排好的。log 按時間排、名字大致按字母序、成績大部分及格只有幾個不及格。TimSort 利用這個特性：

1. **掃描找自然的 run**（連續遞增或遞減的片段）。
2. **短的 run 用 Insertion Sort 補長到 minrun**（通常 32-64）。
3. **用 Merge Sort 的策略合併所有 run**。


幾乎排好的資料？$O(n)$。完全隨機？跟 Merge Sort 一樣 $O(n \log n)$。穩定。代價是 $O(n)$ 空間。

### pdqsort — Go 1.19+、Rust

這是最新的一種，全名 pattern-defeating quicksort。

Introsort 的升級版。除了 Quick Sort + Heap Sort + Insertion Sort，還加了一招：**偵測特殊 pattern**。

- 資料已排好？$O(n)$ 就結束。
- 大量重複值？用 Dutch National Flag 三路分區。
- 隨機資料？照常 Quick Sort。

Go 是 1.19 換過去的，release note 寫「The sorting algorithm has been rewritten to use pattern-defeating quicksort」，改的是 `sort` 套件。泛型的 `slices.Sort()` 要到 1.21 才進標準函式庫，底層同樣是 pdqsort。

---

## 超越比較：$O(n)$ 排序

上面所有演算法都基於「比較」。理論已證明：**比較排序的下界是 $O(n \log n)$**。不可能更快。

但如果不比較呢？

### [Counting Sort](/concept/counting-sort)

知道值的範圍（比如 0-100），開一個計數陣列，掃一遍統計，再按計數輸出。

- Time: $O(n + k)$，k = 值域大小
- Space: $O(k)$

限制：只能排整數，而且 k 不能太大。成績 0-100？完美。年薪 0-$10^9$？別想了。

### [Radix Sort](/concept/radix-sort)

一位一位排。從最低位開始，每一位用 Counting Sort。

<details>
<summary>展開逐步播放器</summary>

<div data-algo-viz="radix-sort" data-input="329,457,657,839,436,720,355,214,908,163,542,781"></div>

</details>

- Time: $O(d \cdot (n + k))$，d = 位數，k = 基數（10）
- Space: $O(n + k)$

32-bit 整數？d = 10，k = 10，等於 $O(10n) = O(n)$。比任何比較排序都快。但常數大、cache 不友善、只能排整數。

---

## 各語言的 sort API

| 語言 | In-place（改原本） | 回傳新的 | 底層演算法 |
|------|-------------------|----------|-----------|
| JavaScript | `arr.sort()` | `arr.toSorted()`（ES2023） | TimSort |
| Python | `list.sort()` | `sorted(iterable)` | TimSort |
| Go | `slices.Sort()` | 自己 clone | pdqsort |
| Java | `Arrays.sort()` | `stream().sorted()` | primitive 走 dual-pivot quicksort，Object 走 TimSort |
| Rust | `vec.sort()` | 自己 clone（`sorted()` 要 itertools） | `sort()` TimSort，`sort_unstable()` pdqsort |
| C++ | `std::sort()` | `std::ranges::to()` 再 sort | Introsort |

三個容易踩到的地方：

JS 的 `sort()` 直接改原陣列，回傳值是同一個 reference，不像 `map()` 給新的。不加比較函式的話它按字串排，`[10, 9]` 會排成 `[10, 9]`。

Python 的 `list.sort()` 回傳 `None` 不是 list，接在變數後面會拿到 None。要新的用 `sorted()`，它還吃任何 iterable。

Go 沒有內建回傳新 slice 的排序，要自己 `slices.Clone()` 再排。穩定排序是 `slices.SortStableFunc()`。

---

## 怎麼選？

不用選。語言幫你選好了。

| 場景 | 用什麼 |
|------|--------|
| 一般排序 | 語言內建 `sort()`，底層已經是混合過的演算法 |
| 需要穩定排序 | Python/JS 預設穩定；Go 用 `SortStableFunc()`；Rust 用 `sort()` |
| 不改原陣列 | Python `sorted()` / JS `toSorted()` / 其他語言先 clone |
| 值域小的整數 | Counting Sort |
| 固定長度的整數/字串 | Radix Sort |
| 幾乎排好的資料 | TimSort 自動處理 |
| 面試手寫 | Quick Sort（最常考）> Merge Sort > Heap Sort |

---

## 哪些 LeetCode 題用到排序

排序本身不是目的。看到「區間」「第 k 個」「重複」「配對」，先想能不能先排一遍，排完之後原本 $O(n^2)$ 的問題常常變成掃一遍就好。

| 題目 | 排序幹嘛 |
|---|---|
| [15. 3Sum](/problem/3sum) | 排好用兩個指標夾，避免重複，$O(n^3)$ → $O(n^2)$ |
| [75. Sort Colors](/problem/sort-colors) | 原地排 0, 1, 2，Dutch National Flag |
| [88. Merge Sorted Array](/problem/merge-sorted-array) | 合併兩個已排好的陣列 |
| [56. Merge Intervals](/problem/merge-intervals) | 按開頭排，重疊的變相鄰，掃一遍就能合併 |
| [253. Meeting Rooms II](/problem/meeting-rooms-ii) | 按開頭排，用 heap 追蹤同時進行的會議數 |
| [435. Non-overlapping Intervals](/problem/non-overlapping-intervals) | 按結尾排，貪心選最早結束的 |
| [853. Car Fleet](/problem/car-fleet) | 按位置排，從最靠近終點的開始掃 |
| [621. Task Scheduler](/problem/task-scheduler) | 按頻率排，最多的先排 |
| [215. Kth Largest Element](/problem/kth-largest-element-in-an-array) | 排好取倒數第 k 個，也可以用 heap |
| [217. Contains Duplicate](/problem/contains-duplicate) | 排好之後重複的相鄰 |
| [347. Top K Frequent Elements](/problem/top-k-frequent-elements) | 按頻率排取前 k 個 |

---

## 總結

**In-place** 意思是排序直接在原陣列上操作，不需要開一份新陣列。空間欄位顯示的是「額外」用了多少記憶體。Quick Sort 的 $O(\log n)$ 是遞迴 call stack，不是開新陣列。Merge Sort 的 $O(n)$ 是合併時需要的暫存空間。

| 演算法 | 最佳 | 平均 | 最糟 | 空間 | In-place | 穩定 | 備註 |
|--------|------|------|------|------|----------|------|------|
| [Bubble Sort](/concept/bubble-sort) | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 是 | 是 | 教學用 |
| [Selection Sort](/concept/selection-sort) | $O(n^2)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 是 | 否 | swap 次數最少 |
| [Insertion Sort](/concept/insertion-sort) | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 是 | 是 | 小資料量、幾乎有序 |
| [Merge Sort](/concept/merge-sort) | $O(n \log n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(n)$ | 否 | 是 | 保證 $O(n \log n)$ |
| [Quick Sort](/concept/quick-sort) | $O(n \log n)$ | $O(n \log n)$ | $O(n^2)$ | $O(\log n)$ | 是 | 否 | 實務最快 |
| [Heap Sort](/concept/heap-sort) | $O(n \log n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(1)$ | 是 | 否 | 保底用 |
| Introsort | $O(n \log n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(\log n)$ | 是 | 否 | Go/C++ 標準 |
| TimSort | $O(n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(n)$ | 否 | 是 | Python/JS/Java 標準 |
| pdqsort | $O(n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(\log n)$ | 是 | 否 | Go 1.19+ / Rust |
| [Counting Sort](/concept/counting-sort) | $O(n+k)$ | $O(n+k)$ | $O(n+k)$ | $O(k)$ | 否 | 是 | 整數、值域小 |
| [Radix Sort](/concept/radix-sort) | $O(dn)$ | $O(dn)$ | $O(dn)$ | $O(n+k)$ | 否 | 是 | 固定長度整數 |

各語言 Array / Slice 的完整操作複雜度，看 [Array 與 Slice 操作複雜度](/concept/array-operations)。
