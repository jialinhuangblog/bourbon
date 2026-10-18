你站在某個樹節點 `target`，問距離你**恰好** K 步的所有節點是誰。距離 = 走過幾條邊。

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

target=5, K=2。答案 [7, 4, 1]：
- 7、4 在 5 的子樹（5→2→7、5→2→4，兩步）
- 1 在 5 的另一邊（5→3→1，兩步）

5 的子樹好辦：DFS 走兩層就抓到 7、4。但 1 怎麼辦？1 在 root 那邊，從 5 要走到 1 必須**經過 root 3**。Tree 沒有「往上的邊」，得自己補。

---

**解題引導**

用 target=5, K=2 想。

**Step 1：5 的子樹找距離 2 簡單，DFS 走兩層就好。但要走到 1，問題在哪？**

*Tree only points downward.*

<span class="spoiler">tree 預設只有 left / right，沒有 parent。從 5 要到 1 必須經過 3（5 的祖先），但沒辦法從 5 走到 3。tree 缺一條「往上的邊」。 / Tree only has left / right, no parent. To walk from 5 to 1 you must pass through 3 (5's ancestor), but the tree gives no way to walk upward. The "up edge" is missing.</span>

**Step 2：怎麼補出往上的邊？**

*Add it ourselves before searching.*

<span class="spoiler">先 DFS 一遍整棵 tree，建一個 parent map：node → parent。然後 tree 就變成 undirected graph，每個 node 有最多三條邊：left、right、parent。 / DFS once to build a parent map: node → parent. Now the tree behaves like an undirected graph with up to three edges per node: left, right, parent.</span>

**Step 3：在 graph 上找距離恰好 K 的所有點，用什麼？**

*Layer by layer.*

<span class="spoiler">BFS。從 target 出發每層擴散一次，第 K 層就是答案。要 visited set 避免回頭走（不然會無限振盪）。 / BFS. Start from target, expand one layer at a time; the K-th layer is the answer. Need a visited set to avoid bouncing back and forth.</span>

想完再看 code。

---

## 解法：建 parent map 後 BFS

```typescript
function distanceK(root: TreeNode | null, target: TreeNode, k: number): number[] {
  // Step 1: DFS 一遍建 parent map
  const parent = new Map<TreeNode, TreeNode | null>();
  const buildParent = (node: TreeNode | null, par: TreeNode | null) => {
    if (!node) return;
    parent.set(node, par);
    buildParent(node.left, node);
    buildParent(node.right, node);
  };
  buildParent(root, null);

  // Step 2: BFS from target，擴散 K 層
  const visited = new Set<TreeNode>([target]);
  let layer: TreeNode[] = [target];
  for (let i = 0; i < k; i++) {
    const next: TreeNode[] = [];
    for (const node of layer) {
      // 三條邊：left、right、parent
      for (const nb of [node.left, node.right, parent.get(node) ?? null]) {
        if (nb && !visited.has(nb)) {
          visited.add(nb);
          next.push(nb);
        }
      }
    }
    layer = next;
  }
  return layer.map(n => n.val);
}
```

- Time: O(n) — DFS 一次 + BFS 最多訪問每個節點一次
- Space: O(n) — parent map + visited set

<details>
<summary>Go 版本</summary>

```go
func distanceK(root *TreeNode, target *TreeNode, k int) []int {
    parent := map[*TreeNode]*TreeNode{}
    var build func(node, par *TreeNode)
    build = func(node, par *TreeNode) {
        if node == nil { return }
        parent[node] = par
        build(node.Left, node)
        build(node.Right, node)
    }
    build(root, nil)

    visited := map[*TreeNode]bool{target: true}
    layer := []*TreeNode{target}
    for i := 0; i < k; i++ {
        next := []*TreeNode{}
        for _, node := range layer {
            for _, nb := range []*TreeNode{node.Left, node.Right, parent[node]} {
                if nb != nil && !visited[nb] {
                    visited[nb] = true
                    next = append(next, nb)
                }
            }
        }
        layer = next
    }
    res := make([]int, 0, len(layer))
    for _, n := range layer { res = append(res, n.Val) }
    return res
}
```

</details>

N=500 的 constraint 很寬，這解法綽綽有餘（$5 \times 10^2$ 操作，<1ms）。

---

**走一遍 — target=5, K=2**

樹：
```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

Step 1 後的 parent map：
```
3 → null
5 → 3,  1 → 3
6 → 5,  2 → 5
0 → 1,  8 → 1
7 → 2,  4 → 2
```

Step 2 BFS：

```
L0: [5]                   visited={5}
L1: 從 5 看三個鄰居 left=6, right=2, parent=3
    全都沒 visit 過
    [6, 2, 3]              visited={5,6,2,3}
L2: 從 6 看：left=null, right=null, parent=5（visited）→ 沒新東西
    從 2 看：left=7, right=4, parent=5（visited）→ 加 7, 4
    從 3 看：left=5（visited）, right=1, parent=null → 加 1
    [7, 4, 1]              visited={5,6,2,3,7,4,1}
```

return [7, 4, 1]。

關鍵看 L2 那行 — 從 3 出發時 left=5 被 visited 擋下，沒有走回頭。沒有 visited set 的話，3 會把 5 又加進來，5 又把它的鄰居再加一次，整個 BFS 會在原地跳。

---

**為什麼用 BFS 不用 DFS？**

DFS 也行，但要傳「現在離 target 多遠」這個參數，而且 tree 變 graph 後 DFS 容易重複走、邏輯複雜。

BFS 天然按距離分層，第 K 層直接收割。問「距離恰好 K」這種題，BFS 是最直接的工具。

---

**跟 [99001 Lock Binary Tree](/problem/lock-binary-tree) 的關聯**

兩題核心都是「**tree 預設沒往上的邊，要往上走得自己補**」。

- 99001 自己設計 Node 類別，可以直接加 `parent` 欄位。
- 0863 不能改題目給的 TreeNode，所以用 hashmap 從外面補：`Map<Node, Node>`。

補完之後 tree 就變 undirected graph：
- 99001 用「往上走檢查祖先」處理 lock 規則
- 0863 用「往上下走擴散 K 步」處理距離查詢

也跟 [1650 LCA III](/problem/lowest-common-ancestor-of-a-binary-tree-iii) 同源 — 1650 題目直接給 parent pointer，省了補 map 這一步。三題練的是同一個直覺：**tree 加 parent 邊就是 graph，graph 上任意走**。

---

## 結論

**Tree 缺往上的邊，DFS 一遍補出 parent map，剩下就是 graph 上的 BFS**。Visited set 防止回頭振盪，第 K 層收割答案。
