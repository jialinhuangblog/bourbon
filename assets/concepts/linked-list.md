---
title: "Linked List"
category: Data Structures
slug: linked-list
subtitle: 節點串節點，插刪 O(1)
date: 2026-02-28T11:51:31
updated: 2026-07-28
---

# Linked List

Array 是一排連號的座位。你要找第 5 個人，直接走到座位 5。$O(1)$。

Linked list 是一群人手牽手。你要找第 5 個人，只能從第 1 個開始，沿著手一路牽過去。$O(n)$。

聽起來比 array 爛。那為什麼要用它？

因為 array 要插入一個人到中間，後面所有人要讓位。$O(n)$。

Linked list 插一個人？解開旁邊兩個人的手，把新人牽進去。$O(1)$。

---

## 長什麼樣？

```
[1] -> [2] -> [3] -> [4] -> null
```

每個節點有兩個東西：一個值，一個指向下一個節點的指標。

```go
type ListNode struct {
    Val  int
    Next *ListNode
}
```

```js
class ListNode {
    constructor(val = 0, next = null) {
        this.val = val
        this.next = next
    }
}
```

最後一個節點的 `next` 是 `null`。走到 null 就知道到底了。

---

## Singly vs Doubly

### Singly Linked List（單向）

```
[1] -> [2] -> [3] -> null
```

只能往前走，沒辦法回頭。LeetCode 的 linked list 題預設就是這種，給的 `ListNode` 只有 `next`。

### Doubly Linked List（雙向）

```
null <- [1] <-> [2] <-> [3] -> null
```

每個節點多一個 `prev` 指標。可以往前也可以往後。

用在哪？LRU Cache。需要 $O(1)$ 刪除中間的節點。單向 list 刪除要先找到前一個節點（$O(n)$）。雙向 list 直接用 `prev` 就找到了。

但前提是：你手上已經拿著那個節點的 reference。如果你只有一個值，不管單向還是雙向，都要從頭找，$O(n)$。所以 LRU Cache 用 hash map 存 `key → node reference`，查 hash map 拿到節點（$O(1)$），再用 `prev` / `next` 把自己拆掉（$O(1)$）。兩個資料結構搭配，才是真正的 $O(1)$ 刪除。

---

## 基本操作

### 走訪

```go
for curr := head; curr != nil; curr = curr.Next {
    fmt.Println(curr.Val)
}
```

從頭走到尾。$O(n)$。

### 插入到頭部

```go
newNode := &ListNode{Val: 0, Next: head}
head = newNode
```

$O(1)$。不用搬任何東西。

### 刪除某個節點

```go
// 刪除 val=3 的節點
prev := head
for prev.Next != nil {
    if prev.Next.Val == 3 {
        prev.Next = prev.Next.Next    // 跳過它
        break
    }
    prev = prev.Next
}
```

找到要刪的節點的前一個，讓前一個直接指向後一個。被刪的節點就斷開了。

為什麼要把「還沒走到底」跟「還沒找到」拆成兩層？因為出迴圈後要做的事不一樣：找到了要改指標，走到底沒找到就什麼都不做。塞在同一個 `for` header 變成 `A && B`，出來還得再補一個 `if` 去分辨是哪個條件先不成立，語意反而糊。

為什麼是這兩個複雜度？`for` 迴圈從 head 一路往後找 `prev.Next.Val == 3`，最差走到尾巴才找到，所以搜尋是 $O(n)$。找到之後，`prev.Next = prev.Next.Next` 就一行，不管 list 多長都是常數時間，所以改指標是 $O(1)$。

---

## Dummy Head：省掉邊界判斷

刪除 head 怎麼辦？沒有「前一個」可以改。

加一個 dummy node 在最前面：

```
dummy -> [1] -> [2] -> [3] -> null
```

dummy 永遠在。不管刪誰，都有「前一個」。

```go
func removeElements(head *ListNode, val int) *ListNode {
    dummy := &ListNode{Next: head}
    prev := dummy
    for prev.Next != nil {
        if prev.Next.Val == val {
            prev.Next = prev.Next.Next
        } else {
            prev = prev.Next
        }
    }
    return dummy.Next
}
```

只要題目可能動到 head，加個 dummy 就省掉一整組邊界判斷。

---

## 快慢指標：Linked List 的招牌技巧

兩個指標。一個走一步（slow），一個走兩步（fast）。

### 找中點

[#876 Middle of the Linked List](/problem/middle-of-the-linked-list)

```
[1] -> [2] -> [3] -> [4] -> [5] -> null
 s
 f

[1] -> [2] -> [3] -> [4] -> [5] -> null
        s
               f

[1] -> [2] -> [3] -> [4] -> [5] -> null
               s
                             f

fast 到底了。slow 在中間。
```

fast 走兩步，slow 走一步。fast 到底時，slow 剛好在中間。

```go
func middleNode(head *ListNode) *ListNode {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
    }
    return slow
}
```

$O(n)$，只走一遍。不需要先數長度。

### 偵測環

[#141 Linked List Cycle](/problem/linked-list-cycle)

```
[1] -> [2] -> [3] -> [4]
               ^      v
              [6] <- [5]
```

如果有環，fast 遲早追上 slow。因為 fast 每步比 slow 多走一格，相對速度 = 1。在環裡繞圈，一定會碰到。

如果沒有環，fast 先到 null。

```go
func hasCycle(head *ListNode) bool {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast { return true }
    }
    return false
}
```

不需要 hash map 記走過的。$O(1)$ 空間。

### 為什麼是 2 倍速？

光判斷「有沒有環」（141）不挑速度，任何 k ≥ 2 都追得上。把 k 鎖死在 2 的是下一題 [142](/problem/linked-list-cycle-ii) 的目標：**用 O(1) 空間找出環的入口**。這個推導解的就是它，放進一個公園場景走完，八步。

#### Step 1: 場景

想像一個河濱公園：門口進去是一段直步道，走到底接上一圈環形跑道，而且是單行道，上了跑道只能順著繞，沒有岔路離開。有環的 list 就是這個形狀，步道是前段、跑道是環：

```
head = 1 -> 2 -> [3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10 -> 11 -> 12 -> 3 (回頭)]
                  ^
                  跑道入口

步道長 a = 2（head 到入口的步數）
跑道一圈 c = 10（環長）
```

你（slow）跟朋友（fast）同時從公園門口出發，朋友配速是你的 k 倍。跑演算法的時候**不知道** a 和 c，目標：找到跑道入口（節點 3）。

出場的字母先分清楚身分，它們不是同一種東西：

- **a**：步道長度。輸入決定，固定但未知。
- **b**：跑道入口到相遇點的距離。相遇發生後才確定。
- **c**：跑道一圈的長度。輸入決定。
- **k**：朋友的倍速。設計演算法時選的，整個推導就是在解它。
- **T**：相遇時你走的總步數；**n**：朋友多繞的圈數。兩個都是跑出來的結果。

推導的玩法：把 a、b、c、n 當「已知但不知道值」的常數，看 k 要滿足什麼條件才能對任何公園都成立。

#### Step 2: 為什麼只能派人走

沿路撒麵包屑、記住每個走過的節點（hash set），第一個重複的就是入口。可以，但 O(n) 空間，太貴。O(1) 空間等於不准做記號，只能派固定幾個人進去走：

- 一個人：上了跑道只能一直繞，沒有參考點，認不出哪裡是入口。
- 兩個人同速：從出發就疊在一起，走不出任何資訊。
- 兩個人不同速：有戲。

#### Step 3: 重逢必在跑道上

朋友比你快，早早走完步道上了跑道；單行道讓他回不了步道。步道上你們只在出發那一刻站同一點（t 步時你在第 t 格、他在第 kt 格）。所以重逢只可能發生在跑道上，而你人在跑道上，代表你已經把整條步道走完了。

再進一步：你踏上跑道那一刻，他離你最多一圈，之後每步逼近一步，所以你第一圈還沒繞完就會被追上。b < c，你的里程裡沒有自己繞的圈。

#### Step 4: 相遇時盤點里程

你走了 T 步，路線是步道加跑道上一小段：`T = a + b`。

他走了 kT 步，路線頭尾跟你一樣，多出來的部分起點終點都在跑道上，只能是整數圈：`kT = a + b + n·c`。

把第一條的 `a + b` 整塊代進第二條，相減：

```
kT = T + n·c
(k - 1)·T = n·c
T = n·c / (k - 1)
```

T 從此有兩個身分：從你的路線看是 `a + b`，從兩人路程差看是 `n·c / (k - 1)`。同一個數。

#### Step 5: 手上的位置只有兩個

公園門口（head，函式參數，一直在手上）跟相遇點（兩人此刻站的地方，指標正指著）。其他節點都走過但沒留下，留下就是撒麵包屑，回到 O(n)。所以能設計的動作只有一種：派人從這兩個位置出發往前走。

#### Step 6: 設計實驗：齊步走

一人回門口、一人留在相遇點，同速齊步走，賭他們在跑道入口碰面。什麼條件下賭贏？

- 門口那位走到入口：剛好 a 步。
- 相遇點那位走到入口：先走 `c - b` 補完這一圈，之後每多繞一圈（+c）都會再經過入口。所以是 `(c - b) + j·c`（j = 0, 1, 2, ...）。

兩人同時踩進入口的條件：`a = (c - b) + j·c`，整理成 `a + b = (j + 1)·c`。**你相遇時的總里程，要是跑道的整數圈。**

#### Step 7: 兩個身分對起來，解出 k

Step 4 說 T 既是 `a + b` 也是 `n·c / (k - 1)`。「a + b 是 c 的整數倍」換上第二個身分，就是 `n / (k - 1)` 要是整數：k − 1 要整除 n。

但 n 是朋友多繞的圈數，由公園的 a、c 決定，可能是任何正整數，寫演算法的人管不到。要對**任何** n 都成立，k − 1 只能是 1。

**k = 2。** 兩倍速是唯一不挑場地的配速。

#### Step 8: 驗證

用 a=2, c=10 跑 2x phase 1：

| t | slow | fast |
|---|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |
| 3 | 4 | 7 |
| 4 | 5 | 9 |
| 5 | 6 | 11 |
| 6 | 7 | 3 |
| 7 | 8 | 5 |
| 8 | 9 | 7 |
| 9 | 10 | 9 |
| 10 | **11** | **11** 相撞 |

T = 10, a + b = 10 = 1·c (n=1)。

Phase 2：slow 放回 head=1，fast 留 11，同速 1 步走：

| step | slow | fast |
|---|---|---|
| 0 | 1 | 11 |
| 1 | 2 | 12 |
| 2 | **3** | **3** 在入口相撞 |

入口 = 3，找到了。

---

### 其他倍速爛在哪

3 倍速需要 `2 整除 n`，n=1 就壞。4 倍速需要 `3 整除 n`，n=1 或 2 都壞。

壞的**不是偵測**（3 倍、4 倍一樣會相遇），是 **phase 2 進死迴圈**。

### 實際跑 3 倍速的壞例子

`1 -> 2 -> [3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 3 (回頭)]`，a=2, c=6。

**Phase 1（3 倍速，找相撞點）**：

| t | slow | fast |
|---|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 4 |
| 2 | 3 | 7 |
| 3 | **4** | **4** 相撞 |

T=3，n=1，相撞點 4，b=1。

**Phase 2（slow 回 head=1，fast 留 4，同速 1 步走）**：

| step | slow | fast | 在環內的差距 |
|---|---|---|---|
| 0 | 1 (前段) | 4 (環 pos 1) | slow 還沒進環 |
| 1 | 2 (前段) | 5 (環 pos 2) | slow 還沒進環 |
| 2 | **3 (入口, pos 0)** | 6 (pos 3) | **差 3** |
| 3 | 4 (pos 1) | 7 (pos 4) | 差 3 |
| 4 | 5 | 8 | 差 3 |
| 5 | 6 | 3 | 差 3 |
| 6 | 7 | 4 | 差 3 |
| ... | | | 永遠差 3 |

slow 進環那一刻，fast 已經在 pos 3，差 3 = c/2。**兩人同速，差距永遠是 3，`while slow != fast` 不會停**。

這個差距 3 不是巧合：`(a + b) mod c = (2 + 1) mod 6 = 3`，剛好是 c/2。Step 6 要求的「總里程是整數圈」在這裡沒被滿足，餘數 3 就變成兩人永遠追不上的固定差距。

比答案錯更糟：debug 半天會懷疑是不是 list 有無窮長。

2 倍速是唯一讓這個死迴圈絕對不發生的整數倍速。

### 找環的入口

[#142 Linked List Cycle II](/problem/linked-list-cycle-ii)

上一題用快慢指標判斷「有沒有環」。這一題更進一步：環從哪裡開始？

分兩個階段。階段一沿用上一題：slow 走 1 步、fast 走 2 步，找到相遇點。階段二**速度改變**：把一個指標移回 head，兩個都改成每次走 1 步。再次碰面的地方就是環入口。

為什麼這樣做有效？要從結構看起。有環的 list 不是整條都在環裡，前面還有一段「前段」（從 head 到環入口，整張圖的形狀像希臘字母 ρ，前段就是 ρ 的直線尾）：

```
[1] -> [2] -> [3] -> [4] -> [5] -> [6] -> [7] -> [8] -> [9]
                                    ^                     |
                                    +---------------------+
```

`[9]` 的 next 指回 `[6]`，形成環。環入口是 `[6]`，不是 `[1]`。目標就是找到 `[6]`。

#### 先用實際例子走一遍

slow 每次 1 步，fast 每次 2 步，都從 `[1]` 出發：

| 步數 | slow | fast | fast 怎麼走 |
|------|------|------|------------|
| 0 | [1] | [1] | |
| 1 | [2] | [3] | [1]->[2]->[3] |
| 2 | [3] | [5] | [3]->[4]->[5] |
| 3 | [4] | [7] | [5]->[6]->[7] |
| 4 | [5] | [9] | [7]->[8]->[9] |
| 5 | [6] | [7] | [9]->[6]->[7] |
| 6 | [7] | [9] | [7]->[8]->[9] |
| 7 | [8] | [7] | [9]->[6]->[7] |
| 8 | [9] | [9] | [7]->[8]->[9] |

第 8 步在 `[9]` 相遇。

現在把三段路取名字：
- `a` = `[1]` 到環入口 `[6]` = **5** 步
- `b` = 環入口 `[6]` 到相遇點 `[9]` = **3** 步
- `c` = 環一圈 `[6]->[7]->[8]->[9]->[6]` = **4** 步

三個數字全部不同。

#### 拿上一節的結論對帳

為什麼移回 head 齊步走就撞在入口，上一節的 Step 4 到 Step 7 已經推完：2 倍速保證**相遇時 slow 的總里程是整數圈**（`a + b = n·c`），所以從相遇點再走 a 步，等於補完這圈、再繞幾整圈，落點必是入口；從 head 走 a 步照定義也是入口。

用這組數字驗：`a + b = 5 + 3 = 8 = 2 × 4`，整數圈成立（n = 2）。從相遇點 `[9]` 走 a = 5 步：`[9]->[6]->[7]->[8]->[9]->[6]`，先到入口再多繞一圈，落點還是 `[6]`。從 head 走 5 步也到 `[6]`。同速同步數，在入口碰面。

---

## 反轉 Linked List

[#206 Reverse Linked List](/problem/reverse-linked-list)

面試最常出現的 linked list 題。

```
原本: [1] → [2] → [3] → null
反轉: [3] → [2] → [1] → null
```

三個指標：prev, curr, next。

```
prev = null
curr = [1] -> [2] -> [3] -> null

步驟 1:
  next = curr.Next = [2]
  curr.Next = prev = null     <- [1] 指向 null
  prev = curr = [1]
  curr = next = [2]

  null <- [1]   [2] -> [3] -> null
          prev  curr

步驟 2:
  next = [3]
  curr.Next = [1]             <- [2] 指向 [1]
  prev = [2]
  curr = [3]

  null <- [1] <- [2]   [3] -> null
                 prev  curr

步驟 3:
  next = null
  curr.Next = [2]             <- [3] 指向 [2]
  prev = [3]
  curr = null

  null <- [1] <- [2] <- [3]
                        prev

curr = null -> 結束。回傳 prev。
```

```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        prev = curr
        curr = next
    }
    return prev
}
```

四行核心。每行改一個指標。練到閉著眼睛能寫。

---

## Array vs Linked List

| | Array | Linked List |
|---|---|---|
| 隨機存取 | $O(1)$ | $O(n)$ |
| 插入/刪除（頭部） | $O(n)$ | $O(1)$ |
| 插入/刪除（中間） | $O(n)$ | $O(1)$（已知位置） |
| 記憶體 | 連續 | 分散 |
| Cache | 好（連續） | 差（跳來跳去） |
| 空間 overhead | 低 | 高（每個節點多一個指標） |

面試場景：array 幾乎永遠贏。Linked list 的優勢（$O(1)$ 插入刪除）在實務中被 cache miss 抵消了。

但 LeetCode 上 linked list 是單獨的一類題。考的是指標操作的功力，不是「你會不會選資料結構」。

---

## 高頻題清單

| 題目 | 核心 |
|------|------|
| [#206 Reverse Linked List](/problem/reverse-linked-list) | 三指標翻轉 |
| [#21 Merge Two Sorted Lists](/problem/merge-two-sorted-lists) | dummy head + 兩指標 |
| [#141 Linked List Cycle](/problem/linked-list-cycle) | 快慢指標 |
| [#142 Linked List Cycle II](/problem/linked-list-cycle-ii) | 快慢相遇後一個回 head |
| [#876 Middle of the Linked List](/problem/middle-of-the-linked-list) | 快慢指標找中點 |
| [#19 Remove Nth Node From End](/problem/remove-nth-node-from-end-of-list) | 快指標先走 n 步 |
| [#23 Merge K Sorted Lists](/problem/merge-k-sorted-lists) | Heap + linked list |
| [#146 LRU Cache](/problem/lru-cache) | Doubly linked list + hash map |
| [#2 Add Two Numbers](/problem/add-two-numbers) | 逐位相加，carry |
| [#234 Palindrome Linked List](/problem/palindrome-linked-list) | 找中點 + 反轉後半 + 比較 |

---

## 總結

Linked list 的題目考的不是資料結構的選擇。考的是**指標操作**。

三個必會技巧：
1. **Dummy head** — 省掉 head 的邊界判斷
2. **快慢指標** — 找中點、偵測環、找倒數第 N 個
3. **反轉** — 三指標翻轉，四行 code

上面那張清單裡，除了 #23 要配 heap、#146 要配 hash map，其他都是這三個技巧的組合。

想學另一種「兩個指標」的技巧？看 [Two Pointers](/concept/two-pointers)。
