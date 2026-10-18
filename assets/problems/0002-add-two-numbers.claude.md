兩個數字用 linked list 倒著存（個位在前），把它們加起來，回傳同樣格式的 linked list。

---

先搞懂題目在說什麼。

`342 + 465 = 807`。大家都會算。但題目不是直接給你數字。它給你兩條 linked list，數字是**倒著存**的。

342 存成 `2 → 4 → 3`。個位數在最前面，百位數在最後面。
465 存成 `5 → 6 → 4`。一樣倒著。

你要回傳的答案 807，也倒著存：`7 → 0 → 8`。

為什麼倒著？想想小學加法。你從個位數開始加，往左進位。倒著存剛好讓你從 list 的頭開始處理，不用先跑到尾巴。

---

## 解法一：轉成數字

最直覺的暴力解：把 linked list 轉回數字，加完，再拆回 linked list。

怎麼轉？`2 → 4 → 3` 要變成 342。

第一個 node 是 2，乘以 1（個位）。第二個 node 是 4，乘以 10（十位）。第三個 node 是 3，乘以 100（百位）。加起來：`2×1 + 4×10 + 3×100 = 342`。

```typescript
function addTwoNumbers(l1: ListNode | null, l2: ListNode | null): ListNode | null {
    let num1 = 0, num2 = 0;
    let base = 1;
    for (let p = l1; p !== null; p = p.next) { // 走過 l1 的每個 node
        num1 += p.val * base;                   // 2×1, 4×10, 3×100
        base *= 10;                             // 每走一步，位數升一級
    }
    base = 1;
    for (let p = l2; p !== null; p = p.next) { // 同樣方式處理 l2
        num2 += p.val * base;
        base *= 10;
    }

    let sum = num1 + num2; // 342 + 465 = 807
    if (sum === 0) {
        return new ListNode(0);
    }

    // 把 807 拆回 linked list：7 → 0 → 8
    const dummy = new ListNode(0);
    let cur = dummy;
    while (sum > 0) {
        cur.next = new ListNode(sum % 10); // 807 % 10 = 7，取個位
        cur = cur.next;
        sum = Math.floor(sum / 10);        // 807 / 10 = 80，砍掉個位
    }
    return dummy.next;
}
```

<details>
<summary>Go 版本</summary>

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    num1, num2 := 0, 0
    base := 1
    for p := l1; p != nil; p = p.Next { // 走過 l1 的每個 node
        num1 += p.Val * base             // 2×1, 4×10, 3×100
        base *= 10                        // 每走一步，位數升一級
    }
    base = 1
    for p := l2; p != nil; p = p.Next { // 同樣方式處理 l2
        num2 += p.Val * base
        base *= 10
    }

    sum := num1 + num2 // 342 + 465 = 807
    if sum == 0 {
        return &ListNode{Val: 0}
    }

    // 把 807 拆回 linked list：7 → 0 → 8
    dummy := &ListNode{}
    cur := dummy
    for sum > 0 {
        cur.Next = &ListNode{Val: sum % 10} // 807 % 10 = 7，取個位
        cur = cur.Next
        sum /= 10                            // 807 / 10 = 80，砍掉個位
    }
    return dummy.Next
}
```

</details>

`sum % 10` 是什麼？取餘數。807 除以 10，餘 7。就是個位數。
`sum / 10` 是什麼？整數除法。807 除以 10，得 80。把個位砍掉，準備取下一位。

這樣一輪一輪拆，807 變成 `7 → 0 → 8`。剛好是倒著的。

- Time: O(n)
- Space: O(n)

能跑。但有個致命問題。

**Overflow。** 題目說每條 list 最多 100 個 node。100 位數的數字。Go 的 `int64` 最多存 19 位。直接爆掉。

---

## 解法二：逐位相加

暴力解的問題不是慢，是根本存不下。那就別轉成數字了。

回想小學怎麼做直式加法：

```
  3 4 2
+ 4 6 5
-------
```

從右邊（個位）開始。`2 + 5 = 7`，寫 7。`4 + 6 = 10`，寫 0 進 1。`3 + 4 + 1(進位) = 8`，寫 8。答案 807。

題目幫你把數字倒過來了。`2 → 4 → 3` 的第一個 node 就是個位。所以你從頭開始走，就是從個位開始加。

同時走兩條 list。每一步：拿兩個數字加起來，再加上一輪的進位。個位放進新 node，十位留給下一輪當進位。

```typescript
function addTwoNumbers(l1: ListNode | null, l2: ListNode | null): ListNode | null {
    const dummy = new ListNode(0);
    let cur = dummy;
    let carry = 0;

    while (l1 || l2 || carry) {
        let sum = carry;
        if (l1) {
            sum += l1.val;
            l1 = l1.next;
        }
        if (l2) {
            sum += l2.val;
            l2 = l2.next;
        }
        carry = Math.floor(sum / 10);
        cur.next = new ListNode(sum % 10);
        cur = cur.next;
    }

    return dummy.next;
}
```

<details>
<summary>Go 版本</summary>

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    dummy := &ListNode{}  // 假的頭，等等回傳 dummy.Next
    cur := dummy
    carry := 0            // 進位，一開始是 0

    for l1 != nil || l2 != nil || carry != 0 {
        sum := carry      // 先把上一輪的進位加進來
        if l1 != nil {
            sum += l1.Val // 加 l1 的這一位
            l1 = l1.Next  // l1 往下走
        }
        if l2 != nil {
            sum += l2.Val // 加 l2 的這一位
            l2 = l2.Next  // l2 往下走
        }
        carry = sum / 10             // 進位：12 / 10 = 1
        cur.Next = &ListNode{Val: sum % 10} // 個位數：12 % 10 = 2
        cur = cur.Next
    }

    return dummy.Next
}
```

</details>

- Time: O(max(m, n)) — 走完較長的那條
- Space: O(max(m, n)) — 新 list 最多多一個 node（最後一個進位）

用 Example 1 走一遍。`l1 = 2→4→3`，`l2 = 5→6→4`，carry = 0。

第一步：`sum = 0 + 2 + 5 = 7`。carry = 0。新 node = 7。
第二步：`sum = 0 + 4 + 6 = 10`。carry = 1。新 node = 0。
第三步：`sum = 1 + 3 + 4 = 8`。carry = 0。新 node = 8。
兩條都走完，carry 也是 0。結束。答案：`7 → 0 → 8`。

兩個容易踩的坑。

第一，迴圈條件的 `carry != 0`。想像 `999 + 1 = 1000`。三步走完之後，l1 和 l2 都是 nil，但 carry 還是 1。少了這個條件，最高位的 1 就不見了。

順帶一提，carry 永遠只會是 0 或 1。因為每一位最大就是 `9 + 9 + 1(上一輪進位) = 19`，不可能到 2。所以你寫 `carry = sum >= 10 ? 1 : 0` 也完全正確。用 `sum / 10` 只是比較通用的寫法，省一個 if。

第二，`dummy node`。建 linked list 的時候，第一個 node 很煩。沒有 dummy 的話，你得在迴圈外面特別處理「還沒有任何 node」的情況。有了 dummy，每一步都是 `cur.Next = 新node`，統一處理。最後回傳 `dummy.Next` 跳過假頭就好。

---

## 解法三：遞迴

有人會寫遞迴版。每一層處理一位，進位傳給下一層。

```typescript
function addTwoNumbers(l1: ListNode | null, l2: ListNode | null): ListNode | null {
    return helper(l1, l2, 0);
}

function helper(l1: ListNode | null, l2: ListNode | null, carry: number): ListNode | null {
    if (l1 === null && l2 === null && carry === 0) {
        return null;
    }
    let sum = carry;
    if (l1 !== null) {
        sum += l1.val;
        l1 = l1.next;
    }
    if (l2 !== null) {
        sum += l2.val;
        l2 = l2.next;
    }
    const node = new ListNode(sum % 10);
    node.next = helper(l1, l2, Math.floor(sum / 10)); // 剩下的交給下一層
    return node;
}
```

<details>
<summary>Go 版本</summary>

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    return helper(l1, l2, 0)
}

func helper(l1, l2 *ListNode, carry int) *ListNode {
    if l1 == nil && l2 == nil && carry == 0 {
        return nil
    }
    sum := carry
    if l1 != nil {
        sum += l1.Val
        l1 = l1.Next
    }
    if l2 != nil {
        sum += l2.Val
        l2 = l2.Next
    }
    node := &ListNode{Val: sum % 10}
    node.Next = helper(l1, l2, sum/10) // 剩下的交給下一層
    return node
}
```

</details>

邏輯一模一樣，只是用 function call stack 取代了 while 迴圈。代價是多吃 O(n) 的 stack 空間。100 層遞迴不會爆，但也沒有任何好處。

---

**Overthinking：如果數字是正著存的呢？**

`342` 存成 `3 → 4 → 2`。高位在前，低位在後。

這時候你不能從頭開始加。因為你不知道進位會不會一路往前影響。想想 `999 + 1`，進位要從個位一路傳到千位。但個位在最後面，你得先走到尾巴才能開始。

兩種做法：先把兩條 list 反轉再加，或者用 stack 把數字倒過來。都比原題多一步。

這就是題目特地說「倒序存放」的原因。它已經幫你排好了加法的順序。

---

## 結論

不要想著轉數字。就模擬小學加法：逐位相加，處理進位。Dummy node 讓你不用煩第一個 node。
