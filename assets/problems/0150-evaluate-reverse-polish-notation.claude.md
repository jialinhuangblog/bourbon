一串算式卡片：`["2","1","+","3","*"]`。這是逆波蘭表示法（Reverse Polish Notation，也叫後綴表示法）：運算子放在兩個運算元的**後面**。`2 1 +` 就是 `2 + 1`，`... 3 *` 就是「前面算出來的結果 乘以 3」。

```
["2","1","+","3","*"]
= (2 + 1) * 3
= 3 * 3
= 9
```

平常寫的 `(2+1)*3` 是中綴（運算子在中間），要靠括號決定先算誰。後綴不用括號，順序本身就決定了運算次序。這種寫法早期的 HP 計算機在用，因為它跟一個資料結構天生契合：stack。

---

**這題為什麼是 stack？**

看運算子做的事：它要抓「前面最近算出來的兩個數」。`2 1 + 3 *` 的 `*`，要的是剛剛的 `3`（也就是 2+1）跟後面的 `3`。「最近的先拿」就是後進先出，正好是 stack。

規則兩條：
- 遇到數字，push 進 stack。
- 遇到運算子，pop 出兩個數算一算，結果 push 回去。

---

## 解法：stack

用 `["4","13","5","/","+"]` 走一遍：

```
"4"   push 4              stack=[4]
"13"  push 13             stack=[4,13]
"5"   push 5              stack=[4,13,5]
"/"   pop 5、pop 13
      13 / 5 = 2（往 0 截斷）
      push 2              stack=[4,2]
"+"   pop 2、pop 4
      4 + 2 = 6
      push 6              stack=[6]

結束，stack 剩一個 → 答案 6
```

有個容易錯的地方：pop 的順序。先 pop 出來的是**右**運算元，後 pop 的才是**左**。算 `13 / 5`，13 是左、5 是右，但 5 在 stack 頂會先被 pop。加法乘法交換律無所謂，但減法除法會算反，一定要 `a = 後pop, b = 先pop`，寫成 `a - b`、`a / b`。

```typescript
function evalRPN(tokens: string[]): number {
    const stack: number[] = [];
    for (const t of tokens) {
        if (t === '+' || t === '-' || t === '*' || t === '/') {
            const b = stack.pop()!;   // 先 pop 的是右運算元
            const a = stack.pop()!;   // 後 pop 的是左運算元
            let r: number;
            if (t === '+') r = a + b;
            else if (t === '-') r = a - b;
            else if (t === '*') r = a * b;
            else r = Math.trunc(a / b);   // 往 0 截斷
            stack.push(r);
        } else {
            stack.push(Number(t));
        }
    }
    return stack[0];
}
```

- Time: $O(n)$ — 每個 token 看一次
- Space: $O(n)$ — stack 最多裝下所有運算元

<details>
<summary>Go 版本</summary>

```go
func evalRPN(tokens []string) int {
    stack := []int{}
    for _, t := range tokens {
        switch t {
        case "+", "-", "*", "/":
            n := len(stack)
            a, b := stack[n-2], stack[n-1] // a 左、b 右
            stack = stack[:n-2]
            var r int
            switch t {
            case "+":
                r = a + b
            case "-":
                r = a - b
            case "*":
                r = a * b
            case "/":
                r = a / b // Go 整數除法本來就往 0 截斷
            }
            stack = append(stack, r)
        default:
            num, _ := strconv.Atoi(t)
            stack = append(stack, num)
        }
    }
    return stack[0]
}
```

</details>

第二個坑是除法的截斷方向。題目說「往 0 截斷」：`13 / 5 = 2.6 → 2`，`6 / -132 = -0.045 → 0`。

TypeScript 這裡不能用 `Math.floor`。`Math.floor(-0.045)` 會得到 -1（往下取整），但題目要的是往 0 截斷、得 0。所以用 `Math.trunc`，它直接砍掉小數，正負都往 0 靠。

Go 反而不用煩：整數除法 `a / b` 本來就往 0 截斷，`-0.045` 那種情況直接得 0，跟題目要求一致。這是少數 Go 比 TypeScript 少踩一個坑的地方。

---

## 結論

後綴表示法配 stack 是天作之合：運算子要「最近的兩個結果」，就是 LIFO。數字 push、運算子 pop 兩個算完 push 回去。兩個坑記住就好：pop 出來的順序（先右後左，減除別算反）、除法往 0 截斷（TypeScript 用 `Math.trunc` 不用 `Math.floor`）。
