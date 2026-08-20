給一個正整數陣列和一個目標值，找出最短的連續子陣列，使得子陣列的和 >= target。沒有就回傳 0。

---

**解題引導**

用 `target = 7, nums = [2, 3, 1, 2, 4, 3]` 想。

**Step 1：最直覺的做法？**<br>
<span class="spoiler">固定一個起點，往右一個一個加，加到和 >= target 就記錄長度。每個起點都試一遍。O(n²)。</span>

**Step 2：暴力解浪費在哪？**<br>
<span class="spoiler">起點從 i 換到 i+1 時，只是少了 nums[i]，但整段又從頭加一遍。中間的和被重複計算了。</span>

**Step 3：怎麼避免重複計算？**<br>
<span class="spoiler">不要從頭加，直接把 nums[i] 減掉就好。用兩個指標維護一個窗口。</span>

**Step 4：窗口什麼時候擴大？什麼時候收縮？**<br>
<span class="spoiler">和 < target 時往右擴大。和 >= target 時從左收縮，試著找更短的。</span>

**Step 5：為什麼 sliding window 在這題成立？**<br>
<span class="spoiler">所有數字都是正整數。加一個正數和一定變大，減一個正數和一定變小。這個單調性讓左右指標只需要單向移動。</span>

**Step 6：時間複雜度？**<br>
<span class="spoiler">left 和 right 各最多走 n 步，每個元素進窗口一次、出窗口一次。O(n)。</span>

想完再往下看 code。

---

## 解法一：暴力（雙層迴圈）

暴力解：固定一個起點，往右一個一個加，加到和 >= target 就停。對每個起點都做一次。

用 `target = 7, nums = [2, 3, 1, 2, 4, 3]` 走一遍。

```
起點 i=0:
  加 nums[0]=2 → sum=2  < 7
  加 nums[1]=3 → sum=5  < 7
  加 nums[2]=1 → sum=6  < 7
  加 nums[3]=2 → sum=8  >= 7 → 找到！長度 4（index 0~3）
  記錄 minLen=4

起點 i=1:
  加 nums[1]=3 → sum=3  < 7
  加 nums[2]=1 → sum=4  < 7
  加 nums[3]=2 → sum=6  < 7
  加 nums[4]=4 → sum=10 >= 7 → 找到！長度 4（index 1~4）
  minLen 還是 4，沒更短

起點 i=2:
  加 nums[2]=1 → sum=1  < 7
  加 nums[3]=2 → sum=3  < 7
  加 nums[4]=4 → sum=7  >= 7 → 找到！長度 3（index 2~4）
  更新 minLen=3

起點 i=3:
  加 nums[3]=2 → sum=2  < 7
  加 nums[4]=4 → sum=6  < 7
  加 nums[5]=3 → sum=9  >= 7 → 找到！長度 3（index 3~5）
  minLen 還是 3

起點 i=4:
  加 nums[4]=4 → sum=4  < 7
  加 nums[5]=3 → sum=7  >= 7 → 找到！長度 2（index 4~5）
  更新 minLen=2

起點 i=5:
  加 nums[5]=3 → sum=3  < 7
  加完了，sum 沒到 7 → 跳過
```

答案 = 2，子陣列 `[4, 3]`。

看到問題了嗎？起點 i=0 算了 `2+3+1+2`。起點 i=1 又重新算了 `3+1+2`。中間 `3+1+2` 這段被重複加了。每換一個起點就從頭加一遍，很浪費。

```typescript
function minSubArrayLen(target: number, nums: number[]): number {
    const n = nums.length;
    let minLen = n + 1;       // 先設一個不可能的大數，代表「還沒找到」
    for (let i = 0; i < n; i++) {
        let sum = 0;
        for (let j = i; j < n; j++) {
            sum += nums[j];
            if (sum >= target) {                      // 和夠大了
                minLen = Math.min(minLen, j - i + 1); // j-i+1 = 這段子陣列的長度（從 i 到 j）
                break;                                // 繼續加只會讓子陣列更長，不可能更短，直接跳出
            }
        }
    }
    if (minLen === n + 1) {  // 如果 minLen 沒被更新過，代表沒有任何子陣列的和 >= target
        return 0;
    }
    return minLen;
}
```

<details>
<summary>Go 版本</summary>

```go
func minSubArrayLen(target int, nums []int) int {
    n := len(nums)
    minLen := n + 1       // 先設一個不可能的大數，代表「還沒找到」
    for i := 0; i < n; i++ {
        sum := 0
        for j := i; j < n; j++ {
            sum += nums[j]
            if sum >= target {          // 和夠大了
                minLen = min(minLen, j-i+1)  // j-i+1 = 這段子陣列的長度（從 i 到 j）
                break                  // 繼續加只會讓子陣列更長，不可能更短，直接跳出
            }
        }
    }
    if minLen == n+1 {  // 如果 minLen 沒被更新過，代表沒有任何子陣列的和 >= target
        return 0
    }
    return minLen
}
```

</details>


`j - i + 1` 是什麼？從 index i 到 index j 有幾個元素。例如 i=0, j=3 → 3-0+1 = 4 個元素。

`break` 為什麼可以跳出？因為所有數字都是正整數。已經 >= target 了，再加下一個正數只會讓 sum 更大、子陣列更長。同一個起點不可能找到更短的了。

- **Time: $O(n^2)$**
- Space: O(1)

$n = 10^5$ 時 $n^2$ 是 $10^{10}$ 次操作，照 $10^7$ 次 ≈ 1 秒換算，大約 1000 秒。明確 TLE。`break` 能救平均情況，但救不了最壞情況（全是 1、target 很大時內層每次都跑到底）。這就是解法二存在的理由。

---

## 解法二：滑動視窗

Sliding window 成立的前提是所有數字都是正整數，為什麼這樣就夠，後面有一節專講。先看它怎麼跑。

---

用 `target = 7, nums = [2, 3, 1, 2, 4, 3]` 走一遍。

兩個指標：`left` 和 `right`。`sum` 追蹤窗口內的和。`minLen` 記錄最短長度。

```
nums = [2, 3, 1, 2, 4, 3]
        L
        R
```

right=0：加入 2。sum = 2。2 < 7，不夠，繼續擴大。

```
[2, 3, 1, 2, 4, 3]
 L
 R
 sum=2 < 7 → 擴大
```

right=1：加入 3。sum = 5。5 < 7，不夠。

```
[2, 3, 1, 2, 4, 3]
 L  R
 sum=5 < 7 → 擴大
```

right=2：加入 1。sum = 6。6 < 7，不夠。

```
[2, 3, 1, 2, 4, 3]
 L     R
 sum=6 < 7 → 擴大
```

right=3：加入 2。sum = 8。8 >= 7，夠了！記錄長度 4。開始收縮左邊。

```
[2, 3, 1, 2, 4, 3]
 L        R
 sum=8 >= 7 → 記錄 minLen=4，收縮左邊
```

收縮：移除 nums[0]=2。sum = 6。left=1。6 < 7，停止收縮。

```
[2, 3, 1, 2, 4, 3]
    L     R
    sum=6 < 7 → 停止收縮，繼續擴大
```

right=4：加入 4。sum = 10。10 >= 7。記錄長度 4。收縮。

```
[2, 3, 1, 2, 4, 3]
    L        R
    sum=10 >= 7 → 記錄 minLen=4，收縮
```

收縮：移除 nums[1]=3。sum = 7。left=2。7 >= 7，還夠！記錄長度 3。繼續收縮。

```
[2, 3, 1, 2, 4, 3]
       L     R
       sum=7 >= 7 → 記錄 minLen=3，繼續收縮
```

收縮：移除 nums[2]=1。sum = 6。left=3。6 < 7，停止。

```
[2, 3, 1, 2, 4, 3]
          L  R
          sum=6 < 7 → 停止收縮
```

right=5：加入 3。sum = 9。9 >= 7。記錄長度 3。收縮。

```
[2, 3, 1, 2, 4, 3]
          L     R
          sum=9 >= 7 → minLen 還是 3，收縮
```

收縮：移除 nums[3]=2。sum = 7。left=4。7 >= 7。記錄長度 2。繼續收縮。

```
[2, 3, 1, 2, 4, 3]
             L  R
             sum=7 >= 7 → 記錄 minLen=2！繼續收縮
```

收縮：移除 nums[4]=4。sum = 3。left=5。3 < 7。停止。

```
[2, 3, 1, 2, 4, 3]
                LR
                sum=3 < 7 → 停止
```

right 到底了。結束。答案 = 2，就是子陣列 `[4, 3]`。

---

完整程式碼：

```typescript
function minSubArrayLen(target: number, nums: number[]): number {
    const n = nums.length;
    let minLen = n + 1;
    let sum = 0;
    let left = 0;

    for (let right = 0; right < n; right++) {
        sum += nums[right]; // 右邊進入窗口

        while (sum >= target) { // 窗口夠大了，收縮
            minLen = Math.min(minLen, right - left + 1);
            sum -= nums[left]; // 左邊離開窗口
            left++;
        }
    }

    return minLen === n + 1 ? 0 : minLen;
}
```

<details>
<summary>Go 版本</summary>

```go
func minSubArrayLen(target int, nums []int) int {
    n := len(nums)
    minLen := n + 1 // 不可能的大數，代表「還沒找到」
    sum := 0
    left := 0

    for right := 0; right < n; right++ {
        sum += nums[right] // 右邊進入窗口

        for sum >= target { // 窗口夠大了，試著收縮
            minLen = min(minLen, right-left+1)
            sum -= nums[left] // 左邊離開窗口
            left++
        }
    }

    if minLen == n+1 { // 從來沒找到和 >= target 的子陣列
        return 0
    }
    return minLen
}
```

</details>


- Time: O(n)
- Space: O(1)

`left` 和 `right` 各最多走 n 步。每個元素進窗口一次、出窗口一次。總共 2n 次操作。

---

**為什麼 sliding window 在這題成立？**

因為所有數字都是正整數。

窗口加一個正數，和一定變大。窗口減一個正數，和一定變小。這個單調性保證了：

1. 和 < target → 左邊縮沒用（只會更小），只能往右擴
2. 和 >= target → 右邊擴沒用（只會更長），應該往左縮

所以 `left` 永遠不需要往回走。兩個指標都是單向移動。O(n)。

---

## 解法三：前綴和 + 二分搜

題目 follow-up 問 O(n log n) 解法。

**怎麼不重算就拿到任意一段的和？** 想像你在開車，`nums` 是每小時開的公里數，`prefix[i]` 是開完前 i 小時的里程表讀數（遞推 `prefix[i+1] = prefix[i] + nums[i]`；多出來的 `prefix[0] = 0` 是出發前的讀數，所以 prefix 比 nums 長一格）：

```
每小時開：       [2, 3, 1, 2,  4,  3]
                 0  1  2  3   4   5    ← nums 的 index

里程表讀數：  [0, 2, 5, 6, 8, 12, 15]
              0  1  2  3  4   5   6    ← prefix 的 index
```

它換來的能力：**任何一段的和，變成兩個讀數相減**，不用重新加中間每一段。第 3 到第 5 小時開了多遠？`prefix[5] - prefix[2] = 12 - 5 = 7`，就是 `[1,2,4]` 的和。一般式：`nums[i..j-1]` 的和 = `prefix[j] - prefix[i]`。

**題目翻成里程表的語言是什麼？** 「最短的連續段，和 >= 7」變成：找兩個讀數，相差 >= 7、相隔小時數最少。固定左讀數 `prefix[i]`，就是**往右找第一個到達 `prefix[i] + 7` 的讀數**，第一個到達的就是最近的。

**為什麼可以二分搜？** 全正數，車一直往前開，讀數嚴格遞增。在排好序的序列裡找「第一個 >= 某值的位置」，是 [704 Binary Search](/problem/binary-search) 的 lower bound 變形。每個起點搜 O(log n)，n 個起點總共 O(n log n)。

逐起點走（`need = prefix[i] + 7`，答案是二分搜結束時的 `lo`，長度 = `lo - i`）：

```
i=0: need=7  → 在 [2,5,6,8,12,15] 找第一個 >= 7 → lo=4（prefix[4]=8），長度 4
i=1: need=9  → lo=5（prefix[5]=12），長度 4
i=2: need=12 → lo=5，長度 3
i=3: need=13 → lo=6（prefix[6]=15），長度 3
i=4: need=15 → lo=6，長度 2  ← 最短！
i=5: need=19 → 讀數最大 15，找不到 → lo 衝到 7 超出範圍，跳過
              （code 尾端的 if (lo <= n) 擋的就是這個）
```

**lo/hi 怎麼收斂到答案？** 展開 i=0 那輪（`lo=1, hi=6`，need=7）：

```
第 1 圈：mid=3，prefix[3]=6 < 7   → 太小，mid 以左全淘汰 → lo=4
第 2 圈：mid=5，prefix[5]=12 >= 7 → 達標但不收工，左邊可能有更早的 → hi=4
第 3 圈：mid=4，prefix[4]=8 >= 7  → 繼續往左壓 → hi=3
lo=4 > hi=3，結束。lo=4。
```

第 2 圈是 lower bound 的關鍵：達標不等於第一個達標，左邊還藏著 prefix[4]=8。範圍每圈砍半（6 → 3 → 1），圈數 = log₂(n)。最後一圈必是 `lo === hi`。判成「太小」就 `lo = hi + 1`，lo 越過 hi，i=5 的收尾走的是這條。判成「達標」就 `hi = lo - 1`，lo 留在原地，上面第 3 圈走的是這條。兩條路都收在 `lo === hi + 1`。不變量是 **lo 左邊全太小、lo 位置起全達標**，所以結束直接讀 lo。

答案 = 2，子陣列 `nums[4..5] = [4, 3]`。跟 sliding window 一樣。

```typescript
function minSubArrayLen(target: number, nums: number[]): number {
    const n = nums.length;
    const prefix = new Array(n + 1).fill(0);
    for (let i = 0; i < n; i++) {
        prefix[i + 1] = prefix[i] + nums[i];
    }

    let minLen = n + 1;
    for (let i = 0; i <= n; i++) {
        const need = prefix[i] + target;
        // 二分搜結束時，lo = 第一個 >= need 的位置
        let lo = i + 1, hi = n;
        while (lo <= hi) {
            const mid = Math.floor((lo + hi) / 2);
            if (prefix[mid] >= need) {
                hi = mid - 1;
            } else {
                lo = mid + 1;
            }
        }
        if (lo <= n) {
            minLen = Math.min(minLen, lo - i);
        }
    }

    if (minLen === n + 1) {
        return 0;
    }
    return minLen;
}
```

<details>
<summary>Go 版本</summary>

```go
func minSubArrayLen(target int, nums []int) int {
    n := len(nums)
    prefix := make([]int, n+1)
    for i := 0; i < n; i++ {
        prefix[i+1] = prefix[i] + nums[i]
    }

    minLen := n + 1
    for i := 0; i <= n; i++ {
        need := prefix[i] + target
        // 二分搜結束時，lo = 第一個 >= need 的位置
        lo, hi := i+1, n
        for lo <= hi {
            mid := (lo + hi) / 2
            if prefix[mid] >= need {
                hi = mid - 1
            } else {
                lo = mid + 1
            }
        }
        if lo <= n {
            minLen = min(minLen, lo-i)
        }
    }

    if minLen == n+1 {
        return 0
    }
    return minLen
}
```

</details>


- Time: O(n log n)
- Space: O(n)

時間慢了，空間多了，這題它輸 sliding window。存在的意義在別處：sliding window 的前提是「擠掉一個數，和一定變小」，一出現負數這條就不成立；里程表相減永遠成立，負數只是里程會倒退。所以有負數的變形題，要從 prefix sum 這條路走（搭 monotonic deque 可以回到 O(n)）。解法三是在幫未來的題鋪路，不是這題的最優解。

---

**Overthinking：如果陣列有負數呢？**

`nums = [2, -1, 3, -2, 5]`，target = 4。

Sliding window 失效。窗口和 >= target 時縮左邊，但移掉負數反而讓和變大。單調性沒了。

正確做法：prefix sum + monotonic deque。O(n)。概念比較進階，但面試官追問時能講出「正整數保證單調性，有負數要換方法」就是加分。

---

## 結論

正整數陣列，sliding window。O(n) 時間 O(1) 空間。核心是正數保證了「加一定變大、減一定變小」的單調性，讓左右指標各走一遍就夠。
