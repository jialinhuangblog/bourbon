一棟房子，門裡面還有門。每個 `(` 是往裡面走進一扇門，每個 `)` 是退出來一扇。走完之後人要站在最外面，而且過程中不能退出一扇沒進去過的門。給你 3 扇門，問所有走得通的走法。

```
n = 3

((()))     一路進到第三層，再一路退出來
(()())     進兩層，退回一層，再進一層，然後全部退出來
(())()     進兩層全部退完，再進一層退掉
()(())     先進一層退掉，再進兩層退完
()()()     進一層退一層，重複三次

共 5 種
```

翻回 code 術語：給 n 對括號，列出所有合法的排列。合法的定義就是上面那兩條規則，只是換成字元在講。

---

**解題引導**

用 `n = 2` 想，答案是 `(())` 跟 `()()`。

**Step 1：一個括號字串要滿足什麼才叫合法？**

*read left to right, closings must never outnumber openings*

<span class="spoiler">從左往右讀，讀到任何一個位置為止，右括號的數量都不能超過左括號。讀到結尾時兩邊數量相等。Read left to right, closings never exceed openings, and they tie at the end.</span>

**Step 2：2n 個位置每格填左或右，全部列出來再逐一檢查，會列出幾個？**

*two choices per slot, and almost none of them survive*

<span class="spoiler">2 的 2n 次方個。n=8 是 65536 個，其中合法的只有 1430 個，其餘的都是產生完才發現不能用。Two to the power of 2n, and only a small fraction survives.</span>

**Step 3：能不能在填的當下就避開，不要等填完才檢查？**

*two counters decide the branch, not a validator afterwards*

<span class="spoiler">記住目前放了幾個左括號、幾個右括號。左括號沒放滿 n 個就可以再放一個，右括號的數量小於左括號的數量才可以放。這兩個條件成立的話，長出來的每一條路都合法。Track both counts and only branch where the rule still holds.</span>

想完再往下看 code。

---

## 解法一：全部列出來再檢查

2n 個位置，每格兩種選擇，遞迴把所有組合都產生出來。長度滿了才檢查合不合法。

檢查的方式是掃描一遍，維護一個計數器，遇到左括號加一、遇到右括號減一。中途變成負數就是有一扇門沒進去過就退出來了，結尾不是零就是有門沒關上。

用 `n = 3` 走，遞迴長出來的 64 個字串裡挑幾個看：

```
"(((((("   depth 一路加到 6，結尾不是 0        → 不合法
"((()))"   depth 走 1,2,3,2,1,0，中途沒負過   → 合法 ✓
"()()()"   depth 走 1,0,1,0,1,0               → 合法 ✓
")))((("   第一個字元就讓 depth = -1          → 不合法
"()))(("   第三個字元讓 depth = -1            → 不合法
```

這五個都是長度 6 的完整字串。`build` 在長度湊滿 2n 之前不會呼叫 `isValid`，中途那些半成品完全沒有被檢查過，它們只是繼續往下長。

```typescript
function generateParenthesis(n: number): string[] {
    const res: string[] = [];

    function isValid(s: string): boolean {
        let depth = 0;   // 目前在第幾層門裡面
        for (const ch of s) {
            depth += ch === '(' ? 1 : -1;
            if (depth < 0) return false;   // 退出一扇沒進去過的門
        }
        return depth === 0;   // 結尾要回到最外面
    }

    // 每一格填 ( 或 )，兩條路都走，長度滿了才檢查
    function build(path: string): void {
        if (path.length === 2 * n) {
            if (isValid(path)) res.push(path);
            return;
        }
        build(path + '(');
        build(path + ')');
    }

    build('');
    return res;
}
```

- Time: $O(2^{2n} \times n)$ — 產生 $2^{2n}$ 個字串，每個檢查 $2n$ 步
- Space: $O(n)$ — 遞迴深度 $2n$，不算輸出

<details>
<summary>Go 版本</summary>

```go
func generateParenthesis(n int) []string {
    res := []string{}

    isValid := func(s string) bool {
        depth := 0
        for _, ch := range s {
            if ch == '(' {
                depth++
            } else {
                depth--
            }
            if depth < 0 { // 退出一扇沒進去過的門
                return false
            }
        }
        return depth == 0
    }

    var build func(path string)
    build = func(path string) {
        if len(path) == 2*n {
            if isValid(path) {
                res = append(res, path)
            }
            return
        }
        build(path + "(")
        build(path + ")")
    }

    build("")
    return res
}
```

</details>

$n$ 上限是 8，$2^{16} \times 16 \approx 1.05 \times 10^6$ 次操作，約 0.1 秒。這題的 constraints 小，暴力解不會 TLE，交上去會過。

問題不在會不會超時。65536 條路裡面只有 1430 條的結果拿得出來，其餘 64106 條都走到底、產生一個完整字串、再被 `isValid` 排除。

---

## 解法二：Backtracking，兩個計數器

浪費的來源是「先產生再檢查」。合法的條件在 Step 1 已經寫出來了，那就在每一步做選擇的時候直接套用，不合法的分支根本不要走進去。

兩個計數器：`opened` 是已經放了幾個左括號，`closed` 是幾個右括號。

- `opened < n` 才能放左括號。左括號總共只有 n 個配額
- `closed < opened` 才能放右括號。右括號的數量一旦追上左括號，再放就會退出一扇沒進去過的門

用 `n = 3` 走一遍，縮排表示遞迴的深度。放 是 `path.push`，撤 是 `path.pop`；同一個縮排的放跟撤是一對，撤完狀態回到放之前那一刻，然後才輪到同層的下一個選擇：

```
backtrack(opened=0, closed=0, path="")
  放( → (1,0) "("
    放( → (2,0) "(("
      放( → (3,0) "((("             opened 用完了，之後只能放 )
        放) → (3,1) "((()"
          放) → (3,2) "((())"
            放) → (3,3) "((()))"    長度到 6 → 收集 ✓
            撤) → (3,2) "((())"
          撤) → (3,1) "((()"
        撤) → (3,0) "((("
      撤( → (2,0) "(("              ((( 整棵走完，回到 (( 換放 )
      放) → (2,1) "(()"
        放( → (3,1) "(()("
          放) → (3,2) "(()()"
            放) → (3,3) "(()())"    收集 ✓
            撤) → (3,2) "(()()"
          撤) → (3,1) "(()("
        撤( → (2,1) "(()"
        放) → (2,2) "(())"
          放( → (3,2) "(())("
            放) → (3,3) "(())()"    收集 ✓
            撤) → (3,2) "(())("
          撤( → (2,2) "(())"
        撤) → (2,1) "(()"
      撤) → (2,0) "(("
    撤( → (1,0) "("
    放) → (1,1) "()"
      放( → (2,1) "()("
        放( → (3,1) "()(("
          放) → (3,2) "()(()"
            放) → (3,3) "()(())"    收集 ✓
            撤) → (3,2) "()(()"
          撤) → (3,1) "()(("
        撤( → (2,1) "()("
        放) → (2,2) "()()"
          放( → (3,2) "()()("
            放) → (3,3) "()()()"    收集 ✓
            撤) → (3,2) "()()("
          撤( → (2,2) "()()"
        撤) → (2,1) "()("
      撤( → (1,1) "()"
    撤) → (1,0) "("
  撤( → (0,0) ""

結果 ["((()))", "(()())", "(())()", "()(())", "()()()"]
```

**這棵樹上一條白走的路都沒有。** 每一條走到底的路都收集到一個答案，沒有出現走完才發現不合法的情況。

原因是那兩個條件不只保證當下這一步合法，也保證剩下的部分一定填得完。任何一個 `opened <= n` 而且 `closed <= opened` 的狀態，都可以先把剩下的左括號補齊，再把右括號全部補上，湊出一個合法字串。所以走進去就一定有東西可以拿。

對照解法一的 `isValid`：那個函式排除的是已經成形的失敗品，這裡的兩個 `if` 是在失敗發生之前就不往那邊走。

```typescript
function generateParenthesis(n: number): string[] {
    const res: string[] = [];
    const path: string[] = [];

    function backtrack(opened: number, closed: number): void {
        if (path.length === 2 * n) {   // 位置填滿，這條路走完了
            res.push(path.join(''));
            return;
        }
        if (opened < n) {              // 左括號還有配額
            path.push('(');
            backtrack(opened + 1, closed);
            path.pop();                // 撤回，換另一個選擇
        }
        if (closed < opened) {         // 右括號追不過左括號才能放
            path.push(')');
            backtrack(opened, closed + 1);
            path.pop();
        }
    }

    backtrack(0, 0);
    return res;
}
```

- Time: $O(C_n \times n)$ — $C_n$ 是第 n 個 Catalan 數，也就是答案的個數；每個答案要 join 出長度 $2n$ 的字串
- Space: $O(n)$ — 遞迴深度加上 `path`，不算輸出

<details>
<summary>Go 版本</summary>

```go
func generateParenthesis(n int) []string {
    res := []string{}
    path := make([]byte, 0, 2*n) // 容量一次開滿，append 不會重新配置

    var backtrack func(opened, closed int)
    backtrack = func(opened, closed int) {
        if len(path) == 2*n {
            res = append(res, string(path))
            return
        }
        if opened < n {
            path = append(path, '(')
            backtrack(opened+1, closed)
            path = path[:len(path)-1] // 撤回
        }
        if closed < opened {
            path = append(path, ')')
            backtrack(opened, closed+1)
            path = path[:len(path)-1]
        }
    }

    backtrack(0, 0)
    return res
}
```

</details>

$n = 8$ 時 $C_8 = 1430$，$1430 \times 16 \approx 2.3 \times 10^4$ 次操作，$0.01$ 秒不到。跟解法一的差別不只是快了 45 倍，是那 64106 條白工整個不存在。

## 解法三：DP，Catalan 分解

前面兩個解法都在一格一格填。換一個切法：任何合法字串的第一個字元一定是 `(`，找到跟它配對的那個 `)`，字串就被切成兩段，括號裡面的 A 跟括號後面的 B，形狀是 `(A)B`。A 跟 B 自己也是合法字串或空字串。這個分解是唯一的，因為配對第一個 `(` 的 `)` 只有一個位置。

所以 k 對括號的答案可以從小的組出來：括號裡面放 i 對、括號後面放 k-1-i 對，i 從 0 跑到 k-1。`dp[k]` 記「k 對括號的所有合法字串」，從 `dp[0]` 一路組到 `dp[n]`，就是 bottom-up DP。

每一層做的事只有一件：對每種切法，把「裡面的每一條」配上「後面的每一條」，組成 `'(' + a + ')' + b`。把這個動作取名叫 `cross`。

起點 `dp[0] = [""]`：陣列裡有一個元素，那個元素是空字串。空字串是「0 對括號」唯一一種合法排列，因為它在陣列裡是一筆真的資料，迴圈才會為它跑一圈，只是 `a` 拼進去的時候不會多出任何字元。要是起點給 `[]`（零個元素），第一層的迴圈一圈都不跑，之後每一層跟著全空。

用 `n = 3` 一層一層填。每一層的切法是「括號裡面放 `inside` 對、括號後面放 `k - 1 - inside` 對」，`a` 取自 `dp[inside]`，`b` 取自 `dp[k - 1 - inside]`。

「切法」指的是 `inside` 的取值。一種切法產出幾條，看兩邊各有幾條相乘，所以切法數跟字串數不一樣。

**k = 1**，一種切法：`dp[0] × dp[0]`，產出 `1×1 = 1` 條

| inside | a | b | `'(' + a + ')' + b` |
|---|---|---|---|
| 0 | `""` | `""` | `()` |

`dp[1] = ["()"]`

**k = 2**，兩種切法：`dp[0] × dp[1]`、`dp[1] × dp[0]`，產出 `1×1 + 1×1 = 2` 條

| inside | a | b | `'(' + a + ')' + b` |
|---|---|---|---|
| 0 | `""` | `"()"` | `()()` |
| 1 | `"()"` | `""` | `(())` |

`dp[2] = ["()()", "(())"]`

**k = 3**，三種切法：`dp[0] × dp[2]`、`dp[1] × dp[1]`、`dp[2] × dp[0]`，產出 `1×2 + 1×1 + 2×1 = 5` 條

| inside | a | b | `'(' + a + ')' + b` |
|---|---|---|---|
| 0 | `""` | `"()()"` | `()()()` |
| 0 | `""` | `"(())"` | `()(())` |
| 1 | `"()"` | `"()"` | `(())()` |
| 2 | `"()()"` | `""` | `(()())` |
| 2 | `"(())"` | `""` | `((()))` |

`dp[3] = ["()()()", "()(())", "(())()", "(()())", "((()))"]`

`1×2 + 1×1 + 2×1 = 5` 就是 Catalan 數的遞迴式。$C_k$ 是「k 對括號有幾條合法字串」，也就是 `dp[k]` 的長度：

```
C_0 = 1     ""
C_1 = 1     ()
C_2 = 2     ()()  (())
C_3 = 5     ()()()  ()(())  (())()  (()())  ((()))
```

代進去：$C_3 = C_0C_2 + C_1C_1 + C_2C_0 = 1{\times}2 + 1{\times}1 + 2{\times}1 = 5$，跟上面三張表的列數 1、2、5 一致。這個解法叫 Catalan 分解，原因在這裡。答案的數量本來就照這個遞迴式長，這個解法只是把數字換成真的字串。

後面幾項是 14、42、132、429、1430，所以 `n = 8` 的答案有 1430 條。

順序跟解法二不同，題目說 any order 所以沒差。因為分解唯一，每條字串只會被組出來一次，不會出現重複。

```typescript
function generateParenthesis(n: number): string[] {
    // 裡面的每一條配上後面的每一條，組成 (A)B
    function cross(insides: string[], afters: string[]): string[] {
        const out: string[] = [];
        for (const a of insides) {
            for (const b of afters) {
                out.push('(' + a + ')' + b);
            }
        }
        return out;
    }

    const dp: string[][] = [[""]];   // dp[k] = k 對括號的所有合法字串
    for (let k = 1; k <= n; k++) {
        let cur: string[] = [];
        for (let inside = 0; inside < k; inside++) {   // 括號裡面放幾對
            cur = cur.concat(cross(dp[inside], dp[k - 1 - inside]));
        }
        dp.push(cur);
    }
    return dp[n];
}
```

- Time: $O(C_n \times n)$ — 每一層的字串都要真的組出來，被最後一層（$C_8 = 1430$ 條、每條長 16）支配
- Space: $O(\sum_k C_k \times k)$ — `dp[0..n]` 每一層整包留在記憶體，$n=8$ 時全部加起來 2056 條字串

<details>
<summary>Go 版本</summary>

```go
func generateParenthesis(n int) []string {
    // 裡面的每一條配上後面的每一條，組成 (A)B
    cross := func(insides, afters []string) []string {
        out := []string{}
        for _, a := range insides {
            for _, b := range afters {
                out = append(out, "("+a+")"+b)
            }
        }
        return out
    }

    dp := [][]string{{""}} // dp[k] = k 對括號的所有合法字串
    for k := 1; k <= n; k++ {
        cur := []string{}
        for inside := 0; inside < k; inside++ { // 括號裡面放幾對
            cur = append(cur, cross(dp[inside], dp[k-1-inside])...)
        }
        dp = append(dp, cur)
    }
    return dp[n]
}
```

</details>

跟解法二比：時間同級，兩邊都被「答案本身有多少字」支配。貴的地方在記憶體，backtracking 的工作空間只有一條 path，這裡是每一層的完整清單都留著，而且就算只要 `dp[n]`，中間每一層還是得整層算完。輪到它好用的情況是題目改問「有幾種」：字串清單換成數量，`C(k) = Σ C(i) × C(k-1-i)` 填幾個數字就結束，backtracking 反而得真的走完 1430 條路才數得出來。

**Overthinking**

[20 Valid Parentheses](/problem/valid-parentheses) 判斷一個字串合不合法要用 stack，因為有三種括號，右括號得知道自己該配對哪一個左括號。這題只有一種括號，配對對象沒有歧義，一個計數器就取代了整個 stack。NeetCode 把 22 放在 Stack 分類是延續 20 的脈絡，實際解法是 backtracking。

## 解法比較表

| 解法 | Time | Space | n=8 走過的路 | 備註 |
|---|---|---|---|---|
| 全部列出再檢查 | $O(2^{2n} \times n)$ | $O(n)$ | 65536 條，1430 條有用 | 過得了，但看得出沒想過 |
| Backtracking | $O(C_n \times n)$ | $O(n)$ | 1430 條，全部有用 | 面試要的解 |
| DP（Catalan 分解） | $O(C_n \times n)$ | $O(\sum C_k \times k)$ | 組出 2056 條（含中間層），全留在記憶體 | 題目改問「有幾種」時最好用 |

## 結論

合法的條件寫得出來，就把它從事後檢查搬到當下的分支判斷。這題剪枝之後整棵遞迴樹上每一條路都通到一個答案。
