桌上兩堆 Scrabble 字母牌，左邊拼著 anagram，右邊拼著 nagaram。問：右邊這堆是不是把左邊整堆打散重排出來的？不能多一張，不能少一張。

```
s = "anagram", t = "nagaram" → true（同一組牌重排）
s = "rat",     t = "car"     → false（c 這張牌左邊沒有）
```

翻回 code 術語：判斷兩個字串裡，每個字母的出現次數是否完全相同。

---

**解題引導**

用 `s = "rat", t = "tar"` 想。

**Step 1：「重新排列」這個條件，真正要比的是什麼？**

*order is a lie here, only the counts matter*

<span class="spoiler">順序不重要，重要的是每個字母各出現幾次。Only the letter counts matter, order is irrelevant.</span>

**Step 2：順序不重要的兩個東西，怎麼變得可以直接比較？**

*give both the same order, any order, just the same one*

<span class="spoiler">各自排序。同一組字母排完序長得一模一樣，直接比對就好。Sort both strings, equal result means anagram.</span>

**Step 3：排序要 O(n log n)。既然只在乎次數，能不能直接數？**

*why sort when you can just count*

<span class="spoiler">數就好。統計 s 每個字母出現幾次，再統計 t，比較兩份統計。掃一遍就完成，O(n)。Count letters in both strings and compare the counts.</span>

**Step 4：需要兩份統計嗎？**

*one ledger, add on one side, subtract on the other*

<span class="spoiler">一份就夠。s 的字母加一，t 的字母減一，最後全部歸零就是 anagram。One counter, increment for s, decrement for t, all zeros means true.</span>

**Step 5：這份統計用 hashmap 還是 array 存？**

*the alphabet is tiny and fixed*

<span class="spoiler">題目限定小寫英文字母，只有 26 種。長度 26 的 array 直接用字母當 index，比 hashmap 少一層 hash 計算。A fixed array of 26 is enough.</span>

想完再往下看 code。

---

## 解法一：排序

最直覺：既然順序不重要，就把順序抹掉。兩個字串各自排序，排完一樣就是 anagram。

用 `s = "rat", t = "tar"` 走一遍：

```
"rat" 排序 → "art"
"tar" 排序 → "art"
"art" == "art" → true

對照組 s = "rat", t = "car"：
"rat" 排序 → "art"
"car" 排序 → "acr"
第二個字元 r != c → false
```

```typescript
function isAnagram(s: string, t: string): boolean {
    if (s.length !== t.length) return false;
    const sorted = (x: string) => [...x].sort().join('');
    return sorted(s) === sorted(t);
}
```

- Time: O(n log n)
- Space: O(n)（複製了一份來排）

<details>
<summary>Go 版本</summary>

```go
func isAnagram(s string, t string) bool {
    if len(s) != len(t) {
        return false            // 長度不同，連數都不用數
    }
    sb := []byte(s)
    tb := []byte(t)
    // Go 不能直接 sort 字串，先轉 []byte
    sort.Slice(sb, func(i, j int) bool { return sb[i] < sb[j] })
    sort.Slice(tb, func(i, j int) bool { return tb[i] < tb[j] })
    return string(sb) == string(tb)
}
```

</details>

$N = 5 \times 10^4$，$O(n \log n) \approx 8 \times 10^5$ 操作 → 約 0.08 秒。不會 TLE，但排序做了多餘的工：我們根本不在乎誰排前面，卻花 $n \log n$ 去製造一個順序，只為了讓兩個字串「可以比較」。

---

## 解法二：counting

Step 1 已經想通了：要比的是次數。那就直接數次數，不排序。

一份計數表，s 的字母加一，t 的字母減一。兩邊字母組成相同的話，加減會互相抵銷，最後整張表歸零。

用同一組 `s = "rat", t = "tar"` 走：

```
count[26] 全 0

i=0: s[0]='r' → count[r]++    t[0]='t' → count[t]--
     表：r:1, t:-1
i=1: s[1]='a' → count[a]++    t[1]='a' → count[a]--
     表：a:0, r:1, t:-1（a 加完馬上被減掉）
i=2: s[2]='t' → count[t]++    t[2]='r' → count[r]--
     表：全 0

全 0 → true
```

對照組 `s = "rat", t = "car"`：

```
i=0: r++ / c--   → r:1, c:-1
i=1: a++ / a--   → 抵銷
i=2: t++ / r--   → t:1, r:0, c:-1

c:-1、t:1 沒歸零 → false
```

```typescript
function isAnagram(s: string, t: string): boolean {
    if (s.length !== t.length) return false;
    const count = new Array(26).fill(0);
    for (let i = 0; i < s.length; i++) {
        count[s.charCodeAt(i) - 97]++;   // 97 是 'a' 的 char code
        count[t.charCodeAt(i) - 97]--;
    }
    return count.every(c => c === 0);
}
```

- Time: O(n)
- Space: O(1)（26 格固定，不隨 n 長大）

<details>
<summary>Go 版本</summary>

```go
func isAnagram(s string, t string) bool {
    if len(s) != len(t) {
        return false            // 這行也保證下面能用同一個 i 走兩個字串
    }
    var count [26]int
    for i := 0; i < len(s); i++ {
        count[s[i]-'a']++       // 'a' 減 'a' 是 0，'r' 減 'a' 是 17，字母自己當 index
        count[t[i]-'a']--
    }
    for _, c := range count {
        if c != 0 {
            return false        // 有一格沒歸零就不是
        }
    }
    return true
}
```

</details>

每個字母至少要看一次才知道它存在，O(n) 是下限，不可能更快。

---

### counting（hashmap）

行，把長度 26 的 array 換成 hashmap 結果一樣。但沒必要：array 用字母直接當 index，hashmap 每次存取都要先算 hash。字元集固定又小的時候，array 就是比較快的那個。

hashmap 真正登場的時機是下面這個 follow-up。

---

**Overthinking：題目的 follow-up，輸入變 Unicode 怎麼辦？**

長度 26 的 array 依賴「小寫英文字母」這個前提。字元換成 Unicode，可能的字元有十幾萬種，開 array 不現實，這時才換 hashmap。

但兩個語言各有一個陷阱：

TypeScript：`charCodeAt` 拿的是 UTF-16 code unit，emoji 這種 surrogate pair 會被切成兩半。改用 `for (const ch of s)`，它按 code point 走。長度也要用 `[...s].length` 才是真正的字元數。

```typescript
function isAnagram(s: string, t: string): boolean {
    if ([...s].length !== [...t].length) return false;
    const counts = new Map<string, number>();
    for (const ch of s) counts.set(ch, (counts.get(ch) ?? 0) + 1);
    for (const ch of t) counts.set(ch, (counts.get(ch) ?? 0) - 1);
    return [...counts.values()].every(c => c === 0);
}
```

<details>
<summary>Go 版本</summary>

```go
func isAnagram(s string, t string) bool {
    counts := map[rune]int{}
    for _, r := range s {        // for range 按 rune 走，不會把一個中文字切成三個 byte
        counts[r]++
    }
    for _, r := range t {
        counts[r]--
    }
    for _, c := range counts {
        if c != 0 {
            return false
        }
    }
    return true
}
```

</details>

Go 的坑在 `s[i]`：拿到的是 byte，不是字元。一個中文字在 UTF-8 裡佔 3 bytes，用 `s[i]` 會把一個字切成三塊分開數。所以上面改用 `for range`，它按 rune 走。

---

## 解法比較表

| 解法 | Time | Space | $N=5 \times 10^4$ 秒數 | 備註 |
|---|---|---|---|---|
| 排序後比較 | O(n log n) | O(n) | 約 0.08 秒 | 好寫好懂，暖身夠用 |
| counting（array 26） | O(n) | O(1) | 約 0.005 秒 | 這題的標準答案 |
| counting（hashmap） | O(n) | O(k) | 約 0.02 秒 | 字元集不固定（Unicode）才需要 |

---

## 結論

順序不重要的比較，用計數代替排序。一份計數表加減互抵，全零就是 anagram。同一個 counting 手法，[49 Group Anagrams](/problem/group-anagrams) 跟 [347 Top K Frequent Elements](/problem/top-k-frequent-elements) 會原樣再用一次。
