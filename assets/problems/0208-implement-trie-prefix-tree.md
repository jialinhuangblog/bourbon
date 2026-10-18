---
id: 208
title: "Implement Trie (Prefix Tree)"
slug: implement-trie-prefix-tree
difficulty: Medium
tags: [Hash Table, String, Design, Trie]
neetcode150_category: Tries
blind75_category: Tree
date_solved: 2026-03-29
languages: [golang, typescript]
time_complexity: O(k)
space_complexity: O(N*26)
"@local": [Trie]
insight: "insert/search/startsWith 都是走字元路徑；差別只在最後看不看 isEnd"
---

# Implement Trie (Prefix Tree)

A [**trie**](https://en.wikipedia.org/wiki/Trie) (pronounced as "try") or **prefix tree** is a tree data structure used to efficiently store and retrieve keys in a dataset of strings. There are various applications of this data structure, such as autocomplete and spellchecker.

Implement the Trie class:

*   `Trie()` Initializes the trie object.
*   `void insert(String word)` Inserts the string `word` into the trie.
*   `boolean search(String word)` Returns `true` if the string `word` is in the trie (i.e., was inserted before), and `false` otherwise.
*   `boolean startsWith(String prefix)` Returns `true` if there is a previously inserted string `word` that has the prefix `prefix`, and `false` otherwise.

**Example 1:**

**Input**
\["Trie", "insert", "search", "search", "startsWith", "insert", "search"\]
\[\[\], \["apple"\], \["apple"\], \["app"\], \["app"\], \["app"\], \["app"\]\]
**Output**
\[null, null, true, false, true, null, true\]

**Explanation**
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // return True
trie.search("app");     // return False
trie.startsWith("app"); // return True
trie.insert("app");
trie.search("app");     // return True

**Constraints:**

*   `1 <= word.length, prefix.length <= 2000`
*   `word` and `prefix` consist only of lowercase English letters.
*   At most `3 * 104` calls **in total** will be made to `insert`, `search`, and `startsWith`.

## Code Template

### Go
```go
type Trie struct {
    
}


func Constructor() Trie {
    
}


func (this *Trie) Insert(word string)  {
    
}


func (this *Trie) Search(word string) bool {
    
}


func (this *Trie) StartsWith(prefix string) bool {
    
}


/**
 * Your Trie object will be instantiated and called as such:
 * obj := Constructor();
 * obj.Insert(word);
 * param_2 := obj.Search(word);
 * param_3 := obj.StartsWith(prefix);
 */
```

### TypeScript
```typescript
class Trie {
    constructor() {
        
    }

    insert(word: string): void {
        
    }

    search(word: string): boolean {
        
    }

    startsWith(prefix: string): boolean {
        
    }
}

/**
 * Your Trie object will be instantiated and called as such:
 * var obj = new Trie()
 * obj.insert(word)
 * var param_2 = obj.search(word)
 * var param_3 = obj.startsWith(prefix)
 */
```
