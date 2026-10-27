判斷一條 linked list 裡有沒有環。有環回傳 true，沒有回傳 false。

環就是某個節點的 next 指回前面走過的節點，形成圈。你只有 head，不知道環在哪。

---

**解題引導**

用這個 list 想：`[1]->[2]->[3]->[4]->[5]`，其中 `[5]` 的 next 指回 `[3]`。

**Step 1：最直覺的方法？怎麼知道「走過這個節點了」？**

<span class="spoiler">用 hash set 記每個走過的節點。走到一個已經在 set 裡的，就是有環</span>

**Step 2：hash set 要 O(n) 空間。能不能 O(1) 空間？**

<span class="spoiler">快慢指標。slow 走 1 步、fast 走 2 步。有環的話 fast 一定追上 slow。沒環的話 fast 先到 null</span>

**Step 3：為什麼 fast 一定追得上 slow？**

<span class="spoiler">兩個都進環後，相對速度 = 1，距離每步縮小 1，不管環多大一定歸零。像操場跑步，快的人一定套圈</span>

想完再往下看 code。

---

用 `[1]->[2]->[3]->[4]->[5]->[3]` 走一遍。`[5]` 指回 `[3]`，環是 `[3]->[4]->[5]->[3]`。

---

## 解法一：Hash Set

走過的節點丟進 set。碰到重複的就是有環，走到 null 就是沒環。

```typescript
function hasCycle(head: ListNode | null): boolean {
    const seen = new Set<ListNode>();
    let curr = head;
    while (curr !== null) {
        if (seen.has(curr)) return true;
        seen.add(curr);
        curr = curr.next;
    }
    return false;
}
```

- Time: O(n)
- Space: **O(n)** — 每個節點都存一份

<details>
<summary>Go 版本</summary>

```go
func hasCycle(head *ListNode) bool {
    seen := map[*ListNode]bool{}
    for curr := head; curr != nil; curr = curr.Next {
        if seen[curr] {
            return true // 見過了，有環
        }
        seen[curr] = true
    }
    return false // 走到 null，沒環
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
curr=[3]  seen={1..5}    -> 見過了！return true
```

題目 follow-up 問：能不能 O(1) 空間？

---

## 解法二：快慢指標

Hash set 的空間是浪費。你不需要記住「走過哪些」，只需要知道「有沒有繞回來」。

兩個指標，不同速度。slow 走 1 步，fast 走 2 步。沒環的話 fast 先到 null。有環的話兩個都進環裡繞，fast 每步比 slow 多走一格（相對速度 = 1），距離每步縮小 1，一定追上。

用同一組 list 走：

```
[1]->[2]->[3]->[4]->[5]->[3]    環：[3]->[4]->[5]->[3]

步數  slow  fast
0     [1]   [1]
1     [2]   [3]
2     [3]   [5]     <- 兩個都進環了（環是 [3]->[4]->[5]）
3     [4]   [4]     <- fast 從 [5] 走兩步：5→3→4，追上！return true
```

第 3 步在 `[4]` 碰面。fast 的第二步經過 `[5]` 指回 `[3]` 的那一跳，這就是「套圈」發生的那一下。

沒有環的情況？`[1]->[2]->[3]->[4]->[5]->null`：

```
步數  slow  fast
0     [1]   [1]
1     [2]   [3]
2     [3]   [5]
3     --    fast.Next == null -> 停

fast 走到底了。return false
```

```typescript
function hasCycle(head: ListNode | null): boolean {
    let slow = head, fast = head;
    while (fast !== null && fast.next !== null) {
        slow = slow!.next;
        fast = fast.next.next;
        if (slow === fast) return true;
    }
    return false;
}
```

<details>
<summary>Go 版本</summary>

```go
func hasCycle(head *ListNode) bool {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next      // 走 1 步
        fast = fast.Next.Next // 走 2 步
        if slow == fast {
            return true       // 追上了，有環
        }
    }
    return false // fast 到 null，沒環
}
```

</details>

- Time: O(n) — fast 在環裡最多繞一圈就追上 slow
- Space: O(1) — 只用兩個指標

---

**為什麼判斷 `fast != nil && fast.Next != nil`？**

fast 走 2 步，可能一次跳過 null。如果 fast 本身是 null，`fast.Next` 就爆了。如果 fast 不是 null 但 `fast.Next` 是 null，`fast.Next.Next` 也爆了。兩個條件缺一不可。

slow 不用檢查。因為 slow 走得比 fast 慢，fast 沒爆的話 slow 一定還在 list 裡。

---

**為什麼剛好是 2 倍速？**

先回答更基本的：為什麼不能同速？兩人都從 head 出發，同速就永遠疊在一起，`slow === fast` 每一步都成立，有環沒環分不出來。必須先拉開、再靠環繞回來重逢，重逢才是「有環」的證據，所以速度一定要不同。

2 倍的保證最乾淨：進環之後 fast 每步比 slow 多走 1 格，兩人距離最多環長 L，每步縮 1，**不可能跳過**，一圈之內必定落在同一格。不用管環多長、從哪進環。

3 倍、4 倍其實也追得上：兩個指標同一個起點出發，奇偶剛好對齊，中間會擦身而過，但多繞幾圈一定撞上。

那教科書為什麼都寫 2 倍？相對速度 1 是「絕不擦身」的最小速度，證明一句話就完；跑更快也不省工，fast 每一步都真的要跟著 `next` 走一次，3 倍速每輪走三次，總工作量沒少；最後是決定性的理由：下一題 [142 Linked List Cycle II](/problem/linked-list-cycle-ii) 要找環的入口，**「從相遇點走前段長度 = 入口」的關係式只在 2 倍速成立**，其他倍速會在 phase 2 陷入死迴圈。完整推導見 [Linked List 概念頁的「為什麼是 2 倍速」](/concept/linked-list)。

141 單獨看可以自由挑速度。這裡的 `2` 是為了接 142 的選擇，不是偵測本身逼出來的。

---

## 結論

Hash set 直覺但吃 O(n) 空間。快慢指標只要兩個變數。有環 fast 追上 slow，沒環 fast 先到 null。

進階：不只判斷有沒有環，還要找環從哪裡開始？看 [142 Linked List Cycle II](/problem/linked-list-cycle-ii)。
