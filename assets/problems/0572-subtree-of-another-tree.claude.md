手上有兩張組織圖。大的那張是整間公司，小的那張是別人給你的一個小組。

問這個小組有沒有原封不動出現在公司裡。找的是「某個人，加上他底下所有的人」，不能只挑一部分來湊。

```
root = [3,4,5,1,2]        subRoot = [4,1,2]
      3                         4
     / \                       / \
    4   5                     1   2
   / \
  1   2

節點 4 加上它底下的 1 跟 2，跟 subRoot 一模一樣 → true
```

```
root = [3,4,5,1,2,null,null,null,null,0]
      3
     / \
    4   5
   / \
  1   2
     /
    0

節點 4 底下多了一個 0。subRoot 沒有它，所以 → false
```

翻成 code 的講法：給 root 跟 subRoot，判斷 root 裡有沒有某個節點，它連同所有子孫剛好等於 subRoot。

---

**解題引導**

拿第二組想，就是多了一個 0 的那棵。

**Step 1：為什麼多一個 0 就不算？**

*a subtree is a node and everything under it. no trimming.*

<span class="spoiler">subtree 的定義是「一個節點加上它全部的子孫」，不能中途剪掉。節點 4 底下有 1、2、0 四個人，subRoot 只有三個，數量就對不上。You take the whole thing or nothing.</span>

**Step 2：怎麼判斷從某個節點開始的子樹，跟 subRoot 一不一樣？**

<span class="spoiler">這就是 100 Same Tree。值相同、左邊相同、右邊相同。The check is exactly Same Tree.</span>

**Step 3：那要在幾個節點上試？**

<span class="spoiler">全部。走訪 root 的每個節點，每個都跟 subRoot 比一次，有一個成立就結束。Every node is a candidate.</span>

**Step 4：這樣要比幾次？有沒有辦法一次比完？**

*flatten both trees into strings, then it is just substring search.*

<span class="spoiler">m 個節點各比一次，每次最多比 n 個節點，就是 O(m × n)。想一次比完的話，把兩棵樹各自壓成一個字串，問題就變成「小字串有沒有出現在大字串裡」。Serialize both, then search.</span>

想完再往下看 code。

---

## 解法一：每個節點試一次

`isSameTree` 直接抄 [100 Same Tree](/problem/same-tree)，這題只是多包一層走訪。

```typescript
function isSubtree(root: TreeNode | null, subRoot: TreeNode | null): boolean {
    if (!root) return false;                       // 走到底都沒找到
    if (isSameTree(root, subRoot)) return true;    // 從我開始剛好對上

    return isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot);
}

function isSameTree(p: TreeNode | null, q: TreeNode | null): boolean {
    if (!p || !q) return p === q;                  // 兩邊都空才算一樣
    if (p.val !== q.val) return false;
    if (!isSameTree(p.left, q.left)) return false;
    return isSameTree(p.right, q.right);
}
```

- Time: $O(m \times n)$ — m 是 root 的節點數，n 是 subRoot 的
- Space: $O(h)$ — call stack

**走一遍。** 第二組，root 是 `[3,4,5,1,2,null,null,null,null,0]`、subRoot 是 `[4,1,2]`：

| 試哪個節點 | isSameTree 比到哪裡就停 | 結果 |
|---|---|---|
| 3 | 值 3 vs 4，第一步就不同 | false |
| 4 | 值 4 對上，1 對上，2 對上，然後 2 的左邊 root 有 0、subRoot 是 null | false |
| 1 | 值 1 vs 4 | false |
| 2 | 值 2 vs 4 | false |
| 0 | 值 0 vs 4 | false |
| 5 | 值 5 vs 4 | false |

全部試完，回傳 false。

節點 4 那一列是唯一比了好幾層才失敗的。其他節點的值跟 subRoot 的根對不上，第一行就結束。

**什麼時候會慢。** root 是一條 2000 個節點、值全是 1 的直線，subRoot 是 1000 個節點、值也幾乎全是 1 的直線。每個節點的值都對得上，`isSameTree` 每次都得往下比很多層才發現不對。實測 1,505,500 次呼叫，換算 0.15 秒。題目上限就是這個規模，還過得去，但要是把上限放大就不行了。

<details>
<summary>Go 版本</summary>

```go
func isSubtree(root *TreeNode, subRoot *TreeNode) bool {
    if root == nil {
        return false
    }
    if isSameTree(root, subRoot) {
        return true // 從我開始剛好對上
    }
    return isSubtree(root.Left, subRoot) || isSubtree(root.Right, subRoot)
}

func isSameTree(p *TreeNode, q *TreeNode) bool {
    if p == nil || q == nil {
        return p == q
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

## 解法二：兩棵樹各壓成一個字串

樹的結構其實可以完整寫成一行文字，只要 null 也寫進去。寫完之後，「subRoot 是不是 root 的子樹」就變成「小字串有沒有出現在大字串裡」。

```typescript
function isSubtree(root: TreeNode | null, subRoot: TreeNode | null): boolean {
    function serialize(node: TreeNode | null): string {
        if (!node) return '#';                                    // null 也要留下記號
        return `(${node.val}${serialize(node.left)}${serialize(node.right)})`;
    }

    return serialize(root).includes(serialize(subRoot));
}
```

- Time: $O(m + n)$ 產生字串，加上一次字串搜尋
- Space: $O(m + n)$ — 兩個字串

**走一遍。** 第一組，root `[3,4,5,1,2]`、subRoot `[4,1,2]`：

```
serialize(root)    = (3(4(1##)(2##))(5##))
serialize(subRoot) = (4(1##)(2##))

大字串從第 3 個字元開始，剛好就是小字串 → true
```

第二組多了一個 0：

```
serialize(root)    = (3(4(1##)(2(0##)#))(5##))
serialize(subRoot) = (4(1##)(2##))

2 後面變成 (0##)# 而不是 ##，找不到 → false
```

**括號跟 `#` 都不能省。** 兩個都是為了同一件事：讓每個節點在字串裡的邊界看得出來。

拿 root 只有一個節點 12、subRoot 只有一個節點 2 來看。省掉括號的話，root 壓成 `12##`、subRoot 壓成 `2##`，`"12##".includes("2##")` 回傳 true，但答案應該是 false。12 的個位數被當成一個獨立的節點讀了。加上括號變成 `(12##)` 跟 `(2##)`，`(2` 對不上 `12`，回傳 false。

`#` 的作用是記住 null 的位置。少了它，`[1,null,2]` 跟 `[1,2]` 會壓成同一個字串，結構不同卻判成相同。

**這個 `includes` 沒有保證線性。** JS 的字串搜尋在最壞情況下仍然是 $O(m \times n)$，只是實作用了跳躍策略，一般輸入很快。面試被追問「怎麼保證 $O(m+n)$」的話，答案是把 `includes` 換成 KMP。

<details>
<summary>Go 版本</summary>

```go
func isSubtree(root *TreeNode, subRoot *TreeNode) bool {
    var serialize func(*TreeNode) string
    serialize = func(node *TreeNode) string {
        if node == nil {
            return "#" // null 也要留下記號
        }
        return "(" + strconv.Itoa(node.Val) + serialize(node.Left) + serialize(node.Right) + ")"
    }

    return strings.Contains(serialize(root), serialize(subRoot))
}
```

</details>

---

**Overthinking**

**為什麼這裡的序列化不能用中序。** 中序走訪把 null 丟掉了，`[1,null,2]` 跟 `[2,1]` 的中序結果都是 `[1,2]`，結構明明不同。前序加上 null 記號才是唯一的：一棵樹壓成的字串只對應那一棵樹。這也是 297 Serialize and Deserialize Binary Tree 在做的事。

**還有一種做法是雜湊。** 對每棵子樹算一個 hash 值往上帶，root 走一趟就知道有沒有哪棵子樹的 hash 等於 subRoot 的。速度是 $O(m + n)$ 而且不用產生大字串，代價是要處理碰撞。面試講到這裡通常就夠了，實際寫的話字串版好驗證得多。

**這幾題的骨架對照表**放在 [104 Maximum Depth](/problem/maximum-depth-of-binary-tree) 的 Overthinking。這題是唯一一個把別題整個當零件用的。

---

## 解法比較表

| 解法 | Time | Space | m=2000 n=1000 直線樹 | 備註 |
|---|---|---|---|---|
| 每個節點試一次 | $O(m \times n)$ | $O(h)$ | 1,505,500 次，0.15 秒 | 好寫、好講，題目規模夠用 |
| 序列化加字串搜尋 | $O(m + n)$ 加搜尋 | $O(m + n)$ | 6,002 次走訪，字串長 8,001 | 快，但邊界符號寫錯就會誤判 |

次數是實際跑計數器數出來的，秒數是次數除以 $10^7$。直線樹是刻意挑的最壞情況：值全部相同，所以每個節點都得往下比很多層才會失敗。

---

## 結論

在 root 的每個節點上呼叫一次 Same Tree，是最直接也最好講的解。想要線性就把兩棵樹壓成前序字串，記得補 null 記號跟節點邊界，不然 12 會被讀成 1 和 2。
