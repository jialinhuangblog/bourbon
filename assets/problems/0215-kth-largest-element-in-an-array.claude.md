給一個陣列和一個數字 k，找出第 k 大的元素。

```
nums = [3, 2, 1, 5, 6, 4], k = 2

排序後：[1, 2, 3, 4, 5, 6]
第 2 大是 5，第 1 大是 6
答案：5
```

---

## 解法一：排序

最直覺：排序，取倒數第 k 個。

```typescript
function findKthLargest(nums: number[], k: number): number {
    nums.sort((a, b) => a - b);
    return nums[nums.length - k];
}
```

- Time: O(n log n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
import "sort"

func findKthLargest(nums []int, k int) int {
    sort.Ints(nums)
    return nums[len(nums)-k]
}
```

</details>

能不能不排序？題目也問了。

---

## 解法二：Min Heap

先講 heap 是什麼（詳見 [Binary Tree 家族](/concept/binary-tree#binary-tree-家族) 和 [Heap](/concept/heap)）。Heap 就是一棵有規則的二元樹，用陣列存：

```
陣列：[1, 3, 5, 7, 8]

對應的樹：
       1        ← 最小的永遠在頂端（min heap）
      / \
     3   5
    / \
   7   8

父子關係靠 index 算：
  父：(i-1) / 2
  左子：2*i + 1
  右子：2*i + 2
```

Min heap 的規則：父節點一定 ≤ 子節點。所以最小的永遠在頂端，取最小值 O(1)，插入或刪除 O(log n)。

不像排序要把全部排好，heap 只保證「頂端是最小（或最大）」，其他位置不管順序。

---

回到這題。排序可以解，但把全部排好只為了取一個值，太浪費。有沒有辦法只追蹤「目前最大的 k 個」就好？

需要一個容器，能隨時告訴你「裡面最小的是誰」，這樣超過 k 個時就把最小的丟掉。Heap 剛好做這件事：push 進去自動維護順序，頂端永遠是最小的，取出 O(log n)。

想法：維護一個大小為 k 的 min heap。掃完整個陣列後，heap 裡最小的那個就是第 k 大。

為什麼？heap 裡永遠只留最大的 k 個元素。最小的那個 = 第 k 大。

用 `nums = [3, 2, 1, 5, 6, 4]`, `k = 2` 走一遍：

**讀 3** → size < k，直接 push

```
  3
```

**讀 2** → push 2，2 < 3 所以 2 浮到頂端。size == k，heap 滿了

```
  2
 /
3
```

**讀 1** → 1 < 頂端 2，不夠資格進前 2 大，丟掉

```
  2       ← 1 連進來的資格都沒有
 /
3
```

**讀 5** → 5 > 頂端 2，pop 2，push 5。5 > 3 所以 3 留在頂端

```
  3
 /
5
```

**讀 6** → 6 > 頂端 3，pop 3，push 6。5 < 6 所以 5 留在頂端

```
  5
 /
6
```

**讀 4** → 4 < 頂端 5，不夠資格，丟掉

```
  5       ← 這就是第 2 大
 /
6
```

heap 頂端 = 5 = 第 2 大 ✓

heap 裡留下的 [5, 6] 就是最大的 2 個，頂端的 5 是這兩個裡最小的 = 第 k 大。

JavaScript 沒有內建 heap。LeetCode 的執行環境預掛了 `MinPriorityQueue`，API 見 [Heap](/concept/heap)：

```typescript
function findKthLargest(nums: number[], k: number): number {
    const pq = new MinPriorityQueue({ compare: (a, b) => a - b });

    for (const n of nums) {
        pq.enqueue(n);
        if (pq.size() > k) {
            pq.dequeue();   // 超過 k 個就丟掉最小的
        }
    }
    return pq.front();
}
```

- Time: O(n log k) — 每個元素做一次 heap 操作 O(log k)
- Space: O(k)

面試官說不能用內建的話，就得自己寫。push 往上浮、pop 往下沉，加起來大約 25 行，是背得起來的量。不要回答「假設有 heap」，下一句就是「那你寫一個」。

<details>
<summary>手寫 MinHeap 版（面試禁用內建時）</summary>

`push` 跟 `pop` 各自只做「放進去 / 拿出來」，往上浮跟往下沉抽成兩個方法。跟 Go 的 `container/heap` 把 `up` / `down` 獨立出來是同一種切法，也跟 [Heap](/concept/heap) 那篇的手寫版命名一致。

```typescript
class MinHeap {
    private heap: number[] = [];

    size() { return this.heap.length; }
    peek() { return this.heap[0]; }

    push(val: number) {
        this.heap.push(val);              // 放到最後
        this._bubbleUp(this.heap.length - 1);
    }

    pop(): number {
        const top = this.heap[0];         // 舊的根是回傳值，不會回到陣列裡
        const last = this.heap.pop()!;
        if (this.heap.length > 0) {
            this.heap[0] = last;          // 把最後一個搬到根
            this._sinkDown(0);
        }
        return top;
    }

    // 跟 parent 比，比 parent 小就換上去，一直換到不能換
    private _bubbleUp(i: number) {
        while (i > 0) {
            const parent = (i - 1) >> 1;
            if (this.heap[parent] <= this.heap[i]) break;
            [this.heap[i], this.heap[parent]] = [this.heap[parent], this.heap[i]];
            i = parent;
        }
    }

    // 跟兩個 child 裡比較小的那個比，比它大就換下去
    private _sinkDown(i: number) {
        const n = this.heap.length;
        while (true) {
            const left = 2 * i + 1;
            const right = 2 * i + 2;
            let smallest = i;
            if (left  < n && this.heap[left]  < this.heap[smallest]) smallest = left;
            if (right < n && this.heap[right] < this.heap[smallest]) smallest = right;
            if (smallest === i) break;    // 父已經是最小，停
            [this.heap[i], this.heap[smallest]] = [this.heap[smallest], this.heap[i]];
            i = smallest;
        }
    }
}

function findKthLargest(nums: number[], k: number): number {
    const heap = new MinHeap();
    for (const n of nums) {
        heap.push(n);
        if (heap.size() > k) heap.pop();
    }
    return heap.peek();
}
```

面試的時候先寫 `push` / `pop` / `findKthLargest` 這三段講完想法，再回頭填 `_bubbleUp` 跟 `_sinkDown`。這樣面試官在第一分鐘就看得到你的解法，不用等你把 20 行迴圈寫完。

</details>

<details>
<summary>Go 版本</summary>

```go
import "container/heap"

func findKthLargest(nums []int, k int) int {
    h := &MinHeap{}
    for _, n := range nums {
        heap.Push(h, n)
        if h.Len() > k {
            heap.Pop(h) // 超過 k 個就把最小的丟掉
        }
    }
    return (*h)[0] // 頂端就是第 k 大
}

// Go 的 heap 需要自己實作 interface
type MinHeap []int

func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MinHeap) Push(x any)        { *h = append(*h, x.(int)) }
func (h *MinHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}
```

</details>

---

**為什麼是 min heap 不是 max heap？**

直覺會想：找最大的，用 max heap。畫出來看差別：

**Max heap**（最大的在頂端）：

```
把 [3, 2, 1, 5, 6, 4] 全部放進去：

       6
      / \
     5   4
    / \  /
   2  3 1

頂端是 6 = 第 1 大
pop 一次拿到 6，再 pop 一次拿到 5 = 第 2 大
```

問題：要把全部 n 個元素都放進去，每次 push O(log n)，總共 O(n log n)。跟排序一樣貴。

**Min heap**（最小的在頂端，只放 k 個）：

```
只維護 2 個元素：

  5
 /
6

頂端 5 = 第 2 大 ✓
```

Min heap 只維護 k 個元素，超過就丟掉最小的。heap 操作是 O(log k) 不是 O(log n)。當 k 遠小於 n 時，差很多。

---

**Go 的 heap 為什麼這麼囉唆？**

Go 的 `container/heap` 是用 interface 設計的，需要實作 5 個方法（`Len`, `Less`, `Swap`, `Push`, `Pop`）。看起來多，但每個方法都只有一行。背一次就好，每題都長一樣。

差別只在 `Less`：
- Min heap: `h[i] < h[j]`
- Max heap: `h[i] > h[j]`

---

## 解法三：Quickselect

解法一排序了整個陣列。要第 2 大的話，其他 n - 2 個誰前誰後根本不用知道，那些工作是白做的。

Quicksort 的 partition 會選定一個 pivot，把比它小的搬到左邊、大的搬到右邊，做完之後 pivot 就停在它排序後該在的 index 上。Quicksort 接著左右兩邊都遞迴，quickselect 只走一邊。

`nums = [3, 2, 1, 5, 6, 4]`，k = 2。第 2 大排序後在 index `6 - 2 = 4`，那就是目標。假設選中的 pivot 是 4：

```
partition 之後   [3, 2, 1, 4, 6, 5]
                           ^
                        index 3

4 停在 index 3，比目標 index 4 小
=> 答案在右邊，下一輪只看 index 4..5
=> 左邊那三個再也不碰
```

```typescript
function findKthLargest(nums: number[], k: number): number {
    const target = nums.length - k;   // 第 k 大 = 排序後的 index n-k
    let l = 0;
    let r = nums.length - 1;

    while (true) {
        const p = partition(nums, l, r);
        if (p === target) return nums[p];
        if (p < target) l = p + 1;    // 目標在右邊
        else r = p - 1;               // 目標在左邊
    }
}

function partition(nums: number[], l: number, r: number): number {
    // 隨機挑 pivot，避免已排序的輸入退化成 O(n²)
    const rand = l + Math.floor(Math.random() * (r - l + 1));
    [nums[rand], nums[r]] = [nums[r], nums[rand]];

    const pivot = nums[r];
    let i = l;
    for (let j = l; j < r; j++) {
        if (nums[j] < pivot) {
            [nums[i], nums[j]] = [nums[j], nums[i]];
            i++;
        }
    }
    [nums[i], nums[r]] = [nums[r], nums[i]];
    return i;
}
```

- Time: 平均 O(n)，最差 O(n²)
- Space: O(1)，就地交換，不另外開陣列

平均 O(n) 的來源：第一輪掃 n 個，pivot 期望落在中間，第二輪只剩一半掃 n/2，接著 n/4。`n + n/2 + n/4 + ... = 2n`。最差是每次選中的 pivot 都剛好是當前範圍的最小或最大值，範圍只縮小一格，掃 n 輪，這就是 pivot 要隨機選的原因。

`nums.length = 10^5`，實際數比較次數：

| k | pop 觸發次數 | min heap 比較 | quickselect 中位數 | quickselect p95 |
|---|---|---|---|---|
| 1 | 99,999 | 99,999 | 188,903 | 333,924 |
| 1,000 | 99,000 | 2,545,194 | 199,179 | 353,493 |
| 10,000 | 90,000 | 3,157,475 | 251,730 | 428,891 |
| 50,000 | 50,000 | 1,827,477 | 327,455 | 509,604 |
| 99,000 | 1,000 | 265,724 | 200,637 | 348,101 |

heap 那兩欄是固定的，同一份輸入跑幾次都一樣。quickselect 每次隨機選 pivot，單次結果從 11 萬到 55 萬都出現過，所以跑 8,001 次取中位數，另外附 p95 讓運氣差的那一端也看得到。

heap 那一欄不是一路往上，在 k = 10,000 附近最高，然後回落。因為 pop 只在 heap 超過 k 個的時候觸發，也就是 `n - k` 次，而每次 pop 要往下沉 log k 層。k 變大的時候 `n - k` 線性變少、`log k` 只慢慢變多，兩個相乘就先升後降。

兩端各自是一種極端。k = 1 的時候 heap 裡永遠只有一個元素，pop 一次都不用比較，99,999 次全來自 push。k = 99,000 的時候只 pop 了 1,000 次，剩下的 99,000 個元素進去就沒再動過。

quickselect 受 k 的影響小很多，中位數從 18.9 萬到 32.7 萬，同一個量級。因為它掃的是陣列本身，k 只決定要往左邊還是右邊找，不決定每輪要做多少工作。

它會改動傳進來的 `nums`。LeetCode 不在意，但面試官可能會問，先講一句比較好。

<details>
<summary>Go 版本</summary>

```go
import "math/rand"

func findKthLargest(nums []int, k int) int {
    target := len(nums) - k
    l, r := 0, len(nums)-1
    for {
        p := partition(nums, l, r)
        if p == target {
            return nums[p]
        }
        if p < target {
            l = p + 1
        } else {
            r = p - 1
        }
    }
}

func partition(nums []int, l, r int) int {
    rd := l + rand.Intn(r-l+1)
    nums[rd], nums[r] = nums[r], nums[rd]

    pivot := nums[r]
    i := l
    for j := l; j < r; j++ {
        if nums[j] < pivot {
            nums[i], nums[j] = nums[j], nums[i]
            i++
        }
    }
    nums[i], nums[r] = nums[r], nums[i]
    return i
}
```

</details>

---

## 解法比較表

| 解法 | Time | Space | n=10^5 秒數 | 備註 |
|---|---|---|---|---|
| 排序 | $O(n \log n)$ | $O(1)$ | 0.15 秒 | 一行，面試可以先講出來當基準 |
| Min heap（size k） | $O(n \log k)$ | $O(k)$ | 0.32 秒 | 資料是串流進來的時候只有這個能用 |
| Quickselect | 平均 $O(n)$，最差 $O(n^2)$ | $O(1)$ | 0.03 秒 | 最快，但會改動輸入 |

秒數是實際數比較次數再除以 $10^7$。heap 那格取最差的 k，掃過 k = 1,000 到 20,000 找到峰值在 k = 9,000、3,202,988 次。quickselect 有隨機性，跑 8,001 次取中位數 251,730 次；運氣差的 p95 是 428,891 次，換算 0.04 秒。樣本數要跑到夠：501 次的中位數是 249,387，換算 0.0249 秒，四捨五入會寫成 0.02，跟 1,001 次以後穩定的 0.03 差一格。

排序 0.15 秒反而比 heap 的 0.32 秒快。$O(n \log n)$ 看起來比 $O(n \log k)$ 差，但 $\log_2 10^5$ 只有 16.6，$k = 9{,}000$ 的 $\log_2 k$ 是 13.1，兩個 log 差不多；而排序只走一趟，heap 每個元素都要 push，超過 k 之後還要 pop，兩趟的常數更大。

所以 heap 在這題的價值不是執行時間，是 $O(k)$ 空間，以及資料還沒到齊就能開始處理。`nums` 如果是一條讀不完的串流，排序跟 quickselect 都做不了。

---

## 結論

排序一行搞定但 O(n log n)，做了一堆用不到的排序。Min heap 只留 k 個最大的，O(n log k)，頂端就是第 k 大，資料是串流進來的時候只有這個解法能用。Quickselect 平均 O(n) 最快，代價是會改動輸入、而且最差 O(n²)。

TypeScript 沒有內建 heap，LeetCode 環境有預掛的 `MinPriorityQueue`；面試禁用的話手寫版大約 25 行。Go 的 heap interface 囉唆但固定，背一次就好。
