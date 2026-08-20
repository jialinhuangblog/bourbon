開票現場。一疊選票 `[1,1,1,2,2,3]`，要選出得票最高的 `k=2` 個號碼。1 號三票、2 號兩票、3 號一票，最高的兩個是 1 和 2。

```
nums = [1,1,1,2,2,3], k = 2
        ↓ 數票
1: ███ (3)
2: ██  (2)
3: █   (1)
        ↓ 取前 2 名
[1, 2]
```

翻回 code 術語：統計每個數字出現幾次，回傳次數最高的 k 個。前半段就是 [242](/problem/valid-anagram)、[49](/problem/group-anagrams) 那個 counting，難的是後半段「怎麼取前 k 名」。

題目 follow-up 特別要求：要比 $O(n \log n)$ 快。

---

**解題引導**

用 `nums = [1,1,1,2,2,3], k = 2` 想。

**Step 1：第一步一定要做什麼？**

*you can't rank what you haven't counted*

<span class="spoiler">先數次數。用 hashmap 統計每個數字出現幾次。Count frequencies first with a hashmap.</span>

**Step 2：有了次數，最直覺怎麼取前 k 名？**

*sort by count, take the top k*

<span class="spoiler">依次數排序，取前 k 個。但排序是 O(n log n)，follow-up 說不夠快。Sort by frequency, but that's O(n log n).</span>

**Step 3：只要前 k 名，需要把全部排好嗎？**

*a heap of size k, or something even cheaper*

<span class="spoiler">不用。維護一個大小 k 的 min-heap（O(n log k)），或用桶排序（O(n)）。A size-k heap, or bucket sort.</span>

**Step 4：次數最大會是多少？這個上限能利用嗎？**

*frequency never exceeds n, so it can be an array index*

<span class="spoiler">最多出現 n 次。開 n+1 個桶，次數當 index，從高往低掃。這就是 O(n)。Frequency ≤ n, so use it as a bucket index.</span>

想完再往下看 code。

---

## 解法一：計數 + 排序

最直覺：數完次數，把所有數字依次數由多到少排序，取前 k 個。

用 `[1,1,1,2,2,3], k=2` 走：

```
數次數：{1:3, 2:2, 3:1}
依次數排序：1(3) > 2(2) > 3(1)
取前 2 個：[1, 2]
```

```typescript
function topKFrequent(nums: number[], k: number): number[] {
    const count = new Map<number, number>();
    for (const n of nums) count.set(n, (count.get(n) ?? 0) + 1);
    return [...count.entries()]
        .sort((a, b) => b[1] - a[1])   // 依次數由多到少
        .slice(0, k)
        .map(e => e[0]);
}
```

- Time: $O(n \log n)$ — 排序主導
- Space: $O(n)$ — 計數表

<details>
<summary>Go 版本</summary>

```go
func topKFrequent(nums []int, k int) []int {
    count := map[int]int{}
    for _, n := range nums {
        count[n]++
    }
    uniq := make([]int, 0, len(count))
    for num := range count {
        uniq = append(uniq, num)
    }
    // 依次數由多到少排 unique 的數字
    sort.Slice(uniq, func(i, j int) bool { return count[uniq[i]] > count[uniq[j]] })
    return uniq[:k]
}
```

</details>

能過。但 follow-up 明講要比 $O(n \log n)$ 快，這個解法剛好卡在它不要的那條線上。問題出在：我們排了「全部」，但只要「前 k 個」。多排的部分是浪費。

---

## 解法二：桶排序

一個數字最多出現 n 次（整個陣列都是它）。次數的範圍是 0 到 n，是有界的小整數。有界的小整數，可以直接當 array 的 index。

開 n+1 個桶，`buckets[f]` 放「出現 f 次的所有數字」。填完桶，從最高頻的桶往低頻掃，湊滿 k 個就停。

用 `[1,1,1,2,2,3], k=2` 走：

```
數次數：{1:3, 2:2, 3:1}

填桶（index = 次數）：
buckets[1] = [3]
buckets[2] = [2]
buckets[3] = [1]

從高往低掃：
f=3: buckets[3]=[1] → res=[1]
f=2: buckets[2]=[2] → res=[1,2]，湊滿 k=2，停
```

```typescript
function topKFrequent(nums: number[], k: number): number[] {
    const count = new Map<number, number>();
    for (const n of nums) count.set(n, (count.get(n) ?? 0) + 1);

    // buckets[f] = 出現 f 次的數字。最多出現 nums.length 次，開這麼多桶
    const buckets: number[][] = Array.from({ length: nums.length + 1 }, () => []);
    for (const [num, freq] of count) buckets[freq].push(num);

    const res: number[] = [];
    for (let f = buckets.length - 1; f >= 0 && res.length < k; f--) {
        for (const num of buckets[f]) {
            res.push(num);
            if (res.length === k) break;
        }
    }
    return res;
}
```

- Time: $O(n)$ — 數次數 $O(n)$、填桶 $O(\text{unique})$、掃桶最多走過 n 個桶跟 n 個數字
- Space: $O(n)$ — 桶

<details>
<summary>Go 版本</summary>

```go
func topKFrequent(nums []int, k int) []int {
    count := map[int]int{}
    for _, n := range nums {
        count[n]++
    }
    buckets := make([][]int, len(nums)+1) // buckets[f] = 出現 f 次的數字
    for num, freq := range count {
        buckets[freq] = append(buckets[freq], num)
    }
    res := []int{}
    for f := len(buckets) - 1; f >= 0 && len(res) < k; f-- {
        for _, num := range buckets[f] {
            res = append(res, num)
            if len(res) == k {
                break
            }
        }
    }
    return res
}
```

</details>

這就是 counting sort 的想法：不比大小，用「值本身當位置」。次數有界（≤ n）是這招能用的前提，換成無界的浮點數就不行了。$O(n)$ 打敗 follow-up 要求。

---

## 解法三：大小 k 的 heap

這題掛在 blind75 的 Heap 分類，經典解就是 heap。維護一個大小 k 的 **min-heap**（top 是最小的），掃過每個 (數字, 次數)：塞進去，超過 k 個就把 top（次數最小的）丟掉。掃完，heap 裡剩的就是次數前 k 大。

為什麼用 min-heap 而不是 max-heap？因為要「淘汰最小的」。 top 放最小值，一超過 k 就 pop 掉 top，留下的自然是大的那 k 個。

TypeScript 沒有內建 priority queue，面試現場常要手刻，這是這題在 TS 的真實摩擦點。

```typescript
function topKFrequent(nums: number[], k: number): number[] {
    const count = new Map<number, number>();
    for (const n of nums) count.set(n, (count.get(n) ?? 0) + 1);

    const heap: [number, number][] = [];  // [freq, num]，用陣列當 min-heap
    const swap = (i: number, j: number) => { [heap[i], heap[j]] = [heap[j], heap[i]]; };
    const up = (i: number) => {
        while (i > 0) {
            const p = (i - 1) >> 1;
            if (heap[p][0] <= heap[i][0]) break;   // 父比自己小，停
            swap(i, p);
            i = p;                                 // 游標跟著上去
        }
    };
    const down = (i: number) => {
        while (true) {
            const l = 2 * i + 1;
            const r = 2 * i + 2;
            let s = i;
            if (l < heap.length && heap[l][0] < heap[s][0]) s = l;
            if (r < heap.length && heap[r][0] < heap[s][0]) s = r;
            if (s === i) break;                    // 父已經最小，停
            swap(i, s);
            i = s;                                 // 游標跟著沉下去
        }
    };

    for (const [num, freq] of count) {
        heap.push([freq, num]);
        up(heap.length - 1);

        // 超過 k 個：top 是全場最小，把它換到尾端才能用 pop 刪，
        // 被換上來墊背的元素再用 down(0) 沉澱回該在的位置
        if (heap.length > k) {
            swap(0, heap.length - 1);
            heap.pop();
            down(0);
        }
    }
    return heap.map(e => e[1]);
}
```

- Time: $O(n \log k)$ — 每個 unique 進出 heap 花 log k
- Space: $O(n + k)$

<details>
<summary>Go 版本</summary>

```go
type pair struct{ num, freq int }
type minHeap []pair

func (h minHeap) Len() int            { return len(h) }
func (h minHeap) Less(i, j int) bool  { return h[i].freq < h[j].freq } // 次數小的在 top
func (h minHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x any)         { *h = append(*h, x.(pair)) }
func (h *minHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}

func topKFrequent(nums []int, k int) []int {
    count := map[int]int{}
    for _, n := range nums {
        count[n]++
    }
    h := &minHeap{}
    for num, freq := range count {
        heap.Push(h, pair{num, freq})
        if h.Len() > k {
            heap.Pop(h) // 超過 k 個，丟掉次數最小的
        }
    }
    res := make([]int, 0, k)
    for _, p := range *h {
        res = append(res, p.num)
    }
    return res
}
```

</details>

**走一遍**。假設 count 數完是 `{1:2, 2:7, 3:3, 4:1}`（Map 照插入順序給值），k = 3。heap 裡裝的是 `[freq, num]`，比大小只看 freq。

```
處理 (1, freq 2)：push [2,1]
  heap = [[2,1]]
  up(0)：i = 0 不進 while，不動

處理 (2, freq 7)：push [7,2]
  heap = [[2,1], [7,2]]
  up(1)：父是 0，2 ≤ 7，父比較小，停
  長度 2 ≤ k，不踢人

處理 (3, freq 3)：push [3,3]
  heap = [[2,1], [7,2], [3,3]]
  up(2)：父是 0，2 ≤ 3，停
  長度 3 = k，不踢人

處理 (4, freq 1)：push [1,4]
  heap = [[2,1], [7,2], [3,3], [1,4]]
  up(3)：父是 1，7 > 1 → 交換，heap = [[2,1], [1,4], [3,3], [7,2]]，i = 1
         父是 0，2 > 1 → 交換，heap = [[1,4], [2,1], [3,3], [7,2]]，i = 0，停
         （freq 1 一路浮到 top：它現在是全場最小）

  長度 4 > k，踢掉最小的：
  swap(0, 3)：heap = [[7,2], [2,1], [3,3], [1,4]]   ← 最小的換到尾端
  pop()：     heap = [[7,2], [2,1], [3,3]]           ← 尾端丟掉，[1,4] 出局
  down(0)：l=1 是 [2,1]，2 < 7 → s = 1；r=2 是 [3,3]，3 不小於 2，s 不變
           交換 0 和 1：heap = [[2,1], [7,2], [3,3]]，i = 1
           l = 2×1+1 = 3 超出長度，停
           （頂上來的 [7,2] 沉回去，新的最小 [2,1] 回到 top）

return [1, 2, 3]   ← freq 2、7、3，前三大 ✓（freq 1 被踢掉）
```

「超過 k 就踢」那行拆開就是 trace 裡的三個動作：先把 top（最小）跟尾端交換、`pop` 丟掉尾端、再 `down(0)` 讓被換上來的元素沉回正確位置。

Go 有 `container/heap`，但要自己實作 5 個 interface method（Len/Less/Swap/Push/Pop），也不算輕鬆。heap 解 $O(n \log k)$ 已經打敗 follow-up，只是還沒到桶排序的 $O(n)$。它的好處是 k 遠小於 n 時省空間，而且是「stream 進來、隨時要前 k 名」這種場景的標準解。

---

## 解法比較表

| 解法 | Time | Space | N=$10^{5}$ 秒數 | 備註 |
|---|---|---|---|---|
| 計數 + 排序 | $O(n \log n)$ | $O(n)$ | 約 0.17 秒 | 好寫，但踩在 follow-up 禁止的線上 |
| 桶排序 | $O(n)$ | $O(n)$ | 約 0.01 秒 | 最快，次數有界才能用 |
| min-heap（大小 k） | $O(n \log k)$ | $O(n+k)$ | 約 0.05 秒 | Heap 分類的經典解，stream 場景首選 |

---

## 結論

先數次數，再取前 k 名。取前 k 名有三條路：排序最直覺但最慢，桶排序用「次數有界」換到 $O(n)$，heap 用 $O(n \log k)$ 且省空間。counting 這個開頭，[242](/problem/valid-anagram) 和 [49](/problem/group-anagrams) 都用過，到這題接上「怎麼選 top k」。
