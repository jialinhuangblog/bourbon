---
title: "Merge Sort"
category: Sorting
slug: merge-sort
subtitle: 切半再合併，穩定 O(n log n)
date: 2026-02-28T11:51:31
updated: 2026-08-04
---

# Merge Sort

你有兩疊已經排好的撲克牌。要合成一疊，怎麼做？

每次比兩疊最上面那張，小的拿走放到新的一疊。重複。兩疊都空了，新的一疊就排好了。

這就是 merge sort。

---

## 先學 merge，再學 sort

merge sort 分兩件事：
1. **merge** — 把兩個已排好的陣列合成一個
2. **sort** — 用遞迴把問題拆小，拆到不能再拆，再 merge 回去


---

## Merge：兩個排好的合成一個

`[1, 5, 9]` 和 `[2, 3, 7]`。左手一疊、右手一疊，每次比兩疊最上面那張，小的放到桌上。

```
左 [1,5,9]  右 [2,3,7]  桌 []
1 vs 2 → 1              桌 [1]
5 vs 2 → 2              桌 [1,2]
5 vs 3 → 3              桌 [1,2,3]
5 vs 7 → 5              桌 [1,2,3,5]
9 vs 7 → 7              桌 [1,2,3,5,7]
右手空了，左手剩 9 整段接上  桌 [1,2,3,5,7,9]
```

每個元素只看一次，$O(n)$。

```typescript
function merge(a: number[], b: number[]) {
  const result: number[] = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    if (a[i] <= b[j]) {          // <= 相等先拿左邊，穩定性從這裡來
      result.push(a[i]); i++;
    } else {
      result.push(b[j]); j++;
    }
  }
  return result.concat(a.slice(i), b.slice(j));
}
```

這裡是 `<=` 不是 `<`，相等的時候先拿左邊那張，相等的元素就維持原順序，這是 merge sort 穩定的來源。`concat` 那行是收尾：其中一邊用完之後，另一邊剩下的整段接上去，不用再比。

---

## 排好的陣列從哪來

要有兩個排好的陣列才能 merge，但一開始手上只有一個沒排好的。

作法是把它切一半，左邊排好、右邊排好，再 merge。

「左邊排好」怎麼排？再切一半。再排。再 merge。

一直切，切到只剩一個元素。一個元素的陣列，天生就是排好的。

這就是遞迴。

---

## 完整的 Merge Sort

```typescript
function merge(a: number[], b: number[]) {
  const result: number[] = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    if (a[i] <= b[j]) {
      result.push(a[i]); i++;
    } else {
      result.push(b[j]); j++;
    }
  }
  return result.concat(a.slice(i), b.slice(j));
}

function mergeSort(nums: number[]): number[] {
  if (nums.length <= 1) return nums;

  const mid = Math.floor(nums.length / 2);
  const left = mergeSort(nums.slice(0, mid));
  const right = mergeSort(nums.slice(mid));
  return merge(left, right);
}
```

`mergeSort` 三行邏輯：
1. 只剩一個？回傳。
2. 切一半，左邊遞迴排。
3. 右邊遞迴排，merge 起來。

## 範例資料 `[38, 27, 43, 3, 9, 82, 10, 1]`

8 個元素，每次對半都切得整齊，merge 樹左右對稱。這組資料最看得出切法只認元素個數，跟資料內容無關。同樣 8 格，換成已經排好的資料，搬動次數還是一樣。

<div data-algo-viz="merge-sort" data-input="38,27,43,3,9,82,10,1"></div>

播放器只錄合併，往下對半切分的過程沒有畫，所以它從第一次 merge 開始，64 步走完 7 次 merge。上面那一列是原陣列，下面那一列是 result 暫存區，兩列 index 對齊。元素被取進 result 之後原本那格畫成空的，正在合併的左半右半用兩條線標出來。

深色要等最後一次 merge 寫回去才出現。前六次 merge 的結果雖然內部排好了，但整段還會被更大的那次 merge 再搬一遍。Insertion sort 的左邊會一路長出一塊排好的區域，merge sort 沒有這種區域，只有一棵樹在往回收。

---

## 走一遍 `[38, 27, 43, 3, 9, 82, 10, 1]`

```
     [38, 27, 43, 3, 9, 82, 10, 1]
        /                    \
  [38, 27, 43, 3]        [9, 82, 10, 1]
    /        \             /        \
[38,27]   [43,3]      [9,82]     [10,1]
  / \       / \         / \        / \
[38][27] [43][3]     [9][82]   [10][1]
```

切到剩一個就往回合。播放器跑完的數字：

```
每次 merge 比了幾次：1 + 1 + 3 + 1 + 1 + 3 + 7 = 17 次
搬進 result 的次數：2 + 2 + 4 + 2 + 2 + 4 + 8 = 24 次
排完：[1, 3, 9, 10, 27, 38, 43, 82]
```

24 次剛好等於 $n \log_2 n = 8 \times 3 = 24$。8 是 2 的次方，切分樹三層都是滿的，每一層都要把八格全部搬過一次。元素個數不是 2 的次方的話會少幾次，因為切分樹會有幾根分支比別人短。

比較次數 17 比搬動次數 24 少，差在每次 merge 的收尾：一邊用完之後，另一邊剩下的整段直接接上去，不用再比。最後那次 merge 比 7 次就搬完 8 格，就是這個原因。

換成已經排好的資料，搬動還是 24 次，比較會降到 12 次。差在常數，$O(n \log n)$ 不會動。

---

## 為什麼是 $O(n \log n)$？

每次切一半，切 $\log n$ 層。每一層做 merge，merge 掃過所有元素，$O(n)$。

```
層 0:  [        n 個        ]           → merge 工作量 = n
層 1:  [   n/2   ] [   n/2   ]         → merge 工作量 = n
層 2:  [n/4][n/4]  [n/4][n/4]          → merge 工作量 = n
...
層 log n:  [1][1][1]...[1]             → merge 工作量 = n
```

每層都是 n，共 $\log n$ 層。總共 $n \log n$。

不管資料長什麼樣。已排好？$n \log n$。倒序？$n \log n$。隨機？$n \log n$。

沒有最糟情況。Quick sort 沒有這個保證。

---

## 代價：空間

`mergeSort` 裡的 `slice` 每呼叫一次就複製一份，`merge` 又 `push` 出一個新的 `result`。整趟下來配置的陣列不只一個，額外空間 $O(n)$，這裡的 $n$ 是元素個數。

同一句 $O(n)$ 底下有兩個不同的量，分開講比較不會誤會。

**同時活著的量。** 峰值在最外層那次 merge：`left` 跟 `right` 各佔 $n/2$ 格，兩個都還被變數指著，正在 `push` 的 `result` 又長到 $n$ 格，加起來大約 $2n$。更深層的那些陣列這時候已經沒有變數指向，等著被回收。要把係數壓到 1 的話得改寫成全程共用一個 buffer、用 index 標範圍，但那樣 code 會難讀很多。

**整趟配置過的總量。** 每一層的 `slice` 加 `result` 加起來大約 $n$ 格，$\log n$ 層就是 $O(n \log n)$ 格。前面走一遍數出來的 24 次搬動，搬進去的就是這些格子。

這些陣列用完就沒人指了，V8 的 GC 會回收，不會累積成峰值。但配置跟回收本身要花時間，這是 merge sort 在 JS 裡跑起來比理論慢的主因之一，尤其資料量大到讓 GC 頻繁作用的時候。

複雜度表寫的 $O(n)$ 講的是峰值那一項。遞迴自己還有 $O(\log n)$ 的 call stack，比 $O(n)$ 小，加起來還是 $O(n)$，所以沒有單獨列。

Quick sort 跟 heap sort 都是原地排，不用這筆開銷。這也是標準函式庫不用純 merge sort 當預設的原因。不過 JS 的 `Array.prototype.sort` 底層是 TimSort，骨幹就是 merge sort，只是做了很多工程優化來少配置幾次記憶體。

---

## 面試考什麼？

merge sort 本身不常直接考。但 **merge 這個操作**是高頻考點。

- [Merge Two Sorted Lists](/problem/merge-two-sorted-lists) — 兩個 sorted linked list 合一個
- [Merge k Sorted Lists](/problem/merge-k-sorted-lists) — k 個 sorted list 合一個（用 heap 優化）
- [Sort an Array](/problem/sort-an-array) — 手寫排序，merge sort 最穩
- [Count of Smaller Numbers After Self](/problem/count-of-smaller-numbers-after-self) — merge sort 變體，在 merge 時統計逆序對

這幾題做的事都一樣：兩個指標，各指一個已排好的序列，比大小，小的先走。

---

## 跟其他 $O(n \log n)$ 比

| | Merge Sort | Quick Sort | Heap Sort |
|---|---|---|---|
| 最糟保證 | $O(n \log n)$ | $O(n^2)$ | $O(n \log n)$ |
| 空間 | $O(n)$ | $O(\log n)$ 平均，最糟 $O(n)$ | $O(1)$ |
| 穩定 | 是 | 否 | 否 |
| Cache | 中 | 好 | 差 |
| 對幾乎排好的資料 | 一樣快 | 可能退化 $O(n^2)$ | 一樣快 |
| 實務速度 | 中等 | 最快 | 最慢 |

Merge sort 是三個裡面唯一穩定的。也是唯一沒有最糟情況的。

但它的空間代價最大。Quick sort 平均只要 $O(\log n)$ 的 call stack，切得不平均的時候最糟是 $O(n)$。Heap sort 是 $O(1)$。Merge sort 要 $O(n)$：排一百萬個元素就要額外一百萬格。

這就是為什麼 TimSort（Python、Java、JS）選了 merge sort 當骨幹：它們需要穩定排序。而 Go、C++、Rust 選了 quick sort 當主力：它們更重視速度，穩定排序另外提供 `SortStable()`。

---

## 複雜度

| | 值 |
|---|---|
| 最佳 | $O(n \log n)$ |
| 平均 | $O(n \log n)$ |
| 最糟 | $O(n \log n)$ |
| 空間 | $O(n)$ |
| In-place | 否 |
| 穩定 | 是 |

---

## 總結

Merge sort 做了一個交易：用空間換穩定性。

不管資料多亂，保證 $O(n \log n)$。不管元素怎麼相等，保證穩定。代價是 $O(n)$ 空間。

Quick sort 更快但最糟會退化。Heap sort 省空間但 cache 不友善。Merge sort 永遠 $O(n \log n)$，但吃記憶體。三個各有代價，看場景要的是哪一項。

回到 [Sorting 總覽](/concept/sorting) 看全貌。想學在速度上贏過 merge sort 的做法？看 [Quick Sort](/concept/quick-sort)。
