---
title: "Shell Sort"
category: Sorting
slug: shell-sort
subtitle: Insertion Sort 的長距離版本
date: 2026-03-04T23:54:03
updated: 2026-08-04
---

# Shell Sort

Insertion sort 一次只能往左移一格。

`[9, 8, 7, 6, 5, 4, 3, 1, 2]` 最後那個 `2` 要搬到 index 1，中間隔著 7 個比它大的元素，所以要做 7 次 shift 才到位。

資料越大、越亂，這種一格一格搬的成本就越高。完全倒序的陣列，insertion sort 要跑 $O(n^2)$。

Donald Shell 在 1959 年提出的作法是：先讓元素一次跳好幾格。

---

## 範例資料 `[8, 6, 7, 2, 5, 1, 4, 3]`

這 8 個元素全部不在正確位置上。最小的 `1` 在 index 5，最大的 `8` 在 index 0，幾乎每個元素離它該在的位置都很遠。

要是用 insertion sort，`1` 要從 index 5 一格一格推到 index 0，推 5 次；`8` 要從 index 0 被推到 index 7，被推 7 次。

---

## Gap-based insertion sort

普通 insertion sort 比較的是相鄰的元素，也就是 gap = 1。把這個 1 換成別的數字，比較相隔 `gap` 格的元素，就是 gap-based insertion sort。

一輪 gap = 4 等於把陣列拆成 4 條互不相干的子序列（index 0 跟 4 一條、1 跟 5 一條、以此類推），每條各自做一次 insertion sort。排完之後每條子序列內部有序，整個陣列還是亂的，但每個元素都離它該在的位置近了一些。

Shell sort 就是這樣一輪一輪做下去：gap = 4，然後 gap = 2，最後 gap = 1。最後那輪就是普通的 insertion sort，只是這時候每個元素早就在附近，不必再做長距離的搬移。

---

## 程式碼

自己拖，或按播放。橘色那根是正在比的元素，被拿起來的值顯示在上面，空出來的格子是它等一下要回去的位置。下面那條線寫的是這一輪的 gap。

gap = 4 的時候一次跨 4 格，`8` 一步就從 index 0 到 index 4。gap 縮到 1 的最後一輪就是普通的 insertion sort。

<div data-algo-viz="shell-sort" data-input="8,6,7,2,5,1,4,3"></div>

```typescript
function shellSort(nums: number[]) {
  const n = nums.length;
  let gap = Math.floor(n / 2);              // Shell 的初始 gap：陣列長度除以 2

  while (gap > 0) {
    // gap-based insertion sort
    for (let i = gap; i < n; i++) {
      const key = nums[i];                  // 拿起這個元素
      let j = i - gap;                      // 往左跳 gap 格
      while (j >= 0 && nums[j] > key) {
        nums[j + gap] = nums[j];            // 比 key 大？往右推 gap 格
        j -= gap;                           // 繼續往左跳
      }
      nums[j + gap] = key;                  // 放到對的位置
    }
    gap = Math.floor(gap / 2);              // 縮小步伐
  }
}
```

跟 insertion sort 的差別只有兩行：
- `gap = Math.floor(n / 2)`，然後每輪 `gap = Math.floor(gap / 2)`
- `i` 從 `gap` 開始，比較 `nums[j]` 和 `nums[j + gap]`，移動幅度是 `gap`

結構完全一樣。只是步伐從 1 變成了動態的 `gap`。

---

## 走一遍 `[8, 6, 7, 2, 5, 1, 4, 3]`

陣列長度 8。Shell 的 gap sequence：4 → 2 → 1。

---

### 第一輪：gap = 4

把陣列切成 4 個子序列：

```
子序列 A（index 0, 4）：[8, 5]
子序列 B（index 1, 5）：[6, 1]
子序列 C（index 2, 6）：[7, 4]
子序列 D（index 3, 7）：[2, 3]
```

對每個子序列做 insertion sort：

```
A：[8, 5] → 5 < 8，8 往右推 → [5, 8]
B：[6, 1] → 1 < 6，6 往右推 → [1, 6]
C：[7, 4] → 4 < 7，7 往右推 → [4, 7]
D：[2, 3] → 3 > 2，不動 → [2, 3]
```

程式碼是直接在原陣列上操作的，不存在「回填」。分成子序列只是幫助理解，實際上 `nums[j+gap] = nums[j]` 就是在原地搬。

排完後陣列變成：

```
前：[8, 6, 7, 2, 5, 1, 4, 3]
後：[5, 1, 4, 2, 8, 6, 7, 3]
```

這輪 shift 了 3 次。整個陣列還是亂的，但每條「隔 4 格的子序列」已經有序了。

`8` 從 index 0 搬到 index 4，一次搬 4 格；`1` 從 index 5 搬到 index 1，也是一次 4 格。要是用 insertion sort，這兩個元素得一格一格移動。

---

### 第二輪：gap = 2

現在把陣列切成 2 個子序列：

```
子序列 E（index 0, 2, 4, 6）：[5, 4, 8, 7]
子序列 F（index 1, 3, 5, 7）：[1, 2, 6, 3]
```

對每個子序列做 insertion sort：

```
E：[5, 4, 8, 7] → 4 < 5，5 往右推；8 不動；7 < 8，8 往右推，7 > 5 停 → [4, 5, 7, 8]
F：[1, 2, 6, 3] → 2 不動；6 不動；3 < 6，6 往右推，3 > 2 停       → [1, 2, 3, 6]
```

排完後陣列變成：

```
前：[5, 1, 4, 2, 8, 6, 7, 3]
後：[4, 1, 5, 2, 7, 3, 8, 6]
```

這輪也 shift 了 3 次。還是有點亂，但每個元素離最終位置最多只差 1 到 2 格。

---

### 第三輪：gap = 1

這就是普通的 insertion sort，從 `[4, 1, 5, 2, 7, 3, 8, 6]` 開始：

```
i=1，key=1：1 < 4 → 1 次 shift
→ [1, 4, 5, 2, 7, 3, 8, 6]

i=2，key=5：5 > 4 → 0 次
→ [1, 4, 5, 2, 7, 3, 8, 6]

i=3，key=2：2 < 5, 2 < 4, 2 > 1 停 → 2 次
→ [1, 2, 4, 5, 7, 3, 8, 6]

i=4，key=7：7 > 5 → 0 次
→ [1, 2, 4, 5, 7, 3, 8, 6]

i=5，key=3：3 < 7, 3 < 5, 3 < 4, 3 > 2 停 → 3 次
→ [1, 2, 3, 4, 5, 7, 8, 6]

i=6，key=8：8 > 7 → 0 次
→ [1, 2, 3, 4, 5, 7, 8, 6]

i=7，key=6：6 < 8, 6 < 7, 6 > 5 停 → 2 次
→ [1, 2, 3, 4, 5, 6, 7, 8]
```

完成。

---

### 比較同一組資料

如果直接用 insertion sort 排 `[8, 6, 7, 2, 5, 1, 4, 3]`：

```
i=1，key=6：6 < 8 → 1 次
i=2，key=7：7 < 8, 7 > 6 停 → 1 次
i=3，key=2：2 < 8, 2 < 7, 2 < 6 → 3 次
i=4，key=5：5 < 8, 5 < 7, 5 < 6, 5 > 2 停 → 3 次
i=5，key=1：1 < 8, 1 < 7, 1 < 6, 1 < 5, 1 < 2 → 5 次
i=6，key=4：4 < 8, 4 < 7, 4 < 6, 4 < 5, 4 > 2 停 → 4 次
i=7，key=3：3 < 8, 3 < 7, 3 < 6, 3 < 5, 3 < 4, 3 > 2 停 → 5 次
```

總共：**22 次 shift**。

Shell sort 三輪加起來：gap=4（3 次）+ gap=2（3 次）+ gap=1（8 次）= **14 次**。

差距看起來不大，因為 n=8 太小了。n 越大，差距越明顯。對完全倒序的陣列，insertion sort 跑 $O(n^2)$ 次 shift；Shell sort 用好的 gap sequence 可以降到 $O(n^{1.3})$ 到 $O(n \log^2 n)$。

---

## Gap Sequence 怎麼選

Shell sort 的效能幾乎完全由 gap sequence 決定。

### Shell 的原始版本（1959）

```
n/2, n/4, n/8, ..., 1
```

最直覺的寫法，但效能不是最好的，worst case $O(n^2)$。

### Knuth 的版本（1973）

```
1, 4, 13, 40, 121, 364, ...
```

計算方式：從 1 開始，每次 $gap = 3 \times gap + 1$，找最大的不超過 $n/3$ 的值當初始 gap。

```typescript
function shellSortKnuth(nums: number[]) {
  const n = nums.length;
  let gap = 1;
  while (gap < n / 3) {                     // 找初始 gap：不超過 n/3 的最大 Knuth gap
    gap = gap * 3 + 1;
  }

  while (gap >= 1) {
    for (let i = gap; i < n; i++) {
      const key = nums[i];
      let j = i - gap;
      while (j >= 0 && nums[j] > key) {
        nums[j + gap] = nums[j];
        j -= gap;
      }
      nums[j + gap] = key;
    }
    gap = Math.floor(gap / 3);              // 縮小：gap = (gap - 1) / 3
  }
}
```

Knuth sequence 的 worst case 是 $O(n^{3/2})$，比 Shell 的原版好。

### Ciura 的版本（2001）

```
1, 4, 10, 23, 57, 132, 301, 701, ...
```

這是目前實測效能最好的 gap sequence。沒有公式，是靠實驗測出來的。Ciura 分析了大量資料，選出讓平均比較次數最少的序列。Worst case 未知，但實際跑起來比 Knuth 快。

```typescript
const CIURA_GAPS = [701, 301, 132, 57, 23, 10, 4, 1];   // 從大到小

function shellSortCiura(nums: number[]) {
  const n = nums.length;
  for (const gap of CIURA_GAPS) {
    if (gap >= n) {                         // 跳過比陣列還大的 gap
      continue;
    }
    for (let i = gap; i < n; i++) {
      const key = nums[i];
      let j = i - gap;
      while (j >= 0 && nums[j] > key) {
        nums[j + gap] = nums[j];
        j -= gap;
      }
      nums[j + gap] = key;
    }
  }
}
```

實作時直接 hardcode 這串數字，比動態計算快。

---

## 為什麼大步伐能讓小步伐變快？

Shell sort 能一輪一輪往下做，前提是這條性質：**h-sorted 的陣列做完 k-sort 之後，還是 h-sorted**。

拿上面走過的資料驗一次。gap = 4 排完是 `[5, 1, 4, 2, 8, 6, 7, 3]`，相隔 4 格的四對都排好了。gap = 2 排完變成 `[4, 1, 5, 2, 7, 3, 8, 6]`，再檢查相隔 4 格的那四對：4 < 7、1 < 3、5 < 8、2 < 6，全部還是排好的。

所以 gap = 2 那輪沒有破壞 gap = 4 的成果。可以放心從大 gap 一路做到小 gap，每一輪都建立在前一輪的結果上。

---

## 複雜度

| Gap Sequence | Best | Average | Worst |
|---|---|---|---|
| Shell 原版（n/2） | $O(n \log n)$ | — | $O(n^2)$ |
| Knuth（$3^k-1$） | $O(n \log n)$ | $O(n^{3/2})$ | $O(n^{3/2})$ |
| Ciura | $O(n \log n)$ | 實測最快 | 未知 |

- 空間：$O(1)$，in-place
- 穩定性：**不穩定**

不穩定是因為大 gap 的 shift 可能跨越相等元素，改變它們的相對順序。

---

## 什麼時候用 shell sort

Merge sort 跟 quick sort 都保證 $O(n \log n)$，Shell sort 最好的情況才到 $O(n \log^2 n)$，worst case 更差。它還留著的理由不是速度：merge sort 需要 $O(n)$ 的輔助空間，quick sort 的遞迴 stack 最深要 $O(\log n)$ 到 $O(n)$，而 Shell sort 只有一個巢狀 for 迴圈，原地操作、沒有遞迴。在嵌入式系統這種記憶體受限的環境，多要一塊記憶體常常不可行，所以有些嵌入式的標準函式庫會選它。

Counting sort 跟 radix sort 雖然可以跑 $O(n)$，但只能排整數，而且範圍不能太大。Shell sort 能排任何可以比較的資料型別。

**用 Shell sort：**
- 記憶體極度受限（嵌入式系統、bare-metal 環境）
- 想要比 insertion sort 好，又不想處理遞迴
- 資料量中等（幾百到幾千筆），實作要簡單

**不用 Shell sort：**
- 需要穩定排序
- 資料量幾十萬筆以上，用 quick sort 或 merge sort
- 整數而且範圍已知，用 counting sort 或 radix sort

---

## 總結

| | Insertion Sort | Shell Sort（Knuth） | Merge Sort | Quick Sort |
|---|---|---|---|---|
| Best | $O(n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(n \log n)$ |
| Average | $O(n^2)$ | $O(n^{3/2})$ | $O(n \log n)$ | $O(n \log n)$ |
| Worst | $O(n^2)$ | $O(n^{3/2})$ | $O(n \log n)$ | $O(n^2)$ |
| Space | $O(1)$ | $O(1)$ | $O(n)$ | $O(\log n)$ |
| Stable | 是 | 否 | 是 | 否 |
| In-place | 是 | 是 | 否 | 是 |
| 遞迴 | 否 | 否 | 是 | 是 |

Shell sort 是 insertion sort 的改良版。它在漸近複雜度上贏不過 merge sort 或 quick sort，但在記憶體最受限的環境裡，它是最簡單而且夠用的選擇。

---

回到 [Sorting 總覽](/concept/sorting) 看全貌。想看 Shell sort 改進的起點？看 [Insertion Sort](/concept/insertion-sort)。
