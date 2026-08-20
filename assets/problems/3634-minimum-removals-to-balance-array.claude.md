給一個陣列和一個 k，「平衡」的定義是 max <= min * k。問最少要刪幾個元素，讓剩下的陣列平衡。

---

**解題引導**

用 `nums = [1, 6, 2, 9], k = 3` 想。

**Step 1：把「刪最少」換個問法？**

*thinking about the flip — minimum remove means maximum keep, same thing*

<span class="spoiler">刪最少 = 留最多。只要找出最大的合法子集，答案就是 n - 它的大小。</span>

**Step 2：子集裡面誰決定合不合法？**

*only the extremes matter — the middle is noise*

<span class="spoiler">只有 min 跟 max 決定。中間的數字完全不影響條件。只要 max 不超過 min 的 k 倍就過。</span>

**Step 3：如果把 nums 排序，最佳解長什麼樣？**

*if the ends fit, why throw away the middle*

<span class="spoiler">排序後，最佳解一定是連續一段。因為只要首尾兩個數字合法，中間任何數字都在 min 和 max 之間，塞回去不會破壞條件，反而省一次刪除。</span>

**Step 4：怎麼找最長的連續合法區段？**

*fix the left, push the right as far as it will go*

<span class="spoiler">排序後，對每個左端 i，找最遠的右端 j 使 nums[j] <= nums[i] * k。最笨就是對每個 i 往右掃。O(n²)。</span>

**Step 5：當 i 往右移，j 會倒退嗎？**

*left grows so the limit grows, right never needs to walk back*

<span class="spoiler">不會。左端變大，nums[i] * k 也變大，上限放寬。原本合法的 j 不會突然變不合法，所以 j 只會往前推。雙指針、單調推進 → sliding window O(n)。</span>

想完再往下看 code。

---

## 解法一：排序 + 暴力掃描

先排序，把問題變成「在排好序的陣列裡，找最長的連續區段 `[i, j]` 使 `nums[j] <= nums[i] * k`」。

用 `nums = [1, 6, 2, 9], k = 3` 走一遍。排序後：`[1, 2, 6, 9]`。

暴力解：對每個左端 i，從 i 開始往右掃，直到 `nums[j] > nums[i] * k` 為止。

```
i=0 (nums[i]=1, limit=3):
  j=0: 1 <= 3 ✓
  j=1: 2 <= 3 ✓
  j=2: 6 <= 3 ✗ 停
  區段長度 = 2

i=1 (nums[i]=2, limit=6):
  j=1: 2 <= 6 ✓
  j=2: 6 <= 6 ✓
  j=3: 9 <= 6 ✗ 停
  區段長度 = 2

i=2 (nums[i]=6, limit=18):
  j=2: 6 <= 18 ✓
  j=3: 9 <= 18 ✓
  區段長度 = 2

i=3 (nums[i]=9):
  區段長度 = 1

最長 = 2
```

答案 = 4 - 2 = 2。刪掉 1 和 9，留下 `[2, 6]`，`6 <= 2 * 3` ✓。

```typescript
function minRemoval(nums: number[], k: number): number {
    nums.sort((a, b) => a - b);
    const n = nums.length;
    let best = 1;                      // 至少留一個
    for (let i = 0; i < n; i++) {
        const limit = nums[i] * k;     // 以 nums[i] 當 min 時，max 不能超過這個
        let j = i;
        while (j < n && nums[j] <= limit) {
            j++;                       // 一路往右推
        }
        // 合法區段是 [i, j-1]
        if (j - i > best) {
            best = j - i;
        }
    }
    return n - best;
}
```

- Time: O(n log n) 排序 + **$O(n^2)$ 掃描**
- Space: O(1)（不算排序）

<details>
<summary>Go 版本</summary>

```go
func minRemoval(nums []int, k int) int {
    sort.Ints(nums)
    n := len(nums)
    best := 1                          // 至少留一個
    for i := 0; i < n; i++ {
        limit := nums[i] * k           // 以 nums[i] 當 min 時，max 不能超過這個
        j := i
        for j < n && nums[j] <= limit {
            j++                        // 一路往右推
        }
        // 合法區段是 [i, j-1]
        if j-i > best {
            best = j - i
        }
    }
    return n - best
}
```

</details>

**外層 n 個起點，內層最壞從 i 走到底。** $n = 10^5$ 時 $O(n^2)$ 就 $10^{10}$，TLE。

---

## 解法二：滑動視窗

看暴力解的內層：每個 i 都從 i 開始重新往右掃。但 i 變大時，`nums[i]` 變大，`limit` 也變大，原本不合法的 j 可能變合法，原本合法的不會突然不合法。

換句話說，**右端 j 只會往前，不會倒退。** 那還有什麼理由每次從 i 重新開始？

乾脆兩個指針：右指針 j 持續往右推，左指針 i 只在超標時才往前縮。經典 sliding window。

```
sorted = [1, 2, 6, 9], k = 3

j=0: nums[0]=1. 1 > nums[i=0]*k = 3? no.
     window [0,0], 長度 1. best=1.

j=1: nums[1]=2. 2 > 1*3=3? no.
     window [0,1], 長度 2. best=2.

j=2: nums[2]=6. 6 > 1*3=3? yes → i++ → i=1.
                6 > 2*3=6? no.
     window [1,2], 長度 2. best=2.

j=3: nums[3]=9. 9 > 2*3=6? yes → i++ → i=2.
                9 > 6*3=18? no.
     window [2,3], 長度 2. best=2.
```

答案 = 4 - 2 = 2。

跟暴力解同樣結果，但 i 和 j 各自單調往前走，總共最多 2n 步。

```typescript
function minRemoval(nums: number[], k: number): number {
    nums.sort((a, b) => a - b);        // 記得 compare function，不然是字串排序
    const n = nums.length;
    let best = 1;
    let i = 0;
    for (let j = 0; j < n; j++) {
        while (nums[j] > nums[i] * k) {
            i++;
        }
        best = Math.max(best, j - i + 1);
    }
    return n - best;
}
```

- Time: **O(n log n)**（排序 dominate，sliding window 本身 O(n)）
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func minRemoval(nums []int, k int) int {
    sort.Ints(nums)
    n := len(nums)
    best := 1
    i := 0
    for j := 0; j < n; j++ {
        for nums[j] > nums[i]*k {
            i++                        // 右端超標了，左端縮一格
        }
        if j-i+1 > best {
            best = j - i + 1
        }
    }
    return n - best
}
```

</details>

---

**能不能更好？**

排序是 O(n log n)，sliding window 是 O(n)。想更快只能打排序的主意。

但這題沒辦法。判斷平衡靠相對大小，不是絕對值，counting sort 之類的也要掃過值域 $10^9$。排序已經是下限。

---

## 解法三：二分搜尋

有些人看到「排序後找最遠的 j」會想用 binary search：對每個 i 在 `[i, n-1]` 二分找最大的 j 使 `nums[j] <= nums[i] * k`。

```typescript
function minRemoval(nums: number[], k: number): number {
    nums.sort((a, b) => a - b);
    const n = nums.length;
    let best = 1;
    for (let i = 0; i < n; i++) {
        const limit = nums[i] * k;
        // 二分找最大的 j 使 nums[j] <= limit
        let lo = i, hi = n - 1, j = i;
        while (lo <= hi) {
            const mid = (lo + hi) >> 1;
            if (nums[mid] <= limit) {
                j = mid;               // 還合法，記下來再往右找
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
        if (j - i + 1 > best) {
            best = j - i + 1;
        }
    }
    return n - best;
}
```

<details>
<summary>Go 版本</summary>

```go
func minRemoval(nums []int, k int) int {
    sort.Ints(nums)
    n := len(nums)
    best := 1
    for i := 0; i < n; i++ {
        limit := nums[i] * k
        // 找最大的 j 使 nums[j] <= limit
        j := sort.Search(n-i, func(x int) bool {
            return nums[i+x] > limit
        }) - 1 + i
        if j-i+1 > best {
            best = j - i + 1
        }
    }
    return n - best
}
```

</details>

- Time: O(n log n) 排序 + **O(n log n) 掃 + 二分**

看起來一樣是 O(n log n)，但比 sliding window 多一個 log。sliding window 的 i、j 各走 n 步共 2n，是真正的 O(n)；binary search 每個 i 都花 log n。常數上 sliding window 贏。

---

**Overthinking：`nums[i] * k` 會不會溢位？**

`nums[i]` 最大 $10^9$，`k` 最大 $10^5$，乘起來 $10^{14}$。

- Go：int 在 64-bit 平台是 int64，上限 $\approx 9.2 \times 10^{18}$。穩。
- TypeScript：Number 是 64-bit float，安全整數 `Number.MAX_SAFE_INTEGER = 2^53 ≈ 9 × 10^15`。$10^{14}$ 還在安全範圍。不用 BigInt。

如果題目把 `nums[i]` 或 `k` 再放大一個數量級，Go 還好，TypeScript 就要考慮 BigInt 或改寫條件判斷（例如 `nums[j] / k > nums[i]`，但要小心整數除法的 off-by-one）。

**Overthinking：如果題目改成「連續子陣列」不能排序呢？**

那就完全不同題了。不能排序的話，min 和 max 跟位置綁死，需要維護窗口內的 min 和 max（monotonic deque 或 multiset），然後才能用 sliding window。複雜度還是 O(n log n) 或 O(n)，但資料結構變複雜。這題允許任意刪，所以排序是合法的前置動作。

---

## 結論

刪最少 = 留最多。排序後，最佳子集一定連續，因為中間的數字是免費的。雙指針一次 pass 找最長合法區段，答案就是 n 減掉它。O(n log n)，排序以外沒辦法更快。
