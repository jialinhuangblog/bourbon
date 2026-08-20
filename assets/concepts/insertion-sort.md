---
title: "Insertion Sort"
category: Sorting
slug: insertion-sort
subtitle: 像整理手牌，插到對的位置
date: 2026-02-28T11:51:31
updated: 2026-08-04
---

# Insertion Sort

打撲克牌的時候，抽到一張新牌，會直接把它插進手上已經排好的牌裡，不會整副重排。這就是 insertion sort。

---

## 範例資料 `[1, 4, 2, 6, 3, 4, 3, 1, 7]`

這組資料有重複值（兩個 1、兩個 3、兩個 4），有些元素離正確位置近，有些離很遠（最後面的 1 要搬到最前面）。

這組資料同時呈現 insertion sort 的兩面：

- **近的元素**：4（i=1）、6（i=3）、7（i=8）幾乎不用搬。內層迴圈跑 0 次。
- **遠的元素**：最後的 1（i=7）要推過 6 張牌。內層迴圈跑 6 次。

兩個 4、兩個 3、兩個 1 是拿來看穩定性的：排完之後它們維持原本的先後。穩定性怎麼來的，[Bubble Sort](/concept/bubble-sort) 那篇講得比較完整。

Insertion sort 的隱藏優勢：**對幾乎排好的資料，它是 $O(n)$**。每個元素離正確位置不超過 k 格，內層每輪最多跑 k 次，總共 $O(nk)$。

對比：**selection sort 不管資料排沒排好，照跑 $n^2$ 次比較。**

對比：**quick sort 碰到幾乎排好的資料，如果 pivot 選最右 → 退化成 $O(n^2)$。**

對比：**merge sort 照跑 $O(n \log n)$。** 資料排沒排好都一樣，它就是要切一半、合一半。

只有 insertion sort 能真正利用「大部分元素已經在附近」這個事實。

---

## 直覺

想像桌上有一排面朝上的牌：`1, 4, 2, 6, 3, 4, 3, 1, 7`。

你左手是「已排好的區域」。一開始只有第一張牌。

每次從桌上拿一張新的，在左手從右往左掃，找到它該在的位置，插進去。

就這樣，沒有遞迴也沒有分治，一張一張插進去而已。

---

## 程式碼

自己拖，或按播放。深色那段是已經排好的，虛線框是「這張牌被拿起來之後留下的洞」。

標著「手上」的那張牌不在陣列裡。`const key = nums[i]` 把值抄走之後，index i 那一格就沒有東西了，畫面上是虛線框，不是 0，也不是原本的值。牌懸在洞的上方，高度照它自己的值畫。

洞還會往左走。跑到 index 4 的 3 那一輪：拿起 3，洞在 index 4；6 往右搬到 index 4，洞變成 3；4 往右搬到 index 3，洞變成 2；2 不用讓位，停，3 填進 index 2。所以 `nums[j + 1] = key` 的 j+1 講的是洞現在在哪，不是這張牌原本在哪。

<div data-algo-viz="insertion-sort" data-input="1,4,2,6,3,4,3,1,7"></div>

```typescript
function insertionSort(nums: number[]) {
  for (let i = 1; i < nums.length; i++) {
    const key = nums[i];                  // 把這張牌拿起來，原位置空出來了
    let j = i - 1;
    while (j >= 0 && nums[j] > key) {     // 從右往左看，比 key 大的？
      nums[j + 1] = nums[j];              // 往右覆寫一格（不是 swap，是搬家）
      j--;                                 // 繼續往左看
    }
    // 迴圈停下來：nums[j] <= key，或 j < 0（到頭了）
    // j+1 就是空出來的洞，把 key 放進去
    nums[j + 1] = key;
  }
}
```

### 外層迴圈

`for i := 1; i < len(nums); i++`。每次拿一張新牌。從 index 1 開始，因為 index 0 天生就是「已排好」。一張牌不需要排。

### 內層迴圈

`for j >= 0 && nums[j] > key`。從右往左掃。碰到比 `key` 大的，往右推一格。碰到比 `key` 小的（或到頭了），停下來。

條件是 `>` 不是 `>=`，相等時不推，所以相等的元素維持原本的相對順序，這就是它穩定的來源。

### 放牌

`nums[j+1] = key`。把 `key` 放到空出來的位置。

不需要 swap，只有往右推跟最後那一次寫入，比 bubble sort 每次都交換省。

---

## 走一遍 `[1, 4, 2, 6, 3, 4, 3, 1, 7]`

上面那個播放器就是這組資料。按下一步走完 52 步，會看到 8 輪外層迴圈：

```
每輪推了幾次：0 + 1 + 0 + 2 + 1 + 3 + 6 + 0 = 13 次
排完：[1, 1, 2, 3, 3, 4, 4, 6, 7]
```

13 次 shift。如果資料完全倒序（worst case），會是 $8+7+6+5+4+3+2+1 = 36$ 次。這組資料不算最差，但那張離家最遠的 `1` 自己就貢獻了 6 次。

i=7 那輪，`1` 從最右邊一路擠到最前面，推了 6 次。還有 i=5 跟 i=6，碰到相等的值就停，後來的那個 4 排在原本那個 4 後面，這是穩定性。

---

## 為什麼幾乎排好的資料這麼快？

如果每個元素離正確位置不超過 k 格，內層迴圈每次最多跑 k 次。外層跑 n 次，總共 $O(nk)$。

當 k 是常數（比如每個元素最多偏 1 格），$O(nk) = O(n)$。線性時間。

這組資料裡，大部分元素偏 1-2 格，只有最後的 `1` 偏了 6 格。所以總共 13 次，遠低於 worst case 的 36 次。

現實世界的資料很常是「幾乎排好」的：

- log 按時間戳排好，偶爾有幾筆亂序
- 名字大致按字母序，偶爾插了幾個
- 上一次排好之後，多了幾筆新資料

這些場景，insertion sort 都是接近 $O(n)$。

---

## 小陣列它最快，但「小」是多小

理論上 quick sort 是 $O(n \log n)$，insertion sort 是 $O(n^2)$。Quick sort 應該贏。

但理論忽略了常數。

Quick sort 每次要選 pivot、做 partition、遞迴呼叫。光是 function call 的開銷，n 小的時候就比實際排序還貴。

Insertion sort 呢？一個 for 迴圈套一個 for 迴圈。沒有遞迴。沒有 function call。存取模式是連續的，CPU cache 命中率極高。

那個交叉點在哪，各家實作自己量出來的答案不一樣：

| 實作 | 閾值 | 出處 |
|---|---|---|
| Go `sort`（pdqsort） | 寫死 12 | `src/sort/zsortfunc.go`，`const maxInsertion = 12` |
| CPython TimSort | 動態，上限 64 | `Objects/listobject.c`，`#define MAX_MINRUN 64`，實際的 minrun 由 `minrun_next()` 依 list 長度算 |

CPython 的數字常被轉述成 32，那個數字出自 `Objects/listsort.txt` 裡的「We pick 32 as a good value in the sweet range」，但同一份文件後面就說固定 32 會讓合併的兩堆大小不平衡，所以改成每次算一個略有變化的 minrun。整個 list 短於 64 的話，minrun 等於 list 長度，也就是整段 binary insertion sort 排完，不進入合併。

手上那個語言切在哪，去原始碼找那個常數就知道，不要相信別人轉述的數字，包括這篇。

---

## 標準函式庫的排序，小區間交給 insertion sort

主流語言的標準排序沒有一個是單一演算法，全是混合的。下面這四種混合排序，處理小區間的那一段都用 insertion sort。

| 主演算法 | 誰在用 | 小區間交給誰 |
|---|---|---|
| pdqsort | Go 1.19+、Rust `sort_unstable()` | insertion sort，Go 寫死 12 |
| TimSort | CPython、JS、Java 的物件陣列 | binary insertion sort，把太短的 run 補到 minrun 長 |
| Dual-pivot quicksort | Java 的 primitive 陣列 | insertion sort |
| Introsort | C++ `std::sort()` | insertion sort |

TimSort 那條先掃出資料裡本來就遞增的片段，短的用 insertion sort 補長，再用 merge sort 把片段合起來。

pdqsort 是 introsort 的升級版，多了 pattern 偵測。Go 換過去是在 1.19，release note 寫「The sorting algorithm has been rewritten to use pattern-defeating quicksort」。

---

## 穩定排序

內層迴圈條件是 `nums[j] > key`，不是 `>=`。相等時不推，所以相等的元素維持原本的先後。

Insertion sort 有得選，是因為它只把元素一格一格往右推，沒有跨距離的搬動。

Selection sort 就沒這個選項。它每輪的 swap 直接跨過中間所有元素，打亂順序的是那次搬動本身，比較符號怎麼寫都一樣。

完整的對照寫在 [Bubble Sort](/concept/bubble-sort) 的穩定排序那節。

---

## 複雜度

| | 值 |
|---|---|
| 最佳 | $O(n)$ — 已排好的資料 |
| 平均 | $O(n^2)$ |
| 最糟 | $O(n^2)$ — 完全倒序 |
| 空間 | $O(1)$ |
| In-place | 是 |
| 穩定 | 是 |

---

## 被問到 insertion sort 有什麼用

面試官問你 insertion sort 有什麼用，別說「教學用」。說：

> 「每個現代語言的標準排序都在小區間用它。因為 n 小的時候，低開銷比低複雜度重要。」

> 「對幾乎排好的資料是 $O(n)$。TimSort 就是利用這個特性。」

手寫一遍 insertion sort 誰都會，講得出它為什麼還活在標準函式庫裡才是差別。

---

## 總結

Insertion sort 長得像 $O(n^2)$ 的笨蛋排序。但它有兩個隱藏技能：

1. 幾乎排好的資料 → $O(n)$
2. 小陣列 → 比 quick sort 快

這兩個技能讓它成為每一個現代混合排序的收尾選擇。

Quick sort 當主力，merge sort 合大區間，heap sort 當保底，短到某個閾值以下的碎片區間全部交給 insertion sort。那個閾值 Go 寫死 12，CPython 依 list 長度算、上限 64，數字是各家自己 benchmark 出來的。

回到 [Sorting 總覽](/concept/sorting) 看全貌。想學把 insertion sort 當收尾的分治排序？看 [Merge Sort](/concept/merge-sort)。
