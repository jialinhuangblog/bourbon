給一個字串，忽略大小寫和非英數字元，判斷正著讀和反著讀是不是一樣。

```
"A man, a plan, a canal: Panama"
 → 去掉符號空格，全轉小寫
 → "amanaplanacanalpanama"
 → 正著反著都一樣 → true

"race a car"
 → "raceacar"
 → 正著 raceacar，反著 racaecar → 不一樣 → false
```

---

## 解法一：清理 + 雙指針

最直覺：先把字串清理好（去掉非英數、轉小寫），再用兩個指標從兩端往中間夾。

```typescript
function isPalindrome(s: string): boolean {
    // 先清理：只留英數，全轉小寫
    const clean: string[] = [];
    for (let i = 0; i < s.length; i++) {
        const c = s.charCodeAt(i);
        if (c >= 65 && c <= 90) {
            // ASCII: A-Z 是 65~90，a-z 是 97~122，中間隔了 [\]^_` 六個符號
            // 所以差距是 97-65=32，不是 26（26 是字母數量）
            clean.push(String.fromCharCode(c + 32)); // 轉小寫
        } else if ((c >= 97 && c <= 122) || (c >= 48 && c <= 57)) {
            clean.push(String.fromCharCode(c));
        }
    }

    // 兩端往中間夾
    let l = 0, r = clean.length - 1;
    while (l < r) {
        if (clean[l] !== clean[r]) {
            return false;
        }
        l++;
        r--;
    }
    return true;
}
```

- Time: O(n)
- Space: O(n)（存了一份清理後的字串）

<details>
<summary>Go 版本</summary>

```go
func isPalindrome(s string) bool {
    // 先清理：只留英數，全轉小寫
    clean := []byte{}
    for i := 0; i < len(s); i++ {
        c := s[i]
        if c >= 'A' && c <= 'Z' {
            // ASCII: A-Z 是 65~90，a-z 是 97~122，中間隔了 [\]^_` 六個符號
            // 所以差距是 97-65=32，不是 26（26 是字母數量）
            clean = append(clean, c+32) // 轉小寫
        } else if (c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') {
            clean = append(clean, c)
        }
    }

    // 兩端往中間夾
    l, r := 0, len(clean)-1
    for l < r {
        if clean[l] != clean[r] {
            return false
        }
        l++
        r--
    }
    return true
}
```

</details>

---

## 解法二：原地雙指針

Space 可以省掉。不用先清理整個字串，直接在原字串上用兩個指標，遇到非英數就跳過。

```
"A man, a plan, a canal: Panama"
 L                             R

L 指到 'A'，R 指到 'a'
→ 都是英數，轉小寫比較：'a' == 'a' ✓

L 往右移，R 往左移
→ L 遇到空格，跳過
→ R 遇到 'P'，停下

繼續比...
```

```typescript
function isPalindrome(s: string): boolean {
    let l = 0, r = s.length - 1;
    while (l < r) {
        while (l < r && !isAlphaNum(s[l])) l++;
        while (l < r && !isAlphaNum(s[r])) r--;
        if (s[l].toLowerCase() !== s[r].toLowerCase()) return false;
        l++;
        r--;
    }
    return true;
}

function isAlphaNum(c: string): boolean {
    return /[a-zA-Z0-9]/.test(c);
}
```

- Time: O(n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func isPalindrome(s string) bool {
    l, r := 0, len(s)-1
    for l < r {
        // 跳過左邊的非英數
        for l < r && !isAlphaNum(s[l]) {
            l++
        }
        // 跳過右邊的非英數
        for l < r && !isAlphaNum(s[r]) {
            r--
        }
        if toLower(s[l]) != toLower(s[r]) {
            return false
        }
        l++
        r--
    }
    return true
}

func isAlphaNum(c byte) bool {
    return (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || (c >= '0' && c <= '9')
}

func toLower(c byte) byte {
    if c >= 'A' && c <= 'Z' {
        return c + 32
    }
    return c
}
```

</details>

---

## 結論

Two Pointers 的基本型。兩個指標從兩端往中間夾，遇到非英數就跳過，比較的時候轉小寫。不需要額外空間。
