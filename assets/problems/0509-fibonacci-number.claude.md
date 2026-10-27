給定 n，回傳第 n 個費波那契數。F(0)=0, F(1)=1, F(n)=F(n-1)+F(n-2)。

---

**解題引導**

用 n=5 想。

**Step 1：題目本身就是什麼？**<br>
<span class="spoiler">遞迴定義。F(n) = F(n-1) + F(n-2)，直接翻成 code 就是暴力解。</span>

**Step 2：暴力遞迴的問題在哪？**<br>
<span class="spoiler">fib(3) 被算了兩次，fib(2) 被算了三次。重複計算，O(2^n)。</span>

**Step 3：怎麼解決重複計算？**<br>
<span class="spoiler">加 memo 陣列，算過就存起來，下次直接回傳。O(n)。這就是 top-down。</span>

**Step 4：top-down 有什麼風險？**<br>
<span class="spoiler">遞迴太深會 stack overflow。而且 Fibonacci 每個狀態都會用到，top-down 的「只算需要的」優勢在這題是零。</span>

**Step 5：能不能不用遞迴？**<br>
<span class="spoiler">從小到大用迴圈算：dp[0]=0, dp[1]=1, dp[2]=dp[1]+dp[0]... 這就是 bottom-up。</span>

**Step 6：dp 陣列每一步只看前兩格，能壓縮嗎？**<br>
<span class="spoiler">兩個變數 prev 和 curr 就夠。空間 O(1)。</span>

想完再往下看 code。

---

## 解法一：暴力遞迴

定義就是遞迴，直接翻成 code：

```typescript
function fib(n: number): number {
    if (n <= 1) return n; // F(0)=0, F(1)=1，base case
    return fib(n - 1) + fib(n - 2); // 定義本身
}
```

- **Time: $O(2^n)$**
- Space: $O(n)$ — call stack 深度

<details>
<summary>Go 版本</summary>

```go
func fib(n int) int {
    if n <= 1 {
        return n // F(0)=0, F(1)=1，base case
    }
    return fib(n-1) + fib(n-2) // 定義本身
}
```

</details>

問題在哪？畫 recursion tree 就看到了：

```
fib(5)
├── fib(4)
│   ├── fib(3)
│   │   ├── fib(2)  ← 算了
│   │   └── fib(1)
│   └── fib(2)      ← 又算一次
└── fib(3)          ← 又算一次
    ├── fib(2)      ← 又算一次
    └── fib(1)
```

`fib(3)` 算了兩次，`fib(2)` 算了三次。n 每增加 1，工作量翻倍。

---

## 解法二：Memo

重複計算的問題，用 memo 記住。

```typescript
function fib(n: number): number {
    const memo = new Array(n + 1).fill(-1);
    function helper(n: number): number {
        if (n <= 1) return n;
        if (memo[n] !== -1) return memo[n]; // 算過了，直接回傳
        memo[n] = helper(n - 1) + helper(n - 2);
        return memo[n];
    }
    return helper(n);
}
```

- Time: $O(n)$
- Space: $O(n)$

<details>
<summary>Go 版本</summary>

```go
func fib(n int) int {
    memo := make([]int, n+1)
    for i := range memo {
        memo[i] = -1 // -1 代表還沒算過
    }
    return helper(n, memo)
}

func helper(n int, memo []int) int {
    if n <= 1 {
        return n
    }
    if memo[n] != -1 {
        return memo[n] // 算過了，直接回傳
    }
    memo[n] = helper(n-1, memo) + helper(n-2, memo)
    return memo[n]
}
```

</details>

每個 `fib(i)` 只算一次。從 $O(2^n)$ 砍到 $O(n)$。

還能再簡化。

---

## 解法三：Bottom-up DP

Top-down 翻成 bottom-up。從小到大算，不用遞迴。

```typescript
function fib(n: number): number {
    if (n <= 1) return n;
    const dp = new Array(n + 1);
    dp[0] = 0; dp[1] = 1;
    for (let i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2]; // 只依賴前兩個
    }
    return dp[n];
}
```

- Time: $O(n)$
- Space: $O(n)$

<details>
<summary>Go 版本</summary>

```go
func fib(n int) int {
    if n <= 1 {
        return n
    }
    dp := make([]int, n+1)
    dp[0], dp[1] = 0, 1
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2] // 只依賴前兩個
    }
    return dp[n]
}
```

</details>

---

## 解法四：滾動兩變數

`dp[i]` 只看 `dp[i-1]` 和 `dp[i-2]`，不需要整個陣列。兩個變數就夠：

```typescript
function fib(n: number): number {
    let prev = 0, curr = 1;
    for (let i = 0; i < n; i++) {
        [prev, curr] = [curr, prev + curr]; // 往前滾一步
    }
    return prev;
}
```

- Time: $O(n)$
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func fib(n int) int {
    prev, curr := 0, 1 // F(0), F(1)
    for i := 0; i < n; i++ {
        prev, curr = curr, prev+curr // 往前滾一步
    }
    return prev
}
```

</details>

走一遍 n=5：

```
初始：prev=0, curr=1
i=0: prev=1, curr=1
i=1: prev=1, curr=2
i=2: prev=2, curr=3
i=3: prev=3, curr=5
i=4: prev=5, curr=8

答案：prev=5 ✓
```

---

**Overthinking**

還有一個 $O(\log n)$ 的矩陣快速冪做法：

$$\begin{bmatrix} F(n+1) \\\\ F(n) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\\\ 1 & 0 \end{bmatrix}^n \begin{bmatrix} 1 \\\\ 0 \end{bmatrix}$$

核心觀察：每乘一次這個矩陣，就往前推一個 Fibonacci 數。

```
乘1次： [1, 1]  → F(2), F(1)
乘2次： [2, 1]  → F(3), F(2)
乘3次： [3, 2]  → F(4), F(3)
乘n次：         → F(n+1), F(n)
```

為什麼是 `[[1,1],[1,0]]`？從 Fibonacci 定義推出來的。我們有 F(n) 和 F(n-1)，乘完要得到 F(n+1) 和 F(n)：

```
F(n+1) = 1×F(n) + 1×F(n-1)  → 上排 [1, 1]
F(n)   = 1×F(n) + 0×F(n-1)  → 下排 [1, 0]（原封不動搬過來）
```

不是魔法，是硬推出來的。

矩陣自乘 n 次，直接算要 n 次。快速冪可以砍到 $O(\log n)$：

```
M^8 = M^4 × M^4   → 只要算 M^4
M^4 = M^2 × M^2   → 只要算 M^2
M^2 = M^1 × M^1   → 一次

總共 3 次，不是 8 次。log₂8 = 3。
```

每次指數砍半，砍 $\log n$ 次就到底，所以是 $O(\log n)$。

但 n 最大才 30，$O(n)$ 已經是 30 次操作，根本不需要。面試提一下就好，別真的去實作。

---

## 結論

兩個變數，一個迴圈。Fibonacci 是所有 DP 題的起點，壓縮變數的技巧會一直用到。
