兩個人各自憑記憶畫了一張家族樹，要對一下畫的是不是同一家。

名字對上還不夠，位置也得一樣。同樣是「阿公底下有一個兒子」，一個人畫在左邊、另一個畫在右邊，那就是兩張不同的圖。

```
p = [1,2,3]        q = [1,2,3]        一樣
    1                  1
   / \                / \
  2   3              2   3

p = [1,2]          q = [1,null,2]     不一樣，2 掛的邊不同
    1                  1
   /                    \
  2                      2

p = [1,2,1]        q = [1,1,2]        不一樣，值的位置對調了
    1                  1
   / \                / \
  2   1              1   2
```

翻成 code 的講法：給兩棵二元樹的 root，判斷結構跟每個節點的值是不是完全相同。

---

**解題引導**

拿第三組 `p = [1,2,1]`、`q = [1,1,2]` 想。

**Step 1：兩棵樹要一樣，root 要滿足什麼？**

*value first, then both sides.*

<span class="spoiler">值要相同，而且左子樹對左子樹一樣、右子樹對右子樹一樣。三個條件同時成立才算。Same value, and both sides match on their own side.</span>

**Step 2：左子樹對左子樹一樣，這句話怎麼判斷？**

<span class="spoiler">又是同一個問題，只是樹變小了。丟給同一個函式。The subproblem is literally the same function.</span>

**Step 3：走到底的時候有幾種情況？**

*two nulls agree. one null does not.*

<span class="spoiler">兩邊同時是 null，那就一樣，回傳 true。只有一邊是 null，結構就對不上了，回傳 false。兩邊都不是 null 才繼續比值。Two nulls match, one null does not.</span>

**Step 4：兩棵樹如果一大一小，最多要比幾個節點？**

<span class="spoiler">比較小的那棵先走完。只要有一邊先變成 null 而另一邊還在，馬上回傳 false。所以是 O(min(m, n))。The smaller tree bounds the work.</span>

想完再往下看 code。

---

## 解法一：遞迴，兩棵樹同步走

兩個指標一起往下移動，每一步都問同樣三件事：值一樣嗎、左邊一樣嗎、右邊一樣嗎。

```typescript
function isSameTree(p: TreeNode | null, q: TreeNode | null): boolean {
    if (!p || !q) return p === q;        // 至少一邊空：兩邊都空才算一樣
    if (p.val !== q.val) return false;   // 值就對不上了，底下不用看
    if (!isSameTree(p.left, q.left)) return false;
    return isSameTree(p.right, q.right);
}
```

- Time: $O(\min(m, n))$ — 小的那棵走完就停
- Space: $O(\min(h_p, h_q))$ — call stack 的深度

第一行的 `p === q` 是在比 reference。兩邊都是 `null` 時相等，回傳 true；一邊是節點、另一邊是 `null` 時不相等，回傳 false。兩種情況剛好被同一行處理掉。

**走一遍。** `p = [1,2,1]`、`q = [1,1,2]`：

```
isSameTree(p:1, q:1)
  值都是 1，繼續
  isSameTree(p:2, q:1)
    值 2 vs 1，不同 → false
  左邊回傳 false → 整個回傳 false
```

右子樹根本沒被走到。第三行一發現左邊不成立就直接 return，`p:1` 跟 `q:2` 那一對永遠不會被比較。

**走一遍，這次是相同的樹。** `p = [1,2,3]`、`q = [1,2,3]`：

```
isSameTree(1, 1)
  isSameTree(2, 2)
    isSameTree(null, null) → true
    isSameTree(null, null) → true
    → true
  isSameTree(3, 3)
    isSameTree(null, null) → true
    isSameTree(null, null) → true
    → true
  → true
```

3 個節點加上 4 個 null 配對，總共 7 次呼叫。實測 100 個節點的完整二元樹對自己，計數器數到 201 次，也就是 2n+1。

<details>
<summary>Go 版本</summary>

```go
func isSameTree(p *TreeNode, q *TreeNode) bool {
    if p == nil || q == nil {
        return p == q // 兩邊都是 nil 才算一樣
    }
    if p.Val != q.Val {
        return false
    }
    if !isSameTree(p.Left, q.Left) {
        return false
    }
    return isSameTree(p.Right, q.Right)
}
```

</details>

---

## 解法二：迭代，自己拿一個 stack

遞迴其實就是借用 call stack 存「還沒比完的那些配對」。自己開一個陣列存，就不用遞迴了。

```typescript
function isSameTree(p: TreeNode | null, q: TreeNode | null): boolean {
    const pending: [TreeNode | null, TreeNode | null][] = [[p, q]];

    while (pending.length > 0) {
        const [a, b] = pending.pop()!;
        if (!a && !b) continue;              // 兩邊都空，這一對過關
        if (!a || !b) return false;          // 只有一邊空，結構不同
        if (a.val !== b.val) return false;

        pending.push([a.left, b.left]);      // 待會再比左邊
        pending.push([a.right, b.right]);    // 待會再比右邊
    }

    return true;                             // 所有配對都比完了
}
```

- Time: $O(\min(m, n))$
- Space: $O(\min(m, n))$ — pending 裡面同時放的配對數

**走一遍。** `p = [1,2,3]`、`q = [1,2,3]`：

| 回合 | pop 出來的配對 | 判斷 | push 進去 | pending 剩下 |
|---|---|---|---|---|
| 1 | `(1, 1)` | 值相同 | `(2,2)`、`(3,3)` | `(2,2) (3,3)` |
| 2 | `(3, 3)` | 值相同 | 兩對 null | `(2,2)` 加 2 對 null |
| 3 | `(null, null)` | 都空，continue | 無 | 少一對 |
| 4 | `(null, null)` | 都空，continue | 無 | `(2,2)` |
| 5 | `(2, 2)` | 值相同 | 兩對 null | 2 對 null |
| 6 | `(null, null)` | 都空，continue | 無 | 1 對 null |
| 7 | `(null, null)` | 都空，continue | 無 | 空的 |
| 8 | 迴圈結束 | 回傳 true | | |

用 `pop()` 拿的是最後 push 進去的，所以先比右子樹。順序跟遞迴版相反，答案不受影響，因為每一對都要比、比的內容也跟順序無關。

<details>
<summary>Go 版本</summary>

```go
func isSameTree(p *TreeNode, q *TreeNode) bool {
    type pair struct{ a, b *TreeNode }
    pending := []pair{{p, q}}

    for len(pending) > 0 {
        cur := pending[len(pending)-1]
        pending = pending[:len(pending)-1] // Go 沒有 pop()，手動做兩步

        if cur.a == nil && cur.b == nil {
            continue
        }
        if cur.a == nil || cur.b == nil {
            return false
        }
        if cur.a.Val != cur.b.Val {
            return false
        }

        pending = append(pending, pair{cur.a.Left, cur.b.Left})
        pending = append(pending, pair{cur.a.Right, cur.b.Right})
    }

    return true
}
```

</details>

---

**Overthinking**

**這題是 [572 Subtree of Another Tree](/problem/subtree-of-another-tree) 的零件。** 572 問的是 subRoot 有沒有出現在 root 的某個位置底下。做法就是在 root 的每個節點上呼叫一次這題的 `isSameTree`，有一個回傳 true 就結束。先把這題寫熟，572 只是多包一層。

**為什麼不能只比中序走訪的結果。** 把兩棵樹都中序走訪成陣列再比較，看起來省事，但會誤判。`[1,2]` 跟 `[1,null,2]` 的中序結果分別是 `[2,1]` 跟 `[1,2]`，這組剛好分得出來；換成兩棵單邊樹 `[1,null,2]` 跟 `[2,1]`，中序都是 `[1,2]`，結構明明不同卻會被判成相同。走訪的序列丟掉了 null 的位置，要補回來就得把 null 也寫進序列，那就是 572 解法二在做的事。

**跟同一族的其他題。** 遞迴帶一個值上來、答案從回傳值拿，這個骨架的五題整理在 [104 Maximum Depth](/problem/maximum-depth-of-binary-tree) 的對照表。

---

## 解法比較表

| 解法 | Time | Space | n=100 操作次數 | 備註 |
|---|---|---|---|---|
| 遞迴 | $O(\min(m,n))$ | $O(h)$ | 201 次呼叫 | 四行，面試講這個 |
| 迭代 stack | $O(\min(m,n))$ | $O(\min(m,n))$ | 201 次 pop | 樹很斜也不會 RangeError |

題目上限只有 100 個節點，兩個都是 0.00002 秒等級，選哪個都行。迭代版的價值在於它示範了遞迴到底在存什麼：那個 pending 陣列裝的，就是 call stack 幫你記住的東西。

---

## 結論

值相同、左邊相同、右邊相同，三個都成立才是同一棵樹。兩個 null 算相同，一個 null 算不同，這一行是全題唯一會寫錯的地方。
