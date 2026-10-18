一個字串 `"leetcode"`，從左到右找第一個「只出現一次」的字母，回傳它的 index。找不到回傳 -1。

```
"leetcode"
 l e e t c o d e
 ↑
 l 只出現一次，而且是最左邊的 → index 0

"loveleetcode"
 l 出現 2 次、o 2 次，都不算
 v 只出現 1 次，在 index 2 → 答案 2
```

翻回 code 術語：統計每個字母出現幾次，再從頭找第一個次數為 1 的。又是 [242](/problem/valid-anagram)、[49](/problem/group-anagrams) 那個 counting，這題把它接上「第一個」這個順序條件。

---

**解題引導**

用 `s = "aabb"` 想。

**Step 1：怎麼知道一個字母是不是「只出現一次」？**

*count every letter first*

<span class="spoiler">先數次數。掃一遍統計每個字母出現幾次。Count each letter's frequency.</span>

**Step 2：有了次數，怎麼找「第一個」只出現一次的？**

*scan left to right, return the first with count 1*

<span class="spoiler">再從頭掃一遍原字串，回傳第一個次數為 1 的 index。順序要照原字串，不是照字母表。Second pass over the string, return the first index whose count is 1.</span>

**Step 3：為什麼要掃兩遍，一遍不行嗎？**

*you can't know it's unique until you've seen the whole string*

<span class="spoiler">不行。第一次遇到某字母時，還不知道後面會不會再出現。必須先數完，才能判斷唯一。Uniqueness needs the full count first.</span>

想完再往下看 code。

---

## 解法：計數 + 二次掃描

兩趟。第一趟數次數，第二趟從頭找第一個次數為 1 的。

為什麼一定要兩趟？因為「第一次遇到 l」的時候，你還不知道後面有沒有第二個 l。要判斷唯一，得先把整個字串數完。所以第一趟純粹建計數表，第二趟才做判斷。

用 `s = "leetcode"` 走：

```
第一趟數次數：
l:1 e:3 t:1 c:1 o:1 d:1

第二趟從頭掃（照原字串順序）：
index 0 'l' → count 1 → 回傳 0
```

再用 `s = "aabb"`：

```
第一趟：a:2 b:2
第二趟：
index 0 'a' count 2，跳過
index 1 'a' count 2，跳過
index 2 'b' count 2，跳過
index 3 'b' count 2，跳過
掃完沒有次數 1 的 → -1
```

```typescript
function firstUniqChar(s: string): number {
    const count = new Array(26).fill(0);
    for (const ch of s) count[ch.charCodeAt(0) - 97]++;   // 第一趟：數次數

    for (let i = 0; i < s.length; i++) {                  // 第二趟：找第一個唯一
        if (count[s.charCodeAt(i) - 97] === 1) return i;
    }
    return -1;
}
```

- Time: $O(n)$ — 兩趟都是線性，加起來還是 $O(n)$
- Space: $O(1)$ — 26 格固定，不隨字串長大

<details>
<summary>Go 版本</summary>

```go
func firstUniqChar(s string) int {
    var count [26]int
    for _, ch := range s {
        count[ch-'a']++ // 第一趟：數次數
    }
    for i := 0; i < len(s); i++ {
        if count[s[i]-'a'] == 1 { // 第二趟：第一個次數 1 的
            return i
        }
    }
    return -1
}
```

</details>

跟 242 一樣，字母集固定只有 26 個，用長度 26 的 array 比 hashmap 快：字母直接當 index，省一層 hash 計算。

第二趟為什麼能用 `s[i]` 直接掃、不怕字元順序亂掉？因為題目限定小寫英文字母（一個字元一個 byte）。第一趟用 `for range`（按 rune 走）純粹是習慣，這裡用哪種都行。換成中文那種多 byte 字元才要小心，那是 242 的 Overthinking 講過的坑。

---

## 結論

counting 家族第三題。242 比較兩個字串、49 拿計數當分組 key，這題把計數接上「第一個」的順序條件：數完次數，再照原字串順序掃一次，第一個次數為 1 的就是答案。兩趟的原因是「唯一」得等整個字串數完才能判斷。
