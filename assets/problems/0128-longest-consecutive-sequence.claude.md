給一個亂序陣列，找最長的連續數字序列有幾個（例如 1,2,3,4 → 4）。

---

**解題引導**

用 `nums = [100, 4, 200, 1, 3, 2]` 思考。答案是 4（序列 1,2,3,4）。

**Step 1：暴力怎麼做？**

*Sort, then scan. Simple. But not O(n).*

<span class="spoiler">排序後掃一遍，碰到斷點就重設計數。O(n log n)，但題目要求 O(n)，不合格。 / Sort and scan, reset count at gaps. O(n log n) — too slow, the problem requires O(n).</span>

**Step 2：為什麼 O(n log n) 不夠？關鍵限制是什麼？**

*The constraint says O(n). That means no sorting. What structure gives O(1) lookup?*

<span class="spoiler">排序本身就是 O(n log n) 的下限。O(n) 代表不能排序，查詢又要快 → hashset，O(1) lookup。 / Sorting is inherently O(n log n). O(n) means no sort allowed — use a hashset for O(1) lookup.</span>

**Step 3：把所有數字放進 set 後，怎麼找連續序列的起點？**

*You don't want to count the same sequence multiple times.*

<span class="spoiler">只從「序列起點」開始數。起點的定義：num-1 不在 set 裡。從起點開始一路 +1 往右數，直到斷掉。 / Only start counting from sequence heads. A head is a number where num-1 is not in the set. Count upward until the chain breaks.</span>

**Step 4：如果不過濾起點，會怎樣？**

*Every number becomes a potential start. What's the worst case?*

<span class="spoiler">每個數字都嘗試往右數，序列裡的每個數都跑一遍，變成 O(n²)。只從起點出發，每個數最多被碰到一次，O(n)。 / Without filtering, each number tries to extend right — O(n²) in the worst case. Filtering to heads ensures each number is visited at most once — O(n).</span>

想完再往下看 code。

---

## 解法一：排序

先排序版（intuitive，但 O(n log n)）：

```typescript
function longestConsecutive(nums: number[]): number {
    if (nums.length === 0) return 0;
    nums.sort((a, b) => a - b);
    let best = 1, cur = 1;  // 非空至少有長度 1；從 0 起跳的話 [5] 會回傳 0（迴圈根本不會跑）
    for (let i = 1; i < nums.length; i++) {
        if (nums[i] === nums[i - 1] + 1) {
            cur++;
        } else if (nums[i] !== nums[i - 1]) {  // 相等就跳過（重複數字）
            cur = 1;
        }
        if (cur > best) best = cur;
    }
    return best;
}
```

<details>
<summary>Go 版本</summary>

```go
func longestConsecutive(nums []int) int {
    if len(nums) == 0 { return 0 }
    sort.Ints(nums)
    best, cur := 1, 1  // 非空至少有長度 1；從 0 起跳的話 [5] 會回傳 0（迴圈根本不會跑）
    for i := 1; i < len(nums); i++ {
        if nums[i] == nums[i-1]+1 {
            cur++
        } else if nums[i] != nums[i-1] {  // 相等就跳過（重複數字）
            cur = 1
        }
        if cur > best { best = cur }
    }
    return best
}
```

</details>

走 `[100, 4, 200, 1, 3, 2]` 排序後 → `[1, 2, 3, 4, 100, 200]`：

| i | nums[i] | cur | best |
|---|---------|-----|------|
| 1 | 2 | 2 | 2 |
| 2 | 3 | 3 | 3 |
| 3 | 4 | 4 | 4 |
| 4 | 100 | 1 | 4 |
| 5 | 200 | 1 | 4 |

答案 4。但 O(n log n)，不合題目要求。

- Time: **O(n log n)**
- Space: O(1)

---

## 解法二：HashSet

想像一排人站著等入場，每個人都有號碼牌（不照順序排）。你要找最長的「連號」群組。

笨方法：把大家排好序再掃。O(n log n)。

聰明方法：**只找「前面沒人」的那位問「你後面有幾個連號的？」** — 也就是只從隊首問，不問中間的人。

怎麼認出隊首？他的號碼 -1 根本不存在（沒有人拿那個號）。

這樣每個號碼最多被問到一次，O(n)。

對應到 code：數字全丟進 hashset，`num-1` 不在 set 裡的就是隊首，從它開始一路 +1 往右數。

```typescript
function longestConsecutive(nums: number[]): number {
    const set = new Set(nums);
    let best = 0;

    for (const n of set) {
        if (set.has(n - 1)) continue;   // 不是起點，跳過

        let cur = 1;
        while (set.has(n + cur)) cur++;  // 往右數
        best = Math.max(best, cur);
    }
    return best;
}
```

<details>
<summary>Go 版本</summary>

```go
func longestConsecutive(nums []int) int {
    set := make(map[int]bool)
    for _, n := range nums {
        set[n] = true
    }

    best := 0
    for n := range set {
        if set[n-1] {
            continue    // 不是起點，跳過
        }
        cur := 1
        for set[n+cur] {
            cur++       // 往右數，只要下一個存在就繼續
        }
        if cur > best { best = cur }
    }
    return best
}
```

</details>

走 `[100, 4, 200, 1, 3, 2]`，set = `{100, 4, 200, 1, 3, 2}`：

| n | n-1 在 set？ | 是起點？ | 往右數 | cur |
|---|------------|---------|--------|-----|
| 100 | 99? 不在 | ✓ | 101? 不在 | 1 |
| 4 | 3? 在 | ✗ | 跳過 | — |
| 200 | 199? 不在 | ✓ | 201? 不在 | 1 |
| 1 | 0? 不在 | ✓ | 2→3→4→5? 不在 | **4** |
| 3 | 2? 在 | ✗ | 跳過 | — |
| 2 | 1? 在 | ✗ | 跳過 | — |

答案 4。

- Time: O(n) — 每個數最多進內層 loop 一次（n+1, n+2... 被查到時它們不是起點，外層早就跳過了）
- Space: O(n)

---

## 解法三：Union-Find

這題本質是「把相鄰的數字合成一組，問最大組有多大」。「連通塊」問題的經典訊號。

HashSet 解法的視角：**從起點往右一路數**。Union-Find 的視角：**把 n 跟 n-1、n+1 合起來，最後看最大組多大**。概念文章 [Union-Find](/concept/union-find) 的定義剛好是這題的結構。

```typescript
class UnionFind {
    parent = new Map<number, number>();
    size = new Map<number, number>();
    max = 0;

    constructor(nums: number[]) {
        for (const n of nums) {
            if (!this.parent.has(n)) {
                this.parent.set(n, n);
                this.size.set(n, 1);
                this.max = 1;
            }
        }
    }

    find(x: number): number {
        if (this.parent.get(x) !== x) {
            this.parent.set(x, this.find(this.parent.get(x)!));  // path compression
        }
        return this.parent.get(x)!;
    }

    union(x: number, y: number): void {
        let rx = this.find(x), ry = this.find(y);
        if (rx === ry) return;
        if (this.size.get(rx)! < this.size.get(ry)!) [rx, ry] = [ry, rx];
        this.parent.set(ry, rx);
        const newSize = this.size.get(rx)! + this.size.get(ry)!;
        this.size.set(rx, newSize);
        if (newSize > this.max) this.max = newSize;
    }
}

function longestConsecutive(nums: number[]): number {
    if (nums.length === 0) return 0;
    const uf = new UnionFind(nums);
    for (const n of nums) {
        if (uf.parent.has(n - 1)) uf.union(n, n - 1);
        if (uf.parent.has(n + 1)) uf.union(n, n + 1);
    }
    return uf.max;
}
```

<details>
<summary>Go 版本</summary>

```go
type UnionFind struct {
    parent map[int]int
    size   map[int]int
    max    int
}

func NewUnionFind(nums []int) *UnionFind {
    parent := make(map[int]int)
    size := make(map[int]int)
    maxSize := 0
    for _, n := range nums {
        if _, ok := parent[n]; !ok {  // 重複的數字只算一次
            parent[n] = n
            size[n] = 1
            maxSize = 1
        }
    }
    return &UnionFind{parent, size, maxSize}
}

func (uf *UnionFind) Find(x int) int {
    if uf.parent[x] != x {
        uf.parent[x] = uf.Find(uf.parent[x])  // path compression
    }
    return uf.parent[x]
}

func (uf *UnionFind) Union(x, y int) {
    rx, ry := uf.Find(x), uf.Find(y)
    if rx == ry { return }
    // union by size：小組掛到大組下
    if uf.size[rx] < uf.size[ry] { rx, ry = ry, rx }
    uf.parent[ry] = rx
    uf.size[rx] += uf.size[ry]
    if uf.size[rx] > uf.max { uf.max = uf.size[rx] }
}

func longestConsecutive(nums []int) int {
    if len(nums) == 0 { return 0 }
    uf := NewUnionFind(nums)
    for _, n := range nums {
        if _, ok := uf.parent[n-1]; ok { uf.Union(n, n-1) }
        if _, ok := uf.parent[n+1]; ok { uf.Union(n, n+1) }
    }
    return uf.max
}
```

</details>

走 `[100, 4, 200, 1, 3, 2]`：

```
Init：6 個獨立組，每組 size=1，max=1

處理 100：99? 沒。101? 沒。不合併。
處理 4：  3? 沒（還沒遇過）。5? 沒。不合併。
處理 200：199? 沒。201? 沒。不合併。
處理 1：  0? 沒。2? 沒。不合併。
處理 3：  2? 沒。4? 有 → union(3,4)，{3,4} size=2，max=2
處理 2：  1? 有 → union(2,1)，{1,2} size=2
         3? 有 → union(2,3)，{3,4} 併進 {1,2}（或反過來）
                            → {1,2,3,4} size=4，max=4
```

Map 裡的 parent 指標最後可能長這樣（path compression 之後）：

```
1 → 2     2 → 2     3 → 2     4 → 2       (全部指向 root = 2)
100 → 100                                   (獨立)
200 → 200                                   (獨立)
```

答案 = uf.max = 4。

- Time: $O(n \times \alpha(n))$，反 Ackermann ≤ 4，視為常數 → O(n)
- Space: O(n)

---

**HashSet 還是 UF？**

| | HashSet | Union-Find |
|---|---|---|
| 時間 | O(n) | $O(n \times \alpha(n))$，實際 ≈ O(n) |
| 空間 | O(n) | O(n) — parent + size 兩個 map |
| 程式碼 | 10 行 | 40+ 行 |
| 常數開銷 | 小 | 大（hashmap lookup × 3、遞迴 find） |
| 實際速度 | 快 | 慢 2-3 倍 |

這題 UF 沒比 HashSet 快，程式碼還複雜。**不是升級，是換視角**。

為什麼還值得看？因為「連通塊」這個結構會反覆出現。200 Number of Islands 的格子相連、684 Redundant Connection 的邊相連、547 Number of Provinces 的人相連——都是同一個骨架。把 128 也翻譯成 UF，之後遇到「誰跟誰一組」的題，UF 就是現成的選項。

---

## 結論

把所有數字丟進 set，只從「num-1 不在 set」的起點往右數。O(n log n) 的排序換成 O(1) 的 set lookup，整體降到 O(n)。
