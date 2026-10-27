---
title: "Trie"
category: Data Structures
slug: trie
subtitle: 字元一層一層往下找，前綴樹
date: 2026-02-28T11:51:31
updated: 2026-08-04
---

# Trie

打開手機。輸入「app」。自動跳出「apple」「application」「append」。

你才打三個字，它就知道你可能要打什麼。

用 hash map 做得到嗎？假設字典裡有十萬個單字，使用者輸入 "app"，就得把十萬個 key 全部掃描一遍，逐一檢查是不是 "app" 開頭。O(n)。

用排序 + binary search？找到第一個 "app" 開頭的，然後往後掃。可以，但要先排好序，而且插入新字很麻煩。

有一種資料結構，天生就是為了「前綴搜尋」而生的。

Trie。念 "try"。又叫 prefix tree。

---

## 長什麼樣？

假設你要存三個字：`["apple", "app", "bat"]`。

```
        root
       /    \
      a      b
      |      |
      p      a
      |      |
      p      t*
     / \
    l   *
    |
    e*
```

`*` 代表「這裡是一個完整的字」。

每個節點不存整個字。只存一個字元。從 root 走到任何 `*` 節點，沿路的字元拼起來就是一個完整的字。

root → a → p → p → `*` = "app"
root → a → p → p → l → e → `*` = "apple"
root → b → a → t → `*` = "bat"

"app" 和 "apple" 共享前三個節點。這就是 trie 省空間的方式：共同前綴只存一次。

---

## 節點結構

```go
type TrieNode struct {
    children [26]*TrieNode    // a-z 各一個位置
    isEnd    bool             // 這個節點是不是某個字的結尾
}
```

為什麼是 26？因為英文小寫字母有 26 個。`children[0]` 是 'a'，`children[1]` 是 'b'，...，`children[25]` 是 'z'。

如果要支援更多字元，可以用 `map[rune]*TrieNode`。但 26 個位置的 array 夠應付大多數 LeetCode 題。

---

## 三個基本操作

### Insert（插入一個字）

一個字元一個字元走。沒有路就開一條新路。走到最後一個字元，標記 `isEnd = true`。

插入 "apple"：

```
開始：root（空的）

'a': root.children[0] == nil → 建新節點
     root → [a]

'p': a.children[15] == nil → 建新節點
     root → [a] → [p]

'p': p.children[15] == nil → 建新節點
     root → [a] → [p] → [p]

'l': p.children[11] == nil → 建新節點
     root → [a] → [p] → [p] → [l]

'e': l.children[4] == nil → 建新節點
     root → [a] → [p] → [p] → [l] → [e]

標記 e.isEnd = true
```

再插入 "app"：

```
'a': root.children[0] 已存在 → 直接走過去
'p': a.children[15] 已存在 → 直接走過去
'p': p.children[15] 已存在 → 直接走過去

標記 p.isEnd = true
```

"app" 和 "apple" 共享了 a → p → p 這三個節點。沒有浪費。

```go
func (t *Trie) Insert(word string) {
    node := t.root
    for _, ch := range word {
        idx := ch - 'a'
        if node.children[idx] == nil {
            node.children[idx] = &TrieNode{}    // 沒路就開路
        }
        node = node.children[idx]
    }
    node.isEnd = true    // 標記：這是一個完整的字
}
```

時間：$O(m)$，m = 字的長度。每個字元看一次。

### Search（查詢完整的字）

跟 insert 一樣一個字元一個字元走。走到最後，檢查 `isEnd`。

查 "app"：

```
'a': root.children[0] 存在 → 走過去
'p': a.children[15] 存在 → 走過去
'p': p.children[15] 存在 → 走過去

p.isEnd == true → 找到了！
```

查 "ap"：

```
'a': root.children[0] 存在 → 走過去
'p': a.children[15] 存在 → 走過去

p.isEnd == false → "ap" 不是一個完整的字
```

查 "cat"：

```
'c': root.children[2] == nil → 直接回 false
```

```go
func (t *Trie) Search(word string) bool {
    node := t.root
    for _, ch := range word {
        idx := ch - 'a'
        if node.children[idx] == nil {
            return false    // 路斷了
        }
        node = node.children[idx]
    }
    return node.isEnd    // 走到底了，但它是不是一個完整的字？
}
```

### StartsWith（查詢前綴）

跟 search 幾乎一樣。差別：走到底不用檢查 `isEnd`。只要路沒斷，前綴就存在。

查 "ap" 是不是某個字的前綴：

```
'a': 存在 → 走
'p': 存在 → 走

路沒斷 → true（"apple" 和 "app" 都是 "ap" 開頭的）
```

```go
func (t *Trie) StartsWith(prefix string) bool {
    node := t.root
    for _, ch := range prefix {
        idx := ch - 'a'
        if node.children[idx] == nil {
            return false
        }
        node = node.children[idx]
    }
    return true    // 路沒斷就好，不管是不是完整的字
}
```

Search 和 StartsWith 的差別就在最後一行。一個查 `isEnd`，一個直接 `true`。

---

## 完整實作

[#208 Implement Trie (Prefix Tree)](/problem/implement-trie-prefix-tree)

```go
type Trie struct {
    root *TrieNode
}

type TrieNode struct {
    children [26]*TrieNode
    isEnd    bool
}

func Constructor() Trie {
    return Trie{root: &TrieNode{}}
}

func (t *Trie) Insert(word string) {
    node := t.root
    for _, ch := range word {
        idx := ch - 'a'
        if node.children[idx] == nil {
            node.children[idx] = &TrieNode{}
        }
        node = node.children[idx]
    }
    node.isEnd = true
}

func (t *Trie) Search(word string) bool {
    node := t.root
    for _, ch := range word {
        idx := ch - 'a'
        if node.children[idx] == nil {
            return false
        }
        node = node.children[idx]
    }
    return node.isEnd
}

func (t *Trie) StartsWith(prefix string) bool {
    node := t.root
    for _, ch := range prefix {
        idx := ch - 'a'
        if node.children[idx] == nil {
            return false
        }
        node = node.children[idx]
    }
    return true
}
```

三個方法的迴圈完全一樣，差別在遇到 nil 的時候要不要提早回傳，以及走到最後回傳什麼。

---

## 為什麼不用 Hash Map？

「我直接用 `map[string]bool` 不就好了？」

可以。Search 是 $O(1)$。比 trie 的 $O(m)$ 還快。

但 StartsWith 呢？

```go
words := map[string]bool{"apple": true, "apply": true, "banana": true}

// search("apple")：一次 hash 定位到 bucket
words["apple"]   // true

// startsWith("app")：直覺上想照樣查
words["app"]     // false！
```

`words["app"]` 是 false，因為 map 裡存的 key 是 `"apple"` 跟 `"apply"`，沒有一個 key 恰好等於 `"app"`。hash 查找的機制是把整個字串丟進 hash function，算出一個 bucket 位置，去那一格看有沒有。而 `hash("app")` 跟 `hash("apple")` 算出來是兩個不相干的數字，落在不同 bucket。兩個字明明只差兩個字元，hash 完彼此毫無關聯，前綴關係在 hash 那一步就不見了。

所以 hash map 要做 StartsWith 只剩一條路：放棄 hash 查找，把所有 key 拿出來逐一比對。

```go
func startsWith(words map[string]bool, prefix string) bool {
    for w := range words {                // n 個 key 全部走一遍
        if strings.HasPrefix(w, prefix) { // 每個最多比對 m 個字元
            return true
        }
    }
    return false
}
```

存了十萬個字就要掃描十萬次。這就是 $O(n \cdot m)$ 的來源：不是做不到，是退化成暴力掃描，hash 給的 $O(1)$ 完全用不上。

Trie 查前綴是從 root 走 `a → p → p` 三步，節點存在就是有這個前綴。$O(m)$，m 是前綴的長度，跟字典裡存了多少字無關。

| | Hash Map | Trie |
|---|---|---|
| Search | $O(1)$ 平均 | $O(m)$ |
| StartsWith | $O(n \cdot m)$ | $O(m)$ |
| Insert | $O(1)$ 平均 | $O(m)$ |
| 空間 | 每個字獨立存 | 共同前綴只存一次 |

如果你不需要前綴搜尋，hash map 更簡單。

如果你需要前綴搜尋，trie 是唯一合理的選擇。

---

## 經典一：搜尋含萬用字元的字

[#211 Design Add and Search Words Data Structure](/problem/design-add-and-search-words-data-structure)

"b.d" 要能匹配 "bad"、"bed"、"bid"。`.` 代表任意字元。

Hash map 做不到。你不知道 `.` 是什麼。

Trie + DFS：碰到 `.` 就分岔，所有 children 都試一次。

```go
func (t *Trie) SearchWithDot(word string) bool {
    return dfs(t.root, word, 0)
}

func dfs(node *TrieNode, word string, i int) bool {
    if node == nil { return false }
    if i == len(word) { return node.isEnd }

    ch := word[i]
    if ch == '.' {
        // 萬用字元：26 條路都試
        for _, child := range node.children {
            if dfs(child, word, i+1) { return true }
        }
        return false
    }

    // 普通字元：走對應的路
    return dfs(node.children[ch-'a'], word, i+1)
}
```

普通字元走一條路，$O(m)$。碰到 `.` 分岔成最多 26 條。最壞 $O(26^m)$。但實際上大部分分岔很快就 nil 了。

---

## 經典二：在字元矩陣裡找字

[#212 Word Search II](/problem/word-search-ii)

給你一個 $m \times n$ 的字元矩陣和一堆單字。找出所有出現在矩陣裡的字。

```
board:
o a a n
e t a e
i h k r

words: ["oath", "pea", "eat", "rain"]
```

暴力：每個字都跑一次 DFS 搜尋。k 個字 × m*n 個起點 × 最長 L 步 = $O(k \cdot m \cdot n \cdot L)$。

Trie 解法：把所有字塞進 trie。然後從矩陣的每個格子開始 DFS，同時在 trie 上走。如果 trie 上走不下去，就剪枝。

```go
func findWords(board [][]byte, words []string) []string {
    trie := Constructor()
    for _, w := range words {
        trie.Insert(w)
    }

    result := []string{}
    rows, cols := len(board), len(board[0])

    var backtrack func(r, c int, node *TrieNode, path []byte)
    backtrack = func(r, c int, node *TrieNode, path []byte) {
        if r < 0 || r >= rows || c < 0 || c >= cols { return }
        ch := board[r][c]
        if ch == '#' { return }    // 已走過

        next := node.children[ch-'a']
        if next == nil { return }    // trie 上走不下去 → 剪枝

        path = append(path, ch)
        if next.isEnd {
            result = append(result, string(path))
            next.isEnd = false    // 避免重複找到同一個字
        }

        board[r][c] = '#'    // 標記走過
        backtrack(r+1, c, next, path)
        backtrack(r-1, c, next, path)
        backtrack(r, c+1, next, path)
        backtrack(r, c-1, next, path)
        board[r][c] = ch     // 恢復

        path = path[:len(path)-1]
    }

    for r := 0; r < rows; r++ {
        for c := 0; c < cols; c++ {
            backtrack(r, c, trie.root, []byte{})
        }
    }
    return result
}
```

沒有 trie，每個單字都要從矩陣的每個格子重搜一次。把單字都插進 trie 之後，矩陣上走一步、trie 上也跟著走一步，trie 上沒有對應的 child 就直接回頭，一次 DFS 把所有單字一起搜完。

---

## 經典三：最長共同前綴

[#14 Longest Common Prefix](/problem/longest-common-prefix)

`["flower", "flow", "flight"]` → "fl"

暴力：逐字元比較。$O(n \cdot m)$。

Trie：把所有字插入。然後從 root 往下走。只要某個節點恰好只有一個 child 且不是 isEnd，就繼續。碰到分叉或 isEnd 就停。

```
root → f → l → o → w → e → r*
                 ↗
               i → g → h → t*
                         ↗
                       w*
```

root → f：只有一個 child → 繼續
f → l：只有一個 child → 繼續
l → 有兩個 children（o 和 i）→ 停

最長共同前綴 = "fl"。

```go
func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 { return "" }

    trie := Constructor()
    for _, s := range strs {
        if s == "" { return "" }    // 有空字串，前綴就是空
        trie.Insert(s)
    }

    prefix := []byte{}
    node := trie.root
    for {
        count := 0
        var nextIdx int
        for i, child := range node.children {
            if child != nil {
                count++
                nextIdx = i
            }
        }
        if count != 1 || node.isEnd { break }    // 分叉或某個字在這裡結束
        prefix = append(prefix, byte(nextIdx+'a'))
        node = node.children[nextIdx]
    }
    return string(prefix)
}
```

這題用 trie 有點殺雞用牛刀，直接逐字元比較更簡單。不過插完之後從 root 往下走，共同前綴就自然排在同一條路上，這件事在別的題目上很好用。

---

## Trie 的空間問題

每個節點有 26 個指標。大部分是 nil。

如果存 10 萬個英文單字，平均長度 10。節點數大約 100 萬。每個節點 26 個指標 × 8 bytes = 208 bytes。總共約 200 MB。

太浪費了。

### 解法一：用 map 取代 array

```go
type TrieNode struct {
    children map[rune]*TrieNode
    isEnd    bool
}
```

只存有用的 children。省空間。但查詢變慢（hash 計算）。

### 解法二：壓縮 Trie（Radix Tree）

把只有一個 child 的連續節點壓成一個。

```
壓縮前：
root → a → p → p → l → e*

壓縮後：
root → "apple"*
```

如果 "apple" 和 "app" 都存在：

```
root → "app"* → "le"*
```

共同前綴 "app" 是一個節點。剩下的 "le" 是另一個。

Linux kernel 的路由表用的就是壓縮 trie（radix tree）。

### LeetCode 上要擔心嗎？

不用。LeetCode 的測資通常不大，26-array 的版本夠快夠好。面試也不會要你寫 radix tree。

---

## Trie vs 其他方案

| 需求 | 最佳選擇 | 為什麼 |
|------|----------|--------|
| 查完整的字 | Hash Map | O(1)，最簡單 |
| 查前綴 | Trie | O(m)，不受字典大小影響 |
| 查含萬用字元 | Trie + DFS | Hash map 做不到 |
| 字典排序 | Trie（DFS 走一遍） | 天然按字母排 |
| 自動補全 | Trie | 找到前綴節點，DFS 列出所有完整字 |
| 拼字檢查 | Trie | 查前綴 + 編輯距離 |

**只要跟前綴有關，就是 trie。**

---

## 高頻題清單

| 題目 | 核心 |
|------|------|
| [#208 Implement Trie](/problem/implement-trie-prefix-tree) | 標準 trie：insert, search, startsWith |
| [#211 Add and Search Words](/problem/design-add-and-search-words-data-structure) | Trie + DFS 處理萬用字元 `.` |
| [#212 Word Search II](/problem/word-search-ii) | Trie + backtracking 在矩陣上找字 |
| [#14 Longest Common Prefix](/problem/longest-common-prefix) | 走到分叉就停 |
| [#648 Replace Words](/problem/replace-words) | 查最短前綴 |
| [#677 Map Sum Pairs](/problem/map-sum-pairs) | Trie 每個節點存 sum |
| [#720 Longest Word in Dictionary](/problem/longest-word-in-dictionary) | 每一步都是完整的字 |
| [#1268 Search Suggestions System](/problem/search-suggestions-system) | 自動補全：前綴 + DFS 列出前三個 |

---

## 總結

Trie 做一件事：**把共同前綴壓在同一條路上。**

Hash map 查完整的字是 $O(1)$。但查前綴是 $O(n)$。

Trie 查前綴是 $O(m)$。m 是前綴長度。存了一百萬個字，查前綴還是只看前綴那幾個字元。

三個操作長得幾乎一樣：一個字元一個字元往下走。差別只在終點檢查什麼。

看到「前綴」「自動補全」「萬用字元搜尋」→ trie。其他的，hash map 就夠了。
