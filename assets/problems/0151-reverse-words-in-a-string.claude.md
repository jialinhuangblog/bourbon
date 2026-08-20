把句子裡的單字順序反過來，順便把多餘的空白清掉。

```
"the sky is blue"       → "blue is sky the"
"  hello world  "       → "world hello"            前後空白要砍掉
"a good   example"      → "example good a"         中間多空白壓成一個
```

---

**解題引導**

用 `"  the sky is blue  "` 思考。

**Step 1：這題其實在做幾件事？**

*Read the examples again. It's not just reversing.*

<span class="spoiler">三件事。去掉前後空白、把中間多個空白壓成一個、反轉單字順序。三件事要一起處理。 / Three things — trim ends, collapse inner multi-spaces to one, reverse word order. All at once.</span>

**Step 2：最直覺的做法？**

*Most languages give you a free lunch for this.*

<span class="spoiler">split 成單字陣列、反轉陣列、再用單一空白 join 回來。split 本身就會吃掉多餘空白（Go 的 strings.Fields、JS 的 split 配正則）。 / Split by whitespace, reverse the array, join with single space. Split already handles messy whitespace.</span>

**Step 3：Follow-up 說如果字串可變，能不能 O(1) 空間？**

*Can you do it without allocating a new array of words?*

<span class="spoiler">能。先反轉整個字串，再把每個單字自己反轉回來。兩次反轉就搞定。 / Reverse the whole string, then reverse each word in place. Two passes, no extra array.</span>

**Step 4：為什麼兩次反轉就對了？**

*Picture "the sky is blue" reversed character by character. What do you get?*

<span class="spoiler">反轉整串後變 "eulb si yks eht" — 單字順序對了（blue 跑到最前面），但每個單字的字元順序反了。再把每個單字自己反轉回來，字元順序也對了。 / After reversing all, word order is correct but each word is backward. Reverse each word individually — now both are correct.</span>

**Step 5：那空白怎麼處理？**

*The input has leading, trailing, and multi-spaces. You need to squash them.*

<span class="spoiler">用兩指標。一個讀、一個寫。讀到非空白就抄過去，讀到空白只抄一次（而且前面已經有東西才抄）。 / Two pointers — one reads, one writes. Copy non-spaces directly. Copy a space only if the previous written char wasn't a space.</span>

想完再往下看 code。

---

## 解法一：內建 split

暴力解：split、reverse、join。三行。

```typescript
function reverseWords(s: string): string {
    return s.trim().split(/\s+/).reverse().join(" ");
    // trim 去前後、split(/\s+/) 吃多空白、reverse 反轉、join 補單空白
}
```

<details>
<summary>Go 版本</summary>

```go
import "strings"

func reverseWords(s string) string {
    words := strings.Fields(s)  // Fields 自動處理多空白和前後空白
    // 雙指標反轉陣列
    for i, j := 0, len(words)-1; i < j; i, j = i+1, j-1 {
        words[i], words[j] = words[j], words[i]
    }
    return strings.Join(words, " ")
}
```

</details>

走 `"  the sky is blue  "`：

```
trim + split     → ["the", "sky", "is", "blue"]
reverse          → ["blue", "is", "sky", "the"]
join(" ")        → "blue is sky the"
```

- Time: O(n)
- Space: O(n) — words 陣列 + 輸出字串

---

## 解法二：手刻 split

split 一行解決，但面試官常追問：「**如果沒有 split 可以用，你要怎麼寫？**」他不是在刁難，是想確認你真的懂 split 在做什麼——不然你只是在背 API。

手刻 split 就是**找到每個單字的邊界**。既然最後要反轉單字順序，不如**從右往左掃**，抓到哪個就直接推到 result，自動倒著排。

```typescript
function reverseWords(s: string): string {
    const result: string[] = [];
    let i = s.length - 1;
    while (i >= 0) {
        // 跳過尾端空白
        while (i >= 0 && s[i] === " ") i--;
        if (i < 0) break;
        // 找單字邊界
        const end = i;
        while (i >= 0 && s[i] !== " ") i--;
        result.push(s.substring(i + 1, end + 1));
    }
    return result.join(" ");
}
```

<details>
<summary>Go 版本</summary>

```go
func reverseWords(s string) string {
    var result []string
    i := len(s) - 1
    for i >= 0 {
        // 跳過尾端空白
        for i >= 0 && s[i] == ' ' { i-- }
        if i < 0 { break }
        // 找單字邊界：end 是尾，掃到 ' ' 或起點停，i+1 是頭
        end := i
        for i >= 0 && s[i] != ' ' { i-- }
        result = append(result, s[i+1:end+1])
    }
    return strings.Join(result, " ")
}
```

</details>

走 `"  the sky is blue  "`（長度 19，i 從 18 開始）：

```
index:  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18
char:   _ _ t h e _ s k y _ i  s  _  b  l  u  e  _  _
```

```
i=18,17：' ' → 跳過尾端空白
i=16→13：非空白 → end=16，掃到 i=12（s[12]=' '）停
          → substring(13, 17) = "blue" → result=["blue"]
i=12：' ' → 跳過
i=11→10：非空白 → end=11，掃到 i=9（s[9]=' '）停
          → substring(10, 12) = "is" → result=["blue","is"]
i=9：' ' → 跳過
i=8→6：非空白 → end=8，掃到 i=5（s[5]=' '）停
         → substring(6, 9) = "sky" → result=["blue","is","sky"]
i=5：' ' → 跳過
i=4→2：非空白 → end=4，掃到 i=1（s[1]=' '）停
         → substring(2, 5) = "the" → result=["blue","is","sky","the"]
i=1,0：' ' → 跳過，i=-1，break
```

substring 的兩個參數剛好對應 `(i+1, end+1)`：**起點用 i+1 是因為 i 停在空白上，單字起點是 i+1；終點用 end+1 是因為 substring 右邊界開區間，要多加 1 才涵蓋到 end 那個字元。**

join(" ") → `"blue is sky the"` ✓

- Time: O(n)
- Space: O(n) — result 陣列 + substring + 輸出

空間複雜度**沒改善**，跟 split 版一樣 O(n)。這版的價值是**沒依賴 library helper**，證明你知道 split 背後就是「找空白邊界切單字」這件事。

---

## 解法三：雙反轉

時間已經是 O(n) 了，不會更快——每個字元至少要看一次。

但空間不是 O(1)。words 陣列存了整個字串一遍。Follow-up 問的就是這個：如果字串可變，能不能原地做？

**反轉兩次等於不反轉**。但如果你第一次反轉整串、第二次只反轉每個單字內部，那單字順序被翻了一次（反了），字元順序被翻了兩次（回來了）。

```
原始：    "the sky is blue"
反轉全部： "eulb si yks eht"        ← 單字順序對了，字元順序反了
反轉每字： "blue is sky the"        ← 兩者都對了
```

加上空白處理，流程變成三步：

1. **清空白**：用雙指標，讀/寫各一個，把多餘空白壓掉
2. **反轉全部**：首尾互換到中間
3. **反轉每字**：找到每個單字的邊界，一個個反轉

Go 的 string 不可變，得先轉 `[]byte`。TS 一樣。嚴格來說兩個語言都做不到真正的 O(1)，但這邏輯可以搬到 C/C++ 的 char array 上，那才是 follow-up 要的答案。

```typescript
function reverseWords(s: string): string {
    const b = s.split("");  // TS 字串不可變，還是要轉 array
    let n = cleanSpaces(b);
    b.length = n;
    reverse(b, 0, n - 1);
    let start = 0;
    for (let i = 0; i <= n; i++) {  // <= n 多跑一格，當最後一個單字的虛擬邊界
        // i === n 擺前面，否則 b[n] 會 undefined
        if (i === n || b[i] === " ") {  // 真空白 or 虛擬結尾，都算邊界
            reverse(b, start, i - 1);    // 反轉 [start, i-1] 這段
            start = i + 1;               // 下一段從空白後面開始
        }
    }
    return b.join("");
}

function cleanSpaces(b: string[]): number {
    let j = 0;  // 寫指標
    for (let i = 0; i < b.length; i++) {
        if (b[i] !== " ") {
            b[j++] = b[i];             // 非空白直接抄
        } else if (j > 0 && b[j-1] !== " ") {
            b[j++] = " ";               // 空白只抄一次，且前面得有東西
        }
    }
    if (j > 0 && b[j-1] === " ") j--;  // 砍掉結尾可能的空白
    return j;
}

function reverse(b: string[], l: number, r: number): void {
    while (l < r) { [b[l], b[r]] = [b[r], b[l]]; l++; r--; }
}
```

<details>
<summary>Go 版本</summary>

```go
func reverseWords(s string) string {
    b := []byte(s)
    // 1. 清空白：雙指標壓多空白、去前後空白
    n := cleanSpaces(b)
    b = b[:n]
    // 2. 反轉全部
    reverse(b, 0, n-1)
    // 3. 反轉每個單字：碰到空白當邊界，reverse 前一段
    start := 0
    for i := 0; i <= n; i++ {  // <= n 多跑一格，當最後一個單字的虛擬邊界
        // i == n 一定要擺前面，不然 b[n] 會越界
        if i == n || b[i] == ' ' {  // 真空白 or 虛擬結尾，都算邊界
            reverse(b, start, i-1)   // 反轉 [start, i-1] 這段
            start = i + 1            // 下一段從空白後面開始
        }
    }
    return string(b)
}

func cleanSpaces(b []byte) int {
    j := 0  // 寫指標
    for i := 0; i < len(b); i++ {
        if b[i] != ' ' {
            b[j] = b[i]  // 非空白直接抄
            j++
        } else if j > 0 && b[j-1] != ' ' {
            b[j] = ' '  // 空白只抄一次，且前面得有東西
            j++
        }
    }
    if j > 0 && b[j-1] == ' ' { j-- }  // 砍掉結尾可能的空白
    return j
}

func reverse(b []byte, l, r int) {
    for l < r {
        b[l], b[r] = b[r], b[l]
        l++
        r--
    }
}
```

</details>

走 `"  the sky is blue  "`：

```
原始（底線代表空白）：
  _ _ t h e _ s k y _ i s _ b l u e _ _

cleanSpaces 後：
  t h e _ s k y _ i s _ b l u e          n=15

reverse(0, 14)：
  e u l b _ s i _ y k s _ e h t

反轉每字（找 ' ' 當邊界）：
  [0..3]  "eulb" → "blue"
  [5..6]  "si"   → "is"
  [8..10] "yks"  → "sky"
  [12..14] "eht" → "the"

最後：
  b l u e _ i s _ s k y _ t h e          "blue is sky the"
```

- Time: O(n) — 三趟掃描，每趟 O(n)
- Space: O(1) — 如果字串可變（不算轉 byte array 的話）

---

**能不能更好？**

不能。至少要看過每個字元一次。O(n) 是下限。

空間已經在 O(1)（忽略語言限制），也到底了。

---

**面試追問：為什麼不用 stack？**

有人會想：從左到右掃單字，每個單字 push 進 stack，最後 pop 出來用空白接。

```typescript
function reverseWords(s: string): string {
    const stack = s.trim().split(/\s+/);  // 其實就是個 array
    const result: string[] = [];
    for (let i = stack.length - 1; i >= 0; i--) {
        result.push(stack[i]);
    }
    return result.join(" ");
}
```

<details>
<summary>Go 版本</summary>

```go
func reverseWords(s string) string {
    stack := strings.Fields(s)  // 其實就是個 slice
    var result []string
    for i := len(stack) - 1; i >= 0; i-- {
        result = append(result, stack[i])
    }
    return strings.Join(result, " ")
}
```

</details>

能動，但本質就是 split + reverse，只是換個名字叫 stack。沒有比較聰明。

真正的差異是**空間**：split 版是 O(n)，in-place 雙反轉版是 O(1)。stack 版還是 O(n)。

---

## 結論

簡單版 split/reverse/join 三行解決，O(n) 空間。面試官問 follow-up，就上「反轉整串 + 反轉每字」的雙反轉技巧，O(1) 空間。這招在字串處理題反覆出現，值得記住。
