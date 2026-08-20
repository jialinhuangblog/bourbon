房子排成一圈，第一間和最後一間相鄰，不能搶相鄰兩間，求最大金額。

---

**解題引導**

前提：你已經會解 [198 House Robber](/problem/house-robber)（直線版）。

用 `[2, 3, 2]` 想。

**Step 1：圓形跟直線差在哪？**<br>
<span class="spoiler">第一間和最後一間變成鄰居。直線版可以同時搶頭尾，圓形不行。</span>

**Step 2：所以限制是什麼？**<br>
<span class="spoiler">頭和尾不能同時搶。至少有一個不搶。</span>

**Step 3：「至少一個不搶」有幾種情況？**<br>
<span class="spoiler">兩種：不搶頭（看 nums[1..n-1]），或不搶尾（看 nums[0..n-2]）。</span>

**Step 4：每種情況是什麼形狀？**<br>
<span class="spoiler">直線。每個子問題就是 198 House Robber。</span>

**Step 5：答案怎麼來？**<br>
<span class="spoiler">跑兩次直線版 House Robber，取 max。試著寫寫看。</span>

想完再往下看 code。

---

## 解法一：暴力（位元列舉）

跟 [198 House Robber](/problem/house-robber) 一樣的題，差在房子變成圓形。第一間和最後一間是鄰居，不能同時搶。

暴力解：每間房子選或不選，窮舉所有組合。只要不相鄰、頭尾不同時選，就是合法的。取最大金額。

用 `nums = [2, 3, 2]` 走一遍（3 間房，排成一圈）：

```
房子編號：  0   1   2
金額：     [2,  3,  2]
相鄰關係：  0-1, 1-2, 2-0（圓形，頭尾也相鄰）

所有組合（每間選或不選）：
  選 {}         → 金額 0
  選 {0}        → 金額 2
  選 {1}        → 金額 3
  選 {2}        → 金額 2
  選 {0,1}      → ✗ 相鄰
  選 {1,2}      → ✗ 相鄰
  選 {0,2}      → ✗ 頭尾相鄰（圓形！）
  選 {0,1,2}    → ✗ 相鄰

合法的：{}, {0}, {1}, {2}
最大金額 = 3（只搶第 1 間）
```

每間房子選或不選，$2^n$ 種組合。n = 20 就超過一百萬種。

組合怎麼表示？用一個整數的二進位：每間房子一個 bit，搶是 1，不搶是 0。3 間房就是 3 個 bit，把它們當成一個數字：

```
mask = 0   二進位 000   都不搶
mask = 1   二進位 001   只搶第 0 間
mask = 2   二進位 010   只搶第 1 間
mask = 5   二進位 101   搶第 0 間和第 2 間
mask = 7   二進位 111   三間全搶
```

bit 由右往左數，最右邊是第 0 間。

`1 << n` 是把 1 往左移 n 格。左移一格等於乘 2，移三格就是 $2 \times 2 \times 2 = 8$，所以 `1 << n` 就是 $2^n$。`mask < (1 << n)` 也就是 `mask < 8`：從 0 數到 7，剛好把 3 個 bit 能表示的每一種組合走完。mask = 8 是二進位 `1000`，用到第 4 個 bit，可是只有 3 間房，沒有意義。

`(mask >> i) & 1` 是檢查清單上的第 i 格。用 mask = 5（`101`）檢查第 2 間有沒有搶：

```
mask      = 101
mask >> 2 = 001   右移 2 格，把第 2 個 bit 移到最右邊
001 & 1   = 1     只保留最右邊那格，其他歸零 → 有搶
```

檢查第 1 間的話：`101 >> 1 = 010`，`010 & 1 = 0`，沒搶。右移是把要檢查的那格移到最右邊，`& 1` 是把其他 bit 全部歸零。`& 1` 不能省：`if (mask >> 1)` 問的是「右移完還有沒有任何 bit」，mask = 4（`100`，只搶第 2 間）右移 1 格是 `10`，非零，會被誤判成第 1 間有搶。

```typescript
function rob(nums: number[]): number {
    const n = nums.length;
    let best = 0;
    for (let mask = 0; mask < (1 << n); mask++) { // 窮舉所有選法
        let valid = true;
        let total = 0;
        for (let i = 0; i < n; i++) {
            if ((mask >> i) & 1) {            // 第 i 間有搶嗎？
                const next = (i + 1) % n;     // 找到鄰居（圓形：最後一間的鄰居是第 0 間）
                if ((mask >> next) & 1) {     // 第 next 間也有搶？不合法，排除
                    valid = false;
                    break;
                }
                total += nums[i];
            }
        }
        if (valid && total > best) {
            best = total;
        }
    }
    return best;
}
```

<details>
<summary>Go 版本</summary>

```go
func rob(nums []int) int {
    n := len(nums)
    best := 0
    for mask := 0; mask < (1 << n); mask++ { // 窮舉所有選法
        valid := true
        total := 0
        for i := 0; i < n; i++ {
            if mask>>i&1 == 1 {               // 第 i 間有搶嗎？
                next := (i + 1) % n           // 找到鄰居（圓形：最後一間的鄰居是第 0 間）
                if mask>>next&1 == 1 {         // 第 next 間也有搶？不合法，排除
                    valid = false
                    break
                }
                total += nums[i]
            }
        }
        if valid && total > best {
            best = total
        }
    }
    return best
}
```

</details>


- **Time: $O(2^n \times n)$**
- Space: $O(1)$

用 `[2, 3, 2]` 把八個 mask 全部走一遍：

```
mask = 0（000，都不搶）
  i=0,1,2 全都沒搶，跳過
  valid=true, total=0 → best = 0

mask = 1（001，搶 {0}）
  i=0: 有搶。鄰居 next=(0+1)%3=1，(1>>1)&1 = 0，沒搶 → total=2
  i=1, i=2: 沒搶，跳過
  valid=true, total=2 → best = 2

mask = 2（010，搶 {1}）
  i=1: 有搶。鄰居 next=2，(2>>2)&1 = 0，沒搶 → total=3
  valid=true, total=3 → best = 3

mask = 3（011，搶 {0,1}）
  i=0: 有搶。鄰居 next=1，(3>>1)&1 = 1，鄰居也有搶！
  valid=false，break → 這輪作廢，best 還是 3

mask = 4（100，搶 {2}）
  i=2: 有搶。鄰居 next=(2+1)%3=0，(4>>0)&1 = 0，沒搶 → total=2
  valid=true, total=2 → 2 < 3，best 不動

mask = 5（101，搶 {0,2}）
  i=0: 有搶。鄰居 next=1，沒搶 → total=2
  i=2: 有搶。鄰居 next=(2+1)%3=0，(5>>0)&1 = 1，第 0 間也有搶！
  valid=false，break → 作廢

mask = 6（110，搶 {1,2}）
  i=1: 有搶。鄰居 next=2，也有搶 → valid=false，break

mask = 7（111，全搶）
  i=0: 有搶。鄰居 next=1，也有搶 → valid=false，break

迴圈結束，return best = 3
```

合法的四種選法 {}、{0}、{1}、{2} 分別在 mask = 0、1、2、4 出現，其他四個全在內層被 `valid=false` 排除。mask = 5 要走到 i=2 才被排除：i=0 檢查的時候鄰居是第 1 間，沒問題；到了 i=2，鄰居 (2+1) % 3 = 0 繞回第 0 間，頭尾同搶在這一步被查出來。沒有 `% n` 的話，第 2 間的鄰居會算成不存在的第 3 間，頭尾同搶就檢查不到了。

n = 100 就完全跑不動。

---

## 解法二：拆兩段直線 DP

圓形的限制只有一個：**頭和尾不能同時搶**。

這代表兩種情況：
- 搶第一間 → 最後一間不能搶，只看 `nums[0..n-2]`
- 不搶第一間 → 最後一間可以搶，只看 `nums[1..n-1]`

每個子問題都是直線版的 House Robber。跑兩次，取最大值。

```typescript
function rob(nums: number[]): number {
    const n = nums.length;
    if (n === 1) return nums[0];

    // 兩種情況取最大：搶頭不搶尾 vs 不搶頭可搶尾
    return Math.max(robRange(nums, 0, n - 2), robRange(nums, 1, n - 1));
}

function robRange(nums: number[], lo: number, hi: number): number {
    let prev = 0, curr = 0;
    for (let i = lo; i <= hi; i++) {
        [prev, curr] = [curr, Math.max(curr, nums[i] + prev)]; // 不搶 vs 搶
    }
    return curr;
}
```

<details>
<summary>Go 版本</summary>

```go
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }
    // 兩種情況取最大：搶頭不搶尾 vs 不搶頭可搶尾
    return max(robRange(nums, 0, n-2), robRange(nums, 1, n-1))
}

func robRange(nums []int, lo, hi int) int {
    prev, curr := 0, 0
    for i := lo; i <= hi; i++ {
        prev, curr = curr, max(curr, nums[i]+prev) // 不搶 vs 搶
    }
    return curr
}
```

</details>


- Time: $O(n)$
- Space: $O(1)$

用 `[2, 3, 2]` 走一遍：

```
情況一：nums[0..1] = [2, 3]
初始：prev=0, curr=0
i=0 (2): prev=0, curr=max(0, 2+0)=2
i=1 (3): prev=2, curr=max(2, 3+0)=3
結果：3

情況二：nums[1..2] = [3, 2]
初始：prev=0, curr=0
i=1 (3): prev=0, curr=max(0, 3+0)=3
i=2 (2): prev=3, curr=max(3, 2+0)=3
結果：3

答案：max(3, 3) = 3 ✓
```

再用 `[1, 2, 3, 1]` 確認：

```
情況一：nums[0..2] = [1, 2, 3]
初始：prev=0, curr=0
i=0 (1): prev=0, curr=1
i=1 (2): prev=1, curr=max(1, 2+0)=2
i=2 (3): prev=2, curr=max(2, 3+1)=4
結果：4

情況二：nums[1..3] = [2, 3, 1]
初始：prev=0, curr=0
i=1 (2): prev=0, curr=2
i=2 (3): prev=2, curr=max(2, 3+0)=3
i=3 (1): prev=3, curr=max(3, 1+2)=3
結果：3

答案：max(4, 3) = 4 ✓
```

---

**為什麼只有這兩種情況？**

頭和尾不能同時搶，所以至少有一個不搶。

- 如果一定不搶頭 → `nums[1..n-1]`
- 如果一定不搶尾 → `nums[0..n-2]`

這兩種已經涵蓋所有可能。最優解一定落在其中一個。

---

## 結論

圓形 = 拆成兩個直線。跑兩次 House Robber，取最大。
