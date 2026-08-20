給一個字串，找出最長的一段連續子字串，裡面沒有重複的字元。回傳長度。

```
"abcabcbb"
 ^^^         "abc" 長度 3，再往右就碰到重複的 a
             "bca" 和 "cab" 也是 3，但不會更長
答案：3

"bbbbb"
 ^           每個都是 b，最長就是 1
答案：1

"pwwkew"
   ^^^       "wke" 長度 3
答案：3
```

substring 是連續的。"pwke" 雖然沒有重複但不連續，那叫 subsequence，不算。

---

## 解法一：暴力

暴力法：檢查每一段子字串有沒有重複。

```typescript
function lengthOfLongestSubstring(s: string): number {
    let best = 0;
    for (let i = 0; i < s.length; i++) {
        const seen = new Set<string>();
        for (let j = i; j < s.length; j++) {
            if (seen.has(s[j])) { // 碰到重複，這段結束
                break;
            }
            seen.add(s[j]);
            if (j - i + 1 > best) {
                best = j - i + 1;
            }
        }
    }
    return best;
}
```

- Time: $O(n^2)$
- Space: O(n)

---

## 解法二：滑動視窗

暴力法的問題：每次碰到重複，左邊界只往右移一格，重新開始掃。很多字元被重複檢查。

用 Sliding Window。右指標一直往右擴，碰到重複時左指標往右縮，直到視窗裡沒有重複為止。

還可以提早結束。`l` 只往右移，所以從現在起最長也只可能是 `s.length - l`。這個值一旦不超過 `best`，剩下的位置全掃也贏不了，迴圈直接結束。

```typescript
function lengthOfLongestSubstring(s: string): number {
    const seen = new Map<string, number>(); // 字元 → 視窗內出現次數
    let best = 0;
    let l = 0;

    for (let r = 0; r < s.length; r++) {
        if (s.length - l <= best) break;    // 剩下的長度贏不了 best
        seen.set(s[r], (seen.get(s[r]) ?? 0) + 1);
        // 視窗內有重複，左指標往右縮
        while (seen.get(s[r])! > 1) {
            seen.set(s[l], seen.get(s[l])! - 1);
            l++;
        }
        best = Math.max(best, r - l + 1);
    }
    return best;
}
```

- Time: O(n) — 左右指標各走一遍
- Space: O(n) — map 最多存 n 個字元

用 `"abcabcbb"` 走一遍：

| r | 字元 | 縮掉誰 | l | window | 長度 | best | 剩下 |
|---|---|---|---|---|---|---|---|
| 0 | a | 沒重複 | 0 | `a` | 1 | 1 | 8 |
| 1 | b | 沒重複 | 0 | `ab` | 2 | 2 | 8 |
| 2 | c | 沒重複 | 0 | `abc` | 3 | 3 | 8 |
| 3 | a | a | 1 | `bca` | 3 | 3 | 7 |
| 4 | b | b | 2 | `cab` | 3 | 3 | 6 |
| 5 | c | c | 3 | `abc` | 3 | 3 | 5 |
| 6 | b | a、b | 5 | `cb` | 2 | 3 | 3 |
| 7 | 剩下 3 ≤ best 3，break | | 5 | | | 3 | 3 |

「剩下」那一欄是 `s.length - l`，也就是從 `l` 到字串結尾還有幾格。

r = 3 到 5 每次只縮一格就夠，因為重複的那個剛好在 window 最左邊。r = 6 的 b 在 window 中間（`abc` 的 b），縮掉 a 之後還是重複，得再縮一次才輪到它。

為什麼左指標只往右不往左？因為左邊已經檢查過了，往回走不會更好。這就是 Sliding Window 能 O(n) 的原因：左右指標各走一遍，不回頭。也因為只往右，「剩下幾格」那一欄只會遞減，提早結束的判斷才成立。

提早結束不改複雜度，還是 O(n)，省的是常數。長度 1000 的字串，前 500 個字元互不重複、後面 `ab` 交替的話，1000 圈變 503 圈。但字元全不同或全相同的時候一圈都省不下來：前者 `l` 一直停在 0，後者 `best` 一直是 1。

<details>
<summary>Go 版本</summary>

```go
func lengthOfLongestSubstring(s string) int {
    seen := map[byte]int{} // 字元 → 視窗內出現次數
    best := 0
    l := 0

    for r := 0; r < len(s); r++ {
        if len(s)-l <= best { // 剩下的長度贏不了 best
            break
        }
        seen[s[r]]++
        // 視窗內有重複，左指標往右縮
        for seen[s[r]] > 1 {
            seen[s[l]]--
            l++
        }
        if r-l+1 > best {
            best = r - l + 1
        }
    }
    return best
}
```

</details>

---

### 記住上次位置，直接跳

上面碰到重複時，左指標一格一格縮。可以直接跳：記住每個字元上次出現的位置，碰到重複時左指標直接跳到「上次出現位置 + 1」。

差別要用重複字元埋在 window 中間的輸入才看得出來。`"abcdec"` 前五步兩邊一樣，`r = 5` 的 `c` 跟 index 2 重複，而 window 是 `abcde`：

```
解法二    while 跑 3 圈，一格一格丟掉 a、b、c，l 才走到 3
這個版本  l = lastSeen.get('c') + 1 = 3，一次指派
```

`"abcabcbb"` 看不出差別，因為那組的重複幾乎都貼在 window 最左邊，縮一格就到位了。

```typescript
function lengthOfLongestSubstring(s: string): number {
    const lastSeen = new Map<string, number>();
    let best = 0;
    let l = 0;

    for (let r = 0; r < s.length; r++) {
        const idx = lastSeen.get(s[r]);
        if (idx !== undefined && idx >= l) {
            l = idx + 1;
        }
        lastSeen.set(s[r], r);
        best = Math.max(best, r - l + 1);
    }
    return best;
}
```

- Time: O(n)
- Space: O(n)

Big O 一樣，但實際跑更快，因為左指標不用一格一格縮了。

變的不是演算法，是 **map 裡存的 value**。同樣一個 map，存 count 只能回答「有沒有重複」，碰到重複只能一格一格縮去找。存 position 能回答「重複的在哪」，直接跳過去。空間一樣是 O(n)，左指標省下的是一格一格移動的那段。

這個選擇不只出現在這題。很多 sliding window 題目都有同一個分岔：map 的 value 存什麼，決定了你碰到問題時能做什麼操作。

解法二那個提早結束的判斷在這裡照樣能用，寫法一模一樣。

---

<details>
<summary>變形：HackerRank 的 session 版</summary>

同一題在 HackerRank 換皮出現過，題名叫 Max Unique Substring Length in a Session：字串裡多一種字元 `*`，把字串切成幾個 session，答案必須落在單一 session 裡面。這條規則不在敘述正文，藏在 Constraints 的定義句：「Each session is defined as a maximal contiguous substring of S without '\*' characters」。看漏的話，window 會把 `*` 當普通字元跨過去，把兩個 session 的字母算成同一段，sample 又只給單一 session 的測資，錯全錯在 hidden case 裡。

解法照舊，只是掃描時多一種事件：碰到 `*`，window 不是縮，是整個作廢，從下一格重新開始。

```typescript
// ... 前面那個提早結束的判斷老樣子
if (s[r] === '*') {
    seen.clear();   // 這個 session 到此為止
    l = r + 1;      // window 從分隔字元的下一格重新開始
    continue;
}
// ... 後面老樣子
```

`"abca*bb"` 的關鍵兩步（前面 `abc` 跟原版一樣，不重複走）：

```
r=3  s[3]='a'   a 重複，縮左邊丟掉 s[0]='a'，l = 1
                window = "bca"，best = 3

r=4  s[4]='*'   seen 清空，l = 5，continue
                window 歸零，best 保留著 3

r=5,6  "bb" 這個 session 最長只有 1，答案還是 3
```

`*` 跟重複字母的差別：重複字母只縮到剛好不重複，`*` 直接把整個 window 作廢。空字串跟全星號不用另外寫特例，迴圈走完 `best` 本來就是 0。

</details>

---

## 結論

Sliding Window 的經典題。右指標擴張，碰到重複就縮左指標。進階版記住每個字元上次出現的位置，碰到重複直接跳過去。
