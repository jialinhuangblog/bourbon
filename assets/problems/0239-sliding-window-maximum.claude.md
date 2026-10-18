八個人站成一排，你手上有一張只能看到三個人的紙框。從最左邊開始，每次往右移一格，每次報出框裡最高的那一個。

```
[1, 3, -1] -3, 5, 3, 6, 7    → 3
1, [3, -1, -3] 5, 3, 6, 7    → 3
1, 3, [-1, -3, 5] 3, 6, 7    → 5
1, 3, -1, [-3, 5, 3] 6, 7    → 5
1, 3, -1, -3, [5, 3, 6] 7    → 6
1, 3, -1, -3, 5, [3, 6, 7]   → 7

答案 = [3, 3, 5, 5, 6, 7]
```

八個元素、框寬 3，滑出 `8 - 3 + 1 = 6` 個 window，所以答案有 6 個數字。

翻回 code 術語：給 `nums` 跟 `k`，回傳每個長度 k 的連續區間的最大值。

---

**解題引導**

用 `[1,3,-1,-3,5,3,6,7]`、k = 3 想。

**Step 1：最直覺怎麼做？**

*for each window, scan k numbers*

<span class="spoiler">每個 window 掃 k 個找最大。O(n×k)。For each window, scan all k numbers to find the max.</span>

**Step 2：window 往右移一格，有多少東西是重複看的？**

*two neighboring windows overlap in k-1 numbers*

<span class="spoiler">相鄰兩個 window 有 k-1 個元素重疊，暴力解把它們重掃了一遍。Neighboring windows share k-1 numbers, and brute force rescans them.</span>

**Step 3：window 往右移一格，新的最大值有哪幾種可能？**

*either the old max stays, or the newcomer takes over, or the old max just left*

<span class="spoiler">三種：舊的最大值還在框內就還是它；新進來的比它大就換人；舊的最大值剛好滑出框，就得找備胎。Old max stays, newcomer wins, or old max expired and a backup takes over.</span>

**Step 4：誰有資格當備胎，誰永遠當不了？**

*someone smaller AND to the left of you is useless forever*

<span class="spoiler">在你左邊又比你小的人永遠當不了答案，因為他比你早滑出框，你在的時候輪不到他。留下來的人從左到右一定是遞減的。Anyone smaller and to your left can never win.</span>

想完再往下看 code。

---

## 解法一：暴力（每個 window 掃一遍）

```typescript
function maxSlidingWindow(nums: number[], k: number): number[] {
    const result: number[] = [];
    for (let i = 0; i + k <= nums.length; i++) {   // i 是 window 左界
        let max = nums[i];
        for (let j = i + 1; j < i + k; j++) {       // 掃完這個 window
            if (nums[j] > max) max = nums[j];
        }
        result.push(max);
    }
    return result;
}
```

- Time: $O(n \times k)$
- Space: $O(1)$（不算 result）

<details>
<summary>Go 版本</summary>

```go
func maxSlidingWindow(nums []int, k int) []int {
    result := []int{}
    for i := 0; i+k <= len(nums); i++ {
        max := nums[i]
        for j := i + 1; j < i+k; j++ {
            if nums[j] > max {
                max = nums[j]
            }
        }
        result = append(result, max)
    }
    return result
}
```

</details>

**走一遍。** `[1,3,-1,-3,5,3,6,7]`、k = 3，前三輪：

```
i=0  max 從 nums[0]=1 起
     j=1  nums[1]=3 > 1   → max=3
     j=2  nums[2]=-1 < 3  → 不動
     result = [3]

i=1  max 從 nums[1]=3 起
     j=2  nums[2]=-1 < 3  → 不動
     j=3  nums[3]=-3 < 3  → 不動
     result = [3, 3]

i=2  max 從 nums[2]=-1 起
     j=3  nums[3]=-3 < -1 → 不動
     j=4  nums[4]=5 > -1  → max=5
     result = [3, 3, 5]
```

i=0 跟 i=1 都看過 `nums[1]` 跟 `nums[2]`，i=1 跟 i=2 都看過 `nums[2]` 跟 `nums[3]`。**每個元素被重掃了 k 次。**

`nums.length` 上限 $10^5$。k 取一半時 `(n-k+1) × k` 最大，約 $2.5 \times 10^9$ 次操作，除以 $10^7$ 大約 **250 秒**。TLE。

---

## 解法二：max-heap（會動，但還不夠快）

暴力的浪費在「每個 window 重新找一次最大值」。丟進 max-heap 就不用重找，heap 的 top 永遠是最大的。

但有個狀況要處理：heap 只管大小，不管誰過期了。top 那個元素可能早就滑出 window 了。

做法是連 index 一起存。取 top 的時候看它的 index 還在不在 window 裡，不在就丟掉再看下一個。這叫 lazy deletion：不主動找過期的元素，等它浮到 top 才處理。

```typescript
function maxSlidingWindow(nums: number[], k: number): number[] {
    const heap = new MaxHeap();   // 存 [值, index]，照值排
    const result: number[] = [];

    for (let i = 0; i < nums.length; i++) {
        heap.push([nums[i], i]);
        if (i < k - 1) continue;                    // 還沒湊滿第一個 window

        while (heap.top()[1] <= i - k) heap.pop();  // top 過期了就丟掉
        result.push(heap.top()[0]);
    }
    return result;
}
```

- Time: $O(n \log n)$，每個元素進 heap 一次、最多出來一次，各 $O(\log n)$
- Space: $O(n)$，最壞情況 heap 裝下全部元素（輸入遞增時沒有人會被提早丟掉）

<details>
<summary>Go 版本</summary>

```go
import "container/heap"

type item struct{ val, idx int }
type maxHeap []item

func (h maxHeap) Len() int            { return len(h) }
func (h maxHeap) Less(i, j int) bool  { return h[i].val > h[j].val }   // 大的在前
func (h maxHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x interface{}) { *h = append(*h, x.(item)) }
func (h *maxHeap) Pop() interface{} {
    old := *h
    n := len(old)
    last := old[n-1]
    *h = old[:n-1]
    return last
}

func maxSlidingWindow(nums []int, k int) []int {
    h := &maxHeap{}
    result := []int{}
    for i, v := range nums {
        heap.Push(h, item{v, i})
        if i < k-1 {
            continue
        }
        for (*h)[0].idx <= i-k {   // top 過期就丟
            heap.Pop(h)
        }
        result = append(result, (*h)[0].val)
    }
    return result
}
```

</details>

**`i - k` 是什麼？** 當前 window 是 `[i-k+1, i]`，所以 index 小於等於 `i-k` 的都在框外。

**走一遍。** 官方那組資料每次的 top 剛好都沒過期，看不到 lazy deletion。換 `[5,3,4,2,1]`、k = 3：

```
i=0  push (5,0)                  裡面有 (5,0)
     i < k-1，跳過

i=1  push (3,1)                  裡面有 (5,0) (3,1)
     i < k-1，跳過

i=2  push (4,2)                  裡面有 (5,0) (3,1) (4,2)，top = (5,0)
     window [0,2]，top 的 index 0 > i-k = -1，沒過期
     result = [5]

i=3  push (2,3)                  裡面有 (5,0) (3,1) (4,2) (2,3)，top = (5,0)
     window [1,3]，top 的 index 0 <= i-k = 0，過期了
     pop 掉 (5,0)                 新的 top = (4,2)，index 2 > 0，沒過期
     result = [5, 4]

i=4  push (1,4)                  top = (4,2)
     window [2,4]，index 2 > i-k = 1，沒過期
     result = [5, 4, 4]
```

上面只列 heap 裡有哪些元素，沒列順序，因為 heap 內部怎麼排是實作細節，只有 top 是最大的這件事有保證。

`(3,1)` 在 i=3 之後其實也過期了，但它沒浮到 top，就一直留在 heap 裡。這就是 lazy 的意思，也是空間會到 $O(n)$ 的原因。

$10^5$ 個元素、$\log_2 10^5 \approx 17$，約 $1.7 \times 10^6$ 次操作，**0.17 秒**。過得了。

**能不能更好？** heap 每次插入都要 $O(\log n)$，可是我們根本不需要「完整的排序」，只需要知道誰是最大的、誰有機會接棒。

---

## 解法三：monotonic deque（$O(n)$）

回到 Step 4 的觀察：**在你左邊又比你小的人，永遠當不了答案。** 他比你早滑出框，只要他還在框裡，你也一定在，答案輪不到他。

所以新元素進來時，把尾巴所有比它小的直接丟掉。剩下的從前到後一定遞減，最前面那個就是當前 window 的最大值。

存的是 index 不是值，因為要判斷有沒有滑出 window，得拿 index 跟 `i-k` 比。

```typescript
function maxSlidingWindow(nums: number[], k: number): number[] {
    const waiting: number[] = [];   // 存 index，對應的值由前到後遞減
    const result: number[] = [];

    for (let i = 0; i < nums.length; i++) {
        // 尾巴比我小的，永遠沒機會了
        while (waiting.length && nums[waiting[waiting.length - 1]] < nums[i]) {
            waiting.pop();
        }
        waiting.push(i);

        if (waiting[0] <= i - k) waiting.shift();   // 頭過期了就丟
        if (i >= k - 1) result.push(nums[waiting[0]]);
    }
    return result;
}
```

- Time: $O(n)$，每個 index 進 `waiting` 一次、出來一次，總共 $2n$ 次操作
- Space: $O(k)$，`waiting` 裡的 index 都在同一個 window 內

<details>
<summary>Go 版本</summary>

```go
func maxSlidingWindow(nums []int, k int) []int {
    waiting := []int{}   // 存 index
    result := []int{}

    for i, v := range nums {
        for len(waiting) > 0 && nums[waiting[len(waiting)-1]] < v {
            waiting = waiting[:len(waiting)-1]   // 從尾巴丟
        }
        waiting = append(waiting, i)

        if waiting[0] <= i-k {
            waiting = waiting[1:]                // 從頭丟
        }
        if i >= k-1 {
            result = append(result, nums[waiting[0]])
        }
    }
    return result
}
```

</details>

**為什麼是 `<` 不是 `<=`？** 相等的時候留著也不會錯，因為兩個都可能是答案，右邊那個晚滑出去。寫 `<=` 把左邊那個丟掉也對，只是 `waiting` 短一點。這題兩種都能過。

**走一遍。** `[1,3,-1,-3,5,3,6,7]`、k = 3。括號裡是那個 index 的值：

```
i=0  waiting 空，push 0            waiting: [0(1)]
     i < k-1，還不輸出

i=1  nums[1]=3 > 1，pop 0
     push 1                        waiting: [1(3)]

i=2  nums[2]=-1 < 3，直接 push 2   waiting: [1(3), 2(-1)]
     頭是 index 1 > i-k = -1，沒過期
     result = [3]

i=3  nums[3]=-3 < -1，push 3       waiting: [1(3), 2(-1), 3(-3)]
     頭是 index 1 > i-k = 0，沒過期
     result = [3, 3]

i=4  nums[4]=5 > -3，pop 3
     5 > -1，pop 2
     5 > 3，pop 1
     push 4                        waiting: [4(5)]
     result = [3, 3, 5]

i=5  nums[5]=3 < 5，push 5         waiting: [4(5), 5(3)]
     result = [3, 3, 5, 5]

i=6  nums[6]=6 > 3，pop 5
     6 > 5，pop 4
     push 6                        waiting: [6(6)]
     result = [3, 3, 5, 5, 6]

i=7  nums[7]=7 > 6，pop 6
     push 7                        waiting: [7(7)]
     result = [3, 3, 5, 5, 6, 7]
```

i=4 那一步一次 pop 掉三個。看起來很貴，但那三個之後永遠不會再被碰到，所以整趟加起來還是 $2n$ 次。

**這組資料看不到從頭丟。** 每次該滑出 window 的元素，早就因為比後來的小而從尾巴被丟掉了。換 `[5,3,4,2,1]`、k = 3：

```
i=0  push 0                        waiting: [0(5)]
i=1  3 < 5，push 1                 waiting: [0(5), 1(3)]
i=2  4 > 3，pop 1；4 < 5，push 2   waiting: [0(5), 2(4)]
     頭是 index 0 > i-k = -1
     result = [5]

i=3  2 < 4，push 3                 waiting: [0(5), 2(4), 3(2)]
     頭是 index 0 <= i-k = 0，過期
     shift                         waiting: [2(4), 3(2)]
     result = [5, 4]

i=4  1 < 2，push 4                 waiting: [2(4), 3(2), 4(1)]
     頭是 index 2 > i-k = 1
     result = [5, 4, 4]
```

5 滑出去的時候 4 接手，這就是 Step 4 說的「備胎」。

**兩端都要動，所以要 deque。** 尾巴丟比自己小的，這部分跟 monotonic stack 一樣；頭丟滑出 window 的，這是 stack 做不到的。TypeScript 的 `shift()` 在陣列上是 $O(n)$，這題 k 不大時無所謂；要嚴謹就自己維護一個左指標，或用真的 deque。

$2 \times 10^5$ 次操作，**0.02 秒**。

---

**Overthinking**

**改成問最小值呢？** 把比較符號反過來就好，`waiting` 變成遞增，頭是最小值。

**k 會變動呢？** 這招就不能用了。「誰永遠當不了答案」的推論建立在「左邊的先滑出去」，k 一變，滑出的順序就不確定。那種情況回去用 heap 或平衡樹。

**面試官問「為什麼不是 $O(n \log k)$？」** 那是 heap 版把過期元素主動刪掉的複雜度。deque 版沒有排序動作，只有進出，所以是 $O(n)$。

---

## 解法比較表

| 解法 | Time | Space | n=10^5 秒數 | 備註 |
|---|---|---|---|---|
| 暴力 | $O(n \times k)$ | $O(1)$ | 約 250 秒 | TLE，但先講出來當基準 |
| Max-heap | $O(n \log n)$ | $O(n)$ | 0.17 秒 | 過得了，lazy deletion 好想 |
| Monotonic deque | $O(n)$ | $O(k)$ | 0.02 秒 | 面試要的答案 |

---

## 結論

這題要的不是「怎麼快速找最大值」，是「怎麼知道誰永遠不可能是答案」。左邊又比你小的那些人一律丟掉，剩下的自然遞減，頭就是答案。
