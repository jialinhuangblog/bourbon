這是 design 題。先建 instance，再對它操作：

```typescript
const trie = new Trie();
trie.insert("apple");
trie.search("apple");   // true
trie.search("app");     // false，"app" 沒被 insert 過
trie.startsWith("app"); // true，"apple" 有這個前綴
trie.insert("app");
trie.search("app");     // true，現在有了
```

<details>
<summary>Go 版本</summary>

```go
trie := Constructor()
trie.Insert("apple")
trie.Search("apple")   // true
trie.Search("app")     // false，"app" 沒被 insert 過
trie.StartsWith("app") // true，"apple" 有這個前綴
trie.Insert("app")
trie.Search("app")     // true，現在有了
```

</details>


`search` 要完全匹配，`startsWith` 只要是前綴就行。

---

## 解法：陣列 children

**資料結構**

每個 node 存兩件事：
1. `children` — 26 個英文字母各一個指標（或用 map）
2. `isEnd` — 這個位置是不是某個完整單字的結尾

insert("apple") 再 insert("app") 之後，結構長這樣：

```
root
 └─[a] node
        └─[p] node
               └─[p] node  isEnd=true  ← "app" 結尾
                      └─[l] node
                             └─[e] node  isEnd=true  ← "apple" 結尾
```

每個 `[字母]` 是 children 陣列的 index，node 裡還有 26 個空槽沒畫出來。`isEnd=true` 代表有個完整單字在這裡結束。

**逐步追蹤**

**insert("apple")**
```
root → a → p → p → l → e
                         isEnd=true
```

**search("apple")** → 走到 e，isEnd=true → `true`

**search("app")** → 走到第三個 p，isEnd=false → `false`

**startsWith("app")** → 走到第三個 p，node 存在 → `true`（不管 isEnd）

**insert("app")** → 走到第三個 p，把 isEnd 設為 true

**search("app")** → 走到第三個 p，isEnd=true → `true`

```typescript
class TrieNode {
    children: (TrieNode | null)[] = new Array(26).fill(null);
    isEnd = false;
}

class Trie {
    private root = new TrieNode();

    insert(word: string): void {
        let node = this.root;
        for (const ch of word) {
            const i = ch.charCodeAt(0) - 97; // 'a'=97
            if (!node.children[i]) node.children[i] = new TrieNode();
            node = node.children[i]!;
        }
        node.isEnd = true;
    }

    search(word: string): boolean {
        let node = this.root;
        for (const ch of word) {
            const i = ch.charCodeAt(0) - 97;
            if (!node.children[i]) return false;
            node = node.children[i]!;
        }
        return node.isEnd;
    }

    startsWith(prefix: string): boolean {
        let node = this.root;
        for (const ch of prefix) {
            const i = ch.charCodeAt(0) - 97;
            if (!node.children[i]) return false;
            node = node.children[i]!;
        }
        return true;
    }
}
```

<details>
<summary>Go 版本</summary>

```go
type TrieNode struct {
    children [26]*TrieNode // a-z 各一個槽
    isEnd    bool
}

type Trie struct {
    root *TrieNode
}

func Constructor() Trie {
    return Trie{root: &TrieNode{}}
}

func (t *Trie) Insert(word string) {
    node := t.root
    for _, ch := range word {
        i := ch - 'a'          // 'a'=0, 'b'=1, ..., 'z'=25
        if node.children[i] == nil {
            node.children[i] = &TrieNode{}
        }
        node = node.children[i] // 往下走
    }
    node.isEnd = true           // 走到底，標記結尾
}

func (t *Trie) Search(word string) bool {
    node := t.root
    for _, ch := range word {
        i := ch - 'a'
        if node.children[i] == nil {
            return false // 路斷了
        }
        node = node.children[i]
    }
    return node.isEnd // 走到底，但要是完整單字
}

func (t *Trie) StartsWith(prefix string) bool {
    node := t.root
    for _, ch := range prefix {
        i := ch - 'a'
        if node.children[i] == nil {
            return false
        }
        node = node.children[i]
    }
    return true // 只要路沒斷就行，不管 isEnd
}
```

</details>


---

**核心**

```
insert：走到底，isEnd = true
search：走到底，return isEnd
startsWith：走到底，return true（不看 isEnd）
```

`search` 和 `startsWith` 只差最後一行。

---

**複雜度**

- `insert` / `search` / `startsWith`：O(k)，k = 字串長度
- Space：O(N × 26)，N = 所有字元總數
