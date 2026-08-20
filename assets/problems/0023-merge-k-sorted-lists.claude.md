K 個人手上各拿一疊撲克牌，每疊已從小到大排好（最小的在最上面）。把 K 疊合成一大疊，最小的在最上面。

```
人 A：1 → 4 → 5
人 B：1 → 3 → 4
人 C：2 → 6
```

每疊就是一條 sorted linked list。「拿一張」= 接一個 ListNode 到結果。「那人翻下一張」= `lists[i] = lists[i].next`。三種解法只差**怎麼選下一張**。

---

## 解法一：暴力掃描

最直覺：跑去每個人面前看一眼最上面那張，找最小的拿走。那個人翻下一張上來。重複。

```
比 1A, 1B, 2C → 拿 1A，A 翻 4A
比 4A, 1B, 2C → 拿 1B，B 翻 3B
比 4A, 3B, 2C → 拿 2C，C 翻 6C
比 4A, 3B, 6C → 拿 3B，B 翻 4B
...
```

每張牌出列前都要走過 K 個人比一輪。N 張牌總共比 $N \times K$ 次。

```typescript
function mergeKLists(lists: Array<ListNode | null>): ListNode | null {
    const dummy = new ListNode();
    let curr = dummy;
    while (true) {
        let minIdx = -1;
        for (let i = 0; i < lists.length; i++) {
            if (!lists[i]) continue;
            if (minIdx === -1 || lists[i]!.val < lists[minIdx]!.val) {
                minIdx = i;               // 記住最小的是哪條
            }
        }
        if (minIdx === -1) break;         // 全部走完了
        curr.next = lists[minIdx];        // 接上最小的
        lists[minIdx] = lists[minIdx]!.next;
        curr = curr.next!;
    }
    return dummy.next;
}
```

- **Time: $O(n \times k)$** — 每張牌都要跟 k 個 head 比一次
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func mergeKLists(lists []*ListNode) *ListNode {
    dummy := &ListNode{}
    curr := dummy
    for {
        minIdx := -1
        for i, l := range lists {
            if l == nil { continue }
            if minIdx == -1 || l.Val < lists[minIdx].Val {
                minIdx = i                // 記住最小的是哪條
            }
        }
        if minIdx == -1 { break }         // 全部走完了
        curr.Next = lists[minIdx]         // 接上最小的
        lists[minIdx] = lists[minIdx].Next
        curr = curr.Next
    }
    return dummy.Next
}
```

</details>

n = $10^4$，k = $10^4$，最差一億次。太慢。

---

## 解法二：Min-heap

瓶頸在「每張牌出列前都要走過 K 個人」。如果**有個盒子能直接告訴你 K 個人手上現在最小的是誰**，就不用每次跑一遍。

那個盒子叫 **min heap**（不熟見 [Heap](/concept/heap)）。

```
[盒子]  一開始把 K 個人最上面那張全丟進來
        盒子的魔法：頂端永遠是當下最小

每輪：
  伸手進盒子拿最小的那張           ← O(log K)
  通知對應的人：「翻下一張給我」
  把翻出來的新牌丟進盒子            ← O(log K)
```

盒子內部用「冒泡 + 下沉」維護「最小在頂」的規則，每次操作 O(log K)，比暴力的 O(K) 快。

```typescript
function mergeKLists(lists: Array<ListNode | null>): ListNode | null {
    // TypeScript 沒有內建 heap，用 MinPriorityQueue（LeetCode 有提供）
    const pq = new MinPriorityQueue<ListNode>({ compare: (a, b) => a.val - b.val });

    for (const l of lists) {
        if (l) pq.enqueue(l);             // K 個 head 全進盒子
    }

    const dummy = new ListNode();
    let curr = dummy;
    while (pq.size() > 0) {
        const node = pq.dequeue()!;       // 拿盒子裡最小的
        curr.next = node;
        curr = curr.next;
        if (node.next) {
            pq.enqueue(node.next);        // 那個人翻下一張進盒子
        }
    }
    return dummy.next;
}
```

<details>
<summary>Go 版本</summary>

```go
import "container/heap"

type MinHeap []*ListNode
func (h MinHeap) Len() int            { return len(h) }
func (h MinHeap) Less(i, j int) bool  { return h[i].Val < h[j].Val }
func (h MinHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *MinHeap) Push(x any)         { *h = append(*h, x.(*ListNode)) }
func (h *MinHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}

func mergeKLists(lists []*ListNode) *ListNode {
    h := &MinHeap{}
    for _, l := range lists {
        if l != nil {
            heap.Push(h, l)               // K 個 head 全進盒子
        }
    }

    dummy := &ListNode{}
    curr := dummy
    for h.Len() > 0 {
        node := heap.Pop(h).(*ListNode)   // 拿盒子裡最小的
        curr.Next = node
        curr = curr.Next
        if node.Next != nil {
            heap.Push(h, node.Next)       // 那個人翻下一張進盒子
        }
    }
    return dummy.Next
}
```

</details>

**Step by step**

input：A = 1→4→5，B = 1→3→4，C = 2→6。盒子內元素標 list 來源（A/B/C）方便看誰是誰：

| 圈 | 盒子 (heap) | 拿走（接到 result） | 那人翻下一張進盒子 |
|---|---|---|---|
| 初始 | [1A, 1B, 2C] | — | — |
| 1 | [1B, 2C, 4A] | 1A | A 翻 4A |
| 2 | [2C, 4A, 3B] | 1B | B 翻 3B |
| 3 | [3B, 4A, 6C] | 2C | C 翻 6C |
| 4 | [4A, 6C, 4B] | 3B | B 翻 4B |
| 5 | [4B, 6C, 5A] | 4A | A 翻 5A |
| 6 | [5A, 6C] | 4B | B 已沒牌 |
| 7 | [6C] | 5A | A 已沒牌 |
| 8 | [] | 6C | C 已沒牌 |

result：1 → 1 → 2 → 3 → 4 → 4 → 5 → 6

盒子大小永遠 ≤ K = 3。每張牌進出盒子各一次。

- Time: O(n log k) — 每張牌 push/pop 各一次，每次 O(log k)
- Space: O(k) — 盒子最多裝 k 張

從 O(nk) 降到 O(n log k)。k = $10^4$ 的時候，差了 700 倍。

---

**概念坑：heap 不會自己沿 `.Next` 走**

容易誤會的點：「都是 ListNode，pop 出 head 之後 next 不就連著進來？」

不會。heap 只存**你親手 push 進去的指標**，pop a1 出來時 a2 只是還躺在記憶體裡的孤兒，heap 不知道它存在。要它進來排序，必須 `heap.Push(h, a1.Next)`。

這也是為什麼盒子永遠只有 K 個 — 每 pop 一張、手動補一張，sift 成本 stable 在 O(log K)。

---

## 解法三：分治

還有一條路，不用盒子，**改打淘汰賽**。

K 個人兩兩配對，每對先把兩疊牌合成一疊（兩個人變一個人）。下一輪剩 K/2 個人，再配對再合。一直打到剩一個人手上一疊牌就結束。

```
輪 1: A B C D E F G H        ← 8 人
       └┬┘ └┬┘ └┬┘ └┬┘
        AB  CD  EF  GH       ← 兩兩合，剩 4 人
輪 2:    └┬┘    └┬┘
          ABCD  EFGH         ← 剩 2 人
輪 3:       └─┬─┘
            ABCDEFGH         ← 決賽，剩 1 人
```

「兩疊已排序的合成一疊」就是 [Merge Two Sorted Lists](/problem/merge-two-sorted-lists)，兩根指針誰小誰先接，O(N+M)。

```typescript
function mergeKLists(lists: Array<ListNode | null>): ListNode | null {
    if (lists.length === 0) return null;
    while (lists.length > 1) {
        const merged: Array<ListNode | null> = [];
        for (let i = 0; i < lists.length; i += 2) {
            if (i + 1 < lists.length) {
                merged.push(mergeTwoLists(lists[i], lists[i + 1]));
            } else {
                merged.push(lists[i]);            // 落單（輪空）的帶到下一輪
            }
        }
        lists = merged;                           // 下一輪
    }
    return lists[0];
}

function mergeTwoLists(a: ListNode | null, b: ListNode | null): ListNode | null {
    const dummy = new ListNode();
    let curr = dummy;
    while (a && b) {
        if (a.val <= b.val) {
            curr.next = a;
            a = a.next;
        } else {
            curr.next = b;
            b = b.next;
        }
        curr = curr.next!;
    }
    curr.next = a ?? b;
    return dummy.next;
}
```

<details>
<summary>Go 版本</summary>

```go
func mergeKLists(lists []*ListNode) *ListNode {
    if len(lists) == 0 { return nil }
    for len(lists) > 1 {
        var merged []*ListNode
        for i := 0; i < len(lists); i += 2 {
            if i+1 < len(lists) {
                merged = append(merged, mergeTwoLists(lists[i], lists[i+1]))
            } else {
                merged = append(merged, lists[i]) // 落單（輪空）的帶到下一輪
            }
        }
        lists = merged                            // 下一輪
    }
    return lists[0]
}

func mergeTwoLists(a, b *ListNode) *ListNode {
    dummy := &ListNode{}
    curr := dummy
    for a != nil && b != nil {
        if a.Val <= b.Val {
            curr.Next = a
            a = a.Next
        } else {
            curr.Next = b
            b = b.Next
        }
        curr = curr.Next
    }
    if a != nil { curr.Next = a }
    if b != nil { curr.Next = b }
    return dummy.Next
}
```

</details>

**Step by step**

input：A = 1→4→5，B = 1→3→4，C = 2→6。三個人打淘汰賽，輪 1 配一對 + 一個人輪空（bye）。

外層 while 每輪兩兩合併，K 條砍成 K/2 條：

```
輪 1: lists = [A, B, C]    length 3 > 1
        i=0: A+B 合 → AB
        i=2: 落單 → C
      lists = [AB, C]      length 2

輪 2: lists = [AB, C]       length 2 > 1
        i=0: AB+C 合 → ABC
      lists = [ABC]         length 1

輪 3: length = 1，跳出
```

內層 `mergeTwoLists(A, B)` 兩根指針推進：

| a | b | 比較 | 接誰 |
|---|---|---|---|
| 1A | 1B | 1 ≤ 1 | 1A |
| 4A | 1B | 4 > 1 | 1B |
| 4A | 3B | 4 > 3 | 3B |
| 4A | 4B | 4 ≤ 4 | 4A |
| 5A | 4B | 5 > 4 | 4B |
| 5A | null | b 空 | a 整條接上 |

得 AB = 1→1→3→4→4→5。`mergeTwoLists(AB, C)` 同理，最終 1→1→2→3→4→4→5→6。

- Time: O(n log k) — 淘汰賽打 log k 輪，每輪掃過所有 n 個節點
- Space: O(1) — 不算遞迴的話，原地改指標

跟 heap 一樣是 O(n log k)，但不需要額外的 heap 空間。實際跑起來也更快——沒有 heap 的 push/pop overhead。

---

**兩種 O(n log k) 怎麼選？**

| | Heap | 分治 |
|---|---|---|
| 比喻 | 一個會自動把最小推到頂的盒子 | 打淘汰賽 |
| 時間 | O(n log k) | O(n log k) |
| 空間 | O(k) | O(1) |
| 實作難度 | 要寫 heap interface（Go） | 只要會 merge two lists |
| 實際速度 | heap 有常數開銷 | 更快 |

面試時分治更好寫，也更好解釋。「兩兩合併打淘汰賽，跟 merge sort 一樣。」面試官秒懂。

---

## 結論

暴力 O(nk) 太慢。用 heap 或分治降到 O(n log k)。分治不需要額外空間，程式碼更短，面試首選。

---

**延伸：這題其實是 K-way merge sort 的簡單版**

這題的本體是 **K-way merge sort**，只是把 input 從「硬碟上的 K 個 sorted 檔案」換成「記憶體裡的 K 條 sorted linked list」。merge 邏輯一模一樣。

**生活比喻**：全班 200 份考卷依分數從低到高排好，但桌子只能放 50 份。

```
階段 1（sort）：
  抓 50 份，桌上排好 → 疊成第 1 疊（最低分在最上面）
  抓下 50 份，桌上排好 → 疊成第 2 疊
  抓下 50 份，桌上排好 → 疊成第 3 疊
  抓最後 50 份，桌上排好 → 疊成第 4 疊

  4 疊內部各自排好，但 4 疊互相之間沒排好

階段 2（merge）：
  桌上只放 4 張：4 疊各自最上面那張（每疊的最低分）
  比 4 張，最低的拿走放結果區，那疊翻下一張上來
  比 4 張，最低的拿走放結果區，那疊翻下一張上來
  ...直到全部翻完
```

桌子永遠只有 4 張卷子，但能正確排好 200 份。**用小空間（heap）處理大資料（無法一次塞進記憶體）的核心招式**。

翻譯回工程：

| 比喻 | LC 23（in-memory 版） | 完整外部排序 |
|---|---|---|
| 4 疊考卷 | 4 條 sorted linked list | 4 個 sorted 檔案 |
| 桌上 4 張 | heap 裡的 4 個 head | heap 裡的 4 個 head |
| 比最低、那疊翻下一張 | pop 最小、push node.next | pop 最小、從那個檔讀下一個進來 |

LC 23 假設**資料已經是 K 條 sorted stream**，跳過階段 1，直接做階段 2 merge。完整的外部排序還要先做階段 1（切塊、各自內部排序、寫回硬碟），但 merge 階段的邏輯跟這題完全一樣。

**面試延伸題**：「100 GB 整數要排序，記憶體只有 1 GB，怎麼做？」

答：切 100 塊每塊 1 GB，各自讀進記憶體 sort 寫回硬碟 → 對 100 個 sorted 檔做 100-way merge（heap 裡永遠 100 個 head，pop 最小寫輸出，那個檔讀下一個補進來）。

merge 階段的 code 跟這題的 heap 解法**幾乎一行不改**，只是 `node.next` 換成「從檔案 reader 讀下一個 int」。所以這題寫熟了，外部排序系統設計題也順手解。
