---
title: "Bit Operators"
category: Math & Bit
slug: operator
subtitle: AND OR XOR SHIFT，位元操作
date: 2026-02-24T23:13:16
updated: 2026-07-29
---

# Bit Operators

每個工程師都學過 `>>` 和 `<<`。然後就忘了。

直到你在 LeetCode 碰到 bit manipulation 題，才發現這些運算子能把 O(n) 壓成 O(1)，把十行程式碼縮成一行。

Part 1 是基本運算子。Part 2 是組合技，把基本運算子串起來解題。Part 3 集中處理容易搞混的術語：反碼、一補數、二補數、反數。

建議順序讀。Basic 零術語，只講操作和效果。術語全部留到 Part 3。

---

# Part 1: 基本運算子

---

## `>> 1` — 除以 2

右移一位 = 除以 2，無條件捨去。最右邊那位被丟掉。

```
13 = 1101
         ↓ 全部往右推一格，最右邊的 1 掉出去
 6 = 0110
```

```js
13 >> 1  // 6
```

Binary search 算 mid 最常用：

```js
const mid = (lo + hi) >> 1
```

為什麼常寫成 `>> 1` 而不是 `Math.floor((lo + hi) / 2)`？`>> 1` 是一條 CPU 指令，省掉 `Math.floor` 的函式呼叫；`(lo + hi) / 2` 還會先產生浮點數再截回整數，`>> 1` 全程整數。快一點點，但差距很小。

`(lo + hi) >> 1` **不能**用來避免溢位。

- Java/C++：`lo + hi` 會先超過 int 上限變成負數，之後 `>>` 是有號右移（保留負號），mid 變負數，直接 `ArrayIndexOutOfBoundsException`。
- JavaScript：`>>` 會先把數字轉成 32-bit 有號整數（ToInt32），所以就算 Number 是 64-bit，`(lo + hi) >> 1` 一樣在 $2^{31}$ 就爆掉。

真正避開溢位有兩條路：`lo + (hi - lo) / 2`（[#704](/problem/binary-search) 用的就是這個，任何語言都對），或 Java 的 `(lo + hi) >>> 1`（無號右移，就算 `lo + hi` 溢位成負值，把那串 bit 當無號除以 2 還是對，這是 JDK 官方的修正）。

### LeetCode

- [#704 Binary Search](/problem/binary-search) — 基本 binary search，mid 用 `lo + (hi - lo) / 2` 避免溢位（`>> 1` 不防溢位，見上）
- [#29 Divide Two Integers](/problem/divide-two-integers) — 不能用除法，只能用 bit shift 做除法
- [#50 Pow(x, n)](/problem/powx-n) — 快速冪，每次 `n >>= 1` 把指數砍半

---

## `<< n` — 乘以 2 的次方

往左推，右邊補 0。推幾格就乘以 2 的幾次方。

```
1 = 0001
         ↓ 左移 3 格
8 = 1000

5 = 00101
          ↓ 左移 2 格，等於 * 4
20= 10100
```

```js
1 << 3   // 8
5 << 2   // 20
```

常用在建 mask 和快速乘法。

### LeetCode

- [#78 Subsets](/problem/subsets) — 用 bitmask 枚舉所有子集，`1 << n` 個組合
- [#190 Reverse Bits](/problem/reverse-bits) — `result = (result << 1) | (n & 1)` 逐位反轉

走一遍 #190 Reverse Bits，反轉 `n = 1101`（簡化成 4 bits）。起始 `result = 0000`，每步做同一件事：`n & 1` 取出最後一位、塞進 `result` 尾端、`n` 右移一位。

- 第 1 步：`n & 1 = 1`，`result = (0000 << 1) | 1 = 0001`，`n` 右移後 `= 0110`
- 第 2 步：`n & 1 = 0`，`result = (0001 << 1) | 0 = 0010`，`n = 0011`
- 第 3 步：`n & 1 = 1`，`result = (0010 << 1) | 1 = 0101`，`n = 0001`
- 第 4 步：`n & 1 = 1`，`result = (0101 << 1) | 1 = 1011`，`n = 0000`

原本 `1101`，反轉 `1011`。
- [#371 Sum of Two Integers](/problem/sum-of-two-integers) — 不能用 `+`，用 XOR 做加法，AND + 左移做進位

---

## `& 1` — 判斷奇偶

AND 只保留兩邊都是 1 的位。`& 1` 就是只看最後一位。

```
7 = 0111
    0001  ← & 1
  ------
    0001  → 1（奇數）

8 = 1000
    0001  ← & 1
  ------
    0000  → 0（偶數）
```

```js
7 & 1  // 1
8 & 1  // 0
```

真的比 `n % 2` 快嗎？幾乎沒差。除法指令確實慢（幾十個 cycle，AND 只要一個），但那是除數為變數時的事；除數是常數 2，編譯器跟 V8 的 JIT 都會把 `% 2` 改寫成位元運算，除法電路根本不會被用到。挑 `& 1` 的理由是語意：它明確在說「我在看最後一位」，讀 code 的人秒懂意圖。

真實的差別在負數：`-3 % 2` 是 `-1`（餘數跟著被除數的符號），`-3 & 1` 是 `1`。用 `n % 2 === 1` 判奇數會漏掉負奇數，`(n & 1) === 1` 不會。

### LeetCode

- [#191 Number of 1 Bits](/problem/number-of-1-bits) — 每次 `n & 1` 取最低位，`n >>= 1` 右移
- [#338 Counting Bits](/problem/counting-bits) — `dp[i] = dp[i >> 1] + (i & 1)`，一行搞定
- [#1009 Complement of Base 10 Integer](/problem/complement-of-base-10-integer) — 逐位取反，`& 1` 取每一位

---

## `|` OR — 設定 bit

OR 只要有一邊是 1，結果就是 1。用來把指定的位「打開」。

```
  0100  (4)
| 0010  (2)
------
  0110  (6)  ← 第 1 位被打開了
```

```js
4 | 2  // 6
```

三種 bit 操作對照：

| 想做的事 | 用什麼 | 公式 | 例（n = 0101） |
|----------|--------|------|----------------|
| 設第 k 位為 1 | OR | `n \| (1 << k)` | k=1：`0101 \| 0010 = 0111` |
| 清第 k 位為 0 | AND + NOT | `n & ~(1 << k)` | k=2：`0101 & 1011 = 0001` |
| 翻轉第 k 位 | XOR | `n ^ (1 << k)` | k=0：`0101 ^ 0001 = 0100` |

權限系統常用 OR 合併 flag：

```js
const READ  = 1   // 001
const WRITE = 2   // 010
const EXEC  = 4   // 100

const perm = READ | EXEC  // 101 = 5
```

檢查權限用 AND：

```js
if (perm & WRITE) { ... }  // 101 & 010 = 000 → 沒有寫入權限
if (perm & READ)  { ... }  // 101 & 001 = 001 → 有讀取權限
```

### LeetCode

- [#190 Reverse Bits](/problem/reverse-bits) — `result = (result << 1) | (n & 1)`，OR 把新的 bit 塞進去
- [#371 Sum of Two Integers](/problem/sum-of-two-integers) — XOR + AND + OR 組合做加法

---

## `^` XOR — 相同為 0，不同為 1

兩個位元一樣就是 0，不一樣就是 1。

```
  0101  (5)
^ 0011  (3)
------
  0110  (6)
  ^^     ← 這兩位不同，所以是 1
```

兩條規則讓它在解題時特別好用：

```
a ^ a = 0    自己 XOR 自己，每位都相同，全變 0
a ^ 0 = a    XOR 0 等於不變
```

所有出現兩次的數互相抵消。剩下的就是答案。

走一遍 #136 Single Number — `[4, 1, 2, 1, 2]`，找出只出現一次的數：

```
result = 0          00000

XOR 4:  00000
      ^ 00100
      -------
        00100       result = 4

XOR 1:  00100
      ^ 00001
      -------
        00101       result = 5

XOR 2:  00101
      ^ 00010
      -------
        00111       result = 7

XOR 1:  00111       ← 1 出現第二次
      ^ 00001
      -------
        00110       result = 6

XOR 2:  00110       ← 2 出現第二次
      ^ 00010
      -------
        00100       result = 4   ← 答案

1 和 2 各 XOR 了兩次，自己消掉自己。只剩 4。
```

### LeetCode

- [#136 Single Number](/problem/single-number) — 全部 XOR 起來，出現兩次的消掉
- [#268 Missing Number](/problem/missing-number) — 把 index 和 value 全部 XOR，缺的那個留下來
- [#137 Single Number II](/problem/single-number-ii) — 每個數出現三次，用位元計數取餘
- [#260 Single Number III](/problem/single-number-iii) — 兩個數各出現一次，XOR 之後用最低位的 1 分群

---

## `~` NOT — 每個 bit 翻轉

把 0 變 1，1 變 0。就這樣。

```
 x = 0000 0101  (5)
~x = 1111 1010  (-6, signed 32-bit)
```

5 取反怎麼是 -6？公式：`~x = -(x + 1)`。永遠成立。現在不需要知道為什麼，Part 3 會解釋。

```js
~5    // -6
~0    // -1
~(-1) // 0
```

### LeetCode

- [#476 Number Complement](/problem/number-complement) — 建 mask `(1 << len) - 1`，XOR 取反
- [#1009 Complement of Base 10 Integer](/problem/complement-of-base-10-integer) — 同上，不含前導零的反轉
- [#371 Sum of Two Integers](/problem/sum-of-two-integers) — 處理負數時會用到 NOT

---

## `>>> 0` — 轉 unsigned（JS 專屬）

JavaScript 專屬。`>>` 和 `<<` 把數字當 signed 32-bit。最高位是 1 時數字會變負數。

> **Python 秘密**：Python 的整數沒有位元上限，`-1` 在記憶體裡是無限長的 1。所以 `-1 & 1 = 1`，`1 << 1 = 2`，`2 << 1 = 4`⋯ 進位永遠不會消失。LeetCode 上很多 Python bitwise 解法都要加 `& 0xFFFFFFFF` 把數字限制在 32-bit，不然會無限迴圈。JS 的 bitwise 已經強制 32-bit，這招在 JS 用不著。

`>>> 0` 強制轉回 unsigned，把同樣的 32 個 bit 當正整數讀。

```
-1 的 32-bit 表示：
1111 1111 1111 1111 1111 1111 1111 1111

signed 讀法  → -1
unsigned 讀法 → 4294967295（每位都是 1 = 2^32 - 1）
```

```js
-1 >>> 0  // 4294967295
```

### LeetCode

- [#190 Reverse Bits](/problem/reverse-bits) — 最後 `return result >>> 0`，不然答案可能是負數

---

# Part 2: 組合技

Basics 組合出來的解題 pattern。每個 trick 都由 Part 1 的運算子拼成。

---

## `n & (n - 1)` — 消掉最低位的 1

`n - 1` 會把最低位的 1 變成 0，它右邊的 0 全變成 1。AND 一下，最低位的 1 就消失了。

```
n     = 1100  (12)
n - 1 = 1011  (11)  ← 最低的 1 那位翻轉，右邊全填 1
      ------
        1000  (8)   ← 最低的 1 被消掉了
```

```js
12 & 11  // 8
```

走一遍 #191 Number of 1 Bits — 數 `n = 27` 裡有幾個 1：

```
count = 0

n = 11011  (27)
n & (n-1):
    11011
  & 11010
  -------
    11010  (26)  → count = 1

n = 11010  (26)
n & (n-1):
    11010
  & 11001
  -------
    11000  (24)  → count = 2

n = 11000  (24)
n & (n-1):
    11000
  & 10111
  -------
    10000  (16)  → count = 3

n = 10000  (16)
n & (n-1):
    10000
  & 01111
  -------
    00000  (0)   → count = 4

n = 0，結束。答案：4
```

每消一次就少一個 1。消幾次就有幾個 1。比逐位檢查快。

### LeetCode

- [#191 Number of 1 Bits](/problem/number-of-1-bits) — 消幾次就有幾個 1
- [#231 Power of Two](/problem/power-of-two) — `n > 0 && (n & (n - 1)) === 0`，一行解
- [#342 Power of Four](/problem/power-of-four) — 先確認是 2 的次方，再檢查 1 在偶數位
- [#461 Hamming Distance](/problem/hamming-distance) — XOR 之後數 1，用 `n & (n-1)` 消

---

## `n & (-n)` — 取最低位的 1（lowbit）

`-n` 是 n 的負數表示。AND 之後只剩最低位的 1。結果一定是 2 的次方。

```
 n  = 0000 1100  (12)
-n  = 1111 0100  (-12)

  0000 1100  (12)
& 1111 0100  (-12)
-----------
  0000 0100  (4)  ← 只剩最低位的 1
```

```js
12 & -12  // 4
```

為什麼會這樣？`-n` 會把最低位的 1 以下的 bit 全部翻轉再歸位，只留那一個 1 跟原本對齊。想搞懂細節，Part 3 有完整推導。

### LeetCode

- [#307 Range Sum Query - Mutable](/problem/range-sum-query-mutable) — Fenwick Tree，`i += i & (-i)` 往上走，`i -= i & (-i)` 往下走
- [#315 Count of Smaller Numbers After Self](/problem/count-of-smaller-numbers-after-self) — Fenwick Tree 應用

---

## `(1 << k) - 1` — 低 k bits 的 mask

先把 1 推到第 k 位，再減 1，下面的位全變成 1。

```
1 << 4  = 1 0000  (16)
減 1    = 0 1111  (15)  ← 低 4 位全是 1
```

拿來當 mask，AND 之後只保留低 k 位：

```
  1011 0110  (182)
& 0000 1111  ← (1 << 4) - 1
-----------
  0000 0110  (6)   ← 只留低 4 位
```

```js
(1 << 4) - 1  // 15 = 0b1111
182 & 0xF     // 6
```

Protobuf varint 的 `tag & 0x7f` 就是 `tag & ((1 << 7) - 1)`：只取低 7 bits。

### LeetCode

- [#476 Number Complement](/problem/number-complement) — 找到最高位的 1，建 mask，XOR 取反
- [#1009 Complement of Base 10 Integer](/problem/complement-of-base-10-integer) — 同上

---

## Bitmask 子集枚舉

用一個整數的每一位代表「選或不選」。n = 3 時有 8 種組合：

```
mask = 000 → 什麼都不選
mask = 001 → 選第 0 個
mask = 010 → 選第 1 個
mask = 011 → 選第 0、1 個
...
mask = 111 → 全選
```

怎麼檢查第 i 位？把 1 左移 i 格，AND 看看：

```
mask   = 0101
1 << 2 = 0100
       ------
         0100 → 不是 0，第 2 位是 1
```

走一遍 — 枚舉 `["a", "b", "c"]` 的所有子集：

```
mask = 000 → 沒選    → []
mask = 001 → 第0位=1 → ["a"]
mask = 010 → 第1位=1 → ["b"]
mask = 011 → 0,1=1   → ["a","b"]
mask = 100 → 第2位=1 → ["c"]
mask = 101 → 0,2=1   → ["a","c"]
mask = 110 → 1,2=1   → ["b","c"]
mask = 111 → 全選    → ["a","b","c"]
```

拿 mask = 101 走一遍內層迴圈：

```
i=0: mask & (1<<0) = 101 & 001 = 001 → 選 "a"
i=1: mask & (1<<1) = 101 & 010 = 000 → 不選
i=2: mask & (1<<2) = 101 & 100 = 100 → 選 "c"
→ ["a", "c"]
```

```js
for (let mask = 0; mask < (1 << n); mask++) {
  for (let i = 0; i < n; i++) {
    if (mask & (1 << i)) { ... }
  }
}
```

[#698 Partition to K Equal Sum Subsets](/problem/partition-to-k-equal-sum-subsets)、[#526 Beautiful Arrangement](/problem/beautiful-arrangement)

---

## Fenwick Tree — lowbit 應用

Fenwick Tree（Binary Indexed Tree）處理的情況：陣列會一直被修改，又會一直被查詢「前 i 格的和」。只存原始陣列，查詢是 O(n)；存 prefix sum，修改是 O(n)。Fenwick 讓兩邊都落在 O(log n)。

它的想法用積木講最順：**預先做好一批積木，每塊積木是一段連續格子的和**。查詢是拿積木鋪地板；修改是把蓋著那一格的積木逐塊更新。

**做積木**

每個 index 一塊積木，右端固定在第 i 格。多寬？看 i 的二進位裡**最右邊的 1** 站在哪一位（bit k → 寬 $2^k$ 格，這個值就是 `lowbit(i)`）：

```
最右邊的 1 在 bit 0（……1）→ 寬 1 格：1, 3, 5, 7（所有奇數）
最右邊的 1 在 bit 1（…10）→ 寬 2 格：2（10）, 6（110）, 10（1010）
最右邊的 1 在 bit 2（…100）→ 寬 4 格：4（100）, 12（1100）, 20（10100）
最右邊的 1 在 bit 3（…1000）→ 寬 8 格：8（1000）, 24（11000）
```

（16 = `10000` 的最右邊 1 在 bit 4，寬 16 格，屬於下一排。）

拿 nums = [3, 5, 2, 6, 1, 4, 7, 2] 把八塊積木全部畫出來，每塊存的是自己蓋住那幾格的和：

```
格子:      1   2   3   4   5   6   7   8

tree[1]  [ 3 ]                              和 = 3
tree[2]  [ 3   5 ]                          和 = 8
tree[3]          [ 2 ]                      和 = 2
tree[4]  [ 3   5   2   6 ]                  和 = 16
tree[5]                  [ 1 ]              和 = 1
tree[6]                  [ 1   4 ]          和 = 5
tree[7]                          [ 7 ]      和 = 7
tree[8]  [ 3   5   2   6   1   4   7   2 ]  和 = 30
```

每塊積木只看自己的二進位決定位置，彼此獨立。看圖還能發現一個性質：積木全部對齊 2 的次方刻度，所以**任兩塊要嘛不相交、要嘛小的完全被大的包住**，找不到只交疊一部分的。這個巢狀性等一下修改要靠它。

**查詢＝鋪地板**

查詢前 7 格的和，就是用積木把 [1..7] 這段地板不重不漏鋪滿。從右端開始，每次拿「以缺口右端結尾」的那塊：

- 缺口 [1..7]：拿 `tree[7]`（寬 1）鋪上 [7]，sum = 7。缺口縮成 [1..6]。
- 缺口 [1..6]：拿 `tree[6]`（寬 2）鋪上 [5..6]，sum = 12。缺口縮成 [1..4]。
- 缺口 [1..4]：拿 `tree[4]`（寬 4）一次鋪滿，sum = 28。完工。

三塊鋪完（驗證：3+5+2+6+1+4+7 = 28）。走法永遠行得通，因為每個位置都有一塊以它結尾的積木：缺口右端縮到哪，那裡就有現成的。

用二進位看，「缺口右端」的變化就是把 7 的 1 一顆一顆熄掉：

```
0111  →  0110  →  0100  →  0000
 [7]     [5..6]   [1..4]    完工

減 lowbit = 熄掉最低位的 1 = 鋪掉一塊
```

7 = 4 + 2 + 1，二進位有幾顆 1 就鋪幾塊，最多 log n 塊。

**修改＝蓋到這格的積木都要跟著改**

改了第 2 格，哪些積木的加總要跟著改？一個尺寸一個尺寸問：size 2 的積木有蓋到這格的嗎？size 4 的呢？size 8 的呢？同尺寸的積木彼此不重疊，所以每個尺寸最多一塊：

```
size 1：[1]、[3]、[5]、[7]  都沒蓋到第 2 格 → 不用動
size 2：[1..2]              蓋到 → tree[2] 跟著改
size 4：[1..4]              蓋到 → tree[4] 跟著改
size 8：[1..8]              蓋到 → tree[8] 跟著改
```

「加 lowbit」的跳法會自動把這份名單走完，缺席的尺寸自己跳過：

- `tree[2]` += delta，跳到 2 + lowbit(2) = 4。
- `tree[4]` += delta，跳到 4 + 4 = 8。
- `tree[8]` += delta，跳到 16，超出 n = 8，結束。

「加 lowbit」跳得到下一塊，是因為加法會進位：最低位的 1 被推往左邊，落點剛好是「包住目前這塊的最小積木」的右端。用二進位看：

```
0010   →   0100   →   1000   →   10000
tree[2]    tree[4]    tree[8]     出界

加 lowbit = 最低位的 1 往左進位 = 跳到下一塊更寬的積木
```

prefix sum 要修改七格的事，這裡只更新三塊。

**收尾：一對鏡像**

- query 是熄燈：把 i 的 1 逐顆熄掉，每熄一顆鋪一塊，熄完地板拼好。
- update 是進位：把最低位的 1 一路往左推，每推一次跳到更寬的一塊，推出界就結束。

兩個方向都在 log n 步內結束，i 的二進位有幾個 1 就走幾步。

```js
function update(i, delta) {
  for (; i <= n; i += i & (-i))
    tree[i] += delta
}
function query(i) {
  let sum = 0
  for (; i > 0; i -= i & (-i))
    sum += tree[i]
  return sum
}
```

[#307 Range Sum Query - Mutable](/problem/range-sum-query-mutable)

---

## 真實世界的例子：Svelte 的 bitmask dirty tracking

Bitmask 不是只出現在 LeetCode。Svelte 的渲染引擎核心就是它。

React 把 diff 引擎跟著 bundle 一起出貨：每次 state 改變，先建新的 Virtual DOM 樹、跟舊樹 diff、再 patch 真實 DOM。Svelte 全部跳過：compiler 在 build 時就分析完 template，直接產生針對性的 DOM 操作，用一個 bitmask 記錄哪些變數變了。

這段 Svelte 原始碼：

```svelte
<script>
  let count = 0;
  let name = "world";
  let active = false;
</script>
<p>{count}</p>
<p>{name}</p>
<div class:active>{name}</div>
```

編譯出來大概長這樣：

```js
// patch 函式：有東西變了才會跑
p(ctx, [dirty]) {
  if (dirty & 1) set_data(t0, ctx[0]);              // bit 0 = count
  if (dirty & 2) set_data(t1, ctx[1]);              // bit 1 = name
  if (dirty & 4) toggle_class(div, "on", ctx[2]);   // bit 2 = active
}
```

每個變數分到一個 bit。`dirty` 是一個整數，編碼「哪些變數變了」：

```
              bit 2    bit 1    bit 0
              active   name     count

都沒變:        0        0        0      → dirty = 0
只有 count:    0        0        1      → dirty = 1
只有 name:     0        1        0      → dirty = 2
count+name:   0        1        1      → dirty = 3
只有 active:   1        0        0      → dirty = 4
三個都變:      1        1        1      → dirty = 7
```

`dirty & 1` 檢查 count 變了沒、`dirty & 2` 檢查 name、`dirty & 4` 檢查 active。每個變數一次 bitwise AND、一條 CPU 指令。變了就更新那個 DOM 節點，沒變就跳過。沒有樹的 diff，沒有 reconciliation。

### 為什麼用 bitmask 不用物件？

```js
// 物件版：屬性查找、heap 配置
if (changed.count) ...
if (changed.name) ...

// bitmask 版：一個整數，每個變數一次 AND
if (dirty & 1) ...
if (dirty & 2) ...
```

整數活在暫存器裡，`&` 是一條 CPU 指令。物件要 heap 配置、屬性查找、垃圾回收。這段每次 state 改變都要跑，差距就有感。

### 超過 32 個變數怎麼辦？

JavaScript 的位元運算是 32-bit，一個 `dirty` 整數管 32 個變數。Svelte compiler 在 build 時數你的變數，超過 32 個就自動拆成多個整數：

```js
// 變數 ≤ 32 個：一個整數
p(ctx, [dirty]) {
  if (dirty & 1) ...
}

// 超過 32 個：compiler 自動拆
p(ctx, [dirty0, dirty1]) {
  if (dirty0 & 1)  ...   // 變數 0–31
  if (dirty1 & 1)  ...   // 變數 32
  if (dirty1 & 2)  ...   // 變數 33
}
```

這件事你永遠不用管，compiler 在 build 時決定。runtime 的成本就是每個變數一次 bitwise AND。

Source: [Virtual DOM is pure overhead — Rich Harris](https://svelte.dev/blog/virtual-dom-is-pure-overhead), [Svelte internals](https://svelte.dev/docs)

---

# Part 3: 繞口令

反碼、一補數、二補數、反數，四個詞長得像，來源不同，講的卻是同一串 bit。這裡集中處理所有混淆點。

用 Q&A 風格。每個問題對應一個常見的困惑。

---

## Q: 反碼是什麼？

反碼 = 一補數 = `~x`。三個名字，同一件事。把每個 bit 翻轉。

```
 x = 0000 0101  (5)
~x = 1111 1010
```

就這樣。沒有加 1，沒有負號。純粹把 0 變 1、1 變 0。

---

## Q: 二補數是什麼？為什麼 `~x + 1` 等於 `-x`？

「取二補數」= 「取反數」= `~x + 1` = `-x`。同一件事，四種說法。

```
 x   = 0000 0101  (5)
~x   = 1111 1010       ← 一補數（反碼）
~x+1 = 1111 1011       ← 二補數 = -5
```

這不是巧合。CPU 就是用這個方式表示負數。`-x` 的 bit pattern 就是 `~x + 1`。

同一個數字 5，取負號的過程分三步：

| 名稱 | 做法 | 結果（8-bit） | 值 |
|------|------|------|------|
| 原碼 | 本身 | `0000 0101` | 5 |
| 一補數（反碼） | 每 bit 翻轉 | `1111 1010` | — |
| 二補數 | 反碼 + 1 | `1111 1011` | -5 |

所以 `-x = ~x + 1`。這是二補數的定義。

也是 Part 1 裡 `~x = -(x + 1)` 的由來：把 `-x = ~x + 1` 移項，得到 `~x = -x - 1 = -(x + 1)`。

---

## Q:「二補數」一詞兩意？

對。這個詞同時指兩件事：

1. **系統**：CPU 用來表示有號整數的編碼方式（two's complement representation）
2. **操作**：對一個數取二補數（`~x + 1`），得到它的負數

講「電腦用二補數表示負數」→ 指系統。講「對 5 取二補數得 -5」→ 指操作。

---

## Q: 「反數」跟「二補數」什麼關係？

反數是數學概念。`a` 的（加法）反數是 `-a`，因為 `a + (-a) = 0`。跟 bit 無關。

在二補數系統裡，`-x` 剛好等於 `~x + 1`。所以反數的值和二補數的值一樣。但它們回答不同的問題：

| 術語 | 來源 | 問的問題 |
|------|------|------|
| 反碼 / 一補數 | 計算機 | 每個 bit 翻轉後長什麼樣？ |
| 二補數 | 計算機 | CPU 怎麼表示負數？ |
| 反數 | 數學 | 加上什麼會等於 0？ |

三個角度描述同一個結果。`x = 5`，我想要 `-5`：

- 程式設計師說：「我寫了 `-x`」
- 數學家說：「這是 5 的加法反元素（反數），因為 `5 + (-5) = 0`」
- CPU 說：「我把每個 bit 翻轉再加 1（二補數），得到 `1111 1011`」

結果都是同一串 bit。區分這些術語只在解釋原理時有用。寫 code 的時候，`-x` 就是 `-x`。

---

## Q: `n & (-n)` vs `n & (~n)` — 長得像但完全不同？

```
n    = 0000 1100  (12)

~n   = 1111 0011      ← 反碼：每 bit 翻轉
-n   = 1111 0100      ← 負數：~n + 1

n & ~n = 0000 0000    ← 永遠是 0（每個 bit 都相反）
n & -n = 0000 0100    ← lowbit（最低位的 1）
```

`-n` 比 `~n` 多加了 1。這個 +1 讓最低位的 1 以下全部翻回來，剛好只保留那一個 1。

容易搞混的點：`-n` 是反數（`n + (-n) = 0`），但 `n & (-n)` 不是 0。

加法和 AND 是不同的運算。反數保證「加」為 0，不保證「AND」為 0：

```
n + (-n) = 0    ← 加法：一定是 0（反數的定義）
n & (-n) ≠ 0    ← AND：取出 lowbit，不是 0（除非 n = 0）
```

| 運算 | 用的是 | 結果 |
|------|--------|------|
| `n & (~n)` | 反碼（一補數），純 bit 翻轉 | 永遠 0 |
| `n & (-n)` | 反數（二補數），`~n + 1` | 最低位的 1 |

---

## Q: 為什麼用二補數而不用一補數？

一補數有個問題：0 有兩種表示。`0000 0000`（+0）和 `1111 1111`（-0）。加法器要特殊處理。

二補數只有一個 0。而且加法器不用管正負號，直接加就對了：

```
   0000 0101   (5)
+  1111 1011   (-5, 二補數)
= 10000 0000   溢位捨掉第 9 bit → 0
```

---

## Q: Go 的 `^` — 一個符號兩種用途？

Go 沒有 `~`。用 `^` 同時當 XOR 和 NOT：

```go
a ^ b   // 二元：XOR
^a      // 一元：NOT（等於其他語言的 ~a）
```

Go 還有專屬的 `&^`（AND NOT / bit clear），一步完成「清除指定的 bit」：

```go
x &^ mask   // 等於 x & (^mask)，等於 C 的 x & ~mask
```

```
  0101 0100
&^0000 1100    ← 清除第 2、3 位
----------
  0101 0000
```

---

## 各語言 NOT 對照表

| 語言 | NOT | AND NOT |
|------|-----|---------|
| C / Java | `~x` | `x & ~mask` |
| JavaScript / TypeScript | `~x` | `x & ~mask` |
| Python | `~x` | `x & ~mask` |
| Go | `^x` | `x &^ mask` |

---

# 附錄

---

## 速查表

| 操作 | 用途 | 常見題目 |
|------|------|----------|
| `n >> 1` | 除以 2 | #704, #29, #50 |
| `n << k` | 乘以 $2^k$ | #78, #190, #371 |
| `n & 1` | 奇偶 | #191, #338 |
| `n \| (1 << k)` | 設第 k 位 | #190 |
| `a ^ b` | XOR 抵消 | #136, #268, #260 |
| `~x` | 翻轉所有 bit | #476, #1009 |
| `>>> 0` | 轉 unsigned | #190 |
| `n & (n-1)` | 消最低位的 1 | #191, #231, #461 |
| `n & (-n)` | 取最低位的 1 | #307, #315 |
| `(1 << k) - 1` | 低 k bits mask | #476, #1009 |

---

## 20 道小思考題

不用寫 code。用腦跑一遍就好。答案藏在每題下面。

---

### Q1. `0 ^ 0 ^ 0` 等於多少？

<details><summary>答案</summary>

`0`。0 XOR 任何次都是 0。三個 0 不會變出 1。

</details>

---

### Q2. `7 & 7` 等於多少？

<details><summary>答案</summary>

`7`。自己 AND 自己，每位都保留。`a & a = a`。

</details>

---

### Q3. 不用 `+` `-` `*` `/`，怎麼判斷兩個數是否相等？

<details><summary>答案</summary>

`a ^ b === 0`。XOR 相同位得 0。全部位都相同，結果就是 0。

</details>

---

### Q4. `n = 16`，`n & (n - 1)` 等於多少？這代表什麼？

<details><summary>答案</summary>

`0`。16 = `10000`，只有一個 1。消掉之後變 0。代表 16 是 2 的次方。

判斷 2 的次方：`n > 0 && (n & (n - 1)) === 0`。

</details>

---

### Q5. `5 ^ 5 ^ 3 ^ 3 ^ 9` 等於多少？你根本不需要算。

<details><summary>答案</summary>

`9`。5 和 5 抵消、3 和 3 抵消。只剩 9。

XOR 讓出現兩次的消掉，剩下的就是答案。

</details>

---

### Q6. `-1` 的所有 bit 長什麼樣？`-1 & n` 等於多少？

<details><summary>答案</summary>

`-1` 的 32 位全是 1：`1111...1111`。`-1 & n = n`。全 1 的 mask AND 任何數都不變。

</details>

---

### Q7. 不用第三個變數，怎麼交換 `a` 和 `b`？

<details><summary>答案</summary>

```
a = a ^ b
b = a ^ b    // b = (a^b)^b = a
a = a ^ b    // a = (a^b)^a = b
```

三次 XOR。但實務上別這樣寫，可讀性太差。面試知道就好。

</details>

---

### Q8. `n = 6`（`110`），怎麼用一步把第 0 位設成 1？

<details><summary>答案</summary>

`n | 1`。OR 會把對應位設成 1。`110 | 001 = 111 = 7`。

通用公式：設第 k 位為 1 → `n | (1 << k)`。

</details>

---

### Q9. `n = 7`（`111`），怎麼用一步把第 1 位設成 0？

<details><summary>答案</summary>

`n & ~(1 << 1)`。先建 mask `~(010) = 101`，再 AND。`111 & 101 = 101 = 5`。

通用公式：清除第 k 位 → `n & ~(1 << k)`。

</details>

---

### Q10. `n = 5`（`101`），怎麼用一步翻轉第 1 位？

<details><summary>答案</summary>

`n ^ (1 << 1)`。XOR 1 = 翻轉。`101 ^ 010 = 111 = 7`。

通用公式：翻轉第 k 位 → `n ^ (1 << k)`。

</details>

---

### Q11. `(1 << 32)` 在 JavaScript 裡等於多少？

<details><summary>答案</summary>

`0`，不是 4294967296。JavaScript 的 bitwise 運算是 32-bit。左移 32 格等於繞一圈回來。

如果要 $2^{32}$，用 `2 ** 32` 或 `Math.pow(2, 32)`。

</details>

---

### Q12. 一個陣列 `[1, 2, 3, ..., n]` 裡少了一個數。只用 XOR 怎麼找？

<details><summary>答案</summary>

把 `1 ^ 2 ^ ... ^ n` 和陣列裡所有數 XOR 在一起。成對的抵消，缺的那個留下來。

跟 LeetCode #268 Missing Number 一模一樣。

</details>

---

### Q13. `n >> 31` 在 32-bit signed integer 裡代表什麼？

<details><summary>答案</summary>

取符號位。正數 → `0`，負數 → `-1`（因為算術右移補符號位）。

可以用來不靠 `if` 判斷正負。

</details>

---

### Q14. `n = 12`（`1100`），`n & (-n)` 等於多少？如果 `n = 0` 呢？

<details><summary>答案</summary>

`n = 12`：`1100 & 0100 = 0100 = 4`。取最低位的 1。

`n = 0`：`0 & 0 = 0`。沒有 1 可以取。Fenwick Tree 從 index 1 開始，永遠不會碰到 0。

</details>

---

### Q15. 為什麼 `a ^ b ^ b === a`？

<details><summary>答案</summary>

XOR 滿足結合律和交換律。`a ^ (b ^ b) = a ^ 0 = a`。

這是 XOR 能「抵消」的數學根據。加密裡的 one-time pad 也是這個原理。

</details>

---

### Q16. `mask = 0b1010`。怎麼數裡面有幾個 1？不准用迴圈。

<details><summary>答案</summary>

用 `n & (n-1)` 消，但你說不准迴圈。那就用 popcount。

JavaScript：`mask.toString(2).split('0').join('').length`（醜但能用）。

Go：`bits.OnesCount(mask)`。

Python：`bin(mask).count('1')`。

答案是 2。

</details>

---

### Q17. 給你 `n = 0b11010`。不算的話，`n & (n - 1)` 做幾次會變 0？

<details><summary>答案</summary>

3 次。`11010` 有 3 個 1。每次消一個。消 3 次變 0。

`n & (n-1)` 做幾次 = 1 的個數。

</details>

---

### Q18. 兩個整數 `a` 和 `b`，怎麼不用 `+` 算出 `a + b`？

<details><summary>答案</summary>

```
sum = a ^ b        // 不進位的加法
carry = (a & b) << 1   // 進位
```

重複做，直到 carry = 0。

`3 + 5`：`011 ^ 101 = 110`，`011 & 101 = 001`，carry = `010`。
再來：`110 ^ 010 = 100`，`110 & 010 = 010`，carry = `100`。
再來：`100 ^ 100 = 000`，`100 & 100 = 100`，carry = `1000`。
`000 ^ 1000 = 1000 (8)`，carry = 0。結束。答案 8。

LeetCode #371。

</details>

---

### Q19. `1 << 0` 等於多少？`1 << -1` 呢？

<details><summary>答案</summary>

`1 << 0 = 1`。左移 0 格，不動。

`1 << -1`：JavaScript 裡，shift 量取 mod 32。`-1 & 31 = 31`。所以等於 `1 << 31 = -2147483648`。

別這樣寫。但面試可能問。

</details>

---

### Q20. 給你一個整數 `n`，一行 code 取出它的低 4 位。再一行取出它的第 4~7 位（從 0 開始數）。

<details><summary>答案</summary>

低 4 位：`n & 0xF`。等於 `n & ((1 << 4) - 1)`。

第 4~7 位：`(n >> 4) & 0xF`。先右移 4 格，把第 4~7 位推到最低位，再取低 4 位。

IP 地址解析、色碼拆分（ARGB）、protocol header parsing 全都用這個 pattern。

</details>
