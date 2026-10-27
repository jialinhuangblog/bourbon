園丁修樹有一條規定：任何一個分岔點，左右兩邊的高度差不能超過一層。

只看整棵樹的最外圍不算數，每個分岔點都要合格。

```
合格：
        3
       / \
      9  20         9 那邊高 1，20 那邊高 2，差 1
        /  \
       15   7

不合格：
        1
       / \
      2   2         節點 1 的左邊高 3、右邊高 1，差 2
     / \
    3   3
   / \
  4   4
```

翻成 code 的講法：給一棵二元樹，判斷是不是 height-balanced，也就是每個節點的左右子樹高度差都在 1 以內。

---

**解題引導**

拿上面那棵不合格的 `[1,2,2,3,3,null,null,4,4]` 想。

**Step 1：這條規定是在管誰？**

*not just the root. every single node.*

<span class="spoiler">每一個節點都要檢查，不是只看 root。有一個節點不合格，整棵樹就不合格。Every node has to pass, not only the top one.</span>

**Step 2：檢查某個節點合不合格，需要知道什麼？**

<span class="spoiler">它左子樹的高度跟右子樹的高度。兩個數字相減，絕對值大於 1 就不合格。Just the two subtree heights.</span>

**Step 3：所以照著寫，code 會長什麼樣？**

*one function that walks, another that measures.*

<span class="spoiler">一個函式算高度，另一個走訪每個節點、在每個節點上呼叫算高度的那個。可以動，但同一段樹被量了很多次。One pass to walk, a nested pass to measure the same nodes again.</span>

**Step 4：走訪的時候本來就會經過每個節點，高度能不能順便帶上來？**

<span class="spoiler">可以。回程的時候回傳高度。問題是還要回報「底下已經有節點不合格」，一個回傳值要載兩種資訊。挑一個高度不可能出現的值當記號，例如 -1。Reuse the return value: a height, or -1 meaning already broken.</span>

想完再往下看 code。

---

## 解法一：每個節點各自量一次高度

照 Step 3 直接翻譯。

```typescript
function isBalanced(root: TreeNode | null): boolean {
    function height(node: TreeNode | null): number {
        if (!node) return 0;
        return Math.max(height(node.left), height(node.right)) + 1;
    }

    if (!root) return true;
    if (Math.abs(height(root.left) - height(root.right)) > 1) return false;

    return isBalanced(root.left) && isBalanced(root.right);
}
```

- Time: $O(n \times h)$ — 每個節點都要重新量一遍自己的子樹
- Space: $O(h)$ — call stack

**走一遍。** 用不合格的那棵 `[1,2,2,3,3,null,null,4,4]`：

| 檢查的節點 | height(左) | height(右) | 差 | 判斷 |
|---|---|---|---|---|
| 1 | 3（2 → 3 → 4） | 1（右邊的 2 是葉子） | 2 | 不合格，直接回傳 false |

第一關就結束了。`isBalanced(root.left)` 那一行根本沒被執行。

**斜的樹反而快，這一點跟直覺相反。** 一條 5000 個節點的直線，root 的左邊高 4999、右邊高 0，差 4999，第一個檢查就回傳 false。實測只花 10,001 次呼叫。

真正花時間的是**合格**的樹，因為每一關都要通過，每一關都要重量一次高度。實測 n=5000 的完整二元樹，123,645 次呼叫。

**所以這一版是 $O(n \log n)$，不是 $O(n^2)$。** 常聽到的那個 $O(n^2)$ 得靠一棵又斜又能一路通過檢查的樹，而這種樹不存在：能全部通過檢查的樹，高度最多就是 $1.44 \log_2 n$ 左右。最接近這個上限的是 Fibonacci 樹，實測 4,180 個節點高度 17，想再高一層得準備 6,764 個節點。那棵 4,180 節點的樹跑出 104,781 次呼叫，比一趟版多 12.5 倍。

倍數不大，但這一版還有一個問題：它的兩個函式在做同一件事，而其中一個的結果被丟掉了。

<details>
<summary>Go 版本</summary>

```go
func isBalanced(root *TreeNode) bool {
    var height func(*TreeNode) int
    height = func(node *TreeNode) int {
        if node == nil {
            return 0
        }
        l := height(node.Left)
        r := height(node.Right)
        if l > r {
            return l + 1
        }
        return r + 1
    }

    if root == nil {
        return true
    }
    diff := height(root.Left) - height(root.Right)
    if diff > 1 || diff < -1 {
        return false
    }

    return isBalanced(root.Left) && isBalanced(root.Right)
}
```

</details>

---

## 解法二：一趟走完，用 -1 當壞掉的記號

`height` 已經走遍了整棵子樹，它其實有機會順便把「這底下有沒有人不合格」一起判斷完。剩下的問題是怎麼把兩件事塞進一個回傳值。

高度永遠是 0 或正整數，所以 -1 空著沒人用。拿它當「底下已經不合格了」的記號。

```typescript
function isBalanced(root: TreeNode | null): boolean {
    function height(node: TreeNode | null): number {
        if (!node) return 0;

        const left = height(node.left);
        if (left === -1) return -1;          // 左邊已經壞了，不用再看

        const right = height(node.right);
        if (right === -1) return -1;         // 右邊已經壞了

        if (Math.abs(left - right) > 1) return -1;   // 壞在我這一層

        return Math.max(left, right) + 1;
    }

    return height(root) !== -1;
}
```

- Time: $O(n)$ — 每個節點進去一次
- Space: $O(h)$ — call stack

**走一遍。** 同一棵不合格的 `[1,2,2,3,3,null,null,4,4]`，照後序（先左、再右、最後自己）：

| 進到哪個節點 | left | right | 判斷 | 回傳 |
|---|---|---|---|---|
| 4（左） | 0 | 0 | 差 0 | 1 |
| 4（右） | 0 | 0 | 差 0 | 1 |
| 3（左） | 1 | 1 | 差 0 | 2 |
| 3（右） | 0 | 0 | 差 0 | 1 |
| 2（左） | 2 | 1 | 差 1 | 3 |
| 2（右） | 0 | 0 | 差 0 | 1 |
| 1 | 3 | 1 | 差 2，不合格 | -1 |

root 收到 -1，回傳 false。

不合格的節點如果在深處，記號會一路往上傳，中間每一層的前兩行 `if` 都會攔下來直接回傳 -1，右子樹連走都不用走。

**這個 -1 是特殊值，不是高度。** 讀 code 的時候看到 `return -1` 要翻譯成「回報失敗」，不是「這棵子樹高度負一」。之所以敢這樣用，是因為高度的值域是非負整數，-1 落在值域外面，不會跟真正的高度混淆。有些語言會改用 tuple 回傳 `(高度, 是否合格)`，意思一樣，只是把兩件事分開放。

<details>
<summary>Go 版本</summary>

```go
func isBalanced(root *TreeNode) bool {
    var height func(*TreeNode) int
    height = func(node *TreeNode) int {
        if node == nil {
            return 0
        }

        left := height(node.Left)
        if left == -1 {
            return -1 // 左邊已經壞了，不用再看
        }

        right := height(node.Right)
        if right == -1 {
            return -1
        }

        diff := left - right
        if diff > 1 || diff < -1 {
            return -1 // 壞在我這一層
        }

        if left > right {
            return left + 1
        }
        return right + 1
    }

    return height(root) != -1
}
```

</details>

---

**Overthinking**

**跟 [543 Diameter](/problem/diameter-of-binary-tree) 的差別只在答案放哪裡。** 兩題都是「一趟後序走訪，回傳高度，順便做別的事」。543 把答案放在函式外面的變數，因為它要收集最大值；這題把答案塞進回傳值，因為它只需要回報成功或失敗。哪一種都行，判斷標準是那個附帶的資訊有沒有辦法用回傳值的值域裝下。

**題目說的 height-balanced 就是 AVL 樹的條件。** AVL 樹靠旋轉維持這個性質，好處是查詢的高度有保證。這題只做檢查，沒有要你修。順著問下去的話：既然合格的樹高度最多 $1.44 \log_2 n$，那 AVL 樹的查詢就是 $O(\log n)$，這就是它存在的理由。

**空樹是合格的。** 題目 Example 3 就是 `[]`，答案 true。兩個版本都靠 `if (!node) return 0` 處理掉，不用特別寫。

**這幾題的骨架對照表**放在 [104 Maximum Depth](/problem/maximum-depth-of-binary-tree) 的 Overthinking。

---

## 解法比較表

| 解法 | Time | Space | n=5000 完整樹 | n=4180 Fibonacci 樹 | 備註 |
|---|---|---|---|---|---|
| 每節點重量高度 | $O(n \log n)$ | $O(h)$ | 123,645 次，0.012 秒 | 104,781 次，0.010 秒 | 能過，但同一段樹量很多次 |
| 一趟走完 | $O(n)$ | $O(h)$ | 10,001 次，0.001 秒 | 8,361 次，0.0008 秒 | 面試要的答案 |

呼叫次數是實際跑計數器數出來的，秒數是次數除以 $10^7$。題目上限只有 5000 個節點，兩個版本都在 0.02 秒以內，樸素版不會 TLE。差 12 倍看得出來，但真正該講的是它的第二趟走訪完全是多餘的。

---

## 結論

每個節點都要檢查，所以一定得走遍全樹。走的時候順手把高度帶回來，發現不合格就回傳 -1 一路往上傳。一個回傳值載兩種意思，前提是特殊值落在正常值域外面。
