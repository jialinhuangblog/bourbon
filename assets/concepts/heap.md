---
title: "Heap / Priority Queue"
category: Data Structures
slug: heap
subtitle: 隨時拿最大或最小，O(log n)
date: 2026-02-28T11:51:31
updated: 2026-08-04
---

# Heap / Priority Queue

**三條規則記住就好：**

1. parent ≤ child（min heap；max heap 反過來）
2. pop：把 last 搬到 top，往下沉
3. push：放到 last，往上冒

---

Queue 是排隊。先來先走。公平。

但醫院急診室不公平。車禍的比感冒的先看。不管誰先來。

Priority queue 就是急診室。每次出來的不是「最早進的」，是「優先權最高的」。

底層怎麼做到？用 heap。

---

## Heap 是什麼？

一棵「幾乎完全」的二元樹，滿足一條規則：

- **Max heap**：每個 parent >= 它的 children。根是最大的。
- **Min heap**：每個 parent <= 它的 children。根是最小的。

```
Min Heap:          Max Heap:
     1                  9
    / \                / \
   3   5              7   6
  / \                / \
 8   4              3   2
```

不是 BST。左 child 不一定比右 child 小。只保證 parent 跟 children 的關係。

上面那個 min heap 攤成陣列是 `[1, 3, 5, 8, 4]`，8 排在 4 前面。8 跟 4 是兄弟，兩個都是 3 的 child，彼此之間沒有 parent 跟 child 的關係，規則管不到。heap 建好了不等於排好了，它只保證 root 是最小的那個。

---

## 用陣列表示

跟 [Heap Sort](/concept/heap-sort) 一樣。不需要指標，不需要 struct。一個陣列就夠。

```
index:  0  1  2  3  4
value: [1, 3, 5, 8, 4]    ← min heap

parent(i) = (i-1) / 2
left(i)   = 2*i + 1
right(i)  = 2*i + 2
```

---

## Push 跟 Pop

### Push：加一個元素

放到陣列最後面。然後往上浮（sift up）。跟 parent 比，如果比 parent 小（min heap），交換。一直浮到不能浮為止。

```
push 2 到 [1, 3, 5, 8, 4]:

[1, 3, 5, 8, 4, 2]
                ↑ 新元素在 index 5

parent = index 2 (值 5)
2 < 5 → 交換
[1, 3, 2, 8, 4, 5]

parent = index 0 (值 1)
2 > 1 → 停

結果: [1, 3, 2, 8, 4, 5]
```

$O(\log n)$。最多浮到根，高度 $\log n$。

### Pop：取出最小（或最大）

根是最小的。拿走它。把最後一個元素搬到根。然後往下沉（sift down）。跟 children 比，如果比 children 大，跟最小的 child 交換。一直沉到不能沉為止。

```
pop from [1, 3, 2, 8, 4, 5]:

拿走 1。把 5 搬到根。
[5, 3, 2, 8, 4]

5 的 children: 3, 2。最小的是 2。5 > 2 → 交換
[2, 3, 5, 8, 4]

5 現在在 index 2，children 要在 index 5、6，超出陣列範圍 → 停

結果: [2, 3, 5, 8, 4]，pop 出來的是 1
```

$O(\log n)$。最多沉到底。

### 時間複雜度

| 操作 | 時間 |
|------|------|
| push | $O(\log n)$ |
| pop（取最小/最大） | $O(\log n)$ |
| peek（看最小/最大） | $O(1)$ |
| 建 heap（從陣列） | $O(n)$ |

peek 是 $O(1)$ 因為最小的永遠在 index 0。不用找。

---

## 各語言怎麼用

誰有內建、預設 min 還是 max、另一種怎麼拿：

| 語言 | 內建 heap | 預設 | 另一種怎麼拿 |
|---|---|---|---|
| Python | `heapq` | 只有 min | push 前加負號，pop 後取負 |
| Go | `container/heap` | 兩種都沒有，自己實作 interface | `Less` 的方向決定 min / max |
| JavaScript | 無 | 沒有 | 手刻 / npm 套件 / LeetCode 內建 `MinPriorityQueue` |

Python 只有 min，加負號變 max；Go 兩種都沒給，方向靠自己寫的 `Less` 決定；JS 連基礎建設都沒有，所以下面要手刻。

<details>
<summary>其他語言（Java / C++ / Rust）</summary>

| 語言 | 內建 heap | 預設 | 另一種怎麼拿 |
|---|---|---|---|
| Java | `PriorityQueue` | min | `Collections.reverseOrder()` 或自訂 comparator |
| C++ | `priority_queue` | max | 傳 `greater<>` comparator |
| Rust | `BinaryHeap` | max | 包一層 `Reverse` |

**Java** — 預設 min，`reverseOrder()` 變 max：

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();                            // min（預設）
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());  // max
minHeap.offer(3); minHeap.offer(1);
minHeap.poll();   // 1，最小先出
minHeap.peek();   // peek
```

**C++** — 預設 max，加 `greater<>` 變 min：

```cpp
#include <queue>
priority_queue<int> maxHeap;                              // max（預設）
priority_queue<int, vector<int>, greater<int>> minHeap;   // min
maxHeap.push(3); maxHeap.push(1);
maxHeap.top();    // 3，最大先出
maxHeap.pop();
```

**Rust** — 預設 max，包 `Reverse` 變 min：

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

let mut max_heap = BinaryHeap::new();
max_heap.push(3);            // max（預設）
max_heap.pop();              // Some(3)，最大先出

let mut min_heap = BinaryHeap::new();
min_heap.push(Reverse(3));   // 包 Reverse 變 min
min_heap.pop();              // Some(Reverse(3))
```

C++、Rust 預設是 max，剛好跟 Python 相反。從 Python 換過去最容易在這裡搞錯方向。

</details>

### Python（最方便）

```python
import heapq

heap = []
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappush(heap, 5)

heapq.heappop(heap)     # 1（最小的）
heap[0]                  # 3（peek）
```

Python 的 heapq 是 **min heap**。要 max heap？push 負數。

```python
heapq.heappush(heap, -3)   # 存 -3
-heapq.heappop(heap)        # pop 出來再取負 = 3
```

醜但管用。

### Go

```go
// Go 的 heap 要自己實作 interface
type MinHeap []int
func (h MinHeap) Len() int {
    return len(h)
}
func (h MinHeap) Less(i, j int) bool {
    return h[i] < h[j]
}
func (h MinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}
func (h *MinHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}
func (h *MinHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}

// 使用
h := &MinHeap{3, 1, 5}
heap.Init(h)
heap.Push(h, 2)
heap.Pop(h)          // 1
```

Go 的 heap 寫起來很囉嗦。但面試要會。

#### 那五個方法是誰規定的

`container/heap` 定義了一個 interface，列出它要的方法：

```go
// container/heap/heap.go
type Interface interface {
    sort.Interface       // 把 sort 的三個方法整組併進來
    Push(x any)
    Pop() any
}

// sort/sort.go
type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

三個加兩個等於五個，跟 `MinHeap` 實作的數量對上。把另一個 interface 直接寫進來的做法叫 embedding，所以光看 `heap.Interface` 那三行會以為只需要 `Push` 跟 `Pop`。

#### MinHeap 跟 container/heap 什麼時候才有關係

沒有註冊，也不用寫 implements。`MinHeap` 的定義裡連 `import "container/heap"` 都沒有，單獨放進一個檔案照樣編得過、跑得動。

發生關係的地方只有一個，`heap.Init(h)` 這一行：

```go
func Init(h Interface)   // 參數型別是那個 interface
```

編譯器在這一行問一次：`*MinHeap` 有沒有那五個方法？少一個就編不過。

```
cannot use h (variable of type *MinHeap) as heap.Interface value in
argument to heap.Init: *MinHeap does not implement heap.Interface
(missing method Push)
```

上面這個訊息是刻意拿掉 `Push` 之後編出來的。`Init` 內部根本不會呼叫 `Push`，它只做 `down`，但參數型別是整個 interface，所以五個一個都不能少。

檢查過了，`h` 就被包成一個 interface value 傳進去，裡面裝的是資料的指標加上一張方法表。package 拿著這張表回頭呼叫。關係只存在於這次呼叫，下一行 `heap.Push(h, 2)` 重新來一次。

#### 上浮跟下沉在哪裡

上面四個方法從頭到尾沒有比較大小，也沒有把元素往上搬。`MinHeap.Push` 只是 append 到尾巴，`MinHeap.Pop` 只是切掉尾巴。

因為 `heap.Push` 跟 `MinHeap.Push` 是兩個不同的函式。前者是 `container/heap` 這個 package 提供的，它才是演算法：

```go
// container/heap 的實作
func Push(h Interface, x any) {
    // 呼叫自己實作的：append 到尾巴
    h.Push(x)
    // package 自己的：往上浮
    up(h, h.Len()-1)
}

func Pop(h Interface) any {
    n := h.Len() - 1
    // 根跟最後一個交換
    h.Swap(0, n)
    // 往下沉
    down(h, 0, n)
    // 呼叫自己實作的：切掉尾巴回傳
    return h.Pop()
}

// heapify，從最後一個非葉節點往回 down
func Init(h Interface) {
    n := h.Len()
    for i := n/2 - 1; i >= 0; i-- {
        down(h, i, n)
    }
}
```

`up` 跟 `down` 內部只呼叫 `Less` 跟 `Swap`。所以分工是這樣：比較規則跟交換動作由自己實作，什麼時候比、跟誰換由 package 決定。`Less` 的方向決定這是 min heap 還是 max heap，原因就在這裡。

<details>
<summary>up 跟 down 的原始碼</summary>

前面「Push：加一個元素」跟「Pop：取出最小」講的往上浮跟往下沉，Go 寫出來就是這兩個。它們是小寫的，package 外面看不到，`go doc` 也不會列。

```go
func up(h Interface, j int) {
    for {
        i := (j - 1) / 2 // parent
        if i == j || !h.Less(j, i) {
            break
        }
        h.Swap(i, j)
        j = i
    }
}

func down(h Interface, i0, n int) bool {
    i := i0
    for {
        j1 := 2*i + 1
        if j1 >= n || j1 < 0 { // j1 < 0 after int overflow
            break
        }
        j := j1 // left child
        if j2 := j1 + 1; j2 < n && h.Less(j2, j1) {
            j = j2 // right child
        }
        if !h.Less(j, i) {
            break
        }
        h.Swap(i, j)
        i = j
    }
    return i > i0
}
```

從頭到尾沒有出現過 `h[i]`，只有 `h.Less` 跟 `h.Swap`。因為拿到的是 interface，看不見裡面是 `[]int` 還是別的型別，只能透過方法問。

`down` 裡那句 `j1 < 0 // after int overflow` 跟 [Binary Search](/problem/binary-search) 的 `(l + r) / 2` 是同一類問題：`2*i + 1` 在 `i` 接近 int 上限時會變成負數。

</details>

這也解釋了 `MinHeap.Pop` 為什麼回傳 `old[n-1]` 而不是 `old[0]`。輪到它執行的時候，根已經被 `h.Swap(0, n)` 交換到尾端，最後一格裝的就是最小的那個。單獨看那五行會覺得「拿最後一個？那不是最小的啊」，因為前面那兩行不在同一個檔案裡。

### JavaScript

JavaScript 沒有內建 heap。分兩種寫法。

#### 寫法一：用 LeetCode 內建（最快）

LeetCode 預掛 `MinPriorityQueue` / `MaxPriorityQueue`（來自 `@datastructures-js/priority-queue`），不用 import：

```js
const pq = new MinPriorityQueue({ compare: (a, b) => a - b });

pq.enqueue(5);
pq.enqueue(2);
pq.enqueue(8);

pq.front();       // 2 (peek，舊版叫 .top())
pq.dequeue();     // 2 (拿走最小)
pq.size();        // 2
pq.isEmpty();     // false
```

`compare` 規則跟 `Array.sort` 一樣：負數代表 a 優先。

```js
new MinPriorityQueue({ compare: (a, b) => a - b })       // min heap
new MinPriorityQueue({ compare: (a, b) => b - a })       // max heap
new MinPriorityQueue({ compare: (a, b) => a.val - b.val })  // 物件 heap
```

> LeetCode 改過幾次 API。舊版用 `priority` 函式，新版用 `compare`。`enqueue/dequeue` 行為怪怪的就看當下官方提示哪個版本。

#### 寫法二：自己實作（面試禁用內建時）

用一個陣列存 heap，父子關係用 index 算出來，push 之後往上浮，pop 之後往下沉。

```js
class MinHeap {
  constructor(compareFn = (a, b) => a - b) {
    this.heap = [];
    this.compare = compareFn;
  }

  size() { return this.heap.length; }
  isEmpty() { return this.heap.length === 0; }
  peek() { return this.heap[0]; }

  push(val) {
    this.heap.push(val);                   // 1. 放到最後
    this._bubbleUp(this.heap.length - 1);  // 2. 往上冒
  }

  pop() {
    if (this.heap.length === 0) return undefined;
    // 舊的根先存起來，它是回傳值，不會再回到陣列裡
    const top = this.heap[0];
    const last = this.heap.pop();
    if (this.heap.length > 0) {
      this.heap[0] = last;                 // 把最後一個搬到根
      this._sinkDown(0);                   // 往下沉
    }
    return top;
  }

  _bubbleUp(i) {
    while (i > 0) {
      const parent = (i - 1) >> 1;         // 父索引 = (i-1) / 2
      if (this.compare(this.heap[i], this.heap[parent]) < 0) {
        [this.heap[i], this.heap[parent]] = [this.heap[parent], this.heap[i]];
        i = parent;
      } else {
        break;                             // 比父大或等，停
      }
    }
  }

  _sinkDown(i) {
    const n = this.heap.length;
    while (true) {
      const left = 2 * i + 1;
      const right = 2 * i + 2;
      let smallest = i;

      if (left < n && this.compare(this.heap[left], this.heap[smallest]) < 0) {
        smallest = left;
      }
      if (right < n && this.compare(this.heap[right], this.heap[smallest]) < 0) {
        smallest = right;
      }

      if (smallest === i) break;           // 父已經是最小，停
      [this.heap[i], this.heap[smallest]] = [this.heap[smallest], this.heap[i]];
      i = smallest;
    }
  }
}
```

用法：

```js
const heap = new MinHeap();
heap.push(5);
heap.push(2);
heap.push(8);
heap.peek();   // 2
heap.pop();    // 2

// 物件 heap
const objHeap = new MinHeap((a, b) => a.val - b.val);
objHeap.push({ val: 3 });
objHeap.push({ val: 1 });
objHeap.pop(); // { val: 1 }
```

#### 兩個常見坑

**坑 1：compare 寫反**
```js
(a, b) => a - b    // min，正確
(a, b) => b - a    // max，正確
(a, b) => a < b    // 錯：回傳 true/false，sort 邏輯不認得
```

**坑 2：物件 heap 沒給 compare 就壞掉**
```js
const heap = new MinHeap();           // 預設 (a, b) => a - b
heap.push({ val: 3 });                // 錯：物件相減 = NaN
```
物件一定要：`new MinHeap((a, b) => a.val - b.val)`。

#### 何時用哪種

| 情境 | 選擇 |
|---|---|
| LeetCode | 內建 `MinPriorityQueue` |
| 面試（in-person） | 先問能不能用現成，不行再寫 |
| 真實生產環境 | npm 套件（如 `heap-js`） |
| 學習 / 理解結構 | 至少手寫一次 |

---

## LeetCode 的三種用法

### 用法一：Top K

[#215 Kth Largest Element](/problem/kth-largest-element-in-an-array)

找第 k 大的元素。

暴力：排序，取第 k 個。$O(n \log n)$。

Min heap of size k：維護一個大小為 k 的 min heap。掃過所有元素，如果比 heap 頂大就替換。掃完之後 heap 頂就是第 k 大。

```
nums = [3, 2, 1, 5, 6, 4], k = 2

掃 3: heap = [3]           (size < k, 直接加)
掃 2: heap = [2, 3]        (size = k)
掃 1: 1 < heap頂(2)，跳過
掃 5: 5 > heap頂(2)，替換  heap = [3, 5]
掃 6: 6 > heap頂(3)，替換  heap = [5, 6]
掃 4: 4 < heap頂(5)，跳過

heap 頂 = 5 = 第 2 大
```

$O(n \log k)$。k 小的時候比排序快很多。

### 用法二：Merge K Sorted

[#23 Merge K Sorted Lists](/problem/merge-k-sorted-lists)

k 個已排好的 linked list，合成一個。

暴力：每次掃 k 個 list 的頭，找最小的。$O(n \cdot k)$。

Min heap：把 k 個 list 的頭丟進 min heap。每次 pop 最小的出來，把它的 next push 進去。

```
lists: [1→4→5, 1→3→4, 2→6]

heap: [1, 1, 2]    (三個 list 的頭)

pop 1 → result: [1]     push 4    heap: [1, 2, 4]
pop 1 → result: [1,1]   push 3    heap: [2, 3, 4]
pop 2 → result: [1,1,2] push 6    heap: [3, 4, 6]
pop 3 → result: [1,1,2,3] push 4  heap: [4, 4, 6]
pop 4 → result: [1,1,2,3,4] push 5 heap: [4, 5, 6]
pop 4 → result: [1,1,2,3,4,4]     heap: [5, 6]
pop 5 → result: [1,1,2,3,4,4,5]   heap: [6]
pop 6 → result: [1,1,2,3,4,4,5,6] heap: []

done
```

$O(n \log k)$。n 個元素，每次 push/pop 是 $O(\log k)$。

Merge K Sorted Lists 就是這樣解的。[Merge Sort](/concept/merge-sort) 處理 merge 兩個，heap 把它推廣到 merge k 個。

### 用法三：Streaming / Data Stream

[#295 Find Median from Data Stream](/problem/find-median-from-data-stream)

資料一筆一筆進來。隨時要能回答中位數。

用兩個 heap：一個 max heap 存小的那一半，一個 min heap 存大的那一半。

```
max heap (小的那半) | min heap (大的那半)
        [2, 1]     |     [3, 5]

中位數 = (max heap 頂 + min heap 頂) / 2 = (2 + 3) / 2 = 2.5
```

每次加入新元素，先丟進某一邊，再 rebalance（確保兩邊大小差不超過 1）。

$O(\log n)$ per insert，$O(1)$ 查中位數。

---

## Heap vs 其他排序方式

| 場景 | 用什麼 | 為什麼 |
|------|--------|--------|
| 全部排好 | sort() | $O(n \log n)$ 一次搞定 |
| 只要最大/最小的 | heap | $O(n)$ 建 heap + $O(\log n)$ 取一個 |
| 要 top K | heap of size k | $O(n \log k)$ |
| 資料持續進來 | heap | 每次 $O(\log n)$ 插入 |
| 資料不變，多次查 | 排序 + index | $O(1)$ 查 |

Heap 的強項是**動態**。資料不斷變化（加入、刪除），heap 每次操作都是 $O(\log n)$。排序是靜態的，改了要重排。

---

## Heap vs Heap Sort vs Heap Memory

三個「heap」，完全不同的東西。

| 名字 | 是什麼 |
|------|--------|
| Heap（資料結構） | 一棵滿足 heap 性質的二元樹。本篇在講的。 |
| Heap Sort | 用 heap 做排序。見 [Heap Sort](/concept/heap-sort)。 |
| Heap Memory | 作業系統管理的記憶體區域。`new`/`malloc` 分配的記憶體在這裡。跟資料結構無關。 |

面試說 "heap" 通常指資料結構。寫 code 說 "allocated on the heap" 指記憶體。別搞混。

---

## 高頻題清單

| 題目 | 核心 |
|------|------|
| [#215 Kth Largest Element](/problem/kth-largest-element-in-an-array) | Min heap of size k |
| [#23 Merge K Sorted Lists](/problem/merge-k-sorted-lists) | Min heap 取 k 個頭的最小 |
| [#347 Top K Frequent Elements](/problem/top-k-frequent-elements) | Count + min heap |
| [#295 Find Median from Data Stream](/problem/find-median-from-data-stream) | 兩個 heap |
| [#703 Kth Largest Element in a Stream](/problem/kth-largest-element-in-a-stream) | Min heap of size k |
| [#973 K Closest Points to Origin](/problem/k-closest-points-to-origin) | Max heap of size k |
| [#621 Task Scheduler](/problem/task-scheduler) | Max heap + cooldown |
| [#355 Design Twitter](/problem/design-twitter) | Merge K sorted（每人的 tweets） |

---

## 總結

Heap 做一件事：**快速拿到最大或最小的。**

不用全排。不用全掃。$O(\log n)$ push，$O(\log n)$ pop，$O(1)$ peek。

看到「第 K 大」「第 K 小」「合併 K 個排好的」「持續進來的資料流」，想到 heap。

它不是 queue（FIFO）。它是 priority queue：每次出來的是最重要的那個，不是最早的。
