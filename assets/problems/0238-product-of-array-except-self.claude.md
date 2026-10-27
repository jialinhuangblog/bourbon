給一個整數陣列，回傳一個新陣列，每個位置的值是「原陣列中除了自己以外所有數的乘積」。不能用除法。

---

## 解法一：暴力（雙層迴圈）

最直覺的做法：對每個位置，把其他所有元素乘起來。

```typescript
function productExceptSelf(nums: number[]): number[] {
    const n = nums.length;
    const ans = new Array(n);
    for (let i = 0; i < n; i++) {
        let product = 1;
        for (let j = 0; j < n; j++) { // 把所有不是自己的乘起來
            if (j !== i) {
                product *= nums[j];
            }
        }
        ans[i] = product;
    }
    return ans;
}
```

- **Time: $O(n^2)$**
- Space: O(1)（不算輸出）

<details>
<summary>Go 版本</summary>

```go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    ans := make([]int, n)
    for i := 0; i < n; i++ {
        product := 1
        for j := 0; j < n; j++ { // 把所有不是自己的乘起來
            if j != i {
                product *= nums[j]
            }
        }
        ans[i] = product
    }
    return ans
}
```

</details>

題目明說要 O(n)。這個過不了。

---

## 解法二：前綴積

內層迴圈在做什麼？把左邊的乘一遍、右邊的乘一遍。每次都重算。很浪費。

先講一個概念：prefix sum（前綴和）。`prefix[i]` 是「前 i 個數字的總和」。每多一個數字，不用從頭加，只要 `prefix[i] = prefix[i-1] + nums[i]`。

Prefix product（前綴積）一模一樣，只是把加法換成乘法。`prefix[i]` 是「前 i 個數字的乘積」。每多一個數字，`prefix[i] = prefix[i-1] × nums[i]`。

用 `[1, 2, 3, 4]` 舉例：

```
prefix sum:     [1, 3, 6, 10]     1, 1+2, 1+2+3, 1+2+3+4
prefix product: [1, 2, 6, 24]     1, 1×2, 1×2×3, 1×2×3×4
```

回到這題。`answer[i]` 要的是「除了自己以外所有數的乘積」。拆開來看：

```
answer[i] = 左邊所有數的乘積 × 右邊所有數的乘積
```

用 `nums = [1, 2, 3, 4]` 畫出來：

```
i=0:  左邊( )      × 右邊(2×3×4)  = 24
i=1:  左邊(1)      × 右邊(3×4)    = 12
i=2:  左邊(1×2)    × 右邊(4)      = 8
i=3:  左邊(1×2×3)  × 右邊( )      = 6
```

左邊的乘積，就是一個從左到右的 prefix product。右邊的乘積，就是一個從右到左的 prefix product。各跑一次就好。

---

用 `nums = [1, 2, 3, 4]` 走一遍。

第一輪：從左到右，算每個位置「左邊所有數的乘積」。

```
nums = [1, 2, 3, 4]

i=0: 左邊沒有數字           → ans[0] = 1
i=1: 左邊有 [1]            → ans[1] = 1
i=2: 左邊有 [1, 2]         → ans[2] = 1 × 2 = 2
i=3: 左邊有 [1, 2, 3]      → ans[3] = 1 × 2 × 3 = 6

ans = [1, 1, 2, 6]  ← 左邊乘積
```

每一步怎麼算？不用重新乘。`ans[i] = ans[i-1] × nums[i-1]`。上一步的結果乘一個數就好。

```
ans[0] = 1             （base case）
ans[1] = ans[0] × nums[0] = 1 × 1 = 1
ans[2] = ans[1] × nums[1] = 1 × 2 = 2
ans[3] = ans[2] × nums[2] = 2 × 3 = 6
```

---

第二輪：從右到左，算每個位置「右邊所有數的乘積」，直接乘進 `ans`。

```
nums = [1, 2, 3, 4]
ans  = [1, 1, 2, 6]  ← 第一輪結果（左邊乘積）
right = 1             ← 追蹤右邊乘積的變數

i=3: 右邊沒有數字           → right = 1
     ans[3] = 6 × 1 = 6    → 左 × 右 = 答案 ✓

i=2: right = 1 × nums[3] = 1 × 4 = 4
     ans[2] = 2 × 4 = 8    → 左(1×2) × 右(4) = 8 ✓

i=1: right = 4 × nums[2] = 4 × 3 = 12
     ans[1] = 1 × 12 = 12  → 左(1) × 右(3×4) = 12 ✓

i=0: right = 12 × nums[1] = 12 × 2 = 24
     ans[0] = 1 × 24 = 24  → 左(無) × 右(2×3×4) = 24 ✓

ans = [24, 12, 8, 6]  ← 最終答案
```

驗證一下：

```
nums[0]=1: 2×3×4 = 24  ✓
nums[1]=2: 1×3×4 = 12  ✓
nums[2]=3: 1×2×4 = 8   ✓
nums[3]=4: 1×2×3 = 6   ✓
```

---

完整程式碼：

```typescript
function productExceptSelf(nums: number[]): number[] {
    const n = nums.length;
    const ans = new Array(n);

    // 從左到右，累積左邊的乘積
    ans[0] = 1;
    for (let i = 1; i < n; i++) {
        ans[i] = ans[i - 1] * nums[i - 1]; // 上一步的結果 × 前一個數
    }

    // 從右到左，累積右邊的乘積
    // n-2 = 倒數第二個，因為最後一個右邊沒有數字，不用處理
    let right = 1;
    for (let i = n - 2; i >= 0; i--) {
        right *= nums[i + 1]; // 右邊乘積多乘一個數
        ans[i] *= right;       // 左 × 右 = 答案
    }

    return ans;
}
```

- Time: O(n)
- Space: O(1)（不算輸出）

<details>
<summary>Go 版本</summary>

```go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    ans := make([]int, n)

    // 第一輪：從左到右，累積左邊的乘積
    ans[0] = 1
    for i := 1; i < n; i++ {
        ans[i] = ans[i-1] * nums[i-1] // 上一步的結果 × 前一個數
    }

    // 第二輪：從右到左，累積右邊的乘積，直接乘進去
    // 為什麼從 n-2 開始？
    // 最後一個元素是 ans[n-1]，它的右邊沒有數字，right = 1，乘了等於沒乘
    // 所以跳過 ans[n-1]，直接從倒數第二個 ans[n-2] 開始
    right := 1
    for i := n - 2; i >= 0; i-- {
        right *= nums[i+1]  // 右邊乘積多乘一個數
        ans[i] *= right      // 左 × 右 = 答案
    }

    return ans
}
```

</details>

---

## 解法三：除法

直覺會想：用除法。先算全部的乘積 `total`，再對每個位置做 `total / nums[i]`。

兩個問題。

第一，題目明確禁止除法。

第二，遇到 0 就爆炸。`nums = [1, 2, 0, 4]`，`total = 0`，`0 / nums[i]` 全部是 0。錯了。要分三種 case 處理：

```
case 1: 沒有 0      → ans[i] = total / nums[i]
case 2: 剛好一個 0  → 只有 0 的位置有答案，其他都是 0
case 3: 兩個以上 0  → 全部都是 0
```

```typescript
// 除法解（題目禁止，但理解為什麼不行很重要）
function productExceptSelf(nums: number[]): number[] {
    let total = 1;
    let zeroCount = 0;
    for (const n of nums) {
        if (n === 0) {
            zeroCount++;
        } else {
            total *= n;
        }
    }

    const ans = new Array(nums.length);
    for (let i = 0; i < nums.length; i++) {
        const n = nums[i];
        if (zeroCount > 1) {
            ans[i] = 0;              // 兩個以上的 0，全部是 0
        } else if (zeroCount === 1) {
            if (n === 0) {
                ans[i] = total;      // 唯一的 0 的位置
            } else {
                ans[i] = 0;          // 有一個 0 所以乘積是 0
            }
        } else {
            ans[i] = total / n;      // 沒有 0，正常除
        }
    }
    return ans;
}
```

<details>
<summary>Go 版本</summary>

```go
// 除法解（題目禁止，但理解為什麼不行很重要）
func productExceptSelf(nums []int) []int {
    total := 1
    zeroCount := 0
    for _, n := range nums {
        if n == 0 {
            zeroCount++
        } else {
            total *= n
        }
    }

    ans := make([]int, len(nums))
    for i, n := range nums {
        if zeroCount > 1 {
            ans[i] = 0              // 兩個以上的 0，全部是 0
        } else if zeroCount == 1 {
            if n == 0 {
                ans[i] = total      // 唯一的 0 的位置
            } else {
                ans[i] = 0          // 有一個 0 所以乘積是 0
            }
        } else {
            ans[i] = total / n      // 沒有 0，正常除
        }
    }
    return ans
}
```

</details>

三個 if 分支，還違反題目規則。不值得。

---

**Overthinking：如果是二維陣列呢？**

面試官可能追問：給一個 $m \times n$ 矩陣，`answer[i][j]` 等於同一行中除了自己以外所有數的乘積 × 同一列中除了自己以外所有數的乘積。

用一個 $2 \times 3$ 矩陣走一遍：

```
matrix = [[1, 2, 3],
          [4, 5, 6]]
```

先對每一行做一維的 product except self：

```
row 0: [1, 2, 3] → rowExcept = [2×3, 1×3, 1×2] = [6, 3, 2]
row 1: [4, 5, 6] → rowExcept = [5×6, 4×6, 4×5] = [30, 24, 20]
```

再對每一列做一維的 product except self：

```
col 0: [1, 4] → colExcept = [4, 1]
col 1: [2, 5] → colExcept = [5, 2]
col 2: [3, 6] → colExcept = [6, 3]
```

最後組合：`answer[i][j] = rowExcept[i][j] × colExcept[i][j]`

```
answer[0][0] = rowExcept[0][0] × colExcept[0][0] = 6 × 4 = 24
answer[0][1] = 3 × 5 = 15
answer[0][2] = 2 × 6 = 12
answer[1][0] = 30 × 1 = 30
answer[1][1] = 24 × 2 = 48
answer[1][2] = 20 × 3 = 60
```

驗證 `answer[0][0]`：同一行除了自己 = $2 \times 3 = 6$，同一列除了自己 = 4。$6 \times 4 = 24$ ✓

每一行、每一列都是一次一維的 product except self。行做 m 次，列做 n 次。本質跟一維完全一樣，只是多跑幾遍。

---

## 結論

Prefix product = 把加法換成乘法的 prefix sum。這題的核心是「乘積可以拆成左半和右半」，左掃一次右掃一次。每一步只需要上一步的結果乘一個數，不用從頭算。
