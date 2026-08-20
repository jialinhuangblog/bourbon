給一個排好序的陣列和一個目標值，找到目標值的 index，不存在就回傳 -1。要求 O(log n)。

```
nums = [-1, 0, 3, 5, 9, 12], target = 9
                     ^
                     index 4 → 回傳 4

nums = [-1, 0, 3, 5, 9, 12], target = 2
         找不到 → 回傳 -1
```

---

## 解法：二分搜尋

O(log n) = 每一步砍掉一半。這就是 binary search 的定義。

看中間的數字，比 target 大就砍右半邊，比 target 小就砍左半邊。重複直到找到或沒得砍。

用 `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9` 走一遍：

```
步驟 1:
[-1, 0, 3, 5, 9, 12]
  L        M       R     mid = (0+5)/2 = 2, nums[2] = 3
                          3 < 9 → target 在右半邊，L = mid + 1 = 3

步驟 2:
[-1, 0, 3, 5, 9, 12]
              L  M   R   mid = (3+5)/2 = 4, nums[4] = 9
                          9 == 9 → 找到了！回傳 4
```

2 步就找到了。陣列長度 6，暴力掃要 5 步。陣列越大差距越明顯：100 萬個元素只要 20 步。

---

完整程式碼：

```typescript
function search(nums: number[], target: number): number {
    let l = 0, r = nums.length - 1;
    while (l <= r) {
        const mid = l + Math.floor((r - l) / 2);
        if (nums[mid] === target) {
            return mid;
        } else if (nums[mid] < target) {
            l = mid + 1;
        } else {
            r = mid - 1;
        }
    }
    return -1;
}
```

- Time: O(log n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func search(nums []int, target int) int {
    l, r := 0, len(nums)-1
    for l <= r {
        mid := l + (r-l)/2 // 避免 (l+r) 溢位
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            l = mid + 1 // target 在右半邊
        } else {
            r = mid - 1 // target 在左半邊
        }
    }
    return -1
}
```

</details>

---

**三個容易寫錯的地方**

**1. `l <= r` 還是 `l < r`？**

`nums = [-1, 0, 3, 5, 9, 12]`，找 12，條件寫 `l < r`：

```
l=0 r=5 → mid=2, nums[2]=3 太小 → l=3
l=3 r=5 → mid=4, nums[4]=9 太小 → l=5
l=5 r=5 → 5 < 5 不成立，跳出迴圈 → 回 -1
```

答案就在 nums[5]，迴圈卻在檢查它之前結束了。`l == r` 代表區間還剩一個元素，`l < r` 把它當成空的。改成 `l <= r`，多跑一輪 mid=5，`nums[5] == 12`，回 5。

單一元素的陣列更明顯：`nums = [5]` 找 5，`l = r = 0`，`l < r` 一開始就不成立，迴圈一次都沒跑。

**2. `mid = (l + r) / 2` 還是 `mid = l + (r - l) / 2`？**

結果一樣，但 `l + r` 在 l 和 r 都很大時會溢位。除法還沒做，加法就先爆了：

```
32-bit int 上限 = 2,147,483,647

l = 2,000,000,000
r = 2,000,000,000

(l + r)           = 4,000,000,000 → 超過上限，溢位，變成負數
(l + r) / 2       = 負數 / 2 = 錯的答案

l + (r - l) / 2   = 2,000,000,000 + 0 = 2,000,000,000 → 正確
```

`r - l` 一定 ≤ `r`，不會爆。面試寫 `l + (r - l) / 2` 是背起來的好習慣。

不過會不會真的溢位，看的不是語言，是 index 用多寬的整數。同一組 `l = r = 2,000,000,000`：

| 語言 | index 常用型別 | `(l + r) / 2` 算出來 |
|---|---|---|
| Java `int` | 32 bits | `-147483648` |
| C `int` | 32 bits | `-147483648`，而且 signed overflow 在 C 是 UB |
| Go `int` | 64 bits | `2000000000` 正確；宣告成 `int32` 才會壞 |
| Rust `usize` | 64 bits | 正確；`i32` 在 debug build 直接 panic，release 才 wrap |
| Swift `Int` | 64 bits | 正確；`Int32` 執行時 trap，process 收 exit code 133 |
| JavaScript `number` | double | 正確到 2^53 |
| Python / Ruby | 任意精度 | 永遠正確 |

64 bits 的 index 溢位不了。表裡的 Go `int`、Swift `Int` 都是有號的，一個 bit 拿去記正負，正數上限是 `2^63 - 1` 而不是 `2^64 - 1`：

```
uint64   0 … 18,446,744,073,709,551,615        上限 2^64 - 1
int64    -9,223,372,036,854,775,808 … 9,223,372,036,854,775,807   上限 2^63 - 1
```

`l` 跟 `r` 最大都是 `n - 1`，`l + r` 最大就是 `2(n - 1)`。要越過 `2^63 - 1`，得 `n > 2^62`，約 4.6×10^18 個元素，一個元素只佔 1 byte 都要 4.6 EB 的記憶體。Rust 的 `usize` 是無號的，門檻還要再往上一倍到 `2^63`。所以剩下 Java 這一類。Java 的陣列長度用 int 定址，上限 2^31-1，`low + high` 剛好能越過 2^31，JDK 自己的 `Arrays.binarySearch` 就寫成 `(low + high) >>> 1`。C/C++ 拿 `int` 當 index 一樣會中，換成 `size_t` 就不會。

TypeScript 那邊也可以用位移寫：

```typescript
const mid = l + ((r - l) >> 1);
```

`>>` 本身就會捨去小數，`Math.floor` 可以省掉。內層那對括號不能省：

```
l = 4, r = 10

l + ((r - l) >> 1)  =  4 + (6 >> 1)  =  4 + 3  =  7   ← 正確
l +  (r - l)  >> 1  =  (4 + 6) >> 1  =  10 >> 1 =  5   ← 少一對括號
```

`+` 的優先級比 `>>` 高，所以少括號的版本先把 `l + (r - l)` 算完，那就是 `r`，整條式子變成 `r >> 1`，跟 l 完全沒關係。`l = 0` 的時候兩種寫法會算出一樣的值，拿 0 當例子看不出差別。

代價是 `>>` 會先把運算元轉成 32 bits 有號整數。JS 的 number 明明是 double，一寫位移就把 32 bits 的限制請回來了：`(2000000000 + 2000000000) >> 1` 在 node 上算出 `-147483648`，跟 Java 的 int 一樣。這題 `nums.length <= 10^4` 離上限很遠，但 index 有機會超過 20 億的場合就不能這樣寫。

另外有一種寫法是 `(l + r) >>> 1`，無號右移。同一組數字換成 `>>>` 就回到 `2000000000`，差別在 `>>` 把 32 bits 的結果當有號數讀，`>>>` 當無號數讀。它在 Java 是必要的補救；在 JS 只是把 `>>` 自己惹出來的問題補回去，不用位移、直接寫 `(l + r) / 2` 本來就沒事。

**3. `l = mid + 1` 還是 `l = mid`？**

用 `mid + 1`。mid 已經檢查過不是 target，不需要再包含它。用 `l = mid` 在某些情況會無限迴圈（當 `l == mid` 時 l 不會前進）。

---

**為什麼這題是所有 Binary Search 題的基底**

更難的 binary search 題（33. Search in Rotated Sorted Array、153. Find Minimum in Rotated Sorted Array）都是在這個框架上加條件。框架不變：

```
l, r = 起點, 終點
for l <= r:
    mid = 中間
    判斷往左還是往右
回傳結果
```

差別只在「判斷往左還是往右」的邏輯變複雜。這題的判斷最單純：比大小。搞懂這題，後面的變體就是換判斷條件而已。

---

## 結論

排好序的陣列找東西，每次看中間砍一半。三個細節：`l <= r`、`l + (r - l) / 2`、`mid + 1`。所有 binary search 變體都從這個框架出發。
