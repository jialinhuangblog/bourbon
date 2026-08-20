給一棵二元樹，一層一層從左到右收集節點值，回傳 `[[第0層], [第1層], ...]`。

---

## 解法一：DFS

DFS 可以做。遞迴的時候帶著「現在在第幾層」，把值塞進對應的 slice。

```typescript
function levelOrder(root: TreeNode | null): number[][] {
    const result: number[][] = [];
    function dfs(node: TreeNode | null, depth: number): void {
        if (!node) return;
        if (depth === result.length) {     // 第一個到這層的節點，開一個新 array
            result.push([]);
        }
        result[depth].push(node.val);
        dfs(node.left, depth + 1);
        dfs(node.right, depth + 1);
    }
    dfs(root, 0);
    return result;
}
```

- Time: O(n)
- Space: O(n)（result + 遞迴 call stack 最深 O(h)）

<details>
<summary>Go 版本</summary>

```go
func levelOrder(root *TreeNode) [][]int {
    var result [][]int
    var dfs func(node *TreeNode, depth int)
    dfs = func(node *TreeNode, depth int) {
        if node == nil {
            return
        }
        if depth == len(result) {          // 第一個到這層的節點，開一個新 slice
            result = append(result, []int{})
        }
        result[depth] = append(result[depth], node.Val)
        dfs(node.Left, depth+1)
        dfs(node.Right, depth+1)
    }
    dfs(root, 0)
    return result
}
```

</details>

能跑。但這是 DFS 硬模擬 BFS 的行為。遞迴順序是先往深走，不是一層一層掃。結果正確，但過程不直覺。

---

## 解法二：BFS

「一層一層」——這就是 BFS。用 queue。

每輪把 queue 裡「這一層」的節點全部拿出來，收集值，同時把下一層的小孩全部放進去。每輪開始前先記住 queue 長度，才知道哪些是「這一層」的。

```typescript
function levelOrder(root: TreeNode | null): number[][] {
    if (!root) return [];
    const result: number[][] = [];
    const queue: TreeNode[] = [root];
    while (queue.length > 0) {
        const size = queue.length;     // 這一層有幾個
        const level: number[] = [];
        for (let i = 0; i < size; i++) {
            const node = queue.shift()!; // 拿出來
            level.push(node.val);
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        result.push(level);
    }
    return result;
}
```

- Time: O(n)
- Space: O(n)（queue 最寬那層最多 n/2 個節點）

<details>
<summary>Go 版本</summary>

```go
func levelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    var result [][]int
    queue := []*TreeNode{root}
    for len(queue) > 0 {
        size := len(queue)             // 這一層有幾個
        level := make([]int, 0, size)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]          // 拿出來
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
    return result
}
```

</details>

複雜度一樣，但 BFS 的思路跟題意完全對齊：一層一層掃，每層收集。不用額外帶 depth 參數。

---

**DFS vs BFS 怎麼選？**

兩個都 O(n)。差別在哪？

DFS 的 space 取決於樹高 h。skewed tree h = n，balanced tree h = log n。但 result 本身就要 O(n)，所以 call stack 的差異被蓋過去了。

BFS 的 queue 最大值是最寬那層。complete binary tree 最後一層有 n/2 個節點。所以 queue 最大也是 O(n)。

實際差異不大。但面試官問 level order，用 BFS 更直覺，不需要解釋「我用 DFS 但帶 depth 模擬 BFS」。

---

## 結論

Level order = BFS。queue 一輪處理一層，`size` 鎖住邊界。DFS 也能做，但 BFS 跟題意天然對齊。
