---
title: "Queue"
category: Data Structures
slug: queue
subtitle: 先進先出，像排隊
date: 2026-02-28T11:51:31
---

# Queue

你在便利商店排隊。前面有三個人。

新來的人排最後面。結帳完的人從前面離開。

先來的先走。**First In, First Out。FIFO。**

Stack 是疊盤子。Queue 是排隊。

---

## 只有兩個操作

| 操作 | 做什麼 | 時間 |
|------|--------|------|
| enqueue（offer） | 放到最後面 | $O(1)$ |
| dequeue（poll） | 拿走最前面的 | $O(1)$ |
| peek/front | 看最前面是什麼（不拿走） | $O(1)$ |

進從後面，出從前面。兩端操作。

Stack 只碰一端。Queue 碰兩端。這是它們唯一的差別。

---

## 各語言怎麼用

### JavaScript

```js
const queue = []
queue.push(1)        // enqueue: [1]
queue.push(2)        //          [1, 2]
queue.push(3)        //          [1, 2, 3]
queue.shift()        // dequeue: 1, queue = [2, 3]
queue[0]             // peek: 2
```

`shift()` 是 $O(n)$：它要把所有元素往前搬一格。如果在乎效能，用 linked list 或自己實作 circular buffer。

LeetCode 不在乎。直接用 array + shift。

### Python

```python
from collections import deque

queue = deque()
queue.append(1)      # enqueue: [1]
queue.append(2)      #          [1, 2]
queue.popleft()      # dequeue: 1, queue = [2]
queue[0]             # peek: 2
```

Python 用 `deque`。`popleft()` 是 $O(1)$。不要用 `list.pop(0)`，那是 $O(n)$。

### Go

```go
queue := []int{}
queue = append(queue, 1)           // enqueue
queue = append(queue, 2)
front := queue[0]                  // peek
queue = queue[1:]                  // dequeue
```

Go 用 slice。`queue[1:]` 嚴格說不是 $O(1)$（底層陣列不回收），但 LeetCode 夠用。

### Java

```java
Queue<Integer> queue = new LinkedList<>();
queue.offer(1);      // enqueue
queue.offer(2);
queue.peek();        // 1
queue.poll();        // 1
```

---

## 你每天都在用 Queue

### 列印排隊

你送了三份文件去印。印表機按順序印。先送的先印。FIFO。

### 訊息佇列

你發了一條訊息。伺服器先處理更早發的。你的排在後面。Kafka、RabbitMQ、SQS 全都是 queue。

### Event Loop

JavaScript 是單執行緒。所有非同步 callback 排進 event queue。主執行緒空了，從 queue 前面拿一個出來跑。先進的先跑。FIFO。

```js
setTimeout(() => console.log('A'), 0)
setTimeout(() => console.log('B'), 0)
// A 先進 queue，所以先印 A 再印 B
```

---

## LeetCode 上的用法：BFS

LeetCode 上的 queue 幾乎都出現在 BFS。

DFS 用 stack（或遞迴）。BFS 用 queue。

為什麼？BFS 要「先處理最早發現的」。最早發現的在 queue 前面。FIFO。

### 走一遍：二元樹層序遍歷

[#102 Binary Tree Level Order Traversal](/problem/binary-tree-level-order-traversal)

```
        3
       / \
      9   20
         / \
        15  7
```

題目要的輸出是分層的 `[[3], [9, 20], [15, 7]]`，不是攤平的 `[3, 9, 20, 15, 7]`。所以 code 裡多一個 `level`：一層的值先收在 `level` 裡，這一層走完才整包放進 `result`。要是不用分層，`level` 可以整個拿掉，內圈直接 `result = append(result, node.Val)`。

```go
func levelOrder(root *TreeNode) [][]int {
    if root == nil { return nil }
    result := [][]int{}
    queue := []*TreeNode{root}

    for len(queue) > 0 {
        size := len(queue)
        level := []int{}
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            level = append(level, node.Val)
            if node.Left != nil { queue = append(queue, node.Left) }
            if node.Right != nil { queue = append(queue, node.Right) }
        }
        result = append(result, level)
    }
    return result
}
```

外圈跑一次等於一層，內圈跑 `size` 次把這一層的節點都拿出來：

```
初始    queue = [3]，result = []

外圈第 1 次
  size = 1              這一層有 1 個
  level = []
  i=0   拿出 3，queue = []
        level = [3]
        3 有 9 跟 20 → queue = [9, 20]
  result = [[3]]

外圈第 2 次
  size = 2              queue 裡是 9 跟 20
  level = []
  i=0   拿出 9，queue = [20]
        level = [9]
        9 沒有 child
  i=1   拿出 20，queue = []
        level = [9, 20]
        20 有 15 跟 7 → queue = [15, 7]
  result = [[3], [9, 20]]

外圈第 3 次
  size = 2
  level = []
  i=0   拿出 15 → level = [15]
  i=1   拿出 7  → level = [15, 7]
  queue = []
  result = [[3], [9, 20], [15, 7]]

queue 空了，len(queue) > 0 不成立，回傳
```

### `size` 為什麼要先記下來

內圈會往 `queue` 塞下一層的節點，所以 `len(queue)` 一直在變。

外圈第 2 次進來時 `queue = [9, 20]`，處理到 20 的時候又塞進 15 跟 7，`len(queue)` 變回 2。內圈要是寫成 `for len(queue) > 0`，它會接著處理 15 跟 7，`level` 就變成 `[9, 20, 15, 7]`，第三層被算進第二層。

先用 `size := len(queue)` 記下這一刻的數量，內圈就只拿走原本那幾個，後面塞進去的留給下一輪。

### BFS 找最短路

[#111 Minimum Depth of Binary Tree](/problem/minimum-depth-of-binary-tree)

DFS 要走遍整棵樹才知道最淺的 leaf 在哪。

BFS 一層一層走。**第一次碰到 leaf，那就是最淺的。** 不用繼續走。

```go
func minDepth(root *TreeNode) int {
    if root == nil { return 0 }
    queue := []*TreeNode{root}
    depth := 1

    for len(queue) > 0 {
        size := len(queue)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            if node.Left == nil && node.Right == nil {
                return depth    // 第一個 leaf，就是答案
            }
            if node.Left != nil { queue = append(queue, node.Left) }
            if node.Right != nil { queue = append(queue, node.Right) }
        }
        depth++
    }
    return depth
}
```

**BFS 天生找最短路。** 因為它一層一層展開，第一次碰到目標時走的步數一定是最少的。

這是 BFS 跟 DFS 最大的差別。DFS 找得到路，但不保證最短。BFS 保證。

---

## Deque：兩端都能進出

普通 queue：後面進，前面出。

Deque（double-ended queue）：前面也能進，後面也能出。

```
front ←→ [1, 2, 3, 4] ←→ back
```

四個操作：pushFront、pushBack、popFront、popBack。全 $O(1)$。

Python 的 `collections.deque`、Java 的 `ArrayDeque` 都是 deque。

### Sliding Window Maximum

[#239 Sliding Window Maximum](/problem/sliding-window-maximum)

給 `nums = [1,3,-1,-3,5,3,6,7]`，window size = 3。一個寬度 3 的 window 從最左邊開始，每次往右移一格，回報它蓋住的三個數字裡最大的那個。

```
[1, 3, -1] -3, 5, 3, 6, 7    max = 3
1, [3, -1, -3] 5, 3, 6, 7    max = 3
1, 3, [-1, -3, 5] 3, 6, 7    max = 5
1, 3, -1, [-3, 5, 3] 6, 7    max = 5
1, 3, -1, -3, [5, 3, 6] 7    max = 6
1, 3, -1, -3, 5, [3, 6, 7]   max = 7

答案 = [3, 3, 5, 5, 6, 7]
```

8 個元素、window 寬 3，滑出 `8 - 3 + 1 = 6` 個 window，所以答案有 6 個數字。

暴力：每個 window 掃一遍找 max。$O(n \cdot k)$。

deque 裡只留最大值不行，因為最大值總有一天會滑出 window，那時候得有人接手。看 `[5, 3, 4, 2]`，k = 3：第一個 window 是 `[5, 3, 4]`，max 是 5；等 5 滑出去、window 變成 `[3, 4, 2]`，答案是 4。所以 4 一路都得留著。

那 3 呢？3 可以丟掉。3 在 4 的左邊，比 4 早滑出去，而且比 4 小。只要 3 還在 window 裡，4 也一定還在，答案輪不到 3。

規則就是這樣來的：**新來的元素，把 deque 尾巴所有比它小的丟掉**，因為那些人比它先出場、又比它小，永遠當不了答案。丟完之後 deque 從前到後一定是遞減的，front 就是當前 window 的最大值。

**deque 存的是 index，不是值。** 因為要判斷 front 有沒有滑出 window，得拿它的 index 跟 `i - k + 1` 比。存值就沒辦法判斷。下面走查裡的 `1(3)` 是「index 1，值 3」的意思。

```
nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3

i=0: deque=[], push 0          deque: [0(1)]
i=1: 3>1, pop 0, push 1       deque: [1(3)]
i=2: -1<3, push 2             deque: [1(3), 2(-1)]
     window [0,2] → max = nums[deque.front] = 3

i=3: -3<-1, push 3            deque: [1(3), 2(-1), 3(-3)]
     window [1,3] → max = 3

i=4: 5>-3, pop 3. 5>-1, pop 2. 5>3, pop 1. push 4
     deque: [4(5)]
     window [2,4] → max = 5

i=5: 3<5, push 5              deque: [4(5), 5(3)]
     window [3,5] → max = 5

i=6: 6>3, pop 5. 6>5, pop 4. push 6
     deque: [6(6)]
     window [4,6] → max = 6

i=7: 7>6, pop 6. push 7
     deque: [7(7)]
     window [5,7] → max = 7

ans = [3, 3, 5, 5, 6, 7]
```

這組資料有一件事看不到：從前面 pop。每次該滑出 window 的那個元素，早就因為比後來的小而從後面被 pop 掉了。換一組就看得到：

```
nums = [5, 3, 4, 2, 1], k = 3

i=0: push 0                    deque: [0(5)]
i=1: 3<5, push 1               deque: [0(5), 1(3)]
i=2: 4>3, pop 1. 4<5, push 2   deque: [0(5), 2(4)]
     window [0,2] → max = 5

i=3: 2<4, push 3               deque: [0(5), 2(4), 3(2)]
     window 變成 [1,3]，front 的 index 是 0，已經小於 1
     popFront                  deque: [2(4), 3(2)]
     → max = 4

i=4: 1<2, push 4               deque: [2(4), 3(2), 4(1)]
     window [2,4]，front 的 index 是 2，還在範圍內
     → max = 4

ans = [5, 4, 4]
```

5 滑出去的時候 4 接手，這就是前面說的「比較小的也要留著」。

從後面 pop 破壞遞減順序的（像 monotonic stack）。從前面 pop 滑出 window 的。兩端都要操作，所以要 deque。

---

## Priority Queue 不是 Queue

名字裡有 queue，但它不是 FIFO。

Priority queue 每次 pop 的是**優先權最高的**，不是最早進的。

底層通常是 heap。詳見 [Heap](/concept/heap)。

別搞混：
- **Queue**：FIFO，排隊
- **Priority Queue**：按優先權，插隊
- **Deque**：兩端都能進出

---

## 高頻題清單

### BFS（queue 的主場）

| 題目 | 核心 |
|------|------|
| [#102 Level Order Traversal](/problem/binary-tree-level-order-traversal) | 一層一層走 |
| [#111 Minimum Depth](/problem/minimum-depth-of-binary-tree) | BFS 碰到 leaf 就是最淺 |
| [#200 Number of Islands](/problem/number-of-islands) | BFS 或 DFS 都行 |
| [#994 Rotting Oranges](/problem/rotting-oranges) | 多起點 BFS |
| [#127 Word Ladder](/problem/word-ladder) | BFS 找最短轉換路徑 |
| [#752 Open the Lock](/problem/open-the-lock) | BFS 搜尋狀態空間 |

### Deque

| 題目 | 核心 |
|------|------|
| [#239 Sliding Window Maximum](/problem/sliding-window-maximum) | monotonic deque |
| [#346 Moving Average from Data Stream](/problem/moving-average-from-data-stream) | 固定大小的 queue |

### 設計題

| 題目 | 核心 |
|------|------|
| [#232 Implement Queue using Stacks](/problem/implement-queue-using-stacks) | 兩個 stack 模擬 queue |
| [#225 Implement Stack using Queues](/problem/implement-stack-using-queues) | 反過來 |
| [#622 Design Circular Queue](/problem/design-circular-queue) | head/tail pointer |

---

## 總結

Queue 做一件事：**按順序處理，先來先走。**

BFS 需要這個性質。一層一層展開，先發現的先處理。所以 BFS 用 queue。

Stack 是 LIFO，處理「最近的」。Queue 是 FIFO，處理「最早的」。

碰到題目先問：我要處理最近的還是最早的？最近的用 stack，最早的用 queue。

想學 queue 的升級版（按優先權處理）？看 [Heap](/concept/heap)。想學 BFS 的完整套路？看 [DFS](/concept/dfs)（裡面有 BFS vs DFS 的比較）。
