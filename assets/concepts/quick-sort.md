---
title: "Quick Sort"
category: Sorting
slug: quick-sort
subtitle: 選 pivot，左小右大，遞迴
date: 2026-02-28T11:51:31
updated: 2026-08-03
---

# Quick Sort

每個學排序的人都會問同一個問題：merge sort 已經 $O(n \log n)$ 了，為什麼還要學別的？

因為 merge sort 要開新陣列。每次 merge 都要分配記憶體。記憶體分配不是免費的。

Quick sort 不開新陣列。它在原地搞定一切。而且實務上，它比 merge sort 快。

---

## 選一個人當標準

想像你是老師。你要幫全班排身高。

你隨便叫一個人站出來。叫他「pivot」。

比他矮的站左邊。比他高的站右邊。一輪下來，pivot 站在他最終的位置上了。

左邊那群人？再叫一個人當 pivot，重複。右邊也一樣。

一直拆，拆到每一邊只剩一個人。排序就完成了。

---

## 範例資料 `[10, 7, 8, 9, 1, 5]`

這組資料是隨機亂序。沒有排好，也沒有倒序。

pivot 選最後一個 = 5。比 5 小的只有 1。比 5 大的有 10, 7, 8, 9。

一次 partition 之後，陣列變成 `[1, 5, 8, 9, 10, 7]`。5 的左邊 1 格，右邊 4 格。不完美（完美是 3:3），但夠看。右邊那四格的順序跟原本不一樣，因為 Lomuto 只保證它們都不小於 pivot，不保證順序。

這組資料看得到 quick sort 每一輪在做什麼：**一次 partition 就把陣列分成兩邊，pivot 直接歸位**。

對比：**merge sort 不管三七二十一，永遠從中間切。** 切的位置跟資料無關。Quick sort 的切法取決於 pivot 的值。選得好，兩邊差不多大。選得差，一邊很大一邊很小。

對比：**insertion sort 碰到這組完全亂序的資料，每張牌都要往左推好幾格。** $O(n^2)$ 跑滿。

對比：**bubble sort 也是 $O(n^2)$ 跑滿。** 每輪只推一個元素到底。

Quick sort 一輪就把所有元素分成「比 pivot 小」和「比 pivot 大」兩群。這是分治的威力。

---

## Lomuto Partition：最直覺的分法

有很多種 partition。面試最常考的叫 Lomuto partition。因為它最好懂。

規則：
1. 選最後一個元素當 pivot
2. 用一個指標 `i` 記錄「下一個小的該放哪」
3. 從左掃到右，碰到比 pivot 小的，就跟 `i` 的位置交換，`i` 往右移一格
4. 掃完之後，把 pivot 跟 `i` 的位置交換。pivot 歸位。

自己拖，或按播放。深色是已經歸位、之後不會再動的格子，藍色是 pivot，橘色是這一步動到的格子（比較的時候一格，swap 的時候兩格），有外框的是 `i` 停的位置。下面那條線標出這一層遞迴負責的 range。

<div data-algo-viz="quick-sort" data-input="10,7,8,9,1,5"></div>

```typescript
function quickSort(nums: number[], lo: number, hi: number) {
  if (lo >= hi) return;

  const pivot = nums[hi];                  // 選最後一個當 pivot
  let i = lo;                              // i = 下一個「小的」該放的位置
  for (let j = lo; j < hi; j++) {
    if (nums[j] < pivot) {
      [nums[i], nums[j]] = [nums[j], nums[i]];
      i++;
    }
  }
  [nums[i], nums[hi]] = [nums[hi], nums[i]];   // pivot 歸位
  quickSort(nums, lo, i - 1);
  quickSort(nums, i + 1, hi);
}
```

呼叫方式：`quickSort(nums, 0, nums.length - 1)`。

### `i` 的意義

`i` 左邊的元素全部比 pivot 小。`i` 到 `j` 之間的元素全部 >= pivot。

把 `i` 想成一道門。小的從門的左邊進來，大的被擋在門的右邊。掃完之後，pivot 站到門的位置，左邊全是小的，右邊全是大的。

---

## 走一遍 `[10, 7, 8, 9, 1, 5]`

上面那個播放器跑的就是這組資料，走完 47 步：

```
partition 次數：4 次
比較次數：5 + 3 + 2 + 1 = 11 次
swap 次數：1 次（迴圈內）+ 4 次（pivot 歸位）= 5 次
遞迴最深：5 層
排完：[1, 5, 7, 8, 9, 10]
```

第一次 partition 的 pivot = 5，掃完只有 1 比它小，所以 5 歸位到 index 1，左邊剩一格、右邊剩四格。整個陣列變成 `[1, 5, 8, 9, 10, 7]`。右邊那四格是 `[8, 9, 10, 7]`，跟原本的 `[10, 7, 8, 9]` 順序不一樣。Lomuto 只保證右邊都不小於 pivot，不保證它們之間的順序。

6 個元素卻遞迴了 5 層，因為接下來每一層挑到的 pivot（7、8、9）都剛好是那段裡最小的，右半每次只少一格。這跟下一節要講的最糟情況是同一個形狀，只是規模小。

深色的格子一旦出現就不會再變，那是 quick sort 跟其他排序最不一樣的地方：pivot 一歸位就永久定案，不像 merge sort 要等最後一次合併。

---

## 最糟情況：$O(n^2)$ 怎麼來的？

拿一個已排好的陣列 `[1, 2, 3, 4, 5]`，pivot 選最後一個，也就是 5。

比 5 小的？1, 2, 3, 4。全部在左邊。右邊是空的。

partition 完成。左邊有 n-1 個元素。右邊 0 個。

下一輪，pivot 是 4。比 4 小的？1, 2, 3。全在左邊。右邊空的。

每一輪只排好一個元素。排 n 個元素需要 n 輪。第一輪掃 n-1 格，第二輪掃 n-2 格，一路遞減，加起來是 $n(n-1)/2$。$O(n^2)$。

**已排序的資料 + 固定選最右當 pivot = 最糟情況。**

同一個演示，input 換成 `[1, 2, 3, 4, 5]`。看下面那條區間線：partition 一次只縮一格，右半永遠是空的，一路縮到底才排完。

<div data-algo-viz="quick-sort" data-input="1,2,3,4,5"></div>

更慘的是，遞迴深度也從 $O(\log n)$ 退化成 $O(n)$。n 夠大，stack overflow。

所以 in-place 跟「空間 $O(\log n)$」是兩件事，不要混在一起講。不另外分配陣列叫 in-place，這件事永遠成立；$O(\log n)$ 講的是 call stack 的深度，只在每次都切得夠平均的時候成立。上面那份 code 左右兩邊都用遞迴，Go 也不做 tail call 優化，最糟就是 $O(n)$ 個 frame 疊在 stack 上。實務的寫法是先遞迴短的那一半，長的那半用迴圈繼續做，這樣深度就有 $O(\log n)$ 的上限，因為每往下一層，要遞迴的 range 至少對半。

而 insertion sort 碰到已排好的資料？$O(n)$。一路掃過去，內層迴圈完全不動。

同一組資料，quick sort 最糟，insertion sort 最佳。完全反過來。

---

## 怎麼避免最糟？

### Random Pivot

不要固定選最後一個。隨機選一個，跟最後一個交換，然後照常跑。

```typescript
function quickSortRandom(nums: number[], lo: number, hi: number) {
  if (lo >= hi) return;

  // 隨機選 pivot，跟最後一個交換
  const r = lo + Math.floor(Math.random() * (hi - lo + 1));
  [nums[r], nums[hi]] = [nums[hi], nums[r]];

  // 接下來跟原本一樣
  const pivot = nums[hi];
  let i = lo;
  for (let j = lo; j < hi; j++) {
    if (nums[j] < pivot) {
      [nums[i], nums[j]] = [nums[j], nums[i]];
      i++;
    }
  }
  [nums[i], nums[hi]] = [nums[hi], nums[i]];
  quickSortRandom(nums, lo, i - 1);
  quickSortRandom(nums, i + 1, hi);
}
```

同一組 `[1, 2, 3, 4, 5]`，這次 pivot 用抽的。跑完按「重抽」再跑一次，抽到的 pivot 不一樣，遞迴層數就跟著不一樣。

<div data-algo-viz="quick-sort-random" data-input="1,2,3,4,5"></div>

五個元素太少，手氣差的機率還不低。跑二十萬次來看：切成 3 層的機率 33%，4 層 53%，跟固定選最右一樣深的 5 層還有 13%。重抽幾次三種都看得到。要走到 5 層，代表每一層抽到的都是那一段的最大或最小值，n 越大這件事越難連續發生。

一行 `Math.random()` 就把最糟情況的機率從「常常發生」降到「幾乎不可能」。

### Median-of-Three

取第一個、中間、最後一個，三個裡面選中位數當 pivot。

好處是不用亂數，而且只要那三個值不同，選到的就不會是這三個裡面最小或最大的那個。已排序、倒序這類自然出現的排列，剛好都會被它切在中間。

C++ 的 introsort 跟 Go 的 pdqsort 都是從這一招起跳，再各自加碼。pdqsort 在 range 大的時候會多取幾個取樣點，還會偵測資料本來就有序的情況。

但兩招處理的東西不一樣。Random pivot 讓最糟情況變成機率事件，跟輸入長什麼樣子無關。Median-of-three 是決定性的，取樣位置公開在原始碼裡，有人想害你，就能反過來湊出一組輸入，讓它每次都選到極端值。它處理的是自然資料裡的排列，不是刻意做出來的輸入。兩者都不能「保證」避開 $O(n^2)$。

---

## 為什麼 Quick Sort 實務上比 Merge Sort 快？

紙面上兩個都是 $O(n \log n)$，quick sort 甚至有 $O(n^2)$ 最糟，但它還是贏。

**In-place。** Quick sort 不開新陣列，merge sort 每次 merge 都要配置一塊。配置跟回收都要時間。

**Cache。** Quick sort 的 partition 是連續掃描，一塊記憶體讀進 cache 之後接下來都是 hit。Merge sort 在兩個陣列之間跳，miss 率高。

**常數小。** 同樣 n log n 次操作，quick sort 每次只做 swap，沒有陣列拷貝。

差距多大取決於語言、元素型別，還有 merge sort 是每次分配新陣列還是共用一塊 buffer。手上那個組合差幾倍，自己量比較準。

| | Quick Sort | Merge Sort | Heap Sort | Insertion Sort |
|---|---|---|---|---|
| 平均 | $O(n \log n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(n^2)$ |
| 最糟 | $O(n^2)$ | $O(n \log n)$ | $O(n \log n)$ | $O(n^2)$ |
| 空間 | $O(\log n)$ 平均，最糟 $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ |
| Cache | 好 | 中 | 差 | 好 |
| 穩定 | 否 | 是 | 否 | 是 |
| 實務速度 | 隨機資料最快 | 中 | 最慢 | 小陣列最快 |

平均最快、cache 最好、不吃額外記憶體，所以標準函式庫拿它當主力。

---

## Hoare Partition：另一種分法

Lomuto 不是唯一的 partition。Hoare partition 發明得更早，swap 次數也更少。

Hoare 的做法：兩個指標從兩端往中間掃。左邊的找到比 pivot 大的停下，右邊的找到比 pivot 小的停下，交換。直到兩個指標交錯。

```typescript
function hoarePartition(nums: number[], lo: number, hi: number): number {
  const pivot = nums[lo];
  let i = lo - 1, j = hi + 1;
  for (;;) {
    do { i++; } while (nums[i] < pivot);
    do { j--; } while (nums[j] > pivot);
    if (i >= j) return j;
    [nums[i], nums[j]] = [nums[j], nums[i]];
  }
}
```

Hoare partition 的 swap 次數比 Lomuto 少。因為它從兩端往中間走，每一次 swap 同時安置一個「大的」跟一個「小的」。Lomuto 一次只處理一個，而且 `i` 跟 `j` 停在同一格的時候還是照換，那一次 swap 是自己跟自己換，白做。

但邊界條件比較難寫對。面試時推薦用 Lomuto，不容易出 bug。

---

## 三路 Partition：大量重複值的解法

如果資料有大量重複值，Lomuto partition 會把所有等於 pivot 的元素全部放到一邊。每輪只排好一個 pivot，其他等於 pivot 的都還在。

Dutch National Flag partition 把陣列分成三段：小於 pivot、等於 pivot、大於 pivot。所有等於 pivot 的元素一次到位。

```
[  < pivot  |  == pivot  |  > pivot  ]
```

LeetCode 75 [Sort Colors](/problem/sort-colors) 就是這個技巧。只有 0, 1, 2 三個值。一次 partition 全部歸位。

pdqsort（Go 1.19 之後的標準排序）就用了這一招。偵測到大量重複值時，自動切換到三路 partition。

---

## 面試怎麼考

手寫排序的第一選擇，因為考點多：partition 寫不寫得對、最糟情況怎麼發生、怎麼避免、為什麼比 merge sort 快。

相關題：

- [Sort an Array](/problem/sort-an-array) — 手寫排序
- [Kth Largest Element](/problem/kth-largest-element-in-an-array) — quickselect
- [Sort Colors](/problem/sort-colors) — 三路 partition，Dutch National Flag

**Quickselect** 找第 k 大不用全排：做一次 partition，pivot 歸位的位置剛好是 k 就結束，在 k 左邊就只遞迴右半，右邊就只遞迴左半。平均每次剩一半，$n + n/2 + n/4 + \dots = 2n$，所以平均 $O(n)$。最糟一樣 $O(n^2)$，pivot 每次都選到極端值的話。

---

## 複雜度

| | 值 |
|---|---|
| 最佳 | $O(n \log n)$ |
| 平均 | $O(n \log n)$ |
| 最糟 | $O(n^2)$ — 已排好 + 固定 pivot |
| 空間 | $O(\log n)$ — call stack 深度，切得平均才成立，最糟 $O(n)$ |
| In-place | 是 |
| 穩定 | 否 |

---

## 總結

Merge sort 用空間換穩定。Quick sort 用穩定性換速度。

沒有額外記憶體分配。連續記憶體存取。常數因子小。這三件事加起來，讓 quick sort 在隨機資料上跑得比誰都快。

代價是兩個：最糟 $O(n^2)$，不穩定。但 random pivot 幾乎消滅了第一個問題。第二個問題，大部分場景不在意。

所以語言標準函式庫的選擇很一致：用 quick sort 當主力，用 heap sort 當保底，用 insertion sort 處理小區間。這就是 introsort。

回到 [Sorting 總覽](/concept/sorting) 看全貌。想學 quick sort 遞迴太深時換上來的那個？看 [Heap Sort](/concept/heap-sort)。
