你有 8 塊正方形磁磚，要在地上拼一個正方形。最大拼得出多大？

```
2 × 2 = 4 塊   夠，還剩 4 塊
3 × 3 = 9 塊   不夠
```

答案是 2。

翻回 code 術語：給一個非負整數 `x`，回傳 $\sqrt{x}$ 向下取整。不能用內建的開根號或次方。

---

**解題引導**

用 `x = 8` 想。

**Step 1：最直覺的做法是什麼？**

*try 1, 2, 3... until it's too big*

<span class="spoiler">從 1 開始往上試，直到 i 乘 i 超過 x，答案就是前一個。Count up until i squared exceeds x.</span>

**Step 2：把 0 到 x 每個數字都問一次「你的平方超過 x 了嗎」，答案排出來長什麼樣？**

*no no no yes yes yes*

<span class="spoiler">前面一段都是「沒超過」，後面一段都是「超過了」，中間只有一個分界點。這種一刀切的排列可以二分。The predicate flips exactly once, so it is binary searchable.</span>

**Step 3：二分的時候要記住什麼？**

*the last one that still fits*

<span class="spoiler">記住最後一個「平方沒超過 x」的候選。找到更好的就更新，最後那個就是答案。Keep the largest mid whose square still fits.</span>

**Step 4：x = 2147483647 的時候，mid 乘 mid 會不會出事？**

<span class="spoiler">在 TypeScript 跟 Go 不會，因為 number 是 double、Go 的 int 是 64 bits，裝得下 46341 的平方約 21.5 億。換成 Java 或 C++ 的 32 bits int 就會溢位，要改寫成除法比較。Overflow only bites in 32-bit int languages.</span>

想完再往下看 code。

---

## 解法一：從 0 一個一個往上數

```typescript
function mySqrt(x: number): number {
    let i = 0;
    while (i * i <= x) {   // 還裝得下就繼續往上
        i++;
    }
    return i - 1;          // 停下來時 i 已經太大，退一格
}
```

- Time: $O(\sqrt{x})$
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func mySqrt(x int) int {
    i := 0
    for i*i <= x {
        i++
    }
    return i - 1
}
```

</details>

**走一遍 `x = 8`：**

```
i=0   0*0=0  <= 8   → i=1
i=1   1*1=1  <= 8   → i=2
i=2   2*2=4  <= 8   → i=3
i=3   3*3=9  >  8   → 停

回傳 3 - 1 = 2
```

**為什麼是 `i - 1` 不是 `i`。** 迴圈停下來的那一刻，`i` 是第一個「平方已經超過 x」的數字。答案要的是最後一個沒超過的，所以退一格。

`x` 上限 $2^{31} - 1 = 2{,}147{,}483{,}647$，$\sqrt{x} \approx 46{,}340$，所以最多數 46,341 次，約 **0.005 秒**。這題其實過得了，但每次都從 0 開始爬很浪費。

---

## 解法二：二分搜尋

把 0 到 x 每個數字都問一次「你的平方超過 x 了嗎」，`x = 8` 的答案排出來是這樣：

```
i:        0    1    2    3    4    5    6    7    8
i*i:      0    1    4    9   16   25   36   49   64
超過 8?   否   否   否   是   是   是   是   是   是
                      ↑
                   分界在這
```

前面一段全是「否」，後面一段全是「是」，中間只翻面一次。這種排列就能二分。

```typescript
function mySqrt(x: number): number {
    let lo = 0, hi = x, ans = 0;
    while (lo <= hi) {
        const mid = Math.floor((lo + hi) / 2);
        if (mid * mid <= x) {
            ans = mid;        // 這個裝得下，先記著，再往右找更大的
            lo = mid + 1;
        } else {
            hi = mid - 1;     // 太大了，往左找
        }
    }
    return ans;
}
```

- Time: $O(\log x)$
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func mySqrt(x int) int {
    lo, hi, ans := 0, x, 0
    for lo <= hi {
        mid := (lo + hi) / 2
        if mid*mid <= x {
            ans = mid
            lo = mid + 1
        } else {
            hi = mid - 1
        }
    }
    return ans
}
```

</details>

**走一遍 `x = 8`：**

```
lo=0 hi=8   mid=4   16 > 8    → hi=3
lo=0 hi=3   mid=1   1 <= 8    → ans=1, lo=2
lo=2 hi=3   mid=2   4 <= 8    → ans=2, lo=3
lo=3 hi=3   mid=3   9 > 8     → hi=2
lo=3 > hi=2 迴圈結束

回傳 ans = 2
```

四步結束。

**`ans` 這個變數為什麼不能省。** 迴圈跳出來的時候 `lo = 3`、`hi = 2`，兩個都不是答案。答案 2 是在第三步經過的，當下不記下來就找不回來了。二分找的是「最後一個滿足條件的」，那種題目都要有一個地方存目前最好的候選。

$x = 2^{31} - 1$ 需要 31 步（實際跑過），對比線性解的 46,341 次。

---

## 解法三：牛頓法

二分是「猜一個數字，看太大還太小」。牛頓法是「猜一個數字，用它算出一個更好的猜測」。

要求 $\sqrt{x}$，也就是求 $g^2 = x$ 的正根。從任意一個猜測 $g$ 出發，下一個猜測取 $g$ 跟 $x/g$ 的平均：

$$g_{next} = \frac{g + x/g}{2}$$

為什麼平均會更接近？$g$ 要是猜大了，$x/g$ 就會偏小，真正的答案夾在兩者中間，取平均就往中間靠。

```typescript
function mySqrt(x: number): number {
    if (x === 0) return 0;          // 不特判會除以零
    let g = x;
    while (g * g > x) {             // 還太大就再逼近一次
        g = Math.floor((g + Math.floor(x / g)) / 2);
    }
    return g;
}
```

- Time: $O(\log x)$，前半段在減半、後半段才二次收斂，下面拆給你看
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func mySqrt(x int) int {
    if x == 0 {
        return 0
    }
    g := x
    for g*g > x {
        g = (g + x/g) / 2
    }
    return g
}
```

</details>

**走一遍 `x = 8`：**

```
g=8   64 > 8   →  g = (8 + 8/8) / 2 = (8 + 1) / 2 = 4
g=4   16 > 8   →  g = (4 + 8/4) / 2 = (4 + 2) / 2 = 3
g=3    9 > 8   →  g = (3 + 8/3) / 2 = (3 + 2) / 2 = 2    ← 8/3 整數除法得 2
g=2    4 <= 8  →  停

回傳 2
```

三步。

**整數除法剛好幫了忙。** `x / g` 取整之後會偏小一點，讓數列從上面單調往下走，第一個平方不超過 x 的就停在正確答案。我把 0 到 200,000 全部跑過一遍對照 `math.isqrt`，沒有一個不同。

**`x === 0` 一定要特判。** 不然第一次迭代就 `x / g` 除以零。

### 牛頓法為什麼不是 $O(\log \log x)$

教科書常說牛頓法是二次收斂，每次迭代正確位數翻倍，$\log \log$ 步就收斂。照這個算，$x = 2^{31} - 1$ 應該 $\log_2 31 \approx 5$ 步就結束。實際跑是 19 步。

把每一步印出來就知道差在哪：

```
step  0  g = 2147483647
step  1  g = 1073741824   前一步的 2.000 分之一
step  2  g =  536870912   前一步的 2.000 分之一
...
step 13  g =     264868   前一步的 1.985 分之一
step 14  g =     136487   前一步的 1.941 分之一
step 15  g =      76110   前一步的 1.793 分之一
step 16  g =      52162   前一步的 1.459 分之一
step 17  g =      46665   前一步的 1.118 分之一
step 18  g =      46342   前一步的 1.007 分之一
step 19  g =      46340   前一步的 1.000 分之一
```

前 14 步每次剛好減半，那不是收斂，是除以二。原因在初始猜測 `g = x` 離答案太遠：`g` 有 21 億而答案只有 4 萬多，這時 `x / g` 幾乎等於 1，`(g + x/g) / 2` 就約等於 `g / 2`。

從 $x$ 一路減半到 $\sqrt{x}$ 要 $\log_2 \sqrt{x} = \frac{1}{2}\log_2 x$ 步，代進去是 $\frac{31}{2} \approx 15.5$，跟實測的 14 步吻合。真正的二次收斂只發生在最後 5 步，也就是倍率從 1.79 掉到 1.000 那一段。

所以牛頓法在這份實作是 $O(\log x)$，跟二分同一級，只是常數比較好（19 對 31）。要拿到 $O(\log \log x)$ 得換初始猜測，例如用 `x` 的 bit 長度先估一個 $2^{\lfloor bits/2 \rfloor}$，跳過那段減半。

---

**Overthinking**

**哪些語言會 overflow。** `mid * mid` 最大是 $46{,}341^2 \approx 2.1 \times 10^9$。

- TypeScript：`number` 是 double，$2^{53}$ 以內的整數精確，安全
- Go：`int` 在 64 bits 平台是 64 bits，安全
- Java / C++ 的 `int`：32 bits，上限約 $2.1 \times 10^9$，剛好會溢位變負數

那些語言要改寫成 `mid <= x / mid`，把乘法換成除法就不會超出範圍。

**`hi` 可以設小一點。** 從 `x` 開始其實浪費，答案不可能超過 46,341。設 `hi = Math.min(x, 46341)` 可以少跑幾步，但 $O(\log x)$ 本來就只有 31 步，省不到什麼。

**要小數點後幾位怎麼辦？** 二分改成在浮點數上做，終止條件從 `lo <= hi` 換成 `hi - lo > 1e-6`。牛頓法在這種場景才真的划算，因為初始猜測可以先用整數解，那時已經很接近答案，直接進入二次收斂，每次迭代正確位數翻倍。

---

## 解法比較表

| 解法 | Time | Space | x=2^31-1 的步數 | 備註 |
|---|---|---|---|---|
| 線性往上數 | $O(\sqrt{x})$ | $O(1)$ | 46,341 | 0.005 秒，過得了但笨 |
| 二分搜尋 | $O(\log x)$ | $O(1)$ | 31 | 面試要的答案 |
| 牛頓法 | $O(\log x)$ | $O(1)$ | 19 | 同一級，常數好一點 |

三個都是 $O(1)$ 空間。牛頓法的 19 步裡有 14 步在做減半，跟二分沒有本質差別，真正快的只有最後 5 步。

---

## 結論

這題真正在考的是「認出可以二分的排列」。`i*i <= x` 這個問句從 0 問到 x，答案只翻面一次，那就能二分。牛頓法看起來高級，但初始猜測從 `x` 出發的話，前面大半的步數也只是在除以二。
