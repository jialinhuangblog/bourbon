公園裡有一堆涼亭，用步道接起來。步道的接法沒有繞圈，任兩個涼亭之間只有一條路可以走。

問最遠的那兩個涼亭，中間隔了幾段步道。

```
      1
     / \
    2   3
   / \
  4   5

最遠的一組是 4 跟 3
走法 4 → 2 → 1 → 3，經過 3 段步道
答案 3
```

數的是步道的段數，不是涼亭的個數。單獨一個涼亭的答案是 0，它哪裡都不用走。

翻成 code 的講法：給一棵二元樹，求任兩個節點之間最長路徑的邊數。

---

**解題引導**

拿上面那棵 `[1,2,3,4,5]` 想。

**Step 1：最長的那條路，長什麼樣子？**

*it goes up for a while, then turns around and goes down.*

<span class="spoiler">從一片葉子往上爬，爬到某個節點之後轉向，再往下走到另一片葉子。那個轉彎的節點，就是這條路的最高點。A path climbs to one node, turns, then descends.</span>

**Step 2：假設轉彎點是節點 X，已經知道 X 左邊最深 2 層、右邊最深 1 層，穿過 X 的最長路徑有幾段？**

<span class="spoiler">2 + 1 = 3 段。左邊往下最多走 2 段，右邊往下最多走 1 段，接起來就是穿過 X 的最長路徑。Left depth plus right depth.</span>

**Step 3：那答案是不是就是 root 的左深加右深？**

*not every path bothers to pass through the top.*

<span class="spoiler">不是。轉彎點可能不是 root。每個節點都有機會當轉彎點，所以每個節點都要算一次，取最大的。The turning point can be any node, not just the root.</span>

**Step 4：每個節點都算一次自己的左深右深，會不會重複算？**

<span class="spoiler">會，而且很嚴重。節點 X 算高度的時候走過它整棵子樹，X 的小孩再算一次又走一遍。一條 10^4 個節點的直線會走到 10^8 次。要改成回程的時候順手把高度帶上來。Recomputing height at every node repeats the same walk.</span>

想完再往下看 code。

---

## 解法一：每個節點各自算一次高度

照 Step 2 直接翻譯。寫一個 `height` 算子樹高度，然後走訪每個節點，在每個節點上問一次「穿過我的路有多長」。

```typescript
function diameterOfBinaryTree(root: TreeNode | null): number {
    let best = 0;

    function height(node: TreeNode | null): number {
        if (!node) return 0;
        return Math.max(height(node.left), height(node.right)) + 1;
    }

    function visit(node: TreeNode | null): void {
        if (!node) return;
        best = Math.max(best, height(node.left) + height(node.right)); // 穿過我的最長路
        visit(node.left);
        visit(node.right);
    }

    visit(root);
    return best;
}
```

- Time: $O(n \times h)$ — 每個節點都要重新走一遍自己的子樹
- Space: $O(h)$ — call stack

`height` 回傳的是節點數。`height(node.left)` 等於「從 node 走到左邊最深的葉子」要經過幾段步道，因為多出來的那一段剛好補上 node 到 node.left 的連線。所以兩邊相加就是邊數，不用再調整。

**走一遍。** `[1,2,3,4,5]`，`visit` 照 root、左、右的順序：

| visit 的節點 | height(左) | height(右) | 相加 | best |
|---|---|---|---|---|
| 1 | 2（2 底下還有 4） | 1（3 是葉子） | 3 | 3 |
| 2 | 1（4） | 1（5） | 2 | 3 |
| 4 | 0 | 0 | 0 | 3 |
| 5 | 0 | 0 | 0 | 3 |
| 3 | 0 | 0 | 0 | 3 |

答案 3。

**問題是重複走。** `visit(1)` 呼叫 `height(2)`，那一趟把 2、4、5 全部走過。接著 `visit(2)` 又呼叫 `height(4)` 跟 `height(5)`，剛剛走過的再走一次。

樹是完整的時候高度只有 14，重複的代價還不算大：n=10000 實測 267,263 次呼叫，換算 0.03 秒。但題目允許一條 10^4 個節點的直線，那時候每個節點的 `height` 都要掃剩下的整條線。同一份 code 實測跑出 100,030,001 次呼叫，換算 10 秒，明確 TLE。

<details>
<summary>Go 版本</summary>

```go
func diameterOfBinaryTree(root *TreeNode) int {
    best := 0

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

    var visit func(*TreeNode)
    visit = func(node *TreeNode) {
        if node == nil {
            return
        }
        if h := height(node.Left) + height(node.Right); h > best {
            best = h // 穿過我的最長路
        }
        visit(node.Left)
        visit(node.Right)
    }

    visit(root)
    return best
}
```

</details>

---

## 解法二：一趟走完，回程順手更新答案

`height` 已經在算高度了，而 `visit` 要的也是高度。兩個函式合成一個：往下走的時候什麼都不做，回程帶著高度上來，經過每個節點的時候順手比一下答案。

```typescript
function diameterOfBinaryTree(root: TreeNode | null): number {
    let best = 0;

    function height(node: TreeNode | null): number {
        if (!node) return 0;

        const left = height(node.left);
        const right = height(node.right);
        best = Math.max(best, left + right);  // 我當轉彎點的話，路有這麼長

        return Math.max(left, right) + 1;     // 回傳給上面的還是高度
    }

    height(root);
    return best;
}
```

- Time: $O(n)$ — 每個節點進去一次
- Space: $O(h)$ — call stack

這個函式回傳的跟收集的是兩個不同的東西。回傳值是高度，給呼叫我的那個節點用；`best` 是答案，放在外面，每經過一個節點更新一次。

**走一遍。** 同一棵 `[1,2,3,4,5]`，照後序的順序（先左、再右、最後自己）：

| 進到哪個節點 | left | right | left + right | best 更新成 | 回傳 |
|---|---|---|---|---|---|
| 4 | 0 | 0 | 0 | 0 | 1 |
| 5 | 0 | 0 | 0 | 0 | 1 |
| 2 | 1 | 1 | 2 | 2 | 2 |
| 3 | 0 | 0 | 0 | 2 | 1 |
| 1 | 2 | 1 | 3 | 3 | 3 |

最後 `height(root)` 回傳 3，但那是巧合，答案要看 `best`。把 root 的右子樹 3 拔掉再跑一次，`best` 會是 2（路徑 4 → 2 → 5），而 `height(root)` 回傳 3。回傳值跟答案在那個例子就分家了。

<details>
<summary>Go 版本</summary>

```go
func diameterOfBinaryTree(root *TreeNode) int {
    best := 0

    var height func(*TreeNode) int
    height = func(node *TreeNode) int {
        if node == nil {
            return 0
        }

        left := height(node.Left)
        right := height(node.Right)
        if left+right > best {
            best = left + right // 我當轉彎點的話，路有這麼長
        }

        if left > right {
            return left + 1
        }
        return right + 1
    }

    height(root)
    return best
}
```

</details>

---

**Overthinking**

**答案不在回傳值裡，這件事值得單獨記。** [104 Maximum Depth](/problem/maximum-depth-of-binary-tree) 的答案就是 root 的回傳值，看完 code 就懂。這題不是。遞迴回傳高度，答案卻是回程路上收集的最大值，兩者只是共用同一趟走訪。同一族的還有「最大路徑和」那類題，都是這個結構：帶一個值上去給父節點，另外拿一個變數在旁邊記全域最好的。

**為什麼回傳的不能直接是 `left + right`。** 因為父節點要的不是「穿過我的路徑長」，而是「從我往下能走多深」。路徑到了我這裡就轉彎了，不能再往上接。硬把 `left + right` 回傳上去，父節點會以為自己底下有一條那麼深的單邊路，答案會偏大。

**跟 [110 Balanced Binary Tree](/problem/balanced-binary-tree) 是同一組。** 110 也是先寫一個每節點重算高度的版本，再合併成一趟。差別在 110 把「不平衡」塞進回傳值當特殊記號（回傳 -1），這題則是另外開一個變數。兩種都是「一趟走完順便做別的事」，兩題連著寫最有感。

**這幾題的骨架對照表**放在 [104](/problem/maximum-depth-of-binary-tree) 的 Overthinking。

---

## 解法比較表

| 解法 | Time | Space | n=10^4 完整樹 | n=10^4 直線樹 | 備註 |
|---|---|---|---|---|---|
| 每節點重算高度 | $O(n \times h)$ | $O(h)$ | 267,263 次，0.03 秒 | 100,030,001 次，10 秒 | 直線樹 TLE |
| 一趟走完 | $O(n)$ | $O(h)$ | 20,001 次，0.002 秒 | 20,001 次，0.002 秒 | 面試要的答案 |

呼叫次數是實際跑計數器數出來的，秒數是次數除以 $10^7$。一趟走完的版本兩種樹的次數一模一樣，因為它只跟節點數有關，樹長什麼樣都沒差。

---

## 結論

最長路徑會在某個節點轉彎，那個節點的左深加右深就是這條路的長度。一趟後序走訪，回傳高度給上面、順手把最大值記在外面，答案就出來了。
