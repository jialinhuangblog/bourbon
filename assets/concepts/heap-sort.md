---
title: "Heap Sort"
category: Sorting
slug: heap-sort
subtitle: 建 heap，每次拔最大的
date: 2026-02-28T11:51:31
updated: 2026-08-04
---

# Heap Sort

Merge sort 需要額外的 $O(n)$ 空間。Quick sort 最糟的情況是 $O(n^2)$。

有沒有一個排序，時間保證 $O(n \log n)$，空間又只要 $O(1)$？

有，heap sort 就是。那為什麼各語言的標準排序，主力都不是它？

---

## Heap 是什麼

Heap 是一棵完全二元樹（complete binary tree）：除了最後一層，每一層都填滿，最後一層從左邊開始填。Max heap 多一條規則，每個節點都不小於它的兩個 child。

```
        9
       / \
      7   6
     / \
    3   2
```

9 不小於 7 跟 6，7 不小於 3 跟 2，滿足。

規則只管上下，不管左右。6 比 7 小，但 6 在 7 的右邊，這完全合法。Heap 不是排好的陣列，它確保 parent 一定不小於 child，就這樣。最常用到的推論是最大的那個一定在 root。

### 建完 heap 不等於排好

把這篇的範例資料 `[4, 10, 3, 5, 1, 8, 2]` 整理成 max heap，結果是：

```
[10, 5, 8, 4, 1, 3, 2]        降序應該是 [10, 8, 5, 4, 3, 2, 1]

           10
        /      \
       5        8
      / \      / \
     4   1    3   2
```

index 1 是 5，index 2 是 8，比較小的那個排在前面。這是合法的 max heap：5 跟 8 是兄弟，兩個都是 10 的 child，彼此之間沒有 parent 跟 child 的關係，規則管不到它們。

所以 heap 給的保證只有一條：**root 是最大的**。第二大的在哪，heap 不知道，也不保證。要拿到第二大的，得先把 root 拿走、剩下的重整，它才會升上來。所以下一節那個「拿走再重整」的迴圈跑不掉，第二大的位置只能這樣問出來。

### 陣列就是樹

不需要真的配置節點跟指標。一個陣列就夠：把樹按照層序（一層一層，每層由左往右）寫進陣列，parent 跟 child 的關係就變成一條算式。

```
index:  0  1  2  3  4
value: [9, 7, 6, 3, 2]

left(i)   = 2i + 1
right(i)  = 2i + 2
parent(i) = Math.floor((i - 1) / 2)
```

index 0 是 root，index 1 跟 2 是它的兩個 child，index 3 跟 4 是 index 1 的 child。

算式成立是因為每一層的節點依序把下一層填滿：第 $d$ 層的第 $k$ 個節點在 index $2^d - 1 + k$，它的 left child 是下一層的第 $2k$ 個，代進去化簡就是 $2i + 1$。

沒有指標，沒有 struct，也不用配置額外記憶體。這是 heap sort 能做到 $O(1)$ 空間的原因。

### 長條圖畫不出樹

演示只畫得出一排長條。但 heap sort 的所有動作都發生在樹上，每一次交換都是 parent 跟 child 換，也就是上下相鄰的兩層在換。長條圖上的距離跟這件事完全無關：這組資料裡 index 0 跟 index 2 換過（差 2 格），index 2 跟 index 5 也換過（差 3 格），兩次都是相鄰兩層。差幾格只反映節點在第幾層，不反映關係遠近。

7 個元素的形狀長這樣，看播放器的時候對照著看：

```
陣列   index:  0    1    2    3    4    5    6

樹                     0
                    /     \
                   1       2
                 /  \     /  \
                3    4   5    6
```

播放器每一步的說明都會把 index 關係寫出來（「index 2 的兩個 child 是 `2*2+1=5` 跟 `2*2+2=6`」），可以對照上面這張圖看是哪三個節點。

---

## 兩個步驟

**第一步：** siftDown 反覆呼叫，把陣列整理成 max heap。

**第二步的每一輪：** root 的最大值換到 heap 的最後一格，siftDown 把 heap 修回來，次大的就到了最前面，等下一輪。

每取出一個，排好的尾巴就長一格，heap 就短一格。兩塊共用同一個陣列，所以不需要額外空間。

### 跟 selection sort 是同一個骨架

「每輪拿一個最大的放到尾巴」這件事，[selection sort](/concept/selection-sort) 也在做。差別只在「找最大的」那一步用什麼方法：

- **Selection sort**：每輪從頭掃到尾找最大值，一輪 $O(n)$，跑 $n$ 輪就是 $O(n^2)$。
- **Heap sort**：先花 $O(n)$ 把陣列整理成 heap，之後每輪直接讀 root 就是最大值，拿走之後只要 $O(\log n)$ 讓下一個最大的浮上來，總共 $O(n \log n)$。

只要一次最大值的話，掃一遍就好，用不到 heap。heap 的價值在拿走之後：不用重新掃全部，補一補就知道下一個是誰。建 heap 那 $O(n)$ 只做一次，之後每一輪都是 $O(\log n)$，不是 $O(n)$。

---

## 完整程式碼

自己拖，或按播放。上面那條 heap 線是還在整理的範圍，右邊那條是已經排好、不會再變動的尾巴。

<div data-algo-viz="heap-sort" data-input="4,10,3,5,1,8,2"></div>

```typescript
function heapSort(nums: number[]) {
  const n = nums.length;

  // 第一步：建 max heap，從最後一個有 child 的節點往回做
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    siftDown(nums, n, i);
  }

  // 第二步：每次把頂端換到尾巴，heap 範圍縮一格
  for (let end = n - 1; end > 0; end--) {
    [nums[0], nums[end]] = [nums[end], nums[0]];
    siftDown(nums, end, 0);
  }
}

// siftDown 只管一件事：把 index i 的值往下沉到它該待的層
function siftDown(nums: number[], n: number, i: number) {
  let largest = i;
  const l = 2 * i + 1, r = 2 * i + 2;
  if (l < n && nums[l] > nums[largest]) {
    largest = l;
  }
  if (r < n && nums[r] > nums[largest]) {
    largest = r;
  }
  if (largest !== i) {
    [nums[i], nums[largest]] = [nums[largest], nums[i]];
    siftDown(nums, n, largest);
  }
}
```

這個函式很多課本叫 `heapify`。但 heapify 在別的地方又指「把整個陣列建成 heap」那件事，一個名字兩個意思。這篇一律用 siftDown 指單一節點往下沉，用「建 heap」指跑完整個迴圈那件事。

### siftDown 做的事

它跟 left child 比一次、跟 right child 比一次，三個裡面誰最大誰就上來。比完要嘛自己最大、不用動，要嘛某個 child 比較大、換下去。建 heap 的時候兩種都出現過。

**最大的是自己，這一支就結束。** 陣列 `[4, 10, 8, 5, 1, 3, 2]`，處理 index 1：

```
index 1 的值是 10
left  child = 2*1+1 = index 3，值 5
right child = 2*1+2 = index 4，值 1

10 比 5 大，也比 1 大 → largest 還是 1（也就是自己）→ 不換，結束
```

不用再往下，因為底下那兩棵子樹早就處理過了，是合法的 heap；現在 parent 又不小於兩個 child，整棵都合法。

**最大的是某個 child，交換，然後從那個 child 的位置再來一次。** 陣列 `[10, 4, 8, 5, 1, 3, 2]`，處理 index 1：

```
index 1 的值是 4
left  child = index 3，值 5
right child = index 4，值 1

5 比 4 大 → largest = 3
換 index 1 跟 3 → [10, 5, 8, 4, 1, 3, 2]
4 現在在 index 3，從那裡繼續 → siftDown(nums, 7, 3)
    index 3 的 child 是 7 跟 8，超出範圍，停
```

被換下去的 4 可能還是比它的新 child 小，所以要繼續往下問。最多走到 leaf，也就是樹的高度 $O(\log n)$。

參數 `n` 不是陣列長度，是「heap 目前到哪裡為止」。取出階段每交換一次就把它減一，尾巴那段自動被排除在外，不需要另外記錄邊界。

### 為什麼從 `Math.floor(n / 2) - 1` 開始

leaf 不需要 siftDown，因為它沒有 child，自己一個節點就滿足 max heap 的條件。所以只要找出「最後一個有 child 的 index」，從那裡往回做就好。

$n = 7$ 的樹：

```
         0
      /     \
     1       2
    / \     / \
   3   4   5   6

index 3 的 left child 是 2*3+1 = 7，陣列只到 index 6，沒有這個節點
index 4、5、6 同理，全是 leaf
有 child 的只有 index 0、1、2
```

$n = 8$ 的話，多出來的那個節點掛在 index 3 底下：

```
         0
      /     \
     1       2
    / \     / \
   3   4   5   6
  /
 7

index 3 的 left child 是 7，存在 → index 3 有 child（只有一個）
index 4 的 left child 是 9，超出 → leaf
```

幾個 $n$ 的結果：

| $n$ | 有 child 的 index | leaf 的 index | 最後一個有 child 的 |
|---|---|---|---|
| 7 | 0, 1, 2 | 3, 4, 5, 6 | 2 |
| 8 | 0, 1, 2, 3 | 4, 5, 6, 7 | 3 |
| 9 | 0, 1, 2, 3 | 4, 5, 6, 7, 8 | 3 |
| 15 | 0 到 6 | 7 到 14 | 6 |
| 16 | 0 到 7 | 8 到 15 | 7 |

最後那一欄就是 $\lfloor n/2 \rfloor - 1$：$7/2 = 3.5$ 取 3 減 1 得 2，$8/2 = 4$ 減 1 得 3，$9/2 = 4.5$ 取 4 減 1 得 3。$n = 8$ 跟 $n = 9$ 起點一樣，因為第 9 個節點是掛在 index 3 的右邊，沒有讓任何新的節點長出 child。

寫成條件就是：index $i$ 有 left child 的意思是 $2i + 1 < n$。$i$ 最大能取到多少，就是最後一個起點。

方向也不能反。siftDown 的前提是「兩棵子樹已經是 heap 了」，只有 index 由大到小往回做，輪到 index $i$ 的時候它的兩棵子樹才處理過。

### 建 heap 是 $O(n)$，不是 $O(n \log n)$

$\lfloor n/2 \rfloor$ 個有 child 的節點各做一次 $O(\log n)$ 的 siftDown，乘起來是 $O(n \log n)$。這樣乘沒有算錯，只是估得太寬鬆。

$n = 15$ 的樹有 4 層，一層一層數：

| 層 | 節點數 | 每個最多能往下沉幾層 | 乘積 |
|---|---|---|---|
| 第 3 層（底層） | 8 | 0（沒有 child） | 0 |
| 第 2 層 | 4 | 1 | 4 |
| 第 1 層 | 2 | 2 | 4 |
| 第 0 層（root） | 1 | 3 | 3 |
| | | 合計 | **11** |

底層 8 個節點就佔了 15 個裡的一半以上，而它們一次都不用往下沉。往上一層，節點數減半，能往下沉的距離才加一。節點多的那幾層走不了幾步，能走很遠的那幾層又沒幾個節點，所以總和是 11，不是「7 個有 child 的節點各往下走 3 層 = 21」。

$n = 7$ 同樣數法：$1 \times 2 + 2 \times 1 + 4 \times 0 = 4$。播放器錄到的實際交換次數是 3，比 4 少。

寫成通式（$h$ 是樹高）：

$$
\sum_{d=0}^{h} 2^d (h - d) = 2^h \sum_{k=0}^{h} \frac{k}{2^k} < 2^h \cdot 2 \le 2n
$$

$\sum_{k \ge 0} k/2^k$ 收斂到 2，跟 $n$ 無關，所以整趟建 heap 的交換次數不超過 $2n$。$n = 15$ 算出來 11，確實小於 30。

### 取出階段每輪都是完整的 $O(\log n)$

建 heap 的交換次數少，是因為大部分節點在底層，往下走不了幾層。取出階段剛好相反。

每次 siftDown 都從 root 出發，而換上 root 的那個值是從陣列尾端拿來的，通常很小，一路往下沉到底層。這篇的例子六輪各往下沉了幾層：

```
end=6  往下沉 2 層   （當下 heap 有 6 格，高度 2）
end=5  往下沉 2 層   （5 格，高度 2）
end=4  往下沉 2 層   （4 格，高度 2）
end=3  往下沉 1 層   （3 格，高度 1）
end=2  往下沉 1 層   （2 格，高度 1）
end=1  往下沉 0 層   （1 格，沒有地方可以去）
```

每一輪都走到了當下 heap 的最底層，一層都沒少。建 heap 那邊只有 root 走完整個高度，其他節點都只動一兩步；取出階段是每一輪都走完。

$n-1$ 次、每次 $O(\log n)$，這段就是完整的 $O(n \log n)$。整體的複雜度由這段決定，因為建 heap 只有 $O(n)$，量級比它小。

---

## 走一遍 `[4, 10, 3, 5, 1, 8, 2]`

上面那個播放器跑的就是這組資料，走完 71 步：

```
建 heap：3 次 siftDown 呼叫（遞迴展開成 6 次）、8 次比較、3 次交換
整趟：20 次 siftDown 呼叫、21 次比較、17 次交換
排完：[1, 2, 3, 4, 5, 8, 10]
```

播放器畫的是一排長條，畫不出樹。要是想看樹的形狀，每一步的說明都把 index 算式寫出來了（例如 index 2 的 child 是 5 跟 6，因為 2i+1 跟 2i+2），可以對照前面那張樹的圖。

### 每一次 siftDown 呼叫

`siftDown` 在 code 裡出現三次，三個都長得很像。分辨的方法是看第 2、3 個參數：**第二個是 heap 的範圍到哪為止，第三個是從哪個 index 開始往下沉**。

20 次呼叫全部列出來。`迴圈` 是外層迴圈叫的，`遞迴` 是 `siftDown` 最後那行 `siftDown(nums, n, largest)` 自己叫的，縮排代表遞迴的深度。

**第一步：建 max heap。** 範圍固定 7，起點 `i` 從 `Math.floor(7 / 2) - 1 = 2` 往 0 走。

```
i=2  迴圈  siftDown(nums, 7, 2)   [4 10 3 5 1 8 2]
             child 是 index 5 的 8 跟 index 6 的 2，8 最大 → 換 index 2 跟 5
             [4 10 8 5 1 3 2]
     遞迴    siftDown(nums, 7, 5)   index 5 的 child 是 11、12，超出範圍，停

i=1  迴圈  siftDown(nums, 7, 1)   [4 10 8 5 1 3 2]
             child 是 index 3 的 5 跟 index 4 的 1，自己的 10 最大 → 不換
             不換就不遞迴，這一支結束

i=0  迴圈  siftDown(nums, 7, 0)   [4 10 8 5 1 3 2]
             child 是 index 1 的 10 跟 index 2 的 8，10 最大 → 換 index 0 跟 1
             [10 4 8 5 1 3 2]
     遞迴    siftDown(nums, 7, 1)   剛換下來的 4 現在在 index 1
               child 是 index 3 的 5 跟 index 4 的 1，5 最大 → 換 index 1 跟 3
               [10 5 8 4 1 3 2]
       遞迴    siftDown(nums, 7, 3)   child 是 7、8，超出範圍，停

建好：[10 5 8 4 1 3 2]        迴圈 3 次、遞迴 3 次
```

**第二步：逐一取出。** 起點固定 0，範圍 `end` 從 6 一路縮到 1。`|` 右邊是已經排好的尾巴。

```
end=6  root 10 換到 index 6      [2 5 8 4 1 3 | 10]
       迴圈  siftDown(nums, 6, 0)   換 index 0 跟 2 → [8 5 2 4 1 3 | 10]
         遞迴  siftDown(nums, 6, 2)   換 index 2 跟 5 → [8 5 3 4 1 2 | 10]
           遞迴  siftDown(nums, 6, 5)   child 超出範圍，停

end=5  root 8 換到 index 5       [2 5 3 4 1 | 8 10]
       迴圈  siftDown(nums, 5, 0)   換 index 0 跟 1 → [5 2 3 4 1 | 8 10]
         遞迴  siftDown(nums, 5, 1)   換 index 1 跟 3 → [5 4 3 2 1 | 8 10]
           遞迴  siftDown(nums, 5, 3)   child 超出範圍，停

end=4  root 5 換到 index 4       [1 4 3 2 | 5 8 10]
       迴圈  siftDown(nums, 4, 0)   換 index 0 跟 1 → [4 1 3 2 | 5 8 10]
         遞迴  siftDown(nums, 4, 1)   換 index 1 跟 3 → [4 2 3 1 | 5 8 10]
           遞迴  siftDown(nums, 4, 3)   child 超出範圍，停

end=3  root 4 換到 index 3       [1 2 3 | 4 5 8 10]
       迴圈  siftDown(nums, 3, 0)   換 index 0 跟 2 → [3 2 1 | 4 5 8 10]
         遞迴  siftDown(nums, 3, 2)   child 超出範圍，停

end=2  root 3 換到 index 2       [1 2 | 3 4 5 8 10]
       迴圈  siftDown(nums, 2, 0)   換 index 0 跟 1 → [2 1 | 3 4 5 8 10]
         遞迴  siftDown(nums, 2, 1)   child 超出範圍，停

end=1  root 2 換到 index 1       [1 | 2 3 4 5 8 10]
       迴圈  siftDown(nums, 1, 0)   範圍只剩 index 0 一格，沒有 child，停

排完：[1 2 3 4 5 8 10]        迴圈 6 次、遞迴 8 次
```

不用背也分得出來哪個是遞迴：**每一行有交換，下一行就是遞迴呼叫；寫「停」的那一行，這一支就結束，回到外層迴圈。** 所以遞迴的次數等於 `siftDown` 內部的交換次數。

迴圈 3 + 6 = 9 次，遞迴 3 + 8 = 11 次，加起來 20 次，就是上面統計的「20 次 siftDown 呼叫」。交換那邊，11 次來自 siftDown 內部，加上第二步那 6 次「root 換到尾巴」，總共 17 次。

建 heap 只呼叫 3 次 siftDown，因為只有 index 2、1、0 有 child。前一節那個逐層數法在 n=7 時算出來是 4，實際量到 3 次交換，對得上。第二步就沒這麼省了：六輪的遞迴次數是 2、2、2、1、1、0，每一輪都走到了當下 heap 的最底層，一層都沒少。

---

## 跟其他 $O(n \log n)$ 比

Merge sort、quick sort、heap sort 都是 $O(n \log n)$。差在哪？

| | Merge Sort | Quick Sort | Heap Sort |
|---|---|---|---|
| 最糟 | $O(n \log n)$ | $O(n^2)$ | $O(n \log n)$ |
| 空間 | $O(n)$ | $O(\log n)$ | $O(1)$ |
| 穩定 | 是 | 否 | 否 |
| 存取模式 | 連續掃描 | 連續掃描 | 按 $2i+1$ 跳 |

Heap sort 紙面上最好看：最糟情況跟空間兩項都是最好的。但實務上跑最慢，原因在最後一列。

---

## 慢在哪裡：cache

CPU 從 RAM 讀資料很慢，所以中間隔了一層 cache：容量小很多，速度快很多。要的資料已經在 cache 裡叫 hit，不在、得去 RAM 拿叫 miss。

跑一趟 RAM 的固定成本很高，所以 CPU 不會只拿要的那幾個 byte。它一次把附近連續的 64 bytes 一起拉回來（x86 上的長度），這一整塊叫一條 cache line。存 int64 的話一個元素 8 bytes，一條 line 剛好裝 8 個。

Quick sort 的 partition 從左往右連續掃陣列，所以拉回來的那一整條 line 會全部用到：

```
讀 nums[0]  miss  去 RAM 拿，順便把 nums[0] 到 nums[7] 整條拉回來
讀 nums[1]  hit   已經在 cache 裡
讀 nums[2]  hit
   ⋮
讀 nums[7]  hit
讀 nums[8]  miss  換下一條 line
```

八次讀取只有一次真的去了 RAM，另外七次都在 cache 裡。

Heap sort 的 siftDown 不是連續掃，是一路往下跳：$i \to 2i+1 \to 4i+3 \to \cdots$，每往下一層 index 就翻倍。

```
int64（8 bytes），cache line 64 bytes，一條裝 8 個元素

index 0   → byte 0     第 0 條 line
index 1   → byte 8     第 0 條 line
index 3   → byte 24    第 0 條 line
index 7   → byte 56    第 0 條 line
index 15  → byte 120   第 1 條 line
index 31  → byte 248   第 3 條 line
index 63  → byte 504   第 7 條 line
```

第 0 條 line 裝的是 index 0 到 7，也就是樹的前三層再加一個節點。所以 index 0、1、3、7 這四步只有第一步要去 RAM，後面三步都 hit，跟 quick sort 的情況一樣。從 index 15 開始就不一樣了：每往下一層換一條新的 line，而且兩條之間的距離翻倍。$n = 100{,}000$ 的時候樹高是 $\lfloor \log_2 100000 \rfloor = 16$，一次 siftDown 最多走 16 層，前四層在同一條 line 裡，後面十幾層每一層都是一次新的 miss。

一次 miss 會慢多少，要看陣列有沒有超出哪一層 cache。L1 一般是 32 到 48 KB，裝 int64 只有四千到六千個元素，所以陣列一過幾千筆，siftDown 的下半段就註定 miss 掉 L1，往 L2、L3 走；再大就到 main memory。每往外一層，延遲多一個檔次，最外面那一層跟 L1 差到兩位數倍率以上。確切數字每代 CPU 都不同，去查該型號的規格，不要用這篇的估。

所以同樣是 $O(n \log n)$ 次操作，兩邊每次操作的實際成本差很多。慢幾倍要看硬體，差多少跟機器有關，自己量比較準。

---

## heap sort 用途

當保底。

Quick sort 正常情況最快，但它的速度來自「pivot 把陣列切成差不多大的兩半」。pivot 如果選最後一個元素，碰到已經排好的 `[1, 2, 3, 4, 5]` 會變成這樣：

```
pivot = 5   比 5 小的有 1 2 3 4   → 左邊 4 個，右邊 0 個
pivot = 4   比 4 小的有 1 2 3     → 左邊 3 個，右邊 0 個
pivot = 3   比 3 小的有 1 2       → 左邊 2 個，右邊 0 個
pivot = 2   比 2 小的有 1         → 左邊 1 個，右邊 0 個
```

右邊每次都是空的，等於沒切。每一輪只排好一個元素，遞迴要走 $n$ 層，比較次數是 $4 + 3 + 2 + 1 = 10$，也就是 $n(n-1)/2$，$O(n^2)$。倒序的 `[5, 4, 3, 2, 1]` 是同一回事，只是每次選到的都是最小的那個。

已經排好或接近排好的資料在現實裡太常見，標準函式庫不能賭這件事不會發生。

**Introsort** 是 C++ 的 `std::sort` 用的混合排序（.NET 也是這個結構）。它的作法：

1. 開局跑 quick sort。
2. 同時數遞迴深度。超過 $2\lfloor \log_2 n \rfloor$ 就代表 pivot 一直選得很爛，正在走向上面那個 $O(n^2)$ 的情況。
3. 這時切換成 heap sort，剩下的部分保證 $O(n \log n)$ 收場。

為什麼保底選 heap sort 而不是同樣保證 $O(n \log n)$ 的 merge sort？因為 `std::sort` 這類介面對外承諾不配置額外記憶體，中途跑去要一塊 $O(n)$ 緩衝區就違約了，而且在遞迴中途配置還可能失敗。Heap sort 只要 $O(1)$，就地做完。

Go 1.19 之後的 `sort` 換成 pdqsort，Rust 的 `sort_unstable` 也是 pdqsort，兩者同樣在遞迴太深的時候退回 heap sort。這幾句我沒有在這裡貼出原始碼行號，要拿去面試講之前，自己開一次 repo 找那個常數跟那個 fallback 分支。

---

## 不穩定，原因跟 selection sort 一樣

拿 `[2, 2, 1]` 當例子，把前面那個 2 叫 A、後面那個叫 B。

建 heap 不動：$\lfloor 3/2 \rfloor - 1 = 0$，siftDown(3, 0) 比較 A 跟 B、A 跟 1，A 都不小於它們，largest 還是自己。

第一輪把 index 0 的 A 跟 index 2 的 1 交換，A 歸位到最後一格，剩下 `[1, B]` 重整，B 升到 index 0。第二輪把 B 換到 index 1。

排完是 `1, B, A`。B 跑到 A 前面了。

一次長距離交換就足以把相等元素的先後順序打亂，這跟 selection sort 不穩定的原因是同一個。

---

## 面試考什麼

Heap sort 本身不太考，heap 這個資料結構才是高頻考點。

- [Sort an Array](/problem/sort-an-array) — 手寫排序
- [Kth Largest Element in an Array](/problem/kth-largest-element-in-an-array) — min heap
- [Top K Frequent Elements](/problem/top-k-frequent-elements)
- [Merge K Sorted Lists](/problem/merge-k-sorted-lists)
- [Find Median from Data Stream](/problem/find-median-from-data-stream)

被問到「heap 憑什麼 $O(1)$ 拿到最大值、$O(\log n)$ 插入」，兩個答案不一樣：最大值在 root，讀一格就有；插入是放到最後一格再往上浮，最多浮一個樹高，所以是 $\log n$。

---

## 複雜度

| | 值 |
|---|---|
| 最佳 | $O(n \log n)$ |
| 平均 | $O(n \log n)$ |
| 最糟 | $O(n \log n)$ |
| 空間 | $O(1)$ |
| In-place | 是 |
| 穩定 | 否 |

建 heap $O(n)$，取出 $n$ 個元素每次 $O(\log n)$，加起來 $O(n \log n)$。

$O(1)$ 空間有個前提：`siftDown` 寫成遞迴的話，call stack 會用掉 $O(\log n)$。改寫成 while 迴圈就真的是 $O(1)$，兩種寫法的比較次數完全一樣。

---

## 總結

Heap sort 在標準函式庫裡的位置是保底，不是主力。它太慢，cache miss 讓它在實務上輸給 quick sort。

但也拿不掉。quick sort 踩到最糟情況、手上又沒有多餘記憶體的時候，能在 $O(1)$ 空間、$O(n \log n)$ 時間內收場的就是它。

回到 [Sorting 總覽](/concept/sorting) 看全貌。想學不靠比較就能排序的方法？看 [Counting Sort](/concept/counting-sort)。
