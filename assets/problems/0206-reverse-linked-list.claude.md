Linked list 只能往前走，不能回頭。把它反轉，就是把每個箭頭的方向倒過來。

原本：`1 → 2 → 3 → 4 → 5 → nil`
反轉後：`nil ← 1 ← 2 ← 3 ← 4 ← 5`

**難在哪**

每個 node 只記得「我的下一個是誰」，不記得「我的上一個是誰」。

要改掉 `curr.Next = prev`，但改完之後就回不去原本的 next 了。所以要先存起來。

<details>
<summary>提示</summary>

每一步要存三個東西：前一個、現在、下一個。
改箭頭之前，先把下一個存起來。

</details>

---

## 解法一：迭代（三指針）

每走一步，把箭頭反轉：

```typescript
function reverseList(head: ListNode | null): ListNode | null {
    let prev: ListNode | null = null;
    let curr: ListNode | null = head;

    while (curr !== null) {
        const next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }

    return prev;
}
```

<details>
<summary>Go 版本</summary>

```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode // 前一個，初始是 nil（新的尾巴）
    curr := head

    for curr != nil {
        next := curr.Next  // 先存下一個，不然等一下找不到
        curr.Next = prev   // 箭頭反轉
        prev = curr        // 前進
        curr = next        // 前進
    }

    return prev // curr 走到 nil 時，prev 就是新的 head
}
```

</details>


**逐步圖解**

```
初始：prev=nil  curr=1→2→3→nil

step1：next=2, 1→nil, prev=1, curr=2
       nil ← 1   2 → 3 → nil

step2：next=3, 2→1, prev=2, curr=3
       nil ← 1 ← 2   3 → nil

step3：next=nil, 3→2, prev=3, curr=nil
       nil ← 1 ← 2 ← 3

curr == nil，停。回傳 prev = 3（新的 head）
```

---

## 解法二：遞迴

```typescript
function reverseList(head: ListNode | null): ListNode | null {
    // base case：空串列，或只剩一個 node
    if (head === null || head.next === null) {
        return head;
    }

    // 先把後面的全部反轉，拿到新的 head
    const newHead = reverseList(head.next);

    // 這時候 head.next 還指著它後面那個 node（現在是新尾巴）
    // 把那個節點的 next 指回 head，完成反轉
    head.next.next = head;
    head.next = null; // head 變成新尾巴，斷掉原本的箭頭

    return newHead;
}
```

<details>
<summary>Go 版本</summary>

```go
func reverseList(head *ListNode) *ListNode {
    // base case：空串列，或只剩一個 node
    if head == nil || head.Next == nil {
        return head
    }

    // 先把後面的全部反轉，拿到新的 head
    newHead := reverseList(head.Next)

    // 這時候 head.Next 還指著它後面那個 node（現在是新尾巴）
    // 把那個節點的 Next 指回 head，完成反轉
    head.Next.Next = head
    head.Next = nil // head 變成新尾巴，斷掉原本的箭頭

    return newHead
}
```

</details>


遞迴的邏輯：先讓後面的人排好，再處理自己。

`newHead` 是新變數，不是新 node。它只是拿一個指向 node 3 的指標，然後一路往上傳，自己從不改變：

```
reverseList(3) 回傳 3        → newHead = 3（node 3 本人）
reverseList(2) 回傳 newHead  → 還是 3
reverseList(1) 回傳 newHead  → 還是 3
```

真正改變 list 結構的是這兩行，不是 newHead：

```typescript
head.next.next = head; // 改箭頭
head.next = null;      // 斷箭頭
```

<details>
<summary>Go 版本</summary>

```go
head.Next.Next = head  // 改箭頭
head.Next = nil        // 斷箭頭
```

</details>


用 `1 → 2 → 3 → nil` 走一遍：

**下潛（call stack 疊起來）**

```
reverseList(1) 呼叫 reverseList(2)
  reverseList(2) 呼叫 reverseList(3)
    reverseList(3)：head.Next == nil，回傳 3   ← base case
```

這時候 list 還沒動，還是 `1 → 2 → 3 → nil`。

**回來（從尾巴開始處理）**

reverseList(2) 拿到 newHead = 3，head = 2：

```
head.Next.Next = head  →  3.Next = 2   （讓 3 指回 2）
head.Next = nil        →  2.Next = nil （斷掉 2→3 的舊箭頭）

現在：1 → 2 ← 3
```

reverseList(1) 拿到 newHead = 3，head = 1：

```
head.Next.Next = head  →  2.Next = 1   （讓 2 指回 1）
head.Next = nil        →  1.Next = nil （斷掉 1→2 的舊箭頭）

現在：nil ← 1 ← 2 ← 3
```

回傳 newHead = 3，完成。

---

記住這兩行：

```typescript
head.next.next = head; // 讓下一個 node 反指回來
head.next = null;      // 把自己原本的箭頭切掉
```

<details>
<summary>Go 版本</summary>

```go
head.Next.Next = head  // 讓下一個 node 反指回來
head.Next = nil        // 把自己原本的箭頭切掉
```

</details>


執行這兩行時 `head.Next` 還是舊的指向（還沒被改），所以 `head.Next.Next` 就是「讓那個 node 指回 head」。

---

**複雜度**

- Time: O(n)，每個 node 走一次
- Space: O(1)（迭代）/ O(n)（遞迴，call stack 深度）

> 面試建議先寫迭代版，清楚易懂。遞迴版當 follow-up 再補。
