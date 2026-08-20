給一個整數 n，把 0 到 n 每個數字轉成二進位，數有幾個 1，收集成一個陣列回傳。

```
n = 5

數字    二進位    有幾個 1
 0       0         0
 1       1         1
 2      10         1
 3      11         2
 4     100         1
 5     101         2

答案：[0, 1, 1, 2, 1, 2]
```

---

## 解法一：逐位數 1

最直覺的做法：對每個數字，一個一個數 1 有幾個。

怎麼數？用 `i & 1` 看最後一位是不是 1，然後 `i >>= 1` 右移一位，重複直到 i 變 0。

```typescript
function countBits(n: number): number[] {
    const ans = new Array(n + 1).fill(0);
    for (let i = 0; i <= n; i++) {
        let count = 0;
        let num = i;
        while (num > 0) {
            count += num & 1; // 最後一位是 1 就 +1
            num >>= 1;        // 右移，看下一位
        }
        ans[i] = count;
    }
    return ans;
}
```

- Time: O(n log n) — 每個數字最多 log n 位
- Space: O(1)（不算輸出）

<details>
<summary>Go 版本</summary>

```go
func countBits(n int) []int {
    ans := make([]int, n+1)
    for i := 0; i <= n; i++ {
        count := 0
        num := i
        for num > 0 {
            count += num & 1 // 最後一位是 1 就 +1
            num >>= 1        // 右移，看下一位
        }
        ans[i] = count
    }
    return ans
}
```

</details>

題目說 O(n log n) 太簡單了，要 O(n)。

---

## 解法二：DP

觀察一下規律。把 0 到 7 的二進位和 1 的數量列出來：

```
0:  000  → 0
1:  001  → 1
2:  010  → 1
3:  011  → 2
4:  100  → 1
5:  101  → 2
6:  110  → 2
7:  111  → 3
```

看 6 (`110`)。把它右移一位變成 3 (`011`)，差別只是少看了最後一位。所以：

```
ans[6] = ans[6 >> 1] + (6 & 1)
       = ans[3]      + 0
       = 2           + 0
       = 2
```

再看 7 (`111`)。右移一位變成 3 (`011`)，最後一位是 1：

```
ans[7] = ans[7 >> 1] + (7 & 1)
       = ans[3]      + 1
       = 2           + 1
       = 3
```

通用公式：

```
ans[i] = ans[i >> 1] + (i & 1)
```

意思是：i 的 1 的數量 = i 去掉最後一位的 1 的數量 + 最後一位本身。

`i >> 1` 一定比 i 小，所以算到 i 的時候 `ans[i >> 1]` 早就算好了。這就是 DP。

---

用 `n = 5` 走一遍：

```
i=0: ans[0>>1] + (0&1) = ans[0] + 0 = 0  (base case)
i=1: ans[1>>1] + (1&1) = ans[0] + 1 = 1
i=2: ans[2>>1] + (2&1) = ans[1] + 0 = 1
i=3: ans[3>>1] + (3&1) = ans[1] + 1 = 2
i=4: ans[4>>1] + (4&1) = ans[2] + 0 = 1
i=5: ans[5>>1] + (5&1) = ans[2] + 1 = 2

ans = [0, 1, 1, 2, 1, 2] ✓
```

---

完整程式碼：

```typescript
function countBits(n: number): number[] {
    const ans = new Array(n + 1).fill(0);
    for (let i = 1; i <= n; i++) {
        ans[i] = ans[i >> 1] + (i & 1);
    }
    return ans;
}
```

- Time: O(n)
- Space: O(1)（不算輸出）

<details>
<summary>Go 版本</summary>

```go
func countBits(n int) []int {
    ans := make([]int, n+1)
    for i := 1; i <= n; i++ {
        ans[i] = ans[i>>1] + (i & 1)
    }
    return ans
}
```

</details>

---

## 結論

`ans[i] = ans[i >> 1] + (i & 1)`。把數字拆成「去掉最後一位」和「最後一位本身」，前者已經算過，後者用 `& 1` 拿。一行 DP 轉移式，一次迴圈。
