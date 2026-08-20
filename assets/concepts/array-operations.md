---
title: "Array 操作複雜度"
category: Data Structures
slug: array-operations
subtitle: push、insert、delete 背後的真實成本
date: 2026-02-25T10:42:28
---

# Array 操作複雜度

你寫 `append()`、`push()`、`slice()` 的時候，知道它背後花多少時間嗎？

Go 跟 JavaScript 的陣列操作各是什麼複雜度，為什麼是那個數字，下面逐項對。

---

## Go：Array 和 Slice 是兩種東西


| | Array | Slice |
|---|---|---|
| 長度 | 固定，編譯時決定 | 動態，可以長 |
| 型別 | `[5]int` 和 `[3]int` 是**不同型別** | `[]int` 就是 `[]int` |
| 傳遞 | 值傳遞（整個複製） | 傳 header（指標 + len + cap） |
| 用途 | 幾乎不用 | 到處都用 |

```go
// Array：長度是型別的一部分
var a [5]int          // 固定 5 個，不能變
var b [3]int          // 跟 a 不同型別，不能互相賦值

// Slice：動態的 view
s := []int{1, 2, 3}  // 底層有個 array，s 是指向它的 window
s = append(s, 4)     // 長度不夠？自動擴容
```

Slice 的本體是三個欄位：

```go
type slice struct {
    array unsafe.Pointer  // 指向底層 array
    len   int             // 目前有幾個元素
    cap   int             // 底層 array 有多大
}
```

`len` 是你看到的長度，`cap` 是底層的容量。`len <= cap`。當 `append` 導致 `len > cap`，Go 會開一個更大的 array，把舊的複製過去。這就是擴容。

---

## Go Slice 操作

### append — 均攤 O(1)

```go
s := []int{1, 2, 3}
s = append(s, 4)       // cap 夠 → 直接放，O(1)
s = append(s, 5, 6, 7) // cap 不夠 → 擴容 + 複製，O(n)
```

大部分時候 cap 夠用，O(1)。偶爾擴容 O(n)，但擴容是倍增的（小 slice 翻倍，大 slice 約 1.25 倍），所以**均攤下來是 O(1)**。

為什麼均攤是 O(1)？看擴容的時機。cap 從 1 開始倍增：1 → 2 → 4 → 8 → 16...，擴容發生在第 2、3、5、9、17... 次 append，也就是 $2^n + 1$ 的時候擴到 $2^{n+1}$。連續 append N 次，總搬運量：

```
1 + 2 + 4 + 8 + ... + N/2 = N - 1
```

N 次操作，總共搬不到 N 次。平均每次操作搬 < 1 個元素。所以均攤 O(1)。

換個角度想：每個元素被搬的次數是有限的。它被放進去之後，只有在擴容時才會被搬一次到新 array。因為 cap 倍增，下一次擴容時 array 已經是兩倍大了，這個元素之後被搬的頻率越來越低。

跟 JavaScript 的 `push()` 一樣的道理。

### 切片 s[lo:hi] — O(1)

```go
s := []int{10, 20, 30, 40, 50}
sub := s[1:3]  // [20, 30]，O(1)
```

不複製。`sub` 跟 `s` 共用同一個底層 array，只是 pointer 偏移、len/cap 不同。所以是 O(1)。

**但這是陷阱：**

```go
s := []int{10, 20, 30, 40, 50}
sub := s[1:3]   // sub = [20, 30]，跟 s 共用底層
sub[0] = 999    // s 也變了！s = [10, 999, 30, 40, 50]
```

改 `sub` 會改到 `s`。如果不想這樣，用 `slices.Clone()`。

### slices.Clone — O(n)

```go
clone := slices.Clone(s)  // 開新 array，完整複製，O(n)
```

### copy — O(n)

```go
dst := make([]int, len(src))
copy(dst, src)             // 複製 min(len(dst), len(src)) 個元素
```

複製幾個就是 O(幾個)。

### slices.Delete — O(n)

```go
s = slices.Delete(s, 2, 4)  // 刪掉 index 2~3
```

中間刪除，後面的元素要往前搬。O(n)。

跟 JS 的 `splice()` 刪除一樣。

### slices.Insert — O(n)

```go
s = slices.Insert(s, 2, 99)  // 在 index 2 插入 99
```

中間插入，後面的元素要往後搬。O(n)。

### 存取 s[i] — O(1)

```go
v := s[3]     // 直接算位址：pointer + 3 * sizeof(int)
s[3] = 100    // 同理
```

連續記憶體，直接算偏移量。

### len / cap — O(1)

```go
n := len(s)   // 直接讀 slice header 的欄位
c := cap(s)   // 同上
```

不需要遍歷。

---

## Go Slice 速查

| 操作 | 時間 | 說明 |
|---|---|---|
| `s[i]` | O(1) | 直接算位址 |
| `append(s, v)` | 均攤 O(1) | 擴容時 O(n)，但倍增所以均攤 O(1) |
| `s[lo:hi]` | O(1) | 不複製，共用底層 array |
| `slices.Clone(s)` | O(n) | 完整複製 |
| `copy(dst, src)` | O(n) | 複製 min(len) 個 |
| `slices.Insert` | O(n) | 後面的元素要搬 |
| `slices.Delete` | O(n) | 後面的元素要搬 |
| `slices.Contains` | O(n) | 線性掃描 |
| `slices.Sort` | O(n log n) | pdqsort（sort 套件從 Go 1.19 起） |
| `len(s)` / `cap(s)` | O(1) | 讀 header |

---

## JavaScript Array 操作

JS 的 Array 不是真正的連續記憶體陣列。它是特殊的物件，key 是數字字串。但 V8 等引擎會在底層優化成真正的陣列（只要你不做奇怪的事，比如 `arr[10000] = 1` 開洞）。

### 尾部操作 — O(1)

```js
const arr = [1, 2, 3];
arr.push(4);    // 尾部加，O(1) 均攤（跟 Go append 同理）
arr.pop();      // 尾部刪，O(1)
```

不需要搬移任何元素。

### 頭部操作 — O(n)

```js
arr.unshift(0);  // 頭部加，O(n)：所有元素往後搬一格
arr.shift();     // 頭部刪，O(n)：所有元素往前搬一格
```

在迴圈裡用 `shift()` 當 queue 是常見寫法，但每次 shift 都要搬整個陣列，n 大了慢到跑不完。要 queue 的話，用一個 head index 記「頭在哪」、只前進不真的刪，或改用 linked list。

### splice — O(n)

```js
arr.splice(2, 1);        // 刪 index 2 的 1 個元素，O(n)
arr.splice(2, 0, 99);    // 在 index 2 插入 99，O(n)
```

中間插刪，後面的元素要搬。跟 Go 的 `slices.Insert` / `slices.Delete` 一樣。

### slice — O(n)

```js
const sub = arr.slice(1, 3);  // 回傳新陣列 [arr[1], arr[2]]，O(n)
```

**JS 的 `slice()` 會複製，跟 Go 的切片不同。** Go 的 `s[1:3]` 是 O(1) 共用底層，JS 的 `slice()` 是 O(k)（k = 切出來的長度）。

這是兩個語言最容易搞混的地方。

### concat — O(n + m)

```js
const merged = a.concat(b);  // 兩個都複製到新陣列
```

### 遍歷類 — O(n)

```js
arr.map(x => x * 2);       // O(n)，回傳新陣列
arr.filter(x => x > 0);    // O(n)，回傳新陣列
arr.reduce((a, b) => a + b); // O(n)
arr.forEach(x => ...);     // O(n)
arr.find(x => x > 3);      // O(n) 最糟，找到就停
arr.every(x => x > 0);     // O(n) 最糟，碰到 false 就停
arr.some(x => x > 0);      // O(n) 最糟，碰到 true 就停
```

這些都是線性掃描。`find`、`every`、`some` 有短路，但最糟還是 O(n)。

### 搜尋類 — O(n)

```js
arr.indexOf(3);      // O(n)，從頭找
arr.lastIndexOf(3);  // O(n)，從尾找
arr.includes(3);     // O(n)，線性掃描
```

都是線性掃描。要 O(1) 查找，用 `Set` 或 `Map`。

### 排序 — O(n log n)

```js
arr.sort((a, b) => a - b);         // in-place，O(n log n)，TimSort
const sorted = arr.toSorted((a, b) => a - b);  // ES2023，回傳新陣列
```

底層是 TimSort。詳見 [Sorting](/concept/sorting)。

### 其他 O(n)

```js
arr.reverse();       // O(n)，in-place 反轉
arr.join(', ');      // O(n)，產生字串
arr.fill(0);         // O(n)，全部填入同一個值
arr.flat();          // O(n)，攤平巢狀陣列
```

---

## JavaScript Array 速查

| 操作 | 時間 | 說明 |
|---|---|---|
| `arr[i]` | O(1) | 直接存取 |
| `push()` | 均攤 O(1) | 尾部加 |
| `pop()` | O(1) | 尾部刪 |
| `unshift()` | **O(n)** | 頭部加，全部搬移 |
| `shift()` | **O(n)** | 頭部刪，全部搬移 |
| `splice()` | O(n) | 中間插刪 |
| `slice()` | O(k) | 複製切片，k = 切出長度 |
| `concat()` | O(n+m) | 合併兩個陣列 |
| `indexOf()` / `includes()` | O(n) | 線性搜尋 |
| `map()` / `filter()` / `reduce()` | O(n) | 遍歷 |
| `sort()` | O(n log n) | TimSort |
| `arr.length` | O(1) | 讀屬性 |

---

## Go vs JavaScript

| | Go Slice | JS Array |
|---|---|---|
| 切片是否複製 | `s[1:3]` **不複製**，共用底層 O(1) | `slice(1,3)` **會複製** O(k) |
| 頭部操作 | 沒有內建，自己搬 O(n) | `shift/unshift` O(n) |
| 尾部新增 | `append` 均攤 O(1) | `push` 均攤 O(1) |
| 排序 | pdqsort | TimSort |
| 記憶體模型 | 真正的連續記憶體 | 引擎優化後才是（大部分時候是） |
| 共用底層的坑 | 切片改值會影響原 slice | 不會，`slice()` 是複製 |

Go 的切片共用底層，所以是 O(1)，但改 sub 會改到原本的 slice。JS 的 slice 複製一份，所以是 O(n)，兩邊不會互相影響。

---

## 結論

尾部操作 O(1)，頭部/中間操作 O(n)。兩個語言都一樣。

Go 要記住 slice 不是 array，切片共用底層。JS 要記住 `shift/unshift` 是 O(n)，別在迴圈裡當 queue 用。

排序的細節看 [Sorting](/concept/sorting)。
