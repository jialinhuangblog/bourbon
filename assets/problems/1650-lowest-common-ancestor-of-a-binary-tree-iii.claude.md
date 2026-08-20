兩個人在大樓裡走樓梯找彼此的最低共同主管。每人手上有「往上走」的指引（parent pointer），但沒看到全公司架構圖（拿不到 root）。

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

p=7, q=8。兩人各自往上走，路線會在某個節點合併，那就是 LCA。本題答案 3。

跟 [0236 LCA](/problem/lowest-common-ancestor-of-a-binary-tree) 的差別：236 從 root 出發 DFS 找；這題沒給 root，但給每個 node 一個 `.parent` 欄位。整個問題從「樹的搜尋」變成「兩條 linked list 的交點」。

---

**解題引導**

用 p=7, q=8 想。

**Step 1：每個節點都有 parent。從 p 往上走、從 q 往上走，會發生什麼？**

*Two paths upward from two starting points.*

<span class="spoiler">兩條路走到 root 的途中會在某個節點合併（LCA），合併後共用一條尾巴到 root。問題變成「兩條 linked list 第一次交叉的點在哪」，跟 [160 Intersection of Two Linked Lists](/problem/intersection-of-two-linked-lists) 是同一題。 / Two upward paths merge at some node (LCA) and share the same tail to root. Becomes "first intersection of two linked lists" — same as 160 Intersection of Two Linked Lists.</span>

**Step 2：HashSet 怎麼做？**

*Brute first.*

<span class="spoiler">把 p 整條祖先鏈塞進 set。q 往上走，第一個出現在 set 裡的就是 LCA。Time O(h)、Space O(h)。 / Push p's whole ancestor chain into a set. Walk q up, first one found in the set is LCA. Time O(h), Space O(h).</span>

**Step 3：要 O(1) space，set 不能用了。改用兩個指標各從 p、q 往上走 — 但兩條路徑長度不一樣，怎麼讓它們在 LCA 同時到？**

*The blocker: different starting depths.*

<span class="spoiler">直接同步走會錯位 — 深的那個還沒走完淺的就先到 root 了。要讓兩個指標走的「總距離」相等，就需要某種補償機制。 / Walking in lockstep desyncs — the deeper pointer is still mid-path when the shallower one hits root. We need both pointers to travel the same total distance somehow.</span>

**Step 4：補償怎麼做？提示 — 兩條路徑「合併」起來會發生什麼？**

*Each pointer walks both paths.*

<span class="spoiler">a 走完 p 那條後跳到 q 起點再走一次，b 走完 q 那條後跳到 p 起點再走一次。兩個指標的總路徑都是 (p→root) + (q→LCA) = (q→root) + (p→LCA) = 一樣長。第二輪某個點兩個指標必定踩到同一個節點，那就是 LCA。 / a finishes p's path then jumps to q's start; b finishes q's path then jumps to p's start. Both total paths are equal: (p→root) + (q→LCA) = (q→root) + (p→LCA). On the second pass they must meet at LCA.</span>

想完再看 code。

---

## 解法一：暴力（兩層迴圈）

最直白：p 的每個祖先，都拿 q 的整條祖先鏈比對一次，第一個對上的就是 LCA。

```typescript
function lowestCommonAncestor(p: Node | null, q: Node | null): Node | null {
  for (let a = p; a; a = a.parent) {        // p 往上走的每個節點
    for (let b = q; b; b = b.parent) {      // 都掃一遍 q 的祖先鏈
      if (a === b) return a;                // 第一個共同祖先就是答案
    }
  }
  return null;
}
```

- Time: $O(h^2)$
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func lowestCommonAncestor(p, q *Node) *Node {
    for a := p; a != nil; a = a.Parent {
        for b := q; b != nil; b = b.Parent {
            if a == b {
                return a
            }
        }
    }
    return nil
}
```

</details>

**兩層迴圈，最壞 $h \times h$。** 好處是完全不吃額外空間。下面兩個解各往一個方向優化 — 一個省時間，一個省空間。

---

## 解法二：HashSet 省時間

內層迴圈每次都重掃 q 的祖先鏈，這是浪費。先把 p 的祖先全塞進 set，q 往上走一次、每步 O(1) 查表。時間掉到 O(h)，代價是 O(h) 的 set。

```typescript
function lowestCommonAncestor(p: Node | null, q: Node | null): Node | null {
  const seen = new Set<Node>();
  for (let cur = p; cur; cur = cur.parent) seen.add(cur);
  for (let cur = q; cur; cur = cur.parent) {
    if (seen.has(cur)) return cur;
  }
  return null;
}
```

- Time: O(h)
- Space: O(h)

<details>
<summary>Go 版本</summary>

```go
func lowestCommonAncestor(p, q *Node) *Node {
    seen := map[*Node]bool{}
    for cur := p; cur != nil; cur = cur.Parent {
        seen[cur] = true
    }
    for cur := q; cur != nil; cur = cur.Parent {
        if seen[cur] {
            return cur
        }
    }
    return nil
}
```

</details>

寫得快、面試夠用。但有一個 follow-up 一定會問：「能不能 O(1) space？」

---

## 解法三：Two pointer 省空間

兩條路徑長度不一樣（p 比 q 更深一點），所以不能直接「同步走」。但**如果讓兩個指標走「p 路徑 + q 路徑」vs「q 路徑 + p 路徑」**，總長度一樣。第二輪在某個點兩個指標必定相遇 — 那個點就是 LCA。

```typescript
function lowestCommonAncestor(p: Node | null, q: Node | null): Node | null {
  let a = p, b = q;
  while (a !== b) {
    a = a === null ? q : a.parent;  // 走完自己的就接對方的起點
    b = b === null ? p : b.parent;
  }
  return a;
}
```

- Time: O(h)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func lowestCommonAncestor(p, q *Node) *Node {
    a, b := p, q
    for a != b {
        if a == nil {
            a = q
        } else {
            a = a.Parent
        }
        if b == nil {
            b = p
        } else {
            b = b.Parent
        }
    }
    return a
}
```

</details>

---

**為什麼第二輪一定相遇？**

先看一棵樹，`p = 7`、`q = 5`，答案是 `2`：

```
          1
         / \
        2   3
       / \
      4   5
     /
    6
   /
  7
```

兩個指標各自走的路：

- a：`7 → 6 → 4 → 2 → 1 → null → 5 → 2 → 1`
- b：`5 → 2 → 1 → null → 7 → 6 → 4 → 2 → 1`

都是 8 步。同時起跑、每回合各走一步，所以任何時刻兩人剩下的步數都一樣：

| step | a | b | 各剩幾步 |
|---:|---|---|---:|
| 0 | 7 | 5 | 8 |
| 1 | 6 | 2 | 7 |
| 2 | 4 | 1 | 6 |
| 3 | 2 | null | 5 |
| 4 | 1 | 7 | 4 |
| 5 | null | 6 | 3 |
| 6 | 5 | 4 | 2 |
| 7 | **2** | **2** | 1 |
| 8 | 1 | 1 | 0 |

兩條路的尾巴是共用的（`… → 2 → 1`）。每個節點只有一個 parent，所以誰走到 `2`，後面就只剩同一條路。剩餘步數又永遠相等，共用段上「剩 1 步」的位置只有 `2` 這一格，兩人只能同時站上去。step 7 相等，迴圈結束，回傳 `2`。

這不是誰追上誰。a 前三步走 `7 → 6 → 4`，b 前三步走 `5 → 2 → 1`，是兩條不同的路，只有尾巴接在一起。

長度為什麼會一樣，把 a 那條拆開數：

```
p → ... → root     lp 步     這裡是 7 → 6 → 4 → 2 → 1，4 步
root → null         1 步     root.parent 就是 null
null → q            1 步     指標從 null 歸位到對方的起點 q
q → ... → root     lq 步     5 → 2 → 1，2 步
                   ────────
                   lp + 2 + lq = 8
```

b 那條是 `lq + 2 + lp`，同樣 8 步。`+2` 少算一步（寫成 `lp + 1 + lq`）就會跟走查數出來的 hops 對不上。

沒有共同祖先時（兩棵不同的樹），兩人同時走到 `null`，`null === null` 成立，回傳 `null`。

---

**走一遍 — p=7, q=8**

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

路徑：
- 7 → 2 → 5 → 3 → null
- 8 → 1 → 3 → null

每行表示**該 iteration 結束後**的 a、b 狀態。code 是 `a = a===null ? q : a.parent`、`b = b===null ? p : b.parent`，所以 a 走到 null 跳到 q=8、b 走到 null 跳到 p=7。

| iter | a | b | 動作 |
|---|---|---|---|
| 0 (init) | 7 | 8 | 起點 |
| 1 | 2 | 1 | 各走 parent |
| 2 | 5 | 3 | 各走 parent |
| 3 | 3 | null | b 已到頂 |
| 4 | null | **7**（跳到 p）| a 也到頂、b 跳 |
| 5 | **8**（跳到 q）| 2 | a 跳、b 走 parent |
| 6 | 1 | 5 | 各走 parent |
| 7 | 3 | 3 ✓ | 對齊，return 3 |

兩個指標各走 7 hops。a 的總路徑：7→2→5→3→null→8→1→3。b 的總路徑：8→1→3→null→7→2→5→3。兩條等長，都在 LCA 收斂。

---

## 解法比較表

| 解法 | Time | Space | 備註 |
|---|---|---|---|
| 暴力（兩層迴圈） | $O(h^2)$ | O(1) | 最直白、不吃額外空間 |
| HashSet | O(h) | O(h) | 寫得快、面試先寫這個 |
| Two pointer | O(h) | O(1) | follow-up 想看的 |

**跟 [99001 Lock Binary Tree](/problem/lock-binary-tree) 的關聯**

兩題都圍繞同一個 pattern：**有 parent pointer 就能往 root 走**。

- 99001：lock 時走祖先鏈檢查有沒有人鎖。
- 1650：兩個節點各走祖先鏈找交點。

差別在 99001 自己設計類別所以可以加 `parent` 欄位，1650 題目本來就給。但底層動作一樣 —「往上走」是 O(h) 的便宜操作。

---

## 結論

**兩條長度不一的路徑要對齊，讓兩個指標都把對方的路徑也走一遍**，總距離自然相等。LCA 在第二輪某個點同時被兩個指標踩到。
