[347 Top K Frequent Elements](/problem/top-k-frequent-elements) 的加強版。一樣找出現次數前 k 高的，但這題多一條排序規則：次數高的排前面，**次數一樣時，字母順序小的排前面**。

```
words = ["i","love","leetcode","i","love","coding"], k = 2

數次數：i:2, love:2, leetcode:1, coding:1
前 2 名都是次數 2：i 和 love
i 和 love 次數相同 → 比字母，"i" < "love"

答案 ["i", "love"]（順序不能反）
```

347 只要「找出」前 k 個，順序隨便。這題要「排好序」回傳，而且平手時的規則跟次數相反：次數要高的、字母要小的。這個「一個要大、一個要小」的雙重規則，是這題所有麻煩的來源。

---

**解題引導**

用 `words = ["i","love","i","coding"], k = 2` 想。

**Step 1：先做什麼？**

*count first, same as 347*

<span class="spoiler">數次數，跟 347 一樣。Count frequencies.</span>

**Step 2：怎麼同時滿足「次數高的前面、字母小的前面」？**

*a comparator: freq descending, then word ascending*

<span class="spoiler">排序時用兩段比較：先比次數（大的在前），平手再比字母（小的在前）。Sort by frequency desc, tie-break by word asc.</span>

**Step 3：follow-up 要 $O(n \log k)$，怎麼辦？**

*a size-k heap, but the tie-break flips*

<span class="spoiler">維護大小 k 的 heap。但淘汰規則要反過來想：淘汰次數低的、以及次數相同時字母大的。Size-k heap with an inverted tie-break.</span>

想完再往下看 code。

---

## 解法一：計數 + 自訂排序

數完次數，把 unique 的字用一個雙段 comparator 排序：先比次數（大的在前），次數相同比字母（小的在前）。取前 k 個。

用 `["i","love","leetcode","i","love","coding"], k=2` 走：

```
次數：{i:2, love:2, leetcode:1, coding:1}

排序（次數降、字母升）：
i(2) 和 love(2) 平手 → "i" < "love" → i 在前
leetcode(1) 和 coding(1) 平手 → "coding" < "leetcode"
結果 [i, love, coding, leetcode]

取前 2 → [i, love]
```

```typescript
function topKFrequent(words: string[], k: number): string[] {
    const count = new Map<string, number>();
    for (const w of words) count.set(w, (count.get(w) ?? 0) + 1);
    return [...count.keys()]
        .sort((a, b) => count.get(b)! - count.get(a)! || a.localeCompare(b))
        .slice(0, k);
}
```

comparator 裡的 `||` 是重點：`count.get(b)! - count.get(a)!` 是次數降序，等於 0（次數相同）時才輪到 `a.localeCompare(b)` 字母升序。

- Time: $O(n \log n)$ — 排序主導
- Space: $O(n)$

<details>
<summary>Go 版本</summary>

```go
func topKFrequent(words []string, k int) []string {
    count := map[string]int{}
    for _, w := range words {
        count[w]++
    }
    uniq := make([]string, 0, len(count))
    for w := range count {
        uniq = append(uniq, w)
    }
    sort.Slice(uniq, func(i, j int) bool {
        if count[uniq[i]] != count[uniq[j]] {
            return count[uniq[i]] > count[uniq[j]] // 次數高的在前
        }
        return uniq[i] < uniq[j] // 次數同，字母小的在前
    })
    return uniq[:k]
}
```

</details>

好寫好懂。但 follow-up 要 $O(n \log k)$，排序是 $O(n \log n)$，還差一步。

---

## 解法二：大小 k 的 heap

維護大小 k 的 min-heap，top 放「最該被淘汰」的那個。超過 k 個就把 top 丟掉，掃完剩下就是前 k 名。

這裡有個反直覺的點：**要找「最好的 k 個」，用的卻是 min-heap（top 最小）**。原因是 heap 只能從 top 拿走東西，你想丟的（最差的那個）就得待在 top 。留最好的 k 個 = 隨時準備丟掉當前最差的 = 讓最差的浮在頂端 = min-heap。大的沉在底部反而安全，不會被碰到。淘汰口在頂端，所以把淘汰對象擠上去。

淘汰規則要反著想。最終答案要「次數高、字母小」在前，那麼「最該淘汰」的就是次數低的；次數相同時，字母**大**的比較該淘汰（因為它排在後面）。這個 tie-break 方向跟解法一相反，是這題最容易寫錯的地方。

```
最終想要：次數高 → 前，字母小 → 前
top（先淘汰）：次數低，或 次數同時字母大
```

掃完 heap 裡是前 k 名，但 top 是裡面最差的。全部 pop 出來是「最差先出」，反轉一下才是「最好在前」。

用 `["i","love","leetcode","i","love","coding"], k=2` 走。heap 存成 array，top = index 0 = 最該淘汰：

```
count = {i:2, love:2, leetcode:1, coding:1}
掃 unique 的順序：i, love, leetcode, coding
淘汰規則：次數低的更該淘汰；次數同，字母大的更該淘汰

push i
  heap: [i]                          size 1 ≤ 2，留著

push love
  love, i 都 count 2，"love" > "i" → love 更該淘汰
  love 往上冒過 i，佔住 top
  heap: [love, i]                    size 2 ≤ 2，留著

push leetcode（count 1）
  leetcode count 1 < love count 2 → 更該淘汰，冒到 top
  heap: [leetcode, i, love]          size 3 > 2 → 丟 top
  丟掉 leetcode
  heap: [love, i]                    top 回到 love

push coding（count 1）
  coding count 1 < love count 2 → 更該淘汰，冒到 top
  heap: [coding, i, love]            size 3 > 2 → 丟 top
  丟掉 coding
  heap: [love, i]

掃完，heap = [love, i]。兩個都 count 2，top love 是這兩個裡較差的（字母大）。

倒出來，最差先出：
  pop love → res = [love]
  pop i    → res = [love, i]
res.reverse() → [i, love]
```

看 leetcode 和 coding：它們一 push 進去就因為 count 低冒到 top，下一步就被丟掉。heap 大小卡死在 2，比現有兩名還差的新人碰一下就被彈掉，永遠只留最好的 k 個。這就是為什麼是 $O(n \log k)$ 而不是把全部排序的 $O(n \log n)$：手上的量始終是 k。

LeetCode 環境預掛了 `MinPriorityQueue`（來自 `@datastructures-js/priority-queue`），不用自己刻 heap。把前面那條淘汰規則塞進 `compare`，heap 本體交給它：

```typescript
function topKFrequent(words: string[], k: number): string[] {
    const count = new Map<string, number>();
    for (const w of words) count.set(w, (count.get(w) ?? 0) + 1);

    // compare 回負數代表 a 先出（先被淘汰）。 top = 最該淘汰
    const pq = new MinPriorityQueue({
        compare: (a: string, b: string) => {
            const countA = count.get(a)!, countB = count.get(b)!;
            if (countA !== countB) return countA - countB;   // 次數低的先出（先淘汰）
            return b.localeCompare(a);        // 次數同，字母大的先出
        }
    });

    for (const w of count.keys()) {
        pq.enqueue(w);
        if (pq.size() > k) pq.dequeue();      // 超過 k，丟掉最該淘汰的
    }
    const res: string[] = [];
    while (!pq.isEmpty()) res.push(pq.dequeue());  // 最差先出
    return res.reverse();                          // 反轉成最好在前
}
```

- Time: $O(n \log k)$ — 每個 unique enqueue / dequeue 大小 k 的 heap
- Space: $O(n)$

`compare` 的方向就是前面講的淘汰規則：次數低的先出、次數同字母大的先出。 top 永遠是最該丟的，超過 k 就 `dequeue`。程式碼從手刻的四十行縮到十行，heap 的細節不用自己管。（`MinPriorityQueue` 各版本 API 有出入，`enqueue`／`dequeue`／`compare` 的細節見 [Heap](/concept/heap) 概念文章。）

那桶排序呢？347 用桶排序拿到 $O(n)$，這題不行那麼乾脆。因為同一個桶（次數相同）裡的字還要按字母排序，取的時候可能只取半個桶。桶排序還是能用，但每個桶要再排一次，程式碼反而更囉嗦，heap 這裡是比較平衡的選擇。

---

## 手刻 heap（面試禁用內建時）

JS 沒有內建 heap，面試不給用現成時得自己刻。核心：一個 array 存 heap，index 算式表示父子，push 往上冒、pop 往下沉。

**heap 怎麼用 array 表示**

heap 是一棵樹，但不用真的建節點、拉指標。用一個 array 就好，父子關係靠位置的算式：

```
        idx0
        /  \
     idx1   idx2
     /  \
  idx3  idx4

array: [ _ , _ , _ , _ , _ ]
         0   1   2   3   4

爸爸找小孩：左小孩 = 2i+1，右小孩 = 2i+2
小孩找爸爸：(i-1) >> 1
```

`>> 1` 是位元右移一位，等於除以 2 無條件捨去（floor）。`3 >> 1 = 1`、`5 >> 1 = 2`。跟 `Math.floor(i/2)` 同結果，但 `/` 在 JS 會給浮點數（`3/2=1.5`），`>>1` 直接給整數，省一個 `Math.floor`。

驗一下：idx1、idx2 算回去 `(1-1)>>1=0`、`(2-1)>>1=0`，爸爸都是 idx0；idx3、idx4 算回去都是 idx1。對得上。`up` 就是拿這行一路往上找爸爸，`down` 反過來拿 `2i+1`、`2i+2` 往下找小孩。位元操作不熟可以看 [Bit Operators](/concept/operator)。

```typescript
function topKFrequent(words: string[], k: number): string[] {
    const count = new Map<string, number>();
    for (const w of words) count.set(w, (count.get(w) ?? 0) + 1);

    // worse(a,b)=true：a 比 b 更靠近 top 。 top 最先被丟，越靠近頂端就越先被淘汰
    const worse = (a: string, b: string): boolean => {
        const countA = count.get(a)!, countB = count.get(b)!;
        if (countA !== countB) return countA < countB;   // 次數低的更靠近 top
        return a > b;                    // 次數同，字母大的更靠近 top
    };
    const heap: string[] = [];
    const swap = (i: number, j: number) => { [heap[i], heap[j]] = [heap[j], heap[i]]; };  // 交換 array 兩格

    // up：把某一格的元素往上冒到它該在的位置（剛加進來的新元素用）
    const up = (i: number) => {
        while (i > 0) {
            const p = (i - 1) >> 1;                 // 爸爸在哪一格
            if (!worse(heap[i], heap[p])) break;    // 我沒有比爸爸更該往上，就停
            swap(i, p); i = p;                      // 跟爸爸換位置，游標跟著上去
        }
    };
    // down：把某一格的元素往下沉到它該在的位置（top 被換掉之後用）
    const down = (i: number) => {
        while (true) {
            let s = i;                              // 先假設自己最該在上面
            const l = 2 * i + 1, r = 2 * i + 2;     // 左右小孩在哪一格
            if (l < heap.length && worse(heap[l], heap[s])) s = l;  // 左小孩更該在上面就換成它
            if (r < heap.length && worse(heap[r], heap[s])) s = r;  // 右小孩再比一次
            if (s === i) break;                     // 最該在上面的還是自己，就停
            swap(i, s); i = s;                      // 跟更該在上面的小孩換，游標跟著沉下去
        }
    };

    for (const w of count.keys()) {
        heap.push(w);                               // 放到最底
        up(heap.length - 1);                        // 再冒到該在的位置
        if (heap.length > k) {                      // 超過 k 個，把 top 那個最該淘汰的丟掉
            swap(0, heap.length - 1);               // top 跟最後一格對調
            heap.pop();                             // 拿掉最後一格，也就是原本的 top
            down(0);                                // 補上來的元素沉到該在的位置
        }
    }
    const res: string[] = [];
    while (heap.length) {                           // 一個一個倒出來，最差的先出來
        res.push(heap[0]);                          // 先接住當前 top
        swap(0, heap.length - 1); heap.pop(); down(0);  // 拿掉 top，讓下一個上來
    }
    return res.reverse();                           // 最差先出，反過來就變成最好在前
}
```

`up` 裡的 `swap(i, p); i = p;` 是一組兩個動作，缺一不可：

- **`swap(i, p)`** 動的是 array 的內容：把 `i`、`p` 兩格的值對調，元素往上換一層。
- **`i = p`** 動的是游標：元素現在在 `p` 格了，`i` 跟著指到 `p`，下一圈才會拿它的新位置去跟新的爸爸比，一路往上爬。
- **少了 `i = p`** 會怎樣：`i` 留在原本那格，下一圈拿的是被換下來的爸爸。爸爸不會比元素更靠近 top，`worse` 回 false 就 `break`，元素只往上爬一層就停下，heap 的性質被破壞。注意是提早停，不是無窮迴圈。

`down` 裡的 `swap(i, s); i = s;` 是同一組動作，只是方向往下：`swap` 把值往下換一層，`i = s` 讓游標跟著沉下去。

`swap(0, heap.length - 1); heap.pop(); down(0);` 在 `for` 和 `while` 各出現一次，是同一個「拿掉 top 」動作：top 跟最後一格對調、`pop` 掉最後一格（原本的 top）、`down(0)` 讓補上來的元素沉到定位。兩處機制一樣，差別在目的：

- **`for` 裡**：`heap.length > k` 時才做，把 top 那個最該淘汰的丟掉。 top 沒人接，讀都不讀就消失。
- **`while` 裡**：前面多一行 `res.push(heap[0])`，先把 top 讀出來存進答案，再拿掉。所以同樣是拿掉 top，這次是收成果，不是丟垃圾。
- **合起來**：`for` 用這動作維持 heap 大小是 k，`while` 用它把 k 個一個一個倒出來（最差先出），最後 `reverse` 變成最好在前。

複雜度跟內建版一樣，$O(n \log k)$ / $O(n)$。差別只在 heap 是你自己刻的。

**Go 版本**

Go 的 `container/heap` 給你框架，但 `Less`／`Push`／`Pop` 要自己實作：

```go
type wordHeap struct {
    words []string
    count map[string]int
}

func (h wordHeap) Len() int { return len(h.words) }
func (h wordHeap) Less(i, j int) bool {
    a, b := h.words[i], h.words[j]
    if h.count[a] != h.count[b] {
        return h.count[a] < h.count[b] // 次數低的在 top（先淘汰）
    }
    return a > b // 次數同，字母大的在 top
}
func (h wordHeap) Swap(i, j int) { h.words[i], h.words[j] = h.words[j], h.words[i] }
func (h *wordHeap) Push(x any)   { h.words = append(h.words, x.(string)) }
func (h *wordHeap) Pop() any {
    old := h.words
    n := len(old)
    x := old[n-1]
    h.words = old[:n-1]
    return x
}

func topKFrequent(words []string, k int) []string {
    count := map[string]int{}
    for _, w := range words {
        count[w]++
    }
    h := &wordHeap{count: count}
    for w := range count {
        heap.Push(h, w)
        if h.Len() > k {
            heap.Pop(h) // 淘汰最差的
        }
    }
    res := make([]string, k)
    for i := k - 1; i >= 0; i-- { // pop 是最差先出，反著填進 res
        res[i] = heap.Pop(h).(string)
    }
    return res
}
```

---

## 解法比較表

| 解法 | Time | Space | 備註 |
|---|---|---|---|
| 計數 + 自訂排序 | $O(n \log n)$ | $O(n)$ | 好寫，comparator 兩段搞定 |
| min-heap（大小 k） | $O(n \log k)$ | $O(n)$ | follow-up 要的解，注意 tie-break 反向 |

---

## 結論

347 的雙胞胎，加了「字母序 tie-break」。排序法把規則塞進兩段 comparator；heap 法要記住 tie-break 方向跟最終順序相反，淘汰的是次數低、字母大的那個。tie-break 一反，整題的坑就在這裡。
