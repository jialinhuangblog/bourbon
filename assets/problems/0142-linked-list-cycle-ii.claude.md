給你一條 linked list，裡面可能有環。有的話，回傳環開始的那個節點。沒有就回傳 null。

環長這樣：某個節點的 next 指回前面某個已經走過的節點，形成圈。你不知道環在哪、多大，只有 head。

---

**解題引導**

用這個 list 想：`[1]->[2]->[3]->[4]->[5]->[6]->[7]->[8]`，其中 `[8]` 的 next 指回 `[4]`。

**Step 1：最直覺的方法？怎麼知道一個節點「之前走過」？**

<span class="spoiler">用 hash set 記每個走過的節點。走到一個已經在 set 裡的，就是環入口</span>

**Step 2：hash set 要 O(n) 空間。能不能 O(1) 空間？**

<span class="spoiler">快慢指標。slow 走 1 步、fast 走 2 步，有環的話一定會相遇</span>

**Step 3：相遇了，但相遇點不是環入口。怎麼找入口？**

<span class="spoiler">把其中一個指標移回 head，兩個都改成走 1 步。再次碰面的地方就是環入口</span>

**Step 4：為什麼這樣就能找到入口？**

<span class="spoiler">數學推導出「head 到環入口的距離」等於「相遇點沿環走回入口的距離（可能多繞幾圈）」。兩邊同速走，同步數，同終點</span>

想完再往下看 code。

---

用 `[1]->[2]->[3]->[4]->[5]->[6]->[7]->[8]->[4]` 走一遍。環入口是 `[4]`。

---

## 解法一：Hash Set

走過的節點丟進 set，碰到重複的就是入口。

```typescript
function detectCycle(head: ListNode | null): ListNode | null {
    const seen = new Set<ListNode>();
    let curr = head;
    while (curr !== null) {
        if (seen.has(curr)) return curr;
        seen.add(curr);
        curr = curr.next;
    }
    return null;
}
```

- Time: O(n)
- Space: **O(n)** — 每個節點都存一份

<details>
<summary>Go 版本</summary>

```go
func detectCycle(head *ListNode) *ListNode {
    seen := map[*ListNode]bool{}
    for curr := head; curr != nil; curr = curr.Next {
        if seen[curr] {
            return curr // 見過了，這就是入口
        }
        seen[curr] = true
    }
    return nil // 走到 null，沒環
}
```

</details>

逐步走：

```
curr=[1]  seen={}        -> 沒見過，加入
curr=[2]  seen={1}       -> 沒見過，加入
curr=[3]  seen={1,2}     -> 沒見過，加入
curr=[4]  seen={1,2,3}   -> 沒見過，加入
curr=[5]  seen={1..4}    -> 沒見過，加入
curr=[6]  seen={1..5}    -> 沒見過，加入
curr=[7]  seen={1..6}    -> 沒見過，加入
curr=[8]  seen={1..7}    -> 沒見過，加入
curr=[4]  seen={1..8}    -> 見過了！回傳 [4]
```

題目 follow-up 問：能不能 O(1) 空間？

---

## 解法二：快慢指標

空間從 O(n) 降到 O(1)。用快慢指標，分兩個階段。

**階段一：偵測環。** slow 走 1 步，fast 走 2 步。有環 fast 一定追上 slow（相對速度 1，距離每步縮小 1，一定歸零）。沒環 fast 先到 null。

**階段二：找入口。速度改變。** 一個移回 head，一個留在相遇點，**兩個都走 1 步**。再次碰面就是環入口。

為什麼階段二有效？四步推理，不用公式：

1. **fast 比 slow 多走的步數，全部花在環裡繞圈。** 兩人從 head 出發，在環裡某個點碰面。多出來的路起點終點一樣，只能是在環裡繞整圈。
2. **fast 走的 = 2 倍 slow 走的，所以多出來的 = slow 走的。** 多出來的是整數圈，所以 slow 的總步數也是整數圈。
3. **slow 走的路 = 前段 + 進環深度 = 整數圈。** 在這個例子：前段 3 步 + 進環 2 步 = 5 步 = 剛好一圈。
4. **前段 + 深度 = 整圈，所以從相遇點走回入口 = 整圈 - 深度 = 前段。** 5 - 2 = 3 = 前段長度。

兩段路一樣長。一個從 head 走，一個從相遇點走，同速，在入口碰面。完整數學推導見 [Linked List - 找環的入口](/concept/linked-list)。

用同一組 list 走：

```
階段一：找相遇點（slow 走 1 步，fast 走 2 步）

步數  slow  fast   fast 怎麼走
0     [1]   [1]
1     [2]   [3]    [1]->[2]->[3]
2     [3]   [5]    [3]->[4]->[5]
3     [4]   [7]    [5]->[6]->[7]
4     [5]   [4]    [7]->[8]->[4]
5     [6]   [6]    [4]->[5]->[6]

第 5 步相遇在 [6]。
```

a=3, b=2, c=5。c-b=3=a。✓

```
階段二：找入口（都走 1 步）

slow 移回 head，fast 留在相遇點。

步數  slow  fast
0     [1]   [6]
1     [2]   [7]
2     [3]   [8]
3     [4]   [4]    <- 碰面！回傳 [4]
```

走 3 步，兩邊同時到 `[4]`。

```typescript
function detectCycle(head: ListNode | null): ListNode | null {
    let slow = head, fast = head;

    while (fast !== null && fast.next !== null) {
        slow = slow!.next;
        fast = fast.next.next;
        if (slow === fast) {
            slow = head;
            while (slow !== fast) {
                slow = slow!.next;
                fast = fast!.next;
            }
            return slow;
        }
    }
    return null;
}
```

<details>
<summary>Go 版本</summary>

```go
func detectCycle(head *ListNode) *ListNode {
    slow, fast := head, head

    // 階段一：找相遇點
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast {
            // 階段二：找入口
            slow = head           // 一個回 head
            for slow != fast {
                slow = slow.Next  // 都走 1 步
                fast = fast.Next
            }
            return slow
        }
    }
    return nil // 沒環
}
```

</details>

- Time: O(n) — 兩個階段加起來，每個節點最多走常數次
- Space: O(1) — 只用兩個指標

---

**為什麼 fast 一定要走 2 步？**

不是隨便選的，是從 phase 2 的需求逼出來的。

設 slow 每步 1 格、fast 每步 k 格。相遇時 slow 走 T 步、fast 走 kT 步，同樣停在相遇點但 fast 多繞 n 圈：

```
T = μ + b
kT = μ + b + n·c
────────────────
(k − 1)·T = n·c
T = n·c / (k − 1)
```

phase 2 把 slow 搬回 head、fast 留相遇點、同速走，要剛好在入口碰頭。等價條件：`T mod c = 0`，也就是 `(k − 1) | n`。

**n 是 fast 多繞的圈數，由輸入決定**，可能是任何正整數。要對所有 n 都成立，`k − 1` 必須能整除所有正整數 — 只有 1 做得到。所以 `k = 2`。

3 倍速（k=3）要求 `2 | n`，n=1 就壞。壞的不是偵測（照樣會相遇），是 phase 2 變死迴圈：slow 和 fast 同速繞，距離差鎖在 `c/2` 這種非零常數，永遠追不上。`while slow != fast` 不會停。

比答案錯更糟 — debug 會懷疑是不是 list 有無窮長。

---

## 結論

Hash set 直覺但吃 O(n) 空間。快慢指標兩個階段——偵測用不同速度，找入口用相同速度——O(1) 空間搞定。階段二把 2 倍速切回 1 倍速，兩個指標才會在入口碰面。
