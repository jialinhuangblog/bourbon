給一個陣列和一個目標值，找出哪兩個數加起來等於目標值，回傳它們的 index。

---

## 解法一：暴力（兩層迴圈）

每個人第一次看到 Two Sum，都寫兩層迴圈。

```typescript
for (let i = 0; i < nums.length; i++) {          // 每個數字
    for (let j = i + 1; j < nums.length; j++) {  // 跟後面的每個數字配對
        if (nums[i] + nums[j] === target) {       // 加起來等於 target？
            return [i, j];                        // 找到了，回傳兩個 index
        }
    }
}
```

- **Time: $O(n^2)$**
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
for i := 0; i < len(nums); i++ {        // 每個數字
    for j := i + 1; j < len(nums); j++ { // 跟後面的每個數字配對
        if nums[i]+nums[j] == target {    // 加起來等於 target？
            return []int{i, j}            // 找到了，回傳兩個 index
        }
    }
}
```

</details>

能跑。但 n = $10^4$ 就是一億次比較。面試官不會滿意。

---

## 解法二：Hash map

內層迴圈在做什麼？找 `target - nums[i]` 有沒有出現過。每次從頭掃一遍來找。很浪費。

用 hash map 記住「見過的值 → index」。查一次 O(1)。

```typescript
function twoSum(nums: number[], target: number): number[] {
    const seen = new Map<number, number>(); // 值 → index
    for (let i = 0; i < nums.length; i++) {
        const j = seen.get(target - nums[i]); // 差值之前出現過嗎？
        if (j !== undefined) return [j, i];    // 出現過，它的 index 跟現在這個就是答案
        seen.set(nums[i], i);                  // 沒出現過，把自己記起來
    }
    return [];
}
```

- Time: O(n)
- Space: O(n)

<details>
<summary>Go 版本</summary>

```go
func twoSum(nums []int, target int) []int {
    seen := make(map[int]int)          // 建一個 map：值 → index
    for i, n := range nums {           // 走過每個數字
        if j, ok := seen[target-n]; ok { // 差值之前出現過嗎？
            return []int{j, i}          // 出現過，它的 index 跟現在這個就是答案
        }
        seen[n] = i                     // 沒出現過，把自己記進 map
    }
    return nil
}
```

</details>

一次遍歷就結束。

---

## 解法三：排序＋雙指針

直覺會想：排序加雙指標。排序後一左一右夾，O(n log n) 時間，空間好像很省？

兩個問題。

第一，排序打亂了原始 index。題目要回傳 index，所以得先把 index 存起來。光是 `pairs` 陣列就吃 O(n) 空間。

第二，`sort.Slice` 本身是 in-place（Go 底層用 introsort），不額外吃空間。但時間是 O(n log n)，比 hash map 的 O(n) 慢。

```typescript
function twoSum(nums: number[], target: number): number[] {
    // 把值和原始 index 綁在一起
    const pairs = nums.map((v, i) => ({ val: v, idx: i })); // 每個元素存 (值, 原始index)

    // 按值排序
    pairs.sort((a, b) => a.val - b.val);

    // 雙指標：一左一右往中間夾
    let lo = 0, hi = pairs.length - 1;
    while (lo < hi) {
        const sum = pairs[lo].val + pairs[hi].val;
        if (sum === target) {                          // 剛好等於 target
            return [pairs[lo].idx, pairs[hi].idx];      // 回傳原始 index
        } else if (sum < target) {
            lo++;                                       // 太小，左邊往右
        } else {
            hi--;                                       // 太大，右邊往左
        }
    }
    return [];
}
```

- Time: O(n log n)
- Space: O(n) — 存 `(value, index)` 配對

<details>
<summary>Go 版本</summary>

```go
func twoSum(nums []int, target int) []int {
    // 把值和原始 index 綁在一起
    type pair struct{ val, idx int }
    pairs := make([]pair, len(nums))     // 建一個 pair 陣列
    for i, v := range nums {
        pairs[i] = pair{v, i}            // 每個元素存 (值, 原始index)
    }

    // 按值排序
    sort.Slice(pairs, func(i, j int) bool {
        return pairs[i].val < pairs[j].val
    })

    // 雙指標：一左一右往中間夾
    lo, hi := 0, len(pairs)-1
    for lo < hi {
        sum := pairs[lo].val + pairs[hi].val
        if sum == target {                        // 剛好等於 target
            return []int{pairs[lo].idx, pairs[hi].idx} // 回傳原始 index
        } else if sum < target {
            lo++                                  // 太小，左邊往右
        } else {
            hi--                                  // 太大，右邊往左
        }
    }
    return nil
}
```

</details>

空間還是 O(n)，時間比 hash map 慢，code 還多寫一倍，不值得。

---

**Overthinking：如果題目只要回傳「值」呢？**

這是一個常見的變形。不要 index，只要兩個數字本身。

排序就不用額外存 index 了。直接 in-place 排序 + 雙指標。

```typescript
function twoSumValues(nums: number[], target: number): number[] {
    nums.sort((a, b) => a - b);              // in-place 排序，不需要額外空間
    let lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        const sum = nums[lo] + nums[hi];
        if (sum === target) {
            return [nums[lo], nums[hi]];      // 回傳值，不是 index
        } else if (sum < target) {
            lo++;
        } else {
            hi--;
        }
    }
    return [];
}
```

- Time: O(n log n)
- Space: O(1) — 不需要額外空間

<details>
<summary>Go 版本</summary>

```go
func twoSumValues(nums []int, target int) []int {
    sort.Ints(nums)                      // in-place 排序，不需要額外空間
    lo, hi := 0, len(nums)-1
    for lo < hi {
        sum := nums[lo] + nums[hi]
        if sum == target {
            return []int{nums[lo], nums[hi]} // 回傳值，不是 index
        } else if sum < target {
            lo++
        } else {
            hi--
        }
    }
    return nil
}
```

</details>

看起來贏了？沒有。原題要 index。排序破壞 index。這條路走不通。

但如果面試官追問「不用額外空間能不能解？」，你可以反問：「可以只回傳值嗎？」這展示你理解問題的邊界在哪。

---

## 結論

Hash map。O(n) 時間 O(n) 空間。沒有更好的了。認清這點也是一種能力。
