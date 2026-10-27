---
title: "Big O 直覺速查"
category: Math & Bit
slug: big-o
subtitle: 時間複雜度一眼判斷
date: 2026-02-25T19:53:22
---

# Big O

面試官問 "What's the time and space complexity?"，時間跟空間兩個都要答。

下面不推數學定義，只列**看到什麼 pattern 就對應什麼複雜度**。

---

## 時間複雜度

### O(1) — 常數

不管 n 多大，固定幾步就完。

```go
v := nums[3]          // 直接算位址，O(1)
m["key"] = 42         // hash map 存取，均攤 O(1)
stack = stack[:len(stack)-1]  // pop，O(1)
```

看到：陣列取值、hash map 操作、stack push/pop → O(1)。

### $O(\alpha(n))$ — 反 Ackermann

Union-find 加上路徑壓縮跟 union by rank 之後，每次操作均攤是 $O(\alpha(n))$。$\alpha$ 是 Ackermann 函數的反函數，成長慢到就算 n 大到 $10^{80}$，$\alpha(n)$ 也不超過 4，所以答題的時候直接說「當成常數」。

```go
func find(x int) int {
    if parent[x] != x {
        parent[x] = find(parent[x])   // 路徑壓縮：回程把整條路上的節點都指到 root
    }
    return parent[x]
}
```

兩個優化要一起用才有這個界。只做路徑壓縮、不做 union by rank 的話，均攤是 $O(\log n)$。

看到：union-find 的 find / union → $O(\alpha(n))$。

### O(log n) — 對數

每一步砍掉一半。

```go
func binarySearch(nums []int, target int) int {
    lo, hi := 0, len(nums)-1
    for lo <= hi {
        mid := lo + (hi-lo)/2
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            lo = mid + 1    // 砍掉左半邊
        } else {
            hi = mid - 1    // 砍掉右半邊
        }
    }
    return -1
}
```

n = 1,000,000 只要 20 步（$\log_2{10^6} \approx 20$）。

看到：每次砍一半、balanced tree 的高度、binary search → O(log n)。

### $O(\sqrt{n})$ — 根號

判斷 n 是不是質數，只要試除到 $\sqrt{n}$ 就夠。因為 $n = a \times b$ 的話，a 跟 b 不可能同時大於 $\sqrt{n}$，所以小的那個一定落在 $\sqrt{n}$ 以內。

```go
func isPrime(n int) bool {
    for i := 2; i*i <= n; i++ {    // 只跑到 √n
        if n%i == 0 {
            return false
        }
    }
    return n > 1
}
```

另一個來源是分塊（sqrt decomposition）：把 n 個元素切成 $\sqrt{n}$ 塊、每塊 $\sqrt{n}$ 個。查詢的時候掃過 $\sqrt{n}$ 塊，更新的時候只動其中一塊，兩邊都是 $O(\sqrt{n})$。

看到：試除法判質數、把陣列切成 $\sqrt{n}$ 塊 → $O(\sqrt{n})$。

### O(n) — 線性

看過每個元素一次。

```go
sum := 0
for _, v := range nums {    // 一層 loop
    sum += v
}
```

看到：一層 for loop、hash map 建表、linked list 遍歷 → O(n)。

### O(n log n) — 排序等級

排序的理論下界。或是「對每個元素做一次 O(log n) 的操作」。

```go
slices.Sort(nums)           // 排序本身 O(n log n)

// 或：對每個元素做一次 binary search
for _, v := range nums {    // O(n) 次
    binarySearch(sorted, v) // 每次 O(log n)
}
```

看到：排序、heap 建立後逐一取出、merge sort 的 merge → O(n log n)。

### O(n log log n) — 篩法等級

比 $O(n \log n)$ 快，但比 $O(n)$ 慢。`log log n` 就是 log 做兩次：`log n` 是「2 要連乘幾次才到 n」，`log log n` 是「2 要連乘幾次才到那個數字」。n 從 16 長到 $2^{64}$，`log log n` 才從 2 變到 6，幾乎是常數。

不常見，但埃拉托斯特尼篩法（Sieve of Eratosthenes）就是這個。在 LeetCode 裡大概只有 204. Count Primes 會碰到。

每個質數 p 標記大約 $n/p$ 個倍數，加總所有質數：

$$\frac{n}{2} + \frac{n}{3} + \frac{n}{5} + \cdots = n \times \underbrace{\left(\frac{1}{2} + \frac{1}{3} + \frac{1}{5} + \cdots\right)}_{\log \log n}$$

$\log \log n$ 成長極慢。$n = 5 \times 10^6$ 時只有 4.5，而 $\sqrt{n} \approx 2236$。所以 $O(n \log \log n) \ll O(n\sqrt{n})$。

看到：質數篩法 → $O(n \log \log n)$。

### $O(n^2)$ — 兩層迴圈

```go
for i := 0; i < n; i++ {
    for j := i + 1; j < n; j++ {   // 對每個 i 掃剩下的
        // ...
    }
}
```

看到：兩層巢狀 loop、brute force 配對 → $O(n^2)$。

### $O(n^3)$ — 三層迴圈

Floyd-Warshall 求所有點對之間的最短路：對每一個中繼點 k，檢查每一組 (i, j) 繞過 k 會不會比原本短。

```go
for k := 0; k < n; k++ {              // 中繼點
    for i := 0; i < n; i++ {
        for j := 0; j < n; j++ {
            if dist[i][k]+dist[k][j] < dist[i][j] {
                dist[i][j] = dist[i][k] + dist[k][j]
            }
        }
    }
}
```

n 稍微一大就跑不動。$n = 500$ 就是 $1.25 \times 10^8$ 次內圈，純迴圈約 120 ms；$n = 1000$ 是 $10^9$ 次，才逼近一秒（實測數字見[規模直覺](/concept/scale-intuition)）。

看到：三層巢狀 loop、Floyd-Warshall、樸素的矩陣相乘 → $O(n^3)$。

### $O(2^n)$ — 指數

每個元素選或不選，所有子集。

```go
// 產生所有子集
func subsets(nums []int) [][]int {
    if len(nums) == 0 {
        return [][]int{{}}
    }
    rest := subsets(nums[1:])       // 不選第一個
    var with [][]int
    for _, s := range rest {
        with = append(with, append([]int{nums[0]}, s...))  // 選第一個
    }
    return append(rest, with...)
}
```

看到：子集問題、遞迴每層分兩支 → $O(2^n)$。

### O(n!) — 階乘

所有排列。

看到：全排列、旅行推銷員 brute force → O(n!)。n = 20 就是 $2.4 \times 10^{18}$。別想了。

---

## 數字對照表

光看符號沒感覺。用數字感受每個複雜度在 n 變大時的差異：

| n | $O(\log n)$ | $O(n)$ | $O(n \log \log n)$ | $O(n \log n)$ | $O(n\sqrt{n})$ | $O(n^2)$ | $O(2^n)$ |
|---|---|---|---|---|---|---|---|
| 10 | 3 | 10 | 17 | 33 | 32 | 100 | 1,024 |
| 100 | 7 | 100 | 273 | 664 | 1,000 | 10,000 | $10^{30}$ |
| 1,000 | 10 | 1,000 | 3,317 | 9,966 | 31,623 | $10^6$ | 太大 |
| $10^5$ | 17 | $10^5$ | 405,395 | 1,660,964 | 31,622,777 | $10^{10}$ | 太大 |
| $10^6$ | 20 | $10^6$ | 4,316,983 | 19,931,569 | $10^9$ | $10^{12}$ | 太大 |
| $10^9$ | 30 | $10^9$ | 4,901,945,847 | 29,897,352,854 | 31,622,776,601,684 | $10^{18}$ | 太大 |


- $\log \log n$ 幾乎不動。$\log_2 10 \approx 3.3$，再取一次 $\log_2 3.3 \approx 1.7$；$\log_2 10^9 \approx 30$，$\log_2 30 \approx 4.9$。n 長了 $10^8$ 倍，$\log \log n$ 只從 1.7 變成 4.9
- $n \log \log n$ 跟 $n$ 幾乎一樣。篩法這麼快就是因為這個
- $n \log n$ 跟 $n^2$ 之間差距巨大。$n = 10^6$ 時差 50,000 倍
- $n\sqrt{n}$ 卡在 $n \log n$ 和 $n^2$ 中間。暴力試除法就在這裡

---

## 空間複雜度

同一套邏輯，問的是「額外用了多少記憶體」。

| 看到什麼 | 空間 |
|---|---|
| 幾個變數 | O(1) |
| hash map / set 存 n 個元素 | O(n) |
| 複製一份陣列 | O(n) |
| 遞迴深度 n 層（linked list） | O(n) — call stack |
| 遞迴深度 log n 層（balanced tree） | O(log n) — call stack |
| 2D DP table $n \times m$ | $O(n \times m)$ |

遞迴的 call stack 也算空間。Quick Sort 看起來 in-place，但平均遞迴深度 O(log n)，最壞情況每次都切出空的一邊，深度變 O(n)，兩種都不是 O(1)。

---

## 均攤 O(1) — Amortized

不是每次都 O(1)，但平均下來是。

最常見的例子：`append()` / `push()`。大部分時候 O(1)，擴容時 O(n)，但因為倍增策略，均攤是 O(1)。

詳見 [Array 與 Slice 操作複雜度](/concept/array-operations)。

---

## 怎麼快速判斷？

面試時不需要精確推導。用這個決策樹：

```
有迴圈嗎？
├── 沒有 → O(1)
├── 一層，跑 n 次 → O(n)
├── 一層，每次砍半 → O(log n)
├── 兩層巢狀 → O(n²)
└── 有遞迴？
    ├── 每層分兩支，深度 n → O(2ⁿ)
    ├── 每層分兩支，深度 log n → O(n log n)（merge sort）
    └── 線性遞迴，深度 n → O(n)
```

然後看有沒有排序：有排序就至少 O(n log n)。

---

## 常見題型對應

| 複雜度 | 典型題型 | 代表技巧 |
|---|---|---|
| O(1) | 數學公式、bit manipulation | XOR、位移 |
| O(log n) | Binary Search | 排序陣列搜尋 |
| O(n) | Two Pointer、Sliding Window、Hash Map | 一次遍歷 |
| O(n log n) | 排序後處理 | Sort + Two Pointer |
| O(n log log n) | 質數篩法 | Sieve of Eratosthenes |
| $O(n^2)$ | 暴力配對、DP | 兩層迴圈 |
| $O(2^n)$ | 子集、回溯 | Backtracking |
| O(n!) | 全排列 | Brute force |

---

## 英文怎麼唸

面試要唸出口的術語，寫法跟唸法對照：

| 寫法 | 唸法 |
|---|---|
| O(1) | O of one |
| $O(\alpha(n))$ | O of alpha n |
| O(log n) | O of log n |
| $O(\sqrt{n})$ | O of root n（完整版 O of square root of n） |
| O(n) | O of n |
| O(n log log n) | O of n log log n |
| O(n log n) | O of n log n |
| $O(n\sqrt{n})$ | O of n root n |
| $O(n^2)$ | O of n squared |
| $O(n^3)$ | O of n cubed |
| $O(2^n)$ | O of two to the n |
| O(n!) | O of n factorial |

log 唸 log，跟 dog 押韻，不是拼三個字母。底數平常不講，真的要指明才說 log base two of n。

想強調「大 O」就講 big O：the big O is n log n。

次方唸「ten to the eighth」這種：$10^5$ 是 ten to the fifth，$10^8$ 是 ten to the eighth，$10^9$ 是 ten to the ninth。也可以直接講數字，a hundred million。

一整句連起來的話：

> Ten to the eighth operations per second is the rule of thumb. At n equals ten to the fifth, an O of n log n solution is about 1.7 million operations, so that is fine. O of n squared would be ten to the tenth, which is way over.

這句裡兩種唸法混著用。$10^5$、$10^8$、$10^{10}$ 是整數次方，唸 ten to the fifth 這種；1.7 million 不是整數次方，硬要唸成 one point seven times ten to the sixth 很拗口，直接講數字就好。

---

## 結論

Big O 不是數學考試。看到 pattern 就對應出複雜度，兩秒內答出來，時間和空間都要說。

排序的細節看 [Sorting](/concept/sorting)。陣列操作的複雜度看 [Array 與 Slice](/concept/array-operations)。
