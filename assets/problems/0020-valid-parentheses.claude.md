給一個只有括號的字串 `()[]{}`，判斷括號是不是合法的。合法 = 每個開括號都有對應的同類型閉括號，而且順序正確。

```
"()"       → true    一對，沒問題
"()[]{}"   → true    三對，各自配對
"(]"       → false   ( 跟 ] 不是同類型
"([)]"     → false   順序錯了，[ 應該先關才對
"([])"     → true    巢狀，內層先關再關外層
```

---

## 解法：Stack

為什麼用 stack？

看 `"([])"`。讀到 `[` 的時候，最近的開括號是 `[`，所以下一個閉括號必須是 `]`。「最近開的要最先關」— 這就是後進先出（LIFO），stack 的定義。

模擬一遍：

```
"([])"

讀 (  → push (       stack: [(]
讀 [  → push [       stack: [(, []
讀 ]  → pop，拿到 [  → ] 配 [ ✓   stack: [(]
讀 )  → pop，拿到 (  → ) 配 ( ✓   stack: []
結束，stack 是空的 → true
```

再看 `"([)]"`：

```
讀 (  → push (       stack: [(]
讀 [  → push [       stack: [(, []
讀 )  → pop，拿到 [  → ) 配 [ ✗ → false
```

第三步就爆了。`[` 還沒關，就先來了 `)`。

---

完整程式碼：

```typescript
function isValid(s: string): boolean {
    const stack: string[] = [];
    const match: Record<string, string> = {
        ')': '(',
        ']': '[',
        '}': '{',
    };

    for (const c of s) {
        if (c === '(' || c === '[' || c === '{') {
            stack.push(c);
        } else {
            if (stack.length === 0) return false;
            const top = stack.pop()!;
            if (top !== match[c]) return false;
        }
    }
    return stack.length === 0;
}
```

- Time: O(n)
- Space: O(n)（最壞情況全是開括號）

<details>
<summary>Go 版本</summary>

```go
func isValid(s string) bool {
    stack := []byte{}
    match := map[byte]byte{
        ')': '(',
        ']': '[',
        '}': '{',
    }

    for i := 0; i < len(s); i++ {
        c := s[i]
        if c == '(' || c == '[' || c == '{' {
            stack = append(stack, c) // 開括號，push
        } else {
            // 閉括號，pop 出來比對
            if len(stack) == 0 {
                return false // 沒有對應的開括號
            }
            // Go 沒有 pop()，手動做兩步：
            top := stack[len(stack)-1]    // 1. 看最上面那個
            stack = stack[:len(stack)-1]  // 2. 砍掉最後一個（等於 pop）
            if top != match[c] {
                return false // 類型不對
            }
        }
    }
    return len(stack) == 0 // stack 要清空才算合法
}
```

</details>

---

**兩個容易漏的 edge case**

**1. 只有閉括號 `")"`**

stack 是空的就要 pop → 直接 return false。

**2. 只有開括號 `"("`**

跑完整個字串，stack 裡還剩東西 → `len(stack) == 0` 是 false。

這兩個 case 都不需要特別寫 if，已經被程式碼裡的兩個 check 自然處理了。

---

## 結論

Stack 的入門題。開括號 push，閉括號 pop 出來比對。最後 stack 要是空的。「最近開的最先關」就是 stack 的 LIFO 特性。
