---
title: "Counting Sort"
category: Sorting
slug: counting-sort
subtitle: 不比較，用數字當 index 直接數
date: 2026-02-28T11:51:31
updated: 2026-08-04
---

# Counting Sort

靠比較大小來排序，時間下界是 $O(n \log n)$，這是證明過的。理由是每次比較只有「大於」跟「不大於」兩種結果，$n$ 個元素有 $n!$ 種可能的排列，要從 $n!$ 種裡面挑出正確的那一種，至少得問 $\log_2(n!)$ 次，而 $\log_2(n!)$ 大約就是 $n \log_2 n$。

Merge sort、heap sort 都停在這個下界上，quick sort 平均也是。

想要更快，就不能靠比較。

---

## 不比較怎麼排？

假設老師要排全班成績。成績 0 到 100 分。

她不需要兩兩比。她只需要：

1. 準備 101 個桶子，標號 0 到 100。
2. 每個學生把自己的成績丟進對應的桶子。
3. 從桶子 0 開始，一個一個倒出來。

排好了。沒有比較任何兩個成績。

這就是 counting sort：**每個數字就是自己的 index**。值是 4，放進 `count[4]`。不需要比誰大誰小。

---

## 範例資料 `[4, 2, 2, 8, 3, 3, 1]`

這組輸入有重複值（兩個 2、兩個 3）。值域小（1 到 8）。

Counting sort 最適合的就是這種資料：**值域小、重複值多**。比較排序碰到重複值不會變快，counting sort 碰到重複值只是同一格 count 加 1，$O(1)$。

要是這組資料交給 quick sort，pivot 選到 2 或 3 就會有一堆相等元素擠在一起，partition 切不出平衡的兩半。counting sort 不受影響，因為它根本不看兩個元素誰大。

---

## 程式碼

自己拖，或按播放。上排在計數階段是 `nums`，橘色那根是正在數的元素，同時下排對應的那一格會亮起來。數完之後上排清空換成 `result`，一格一格填，下排改用游標標現在掃到 `count` 的哪一格。

下排裝的是「出現幾次」，最多才 2，長條全部貼著地，那一排看數字就好。

<div data-algo-viz="counting-sort" data-input="4,2,2,8,3,3,1"></div>

```typescript
function countingSort(nums: number[], maxVal: number): number[] {
  const count = new Array<number>(maxVal + 1).fill(0);
  for (const v of nums) {                    // 計數
    count[v]++;
  }
  const result: number[] = [];
  for (let v = 0; v <= maxVal; v++) {        // 按順序輸出
    for (let i = 0; i < count[v]; i++) {
      result.push(v);
    }
  }
  return result;
}
```

排成績就傳 `maxVal = 100`，count 開 101 格；上面那組 `[4, 2, 2, 8, 3, 3, 1]` 最大是 8，傳 8，count 開 9 格。`maxVal` 是呼叫的人要給的最大值，決定 count 開幾格，因為 count 的 index 就是值本身，最大的那個值得有位子。事先不知道最大值的話，先掃一遍 `Math.max(...nums)` 也行，多這一遍還是 $O(n)$。

兩個迴圈。第一個掃過原始資料，數每個值出現幾次。第二個掃過 count 陣列，按順序輸出。

整個過程沒有比較大小，也沒有交換。

---

## 走一遍 `[4, 2, 2, 8, 3, 3, 1]`

值域 0 到 8。開一個長度 9 的 count 陣列。

### 第一步：計數

```
掃過每個元素，對應的桶子 +1：

  nums[0] = 4 → count[4]++
  nums[1] = 2 → count[2]++
  nums[2] = 2 → count[2]++
  nums[3] = 8 → count[8]++
  nums[4] = 3 → count[3]++
  nums[5] = 3 → count[3]++
  nums[6] = 1 → count[1]++

count 陣列：
  index:  0  1  2  3  4  5  6  7  8
  count: [0, 1, 2, 2, 1, 0, 0, 0, 1]
              ↑  ↑  ↑  ↑           ↑
              1  2  3  4           8
           1次 2次 2次 1次        1次
```

### 第二步：按 count 輸出

```
index 0: count=0，跳過
index 1: count=1，輸出 1       → [1]
index 2: count=2，輸出 2, 2    → [1, 2, 2]
index 3: count=2，輸出 3, 3    → [1, 2, 2, 3, 3]
index 4: count=1，輸出 4       → [1, 2, 2, 3, 3, 4]
index 5: count=0，跳過
index 6: count=0，跳過
index 7: count=0，跳過
index 8: count=1，輸出 8       → [1, 2, 2, 3, 3, 4, 8]

done: [1, 2, 2, 3, 3, 4, 8]
```

7 個元素，掃了兩遍（一遍計數、一遍輸出）。加上 count 陣列長度 9。總共 $O(n + k)$。

n = 7，k = 9。如果 n = 100,000 且成績還是 0-100，那 k = 101。$O(100,000 + 101) = O(n)$。

比較排序至少要 100,000 * 17 = 1,700,000 次比較。Counting sort 掃兩遍就結束。

---

## 跟比較排序的差別

| | Quick Sort | Merge Sort | Counting Sort |
|---|---|---|---|
| 對 `[4,2,2,8,3,3,1]` | 需要比較和 swap | 需要比較和 merge | 掃兩遍，不比較 |
| 重複值多 | pivot 可能退化 | 無影響 | 更快（count++ 就好） |
| 值域大 | 無影響 | 無影響 | 開不下（count 陣列超出記憶體） |

比較排序不在乎值本身是什麼，只看大小關係。

Counting sort 在乎。值域決定 count 陣列的大小。

---

## 什麼時候用？什麼時候不用？

**用：**
- 值域小。成績 0-100。字元 0-127。月份 1-12。
- 資料量大。n = 1,000,000 筆成績。
- 需要線性時間。

**不用：**
- 值域大。成績 0-100 沒問題，但要是資料變成身分證字號 0 到 $10^9$，count 陣列就要 10 億格、4 GB 記憶體，光是排序就把記憶體用完了。
- 不是整數。浮點數跟字串都不行，因為值沒辦法直接當 index。
- 值域遠大於 n。n = 100 筆資料，值域 0 到 100,000？count 陣列大部分是 0。浪費。

經驗法則：k（值域）不超過 n 的幾倍時，counting sort 才值得。

---

## 穩定版 Counting Sort

三個學生按分數排：

```
[(小明, 3), (小華, 1), (阿德, 3)]
```

上面那個簡化版排不了這組資料。`count[3] = 2` 記得的只有「兩個人考 3 分」，記不得是小明跟阿德。輸出那一步是 `result.push(v)`，印兩個 3 出來就結束，名字在計數的時候就丟掉了。

排裸數字看不出差別，因為 3 就是 3，誰先誰後長得一樣。要按分數排學生、按日期排訂單，簡化版就不能用。

穩定版改成搬原本的元素，所以它得先知道每個值該落在 output 的哪幾格。累加那一步算的就是這個。

```typescript
function stableCountingSort(nums: number[], maxVal: number): number[] {
  const count = new Array<number>(maxVal + 1).fill(0);
  for (const v of nums) {
    count[v]++;
  }
  // 累加：count[i] = 值 <= i 的元素有幾個
  for (let i = 1; i <= maxVal; i++) {
    count[i] += count[i - 1];
  }
  // 從後往前放，保證穩定
  const output = new Array<number>(nums.length);
  for (let i = nums.length - 1; i >= 0; i--) {
    const v = nums[i];
    count[v]--;
    output[count[v]] = nums[i];
  }
  return output;
}
```

code 為了單純還是排數字。換成物件也一樣：`count` 的 index 改用 `nums[i].score`，搬進 output 的還是 `nums[i]` 本體。

回到那三個學生。累加之後 `count[1] = 1`、`count[2] = 1`、`count[3] = 3`。`count[3] = 3` 的意思是「分數小於等於 3 的有 3 個人」，所以考 3 分的最後一個要放在 index 2。

接著從後往前掃原陣列：

```
阿德(3)：count[3] 減一變 2 → output[2] = 阿德
小華(1)：count[1] 減一變 0 → output[0] = 小華
小明(3)：count[3] 減一變 1 → output[1] = 小明

output = [小華(1), 小明(3), 阿德(3)]
```

小明原本在阿德前面，排完還在前面。

為什麼非得從後往前？因為 `count[v]` 一路遞減，先被處理的那個人拿到比較後面的位子。改成從前往後掃：

```
小明(3)：count[3] 減一變 2 → output[2] = 小明
小華(1)：count[1] 減一變 0 → output[0] = 小華
阿德(3)：count[3] 減一變 1 → output[1] = 阿德

output = [小華(1), 阿德(3), 小明(3)]
```

阿德跑到小明前面，兩個同分的人對調了。

Radix sort 的每一位排序就是用這個穩定版。它排完個位再排十位，十位相同的那幾個元素靠的是個位那輪留下來的順序，順序一亂結果就錯。

---

## 複雜度

| | 值 |
|---|---|
| 最佳 | $O(n + k)$ |
| 平均 | $O(n + k)$ |
| 最糟 | $O(n + k)$ |
| 空間 | $O(k)$ |
| In-place | 否 |
| 穩定 | 是（穩定版） |

k = 值域大小（count 陣列的長度）。如果 k 跟 n 差不多大，$O(n + k) = O(n)$，線性。

但如果 k 遠大於 n？想像 `[1, 3, 5, 10000, 3, 2, 5, 4380435435]`：只有 8 筆資料，count 陣列卻要開到 44 億格。64-bit 系統一個 int 佔 8 bytes，44 億 $\times$ 8 = 33 GB。一般電腦 16 GB RAM 根本開不出來。比較排序只要 $O(8 \times 3) = 24$ 次比較就排完了。

---

## 面試考什麼？

直接考 counting sort 的題不多。但前面提過，radix sort 每一位的排序就是用它。

相關的題：

- [Sort Colors](/problem/sort-colors) — 值域只有 0, 1, 2。counting sort 的特例（也可以用 Dutch National Flag）
- [Top K Frequent Elements](/problem/top-k-frequent-elements) — 先 count，再找 top k
- [Sort an Array](/problem/sort-an-array) — 如果值域小，counting sort 是最快的

面試官不太會要求手寫 counting sort。但要是題目的值域很小，主動提出用 counting sort 會加分。

---

## 總結

比較排序的下界是 $O(n \log n)$，counting sort 不受這個下界限制，因為它不做比較。

它只計數，掃兩遍就結束，$O(n + k)$。

代價是限制：只能排整數，值域不能太大。但在對的場景（值域小、資料量大、重複值多），它比任何比較排序都快。

回到 [Sorting 總覽](/concept/sorting) 看全貌。想學怎麼把 counting sort 推廣到更大的數字？看 [Radix Sort](/concept/radix-sort)。
