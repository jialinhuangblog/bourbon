給一棵二元樹，一層一層走，但方向交替：第 0 層左到右、第 1 層右到左、第 2 層左到右……回傳每層的節點值。

---

## 解法一：BFS 事後反轉

如果你做過 102 Level Order Traversal，這題就是在那之上加一個條件：奇數層要反過來。

最直覺的做法：先跑一遍標準 BFS level order，拿到每層的結果，再把奇數層 reverse。

```typescript
function zigzagLevelOrder(root: TreeNode | null): number[][] {
    if (!root) return [];
    const result: number[][] = [];
    const queue: TreeNode[] = [root];
    while (queue.length > 0) {
        const size = queue.length;
        const level: number[] = [];
        for (let i = 0; i < size; i++) {
            const node = queue.shift()!;
            level.push(node.val);
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        result.push(level);
    }
    // 回頭把奇數層反轉
    for (let i = 1; i < result.length; i += 2) {
        result[i].reverse();
    }
    return result;
}
```

- Time: O(n)（每個節點處理一次，reverse 加起來也是 O(n)）
- Space: O(n)

<details>
<summary>Go 版本</summary>

```go
func zigzagLevelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    var result [][]int
    queue := []*TreeNode{root}
    for len(queue) > 0 {
        size := len(queue)
        level := make([]int, 0, size)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            level = append(level, node.Val)
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        result = append(result, level)
    }
    // 回頭把奇數層反轉
    for i := 1; i < len(result); i += 2 {
        slices.Reverse(result[i])
    }
    return result
}
```

</details>

用這棵樹走一遍（之後三個解法都用它）：

```
        1
      /   \
     2     3
    / \     \
   4   5     7
```

```
標準 BFS 收集：[[1], [2,3], [4,5,7]]
反轉奇數層：    [[1], [3,2], [4,5,7]]   ← 只有第 1 層被 reverse
```

能過。但走了兩趟：先收集，再反轉。**能不能一趟搞定？**

---

## 解法二：BFS 一趟填位

反轉的本質是什麼？偶數層從左邊開始塞，奇數層從右邊開始塞。那我在 BFS 的過程中，直接控制「塞進 level 的位置」就好了。

偶數層：`level[i] = node.Val`——從左往右填。
奇數層：`level[size-1-i] = node.Val`——從右往左填。

queue 的入隊順序不變（永遠 left 再 right），只改收集順序。

```typescript
function zigzagLevelOrder(root: TreeNode | null): number[][] {
    if (!root) return [];
    const result: number[][] = [];
    const queue: TreeNode[] = [root];
    let leftToRight = true;
    while (queue.length > 0) {
        const size = queue.length;
        const level = new Array(size);
        for (let i = 0; i < size; i++) {
            const node = queue.shift()!;
            // 決定填入位置
            const idx = leftToRight ? i : size - 1 - i;
            level[idx] = node.val;
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        result.push(level);
        leftToRight = !leftToRight;
    }
    return result;
}
```

- Time: O(n)
- Space: O(n)

<details>
<summary>Go 版本</summary>

```go
func zigzagLevelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    var result [][]int
    queue := []*TreeNode{root}
    leftToRight := true
    for len(queue) > 0 {
        size := len(queue)
        level := make([]int, size)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            // 決定填入位置
            if leftToRight {
                level[i] = node.Val
            } else {
                level[size-1-i] = node.Val
            }
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        result = append(result, level)
        leftToRight = !leftToRight
    }
    return result
}
```

</details>

同一棵樹走一遍，重點看奇數層的 idx 怎麼算：

```
第 0 層  leftToRight=true   size=1
  i=0 拿到 1 → idx=0 → level=[1]                     入隊 2, 3

第 1 層  leftToRight=false  size=2
  i=0 拿到 2 → idx=2-1-0=1 → level=[ , 2]            入隊 4, 5
  i=1 拿到 3 → idx=2-1-1=0 → level=[3, 2]            入隊 7
                              ↑ 先到的 2 反而坐後面的位子

第 2 層  leftToRight=true   size=3
  i=0,1,2 拿到 4, 5, 7 → 依序填 idx 0, 1, 2 → [4, 5, 7]

result = [[1], [3, 2], [4, 5, 7]]
```

複雜度一樣，但只走一趟。省掉 reverse 那圈迴圈。

---

**用 deque 呢？**

有些解法用雙端佇列，奇數層從前面 pop、偶數層從後面 pop，同時交替 push 方向。能做，但邏輯容易繞暈。你要同時追蹤「pop 方向」和「push 子節點的順序」，而且 bug 很難 debug。

上面的解法只改一行——填入位置。queue 順序永遠不變。

---

## 解法三：DFS

BFS 順手是因為它天生一層一層走。**那 DFS 做得到嗎？** 做得到：帶著 depth 遞迴（跟 102 的 DFS 版一樣），每個節點知道自己在第幾層，把值歸進 `result[depth]`。歸檔方式有兩條路：一律 append、收集完把奇數層 reverse（解法一的策略搬過來）；或是照 depth 奇偶決定 append 還是 unshift 往前插。下面的 code 是第二條。

```typescript
function zigzagLevelOrder(root: TreeNode | null): number[][] {
    const result: number[][] = [];
    function dfs(node: TreeNode | null, depth: number): void {
        if (!node) return;
        if (depth === result.length) result.push([]);
        if (depth % 2 === 0) {
            result[depth].push(node.val);
        } else {
            result[depth].unshift(node.val); // 往前插
        }
        dfs(node.left, depth + 1);
        dfs(node.right, depth + 1);
    }
    dfs(root, 0);
    return result;
}
```

- Time: **最壞 $O(n^2)$**。偶數層的 push 是 O(1)，但奇數層的 unshift（Go 的 prepend 同理）每次要搬移該層已經放進去的元素，一層 k 個節點的總成本是 $1 + 2 + \cdots + k \approx k^2/2$。最寬的一層可能有接近 n/2 個節點，光那一層就是 $n^2$ 等級。
- Space: O(n)

<details>
<summary>Go 版本</summary>

```go
func zigzagLevelOrder(root *TreeNode) [][]int {
    var result [][]int
    var dfs func(node *TreeNode, depth int)
    dfs = func(node *TreeNode, depth int) {
        if node == nil {
            return
        }
        if depth == len(result) {
            result = append(result, []int{})
        }
        if depth%2 == 0 {
            result[depth] = append(result[depth], node.Val)
        } else {
            // prepend：往前插
            result[depth] = append([]int{node.Val}, result[depth]...)
        }
        dfs(node.Left, depth+1)
        dfs(node.Right, depth+1)
    }
    dfs(root, 0)
    return result
}
```

</details>

同一棵樹，DFS 的訪問順序是先左子樹到底再右子樹（1 → 2 → 4 → 5 → 3 → 7），跟層的順序交錯，但每個值都照 depth 歸位：

```
訪問 1（depth 0，偶）push    → [[1]]
訪問 2（depth 1，奇）unshift → [[1], [2]]
訪問 4（depth 2，偶）push    → [[1], [2], [4]]
訪問 5（depth 2，偶）push    → [[1], [2], [4,5]]
訪問 3（depth 1，奇）unshift → [[1], [3,2], [4,5]]      ← 3 插到 2 前面
訪問 7（depth 2，偶）push    → [[1], [3,2], [4,5,7]]
```

這題 n ≤ 2000，最壞大約 $2000^2 / 8 = 5 \times 10^5$ 次搬移，照樣瞬間跑完，LeetCode 上感覺不出來；但面試被追問複雜度時要答得出這層。想留 DFS 又想要 O(n)，就走開頭說的第一條路。

---

## 結論

102 的 BFS 模板，加一個 `leftToRight` flag 控制填入方向。queue 順序不變，只改 index。這題就是 level order 的一行變形。
