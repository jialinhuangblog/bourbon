一疊單字卡攤在桌上：eat、tea、tan、ate、nat、bat。規則很簡單：能靠重排字母變成彼此的，丟同一堆。

```
["eat","tea","tan","ate","nat","bat"]
        ↓
["eat","tea","ate"]   ← 這三張都是 a,e,t
["tan","nat"]         ← 這兩張都是 a,n,t
["bat"]               ← 自己一堆
```

翻回 code 術語：把互為 anagram 的字串分組。難的不是判斷「兩個是不是 anagram」（[242 Valid Anagram](/problem/valid-anagram) 講過了），是怎麼讓「同一組」的字自動聚在一起。

---

**解題引導**

用 `["eat","tea","bat"]` 想。

**Step 1：怎麼讓 eat 和 tea 被認出是同一組？**

*same letters, different order, need something order-blind*

<span class="spoiler">給每個字算一個「不管順序的身分證」。同一組 anagram 算出來要一樣。Give each string a canonical key that ignores order.</span>

**Step 2：什麼東西「重排後不變」，可以當這張身分證？**

*sort the letters, or count them*

<span class="spoiler">兩條路：把字母排序（eat→aet, tea→aet），或數每個字母幾次（都是 a1 e1 t1）。兩者對同一組都一樣。Sorted string, or letter-count signature.</span>

**Step 3：算完身分證，怎麼分組？**

*a map from key to bucket*

<span class="spoiler">用 hashmap，key 是身分證，value 是一個 list。掃過每個字，算 key，丟進對應的 list。A map from key to list of strings.</span>

想完再往下看 code。

---

## 解法一：排序當 key

一組 anagram 排序後長得一模一樣。`eat` 排成 `aet`，`tea` 也排成 `aet`。那就拿排序後的字串當 key，同 key 的丟進同一個 list。

用 `["eat","tea","bat"]` 走一遍：

```
"eat" → 排序 "aet" → map["aet"] = ["eat"]
"tea" → 排序 "aet" → map["aet"] = ["eat","tea"]   ← 撞上同一個 key
"bat" → 排序 "abt" → map["abt"] = ["bat"]

最後 map 的 values：[["eat","tea"], ["bat"]]
```

```typescript
function groupAnagrams(strs: string[]): string[][] {
    const groups = new Map<string, string[]>();
    for (const s of strs) {
        const key = [...s].sort().join('');   // "eat" → "aet"
        if (!groups.has(key)) groups.set(key, []);
        groups.get(key)!.push(s);
    }
    return [...groups.values()];
}
```

- Time: $O(n \cdot k \log k)$ — n 個字，每個排序花 $k \log k$（k 是字串長度）
- Space: $O(n \cdot k)$ — 所有字都存進 map

<details>
<summary>Go 版本</summary>

```go
func groupAnagrams(strs []string) [][]string {
    groups := map[string][]string{}
    for _, s := range strs {
        b := []byte(s)
        // Go 不能直接 sort 字串，先轉 []byte
        sort.Slice(b, func(i, j int) bool { return b[i] < b[j] })
        key := string(b)                          // 排序後的字串當 key
        groups[key] = append(groups[key], s)
    }
    res := [][]string{}
    for _, g := range groups {
        res = append(res, g)
    }
    return res
}
```

</details>

$N = 10^4$、$k \le 100$，$O(n \cdot k \log k) \approx 10^4 \times 700 \approx 7 \times 10^6$ 操作 → 約 0.7 秒。能過，但排序是多餘的工。我們根本不在乎字母誰排前面，只是借排序製造一個「不管順序」的 key。

---

## 解法二：字母計數當 key

242 的老招：只在乎每個字母幾次，就別排序，直接數。26 個字母的次數就是一組 anagram 的簽名，`eat` 和 `tea` 數出來都是「a 一個、e 一個、t 一個」。

問題是計數陣列怎麼當 key。TypeScript 的 Map 用陣列當 key 會比參考，`[1,0,1]` 和另一個 `[1,0,1]` 是不同物件，撞不到一起。所以要把陣列壓成字串。

```
"eat" → count = [a:1, e:1, t:1] → key "1#0#0#0#1#...#1#..."
"tea" → count = [a:1, e:1, t:1] → 同一個 key，撞上
```

```typescript
function groupAnagrams(strs: string[]): string[][] {
    const groups = new Map<string, string[]>();
    for (const s of strs) {
        const count = new Array(26).fill(0);
        for (const ch of s) count[ch.charCodeAt(0) - 97]++;
        const key = count.join('#');          // 用 # 隔開，見下方說明
        if (!groups.has(key)) groups.set(key, []);
        groups.get(key)!.push(s);
    }
    return [...groups.values()];
}
```

`count[ch.charCodeAt(0) - 97]` 是把小寫字母壓成 0~25 的 index。`charCodeAt(0)` 取字元的數字碼（`'a'`=97、`'z'`=122），減 97 讓 `'a'` 對到第 0 格。JS 沒有 char 型別，單一字母也是長度 1 的字串，所以要 `charCodeAt(0)` 取第 0 位。

字元跟數字碼可以來回轉：

```typescript
'c'.charCodeAt(0) - 97         // 2      字母 → 0~25
String.fromCharCode(2 + 97)    // 'c'    0~25 → 字母
```

`fromCharCode` 是 `String` 的靜態方法，不是掛在字串實例上（`"c".fromCharCode(...)` 不存在）。要處理 emoji、罕見漢字那種超出範圍的字元，改用 `codePointAt` / `String.fromCodePoint` 那一對。

為什麼 `join('#')` 要加分隔符？沒有的話，`[1,12]` 變成 `"112"`，`[11,2]` 也變成 `"112"`，兩個不同的計數撞成同一個 key。加了 `#` 變成 `"1#12"` 和 `"11#2"`，分得開。

- Time: $O(n \cdot k)$ — 每個字掃一遍數字母，省掉排序的 $\log k$
- Space: $O(n \cdot k)$

<details>
<summary>Go 版本</summary>

```go
func groupAnagrams(strs []string) [][]string {
    // Go 的固定長度 array 本身可比較，能直接當 map key，不用轉字串
    groups := map[[26]int][]string{}
    for _, s := range strs {
        var count [26]int
        for _, ch := range s {
            count[ch-'a']++
        }
        groups[count] = append(groups[count], s)   // 計數陣列直接當 key
    }
    res := [][]string{}
    for _, g := range groups {
        res = append(res, g)
    }
    return res
}
```

</details>

Go 這裡比 TypeScript 乾淨：`[26]int` 是固定長度 array，Go 允許可比較的 array 當 map key，兩個內容一樣的 array 就是同一個 key。不像 slice（`[]int`）不能當 key，array 可以。省掉了 TypeScript 那段壓字串。

$N = 10^4$、$k \le 100$，$O(n \cdot k) \approx 10^6$ 操作 → 約 0.1 秒，比排序快一個量級。

---

## 解法比較表

| 解法 | Time | Space | $N=10^4$ 秒數 | 備註 |
|---|---|---|---|---|
| 排序當 key | $O(n \cdot k \log k)$ | $O(n \cdot k)$ | 約 0.7 秒 | 好寫好記，面試先講這個 |
| 計數當 key | $O(n \cdot k)$ | $O(n \cdot k)$ | 約 0.1 秒 | 字母集固定（26 個）才划算 |

k 大的時候計數法優勢明顯；k 很小（例如都是兩三個字母）時，排序的 log k 幾乎是常數，兩者差不多。

---

## 結論

分組就是給每個字算一張「不管順序的身分證」。排序或計數都行，計數快一點。同一個計數手法，[242 Valid Anagram](/problem/valid-anagram) 拿來比較兩個字，這裡拿來當分組 key，[347 Top K Frequent Elements](/problem/top-k-frequent-elements) 還會再用一次。
