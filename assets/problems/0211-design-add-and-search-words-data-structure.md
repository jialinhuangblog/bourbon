---
id: 211
title: "Design Add and Search Words Data Structure"
slug: design-add-and-search-words-data-structure
difficulty: Medium
tags: [String, Depth-First Search, Design, Trie]
neetcode150_category: Tries
blind75_category: Tree
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Design Add and Search Words Data Structure

Design a data structure that supports adding new words and finding if a string matches any previously added string.

Implement the `WordDictionary` class:

*   `WordDictionary()` Initializes the object.
*   `void addWord(word)` Adds `word` to the data structure, it can be matched later.
*   `bool search(word)` Returns `true` if there is any string in the data structure that matches `word` or `false` otherwise. `word` may contain dots `'.'` where dots can be matched with any letter.

**Example:**

**Input**
\["WordDictionary","addWord","addWord","addWord","search","search","search","search"\]
\[\[\],\["bad"\],\["dad"\],\["mad"\],\["pad"\],\["bad"\],\[".ad"\],\["b.."\]\]
**Output**
\[null,null,null,null,false,true,true,true\]

**Explanation**
WordDictionary wordDictionary = new WordDictionary();
wordDictionary.addWord("bad");
wordDictionary.addWord("dad");
wordDictionary.addWord("mad");
wordDictionary.search("pad"); // return False
wordDictionary.search("bad"); // return True
wordDictionary.search(".ad"); // return True
wordDictionary.search("b.."); // return True

**Constraints:**

*   `1 <= word.length <= 25`
*   `word` in `addWord` consists of lowercase English letters.
*   `word` in `search` consist of `'.'` or lowercase English letters.
*   There will be at most `2` dots in `word` for `search` queries.
*   At most `104` calls will be made to `addWord` and `search`.

## Code Template

### Go
```go
type WordDictionary struct {
    
}


func Constructor() WordDictionary {
    
}


func (this *WordDictionary) AddWord(word string)  {
    
}


func (this *WordDictionary) Search(word string) bool {
    
}


/**
 * Your WordDictionary object will be instantiated and called as such:
 * obj := Constructor();
 * obj.AddWord(word);
 * param_2 := obj.Search(word);
 */
```

### TypeScript
```typescript
class WordDictionary {
    constructor() {
        
    }

    addWord(word: string): void {
        
    }

    search(word: string): boolean {
        
    }
}

/**
 * Your WordDictionary object will be instantiated and called as such:
 * var obj = new WordDictionary()
 * obj.addWord(word)
 * var param_2 = obj.search(word)
 */
```
