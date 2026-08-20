兩個人各自把去超市要買的東西寫成一張清單，順序是他們打算走過貨架的順序。

問兩張清單裡，最多有幾樣東西是兩個人都要買、而且在兩張清單上的先後順序一致的。中間可以跳過別的東西，但不能調換順序。

```
text1 = "abcde"
text2 = "ace"

a b c d e
a - c - e     a、c、e 三樣在兩邊的順序都一樣

答案 3
```

`"ace"` 在 `"abcde"` 裡是把 b 跟 d 刪掉留下來的，這種叫 subsequence，跟 substring 不同：substring 要連在一起，subsequence 只要順序不變。

翻成 code 的講法：給兩個字串，求最長共同子序列的長度。

---

**解題引導**

拿 `text1 = "abcde"`、`text2 = "ace"` 想。

**Step 1：先看兩個字串的最後一個字元，e 跟 e。它們相同的話，代表什麼？**

*a matching pair can always be taken. no reason to leave it.*

<span class="spoiler">這對 e 一定可以算進答案裡。把兩邊的 e 都砍掉，剩下 "abcd" 跟 "ac" 的答案再加 1 就是。A matching last pair always belongs to some optimal answer.</span>

**Step 2：換成 text1 = "abcd"、text2 = "ace"，最後一個字元 d 跟 e 不同。這時候怎麼辦？**

*one of them has to go. try both.*

<span class="spoiler">d 跟 e 不可能同時是配對的一方，至少有一個用不到。所以砍掉 d 試一次、砍掉 e 試一次，取兩個結果裡大的那個。Drop one or the other, take the better result.</span>

**Step 3：這兩件事寫成一個二維陣列會長什麼樣？**

<span class="spoiler">開一個 (m+1) × (n+1) 的二維陣列，f[i][j] 是 text1 前 i 個字元跟 text2 前 j 個字元的答案。字元相同的話，f[i][j] 等於左上角 f[i-1][j-1] 加一；不同的話，f[i][j] 等於 max(上面 f[i-1][j], 左邊 f[i][j-1])。A 2-D table indexed by prefix lengths.</span>

**Step 4：第 0 個 row 跟第 0 個 column 填什麼？**

<span class="spoiler">全部填 0。一邊是空字串的話，共同子序列只能是空的。An empty prefix has nothing in common with anything.</span>

想完再往下看 code。

---

## 解法一：照著兩種情況寫遞迴

`lcs(i, j)` 是「text1 從 i 開始、text2 從 j 開始」的答案。

```typescript
function longestCommonSubsequence(text1: string, text2: string): number {
    function lcs(i: number, j: number): number {
        if (i === text1.length || j === text2.length) return 0;   // 有一邊看完了

        if (text1[i] === text2[j]) return lcs(i + 1, j + 1) + 1;  // 配對成功，兩邊都往前

        return Math.max(lcs(i + 1, j), lcs(i, j + 1));            // 各砍一個試試看
    }

    return lcs(0, 0);
}
```

- Time: $O(2^{m+n})$ — 字元不同的時候分岔成兩個呼叫
- Space: $O(m + n)$ — call stack

**走一遍。** `text1 = "ab"`、`text2 = "ba"`：

```
lcs(0,0)   'a' vs 'b' 不同
├── lcs(1,0)   'b' vs 'b' 相同 → lcs(2,1) + 1 = 0 + 1 = 1
└── lcs(0,1)   'a' vs 'a' 相同 → lcs(1,2) + 1 = 0 + 1 = 1
→ max(1, 1) = 1
```

答案 1，`"a"` 或 `"b"` 都行。

**兩個字串沒有共同字元的時候最慢。** 每一格都走「不同」那條，每次分岔成兩個。實測兩個長度 14、完全沒有共同字元的字串，80,233,199 次呼叫，換算 8 秒。題目允許長度 1000。

<details>
<summary>Go 版本</summary>

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    var lcs func(i, j int) int
    lcs = func(i, j int) int {
        if i == len(text1) || j == len(text2) {
            return 0 // 有一邊看完了
        }

        if text1[i] == text2[j] {
            return lcs(i+1, j+1) + 1 // 配對成功，兩邊都往前
        }

        a := lcs(i+1, j)
        b := lcs(i, j+1)
        if a > b {
            return a
        }
        return b
    }

    return lcs(0, 0)
}
```

</details>

---

## 解法二：填一個二維陣列

`f[i][j]` 改成「text1 的前 i 個字元、text2 的前 j 個字元」的答案。多開一個 row 一個 column 放空字串的情況，邊界就不用另外寫。

```typescript
function longestCommonSubsequence(text1: string, text2: string): number {
    const m = text1.length;
    const n = text2.length;
    const f: number[][] = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));

    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (text1[i - 1] === text2[j - 1]) {
                f[i][j] = f[i - 1][j - 1] + 1;                    // 左上角加一
            } else {
                f[i][j] = Math.max(f[i - 1][j], f[i][j - 1]);     // 上面跟左邊挑大的
            }
        }
    }

    return f[m][n];
}
```

- Time: $O(m \times n)$ — 每格填一次
- Space: $O(m \times n)$ — 二維陣列

`f[i][j]` 講的是「前 i 個字元」，所以第 i 個字元的 index 是 `i - 1`。這個差一位是陣列多開一個 row 一個 column 換來的，代價是每次讀字元都要記得減一。

**走一遍。** `text1 = "abcde"`、`text2 = "ace"`。每一格直接寫計算方法：`↖X+1` 表示跟左上角字元相同，抄左上角的值 X 加一；`max(A,B)` 表示不同，A 是上面、B 是左邊，取比較大的。第 0 個 row 跟第 0 個 column 是邊界，固定是 0。

| | j=0 | j=1 (`a`) | j=2 (`c`) | j=3 (`e`) |
|---|---|---|---|---|
| i=0 | 0 | 0 | 0 | 0 |
| i=1 (`a`) | 0 | ↖0+1=**1** | max(0,1)=1 | max(0,1)=1 |
| i=2 (`b`) | 0 | max(1,0)=1 | max(1,1)=1 | max(1,1)=1 |
| i=3 (`c`) | 0 | max(1,0)=1 | ↖1+1=**2** | max(1,2)=2 |
| i=4 (`d`) | 0 | max(1,0)=1 | max(2,1)=2 | max(2,2)=2 |
| i=5 (`e`) | 0 | max(1,0)=1 | max(2,1)=2 | ↖2+1=**3** |

`f[5][3] = 3`。三格 `↖X+1` 剛好對到 `a`、`c`、`e` 三次相同，也就是最長共同子序列本身；其他格都是 `max`，沒有貢獻新的配對。

**為什麼這樣填陣列，配對到的字元順序不會亂。** 三個配對發生在 (i=1,j=1)、(i=3,j=2)、(i=5,j=3)：i 跟 j 各自是 1→3→5、1→2→3，一路往大走，沒有回頭。原因在 f[i][j] 的定義：它只看 text1 前 i 個字元跟 text2 前 j 個字元，填陣列時 i 從 1 掃到 m、j 從 1 掃到 n，本來就只會往後走。配對成功那一步（左上角 f[i-1][j-1] 加一）發生在某個 (i,j)，下一次配對成功一定發生在更大的 i 跟更大的 j，因為前面的字元已經算進前綴裡，不會再被拿出來重新配對。兩邊配對到的字元位置各自嚴格遞增，剛好就是 subsequence「順序不能調換」的定義。要是不比對前綴、只看兩個字元值是否相同就配對，就可能把 text2 後面的字元配到 text1 前面的字元，那樣配出來的就不是合法 subsequence。

**為什麼配對成功不用跟上面左邊比一下。** 直覺會想「說不定不拿這對 e，答案更大」。不會。`f[i-1][j-1] + 1` 至少跟 `f[i-1][j]` 一樣大，因為 `f[i-1][j]` 頂多比 `f[i-1][j-1]` 多 1，而我們這邊也加了 1。拿下這一對永遠不吃虧，所以直接拿。

<details>
<summary>Go 版本</summary>

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    f := make([][]int, m+1)
    for i := range f {
        f[i] = make([]int, n+1)
    }

    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if text1[i-1] == text2[j-1] {
                f[i][j] = f[i-1][j-1] + 1 // 左上角加一
            } else if f[i-1][j] > f[i][j-1] {
                f[i][j] = f[i-1][j] // 上面比較大
            } else {
                f[i][j] = f[i][j-1] // 左邊比較大
            }
        }
    }

    return f[m][n]
}
```

</details>

### 只保留兩個 row

填 `f[i][j]` 只用得到左上、上、左三格，這三格都在目前這個 row 或上一個 row。只要保留這兩個 row，更早的 row 都可以丟掉。

```typescript
function longestCommonSubsequence(text1: string, text2: string): number {
    const m = text1.length;
    const n = text2.length;
    let prev = new Array(n + 1).fill(0);   // 上一個 row
    let cur = new Array(n + 1).fill(0);    // 這一輪要寫的 row

    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (text1[i - 1] === text2[j - 1]) {
                cur[j] = prev[j - 1] + 1;                    // prev[j-1] 就是左上角
            } else {
                cur[j] = Math.max(prev[j], cur[j - 1]);      // prev[j] 是上面，cur[j-1] 是左邊
            }
        }
        [prev, cur] = [cur, prev];   // 這個 row 變成下一輪的上一個 row
    }

    return prev[n];
}
```

- Time: $O(m \times n)$
- Space: $O(n)$

**走一遍。** 同樣 `"abcde"` 跟 `"ace"`。每一輪印出這一輪讀的 `prev` 跟寫出來的 `cur`，寫完兩者互換，`cur` 才變成下一輪要讀的 `prev`。

i=1（比較 `a`）：

| | j=0 | j=1 (`a`) | j=2 (`c`) | j=3 (`e`) |
|---|---|---|---|---|
| prev（讀，第 0 個 row 的邊界） | 0 | 0 | 0 | 0 |
| cur（寫） | 0 | 1 | 1 | 1 |

`a` 對 `a` 相同，`cur[1] = prev[0] + 1 = 1`。後兩格不同，跟上面（prev）比、跟左邊（cur 剛寫的）比，都是 1。交換後 `prev = [0, 1, 1, 1]`。

i=2（比較 `b`）：

| | j=0 | j=1 (`a`) | j=2 (`c`) | j=3 (`e`) |
|---|---|---|---|---|
| prev（讀） | 0 | 1 | 1 | 1 |
| cur（寫） | 0 | 1 | 1 | 1 |

`b` 跟三個字元都不同，每格都是 max(上面, 左邊)，整個 row 照抄上一輪。交換後 `prev = [0, 1, 1, 1]`。

i=3（比較 `c`）：

| | j=0 | j=1 (`a`) | j=2 (`c`) | j=3 (`e`) |
|---|---|---|---|---|
| prev（讀） | 0 | 1 | 1 | 1 |
| cur（寫） | 0 | 1 | 2 | 2 |

`c` 對 `c` 相同（j=2），`cur[2] = prev[1] + 1 = 2`。交換後 `prev = [0, 1, 2, 2]`。

i=4（比較 `d`）：

| | j=0 | j=1 (`a`) | j=2 (`c`) | j=3 (`e`) |
|---|---|---|---|---|
| prev（讀） | 0 | 1 | 2 | 2 |
| cur（寫） | 0 | 1 | 2 | 2 |

`d` 不對任何字元，整個 row 照抄上一輪。交換後 `prev = [0, 1, 2, 2]`。

i=5（比較 `e`）：

| | j=0 | j=1 (`a`) | j=2 (`c`) | j=3 (`e`) |
|---|---|---|---|---|
| prev（讀） | 0 | 1 | 2 | 2 |
| cur（寫） | 0 | 1 | 2 | 3 |

`e` 對 `e` 相同（j=3），`cur[3] = prev[2] + 1 = 3`。交換後 `prev = [0, 1, 2, 3]`，答案是 `prev[3] = 3`。

五輪的 `cur` 跟二維陣列的五個 row 一一對應。交換完之後答案在 `prev`，不是 `cur`：最後一輪的 `cur` 寫完就變成新的 `prev`，原本的 `prev` 變成 `cur`，下一輪會被整個蓋掉。

**不用清空 `cur` 也對。** 內層迴圈每輪都會把 `cur[1]` 到 `cur[n]` 全部重新算一次，殘留的舊資料在被讀到之前就已經被蓋掉；`cur[0]` 從頭到尾沒人寫它，永遠是初始值 0。實測跑五萬組隨機字串，加不加這行清空結果都一樣。

<details>
<summary>Go 版本</summary>

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    prev := make([]int, n+1)
    cur := make([]int, n+1)

    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if text1[i-1] == text2[j-1] {
                cur[j] = prev[j-1] + 1 // prev[j-1] 就是左上角
            } else if prev[j] > cur[j-1] {
                cur[j] = prev[j]
            } else {
                cur[j] = cur[j-1]
            }
        }
        prev, cur = cur, prev // 這個 row 變成下一輪的上一個 row
    }

    return prev[n]
}
```

</details>

---

**Overthinking**

**要印出那個子序列本身，而不是長度。** 從 `f[m][n]` 往回走：字元相同就把它記下來、往左上角跳；不同就往 max(上面, 左邊) 那格跳。走到邊界為止，把記下來的字元倒過來就是答案。這也是二維陣列比一維滾動值錢的地方，滾動版把中間的 row 丟掉了，回溯不了。

**跟 72 Edit Distance 是雙胞胎。** 一樣的陣列、一樣的三個來源，差別在填法：LCS 配對成功時 `+1` 求最大，Edit Distance 是字元相同就直接抄左上角、不同就三個來源取最小再 `+1`。認出「兩個字串互相比較」這種題型，就知道要開二維陣列，每格看左上角、上面、左邊這三格。

**跟 [62 Unique Paths](/problem/unique-paths) 的對照。**

| | 62 Unique Paths | 1143 LCS |
|---|---|---|
| 陣列的兩個維度 | 格子的 row 與 column | 兩個字串各自的前綴長度 |
| 每格看幾個來源 | 2 個（上、左） | 3 個（左上、上、左） |
| 中間的運算 | 加法（數方法數） | max（挑最好的） |
| 邊界 | 第 0 個 row 與第 0 個 column 填 1 | 第 0 個 row 與第 0 個 column 填 0 |
| 能不能只留一個 row | 可以，只用上跟左 | 要兩個 row，左上角在上一個 row |

最後一行是實際寫的時候會遇到的：62 的兩個來源裡，「上面」那格在一維陣列裡剛好還沒被覆寫，所以一個 row 就夠。LCS 要的左上角是上一個 row 的 `j-1`，而那一格在這一輪早就被蓋掉了，所以得留兩個 row。

**加法跟 max 的差別**在 [70 Climbing Stairs](/problem/climbing-stairs) 的對照表也講過：數方法數就加起來，挑最好的就 min 或 max。這條線在一維二維都成立。

---

## 解法比較表

| 解法 | Time | Space | 操作次數 | 備註 |
|---|---|---|---|---|
| 樸素遞迴 | $O(2^{m+n})$ | $O(m+n)$ | 兩邊長度 14 無共同字元：80,233,199 次，8 秒 | TLE |
| 二維陣列 | $O(m \times n)$ | $O(m \times n)$ | 兩邊長度 1000：1,000,000 次，0.1 秒 | 想印出子序列本身就得留這個陣列 |
| 兩個 row 滾動 | $O(m \times n)$ | $O(n)$ | 同上 | 只要長度的話用這個 |

次數是實際跑計數器數出來的，秒數是次數除以 $10^7$。二維陣列在題目上限要 1001 × 1001 個數字，約 8 MB，還在限制內，所以不壓縮也過得了。

---

## 結論

兩個字元相同，f[i][j] 就是左上角那格加一；不同就是 max(上面, 左邊)。陣列多開一個 row 一個 column 放空字串的情況，邊界就自動是 0。滾動只要留兩個 row，因為左上角住在上一個 row。
