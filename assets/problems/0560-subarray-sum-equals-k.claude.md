給陣列跟 k，數有幾個連續子陣列的和等於 k。

「我不知道你在哪，但我知道你存在過 — 所以你跟我之間一定夾著一段和為 k 的子陣列。」

---

**解題引導**

用 `nums = [1, 2, 3, -2, 5], k = 3` 思考。答案是 4：`[1,2]`、`[3]`、`[2,3,-2]`、`[-2,5]`。

**Step 1：能不能 sliding window？**

*With negatives in the array, expanding right doesn't always increase the sum.*

<span class="spoiler">不行。有負數，sliding window 的單調性不成立，往右擴窗總和可能變小，left/right 誰該動沒依據。 / No. Negatives break monotonicity — expanding right can decrease the sum, so there's no rule for when to move left or right.</span>

**Step 2：暴力怎麼寫？**

*Try every starting point, keep adding right, check each time.*

<span class="spoiler">枚舉每個起點 i，從 i 累加到 j，每次累加後檢查是否等於 k。O(n²)。 / Fix a start i, extend j right while accumulating, check at each step. O(n²).</span>

**Step 3：走到位置 j 時，你真正在問的問題是什麼？**

*You know the total so far. What past total would make the gap exactly k?*

<span class="spoiler">「之前有沒有某個位置，當時累積值 = 現在累積值 - k？」如果有，從那個位置到現在這段的和就是 k。 / Has any past position had a running sum equal to (current - k)? If yes, the gap between that point and now sums to k.</span>

**Step 4：「之前有沒有某個值」→ 什麼資料結構？**

*You've seen this question before.*

<span class="spoiler">hashmap。存「每個累積值出現過幾次」。走到 j，查 map 有幾個值等於「當前累積 - k」，有幾個就代表有幾個合法起點。 / A hashmap storing how many times each running sum has appeared. At j, look up (running - k) — the count is the number of valid subarrays ending here.</span>

**Step 5：hashmap 要存 index 還是 count？**

*The same running sum can appear more than once — each is a different starting point.*

<span class="spoiler">count。因為同一個累積值可能出現多次（有負數時），每一次都是一個合法起點，全部都要算。 / Count. The same running sum can appear at multiple positions — each one is a distinct valid starting point, all must be counted.</span>

**Step 6：map 初始化要放什麼？**

*What if the answer subarray starts from index 0?*

<span class="spoiler">map 放一組：key 是 0、value 是 1。代表「從頭什麼都還沒加時，累積是 0，這算一次出現」。如果當前累積剛好等於 k，查「當前 - k」就是 0，這一次必須找得到，才能算進「從 index 0 開始到現在」這個子陣列。 / Seed the map with {0: 1}. It represents the state before index 0 — a running sum of zero, which already happened once. Without it, subarrays starting from index 0 would be missed.</span>

想完再往下看 code。

---

## 解法一：暴力（枚舉起點）

先暴力。枚舉起點 i，從 i 累加到 j，每次檢查。

```typescript
function subarraySum(nums: number[], k: number): number {
    let count = 0;
    for (let i = 0; i < nums.length; i++) {
        let sum = 0;
        for (let j = i; j < nums.length; j++) {
            sum += nums[j];
            if (sum === k) count++;
        }
    }
    return count;
}
```

<details>
<summary>Go 版本</summary>

```go
func subarraySum(nums []int, k int) int {
    count := 0
    for i := 0; i < len(nums); i++ {
        sum := 0
        for j := i; j < len(nums); j++ {
            sum += nums[j]        // 從 i 開始累加，不用每次從頭算
            if sum == k {
                count++
            }
        }
    }
    return count
}
```

</details>

走 `nums = [1, 2, 3, -2, 5], k = 3`，只挑命中的列出：

| i | j | 子陣列 | sum |
|---|---|--------|-----|
| 0 | 1 | `[1,2]` | 3 ✓ |
| 1 | 3 | `[2,3,-2]` | 3 ✓ |
| 2 | 2 | `[3]` | 3 ✓ |
| 3 | 4 | `[-2,5]` | 3 ✓ |

答案 4。

- Time: **$O(n^2)$**
- Space: O(1)

---

## 解法二：前綴和 + hashmap

想像你開車從 A 出發，每經過一個城市就記下里程表讀數：A=0, B=1, C=3, D=6, E=4, F=9。

現在站在 F（里程 9），想找「哪一段路剛好是 k=3」。不用回頭把每段都量一遍 — 直接問：**「之前有沒有哪個城市的里程是 9-3=6？」** 有的話，從那裡到 F 就是 3。

這就是從左到右掃一次的思路。走到每個位置時，記下累積總和（里程），然後查「之前有沒有 running-k 這個值出現過」。

```
nums =     [1, 2, 3, -2, 5]
累積(走到該位置後) =
            1  3  6   4  9
```

從 index i 到 index j（含）的子陣列和
= (走到 j 後的累積) - (走到 i-1 後的累積)

舉例：想要 `[2, 3, -2]`（index 1..3）的和：
= 走到 3 後累積 - 走到 0 後累積
= 4 - 1 = 3 ✓

所以問題變成：**對每個 j，問「之前有沒有某個累積值 = 當前累積 - k？」**

舉例：走到 index 3，當前累積 = 4，k = 3。查「之前有沒有累積值 = 4 - 3 = 1？」有 — 走到 index 0 時累積就是 1。命中。

「之前有沒有某個值」= hashmap。走到一個位置：
1. 算當前累積
2. 查 map：`當前 - k` 出現過幾次？加進答案
3. 把當前累積記進 map（次數 +1）

```typescript
function subarraySum(nums: number[], k: number): number {
    let count = 0;
    let running = 0;
    const seen = new Map<number, number>();
    seen.set(0, 1);             // 重要：起點前的「空累積 = 0」算一次
    for (const n of nums) {
        running += n;
        count += seen.get(running - k) ?? 0;
        seen.set(running, (seen.get(running) ?? 0) + 1);
    }
    return count;
}
```

<details>
<summary>Go 版本</summary>

```go
func subarraySum(nums []int, k int) int {
    count := 0
    running := 0
    seen := map[int]int{0: 1}   // 重要：起點前的「空累積 = 0」算一次
    for _, n := range nums {
        running += n
        count += seen[running-k] // 之前有幾個位置的累積等於 running-k
        seen[running]++          // 記下當前累積，給後面的人查
    }
    return count
}
```

</details>

走 `nums = [1, 2, 3, -2, 5], k = 3`：

| 讀到 | running | 要找的值 (running - k) | 之前出現過幾次 | count | map 更新後 |
|------|--------|------------------|--------------|-------|-------------|
| (init) | 0 | | | 0 | `{0:1}` |
| 1 | 1 | -2 | 0 | 0 | `{0:1, 1:1}` |
| 2 | 3 | **0** | **1** | 1 | `{0:1, 1:1, 3:1}` |
| 3 | 6 | **3** | **1** | 2 | `{0:1, 1:1, 3:1, 6:1}` |
| -2 | 4 | **1** | **1** | 3 | `{..., 4:1}` |
| 5 | 9 | **6** | **1** | 4 | `{..., 9:1}` |

每一次 count 增加對應的子陣列：

- 讀到 2 時 running=3，查到 0 → 子陣列 `[1, 2]`（從開頭到這裡）
- 讀到 3 時 running=6，查到 3 → 子陣列 `[3]`（從累積=3 之後到這裡）
- 讀到 -2 時 running=4，查到 1 → 子陣列 `[2, 3, -2]`（從累積=1 之後到這裡）
- 讀到 5 時 running=9，查到 6 → 子陣列 `[-2, 5]`（從累積=6 之後到這裡）

答案 4。一次 pass。

- Time: **O(n)**
- Space: O(n)

**為什麼 map 要 `{0: 1}` 初始化？**

如果某個 running 剛好 = k，對應的子陣列是「從 index 0 到現在」，起點前的累積是 0。但我們還沒走進迴圈記過 0，所以要預先放進去。

上面 table 第一個命中：讀到 2 時 running=3，查 `3 - 3 = 0`。這個 0 就是預先放的。拿掉初始化就漏掉 `[1, 2]` 這個答案。

**為什麼存 count 不存 index？**

一個 running 值可能出現好幾次，每一次都代表一個獨立的合法起點。

用 `nums = [1, -1, 1, 2], k = 1` 示範：

| 讀到 | running | 要找的值 (running - k) | 之前出現幾次 | count | map 更新後 |
|------|---------|------------------------|-------------|-------|------------|
| (init) | 0 | | | 0 | {0:1} |
| 1 | 1 | 0 | 1 | 1 | {0:1, 1:1} |
| -1 | 0 | -1 | 0 | 1 | **{0:2**, 1:1} |
| **1** | **1** | **0** | **2** | **3** | {0:2, 1:2} |
| 2 | 3 | 2 | 0 | 3 | {0:2, 1:2, 3:1} |

第三行（讀到第二個 1）：要找 running-k = 0，map 裡 0 出現了 **2** 次 → count 一次加 2。

為什麼 0 出現兩次？
- 第一次：init 放的（代表「從一開始就累積 = 0」）
- 第二次：讀到 -1 後 running 歸 0，存進 map

對應兩條子陣列：
- 從「init 的 0」到現在 → `[1, -1, 1]`，sum=1 ✓
- 從「-1 之後的 0」到現在 → `[1]`，sum=1 ✓

如果 map 只存 index（或只記「有/沒有」），兩條只能算一條。存 count 才能一次抓到全部配對。

---

## 結論

「連續子陣列 + 可有負數 + 和等於 k」 → 累積和 + hashmap 查配對。套路固定。
