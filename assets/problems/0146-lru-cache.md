---
id: 146
title: "LRU Cache"
slug: lru-cache
difficulty: Medium
tags: [Hash Table, Linked List, Design, Doubly-Linked List]
neetcode150_category: Linked List
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# LRU Cache

Design a data structure that follows the constraints of a **[Least Recently Used (LRU) cache](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU)**.

Implement the `LRUCache` class:

*   `LRUCache(int capacity)` Initialize the LRU cache with **positive** size `capacity`.
*   `int get(int key)` Return the value of the `key` if the key exists, otherwise return `-1`.
*   `void put(int key, int value)` Update the value of the `key` if the `key` exists. Otherwise, add the `key-value` pair to the cache. If the number of keys exceeds the `capacity` from this operation, **evict** the least recently used key.

The functions `get` and `put` must each run in `O(1)` average time complexity.

**Example 1:**

**Input**
\["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"\]
\[\[2\], \[1, 1\], \[2, 2\], \[1\], \[3, 3\], \[2\], \[4, 4\], \[1\], \[3\], \[4\]\]
**Output**
\[null, null, null, 1, null, -1, null, -1, 3, 4\]

**Explanation**
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // cache is {1=1}
lRUCache.put(2, 2); // cache is {1=1, 2=2}
lRUCache.get(1);    // return 1
lRUCache.put(3, 3); // LRU key was 2, evicts key 2, cache is {1=1, 3=3}
lRUCache.get(2);    // returns -1 (not found)
lRUCache.put(4, 4); // LRU key was 1, evicts key 1, cache is {4=4, 3=3}
lRUCache.get(1);    // return -1 (not found)
lRUCache.get(3);    // return 3
lRUCache.get(4);    // return 4

**Constraints:**

*   `1 <= capacity <= 3000`
*   `0 <= key <= 104`
*   `0 <= value <= 105`
*   At most `2 * 105` calls will be made to `get` and `put`.

## Code Template

### Go
```go
type LRUCache struct {
    
}


func Constructor(capacity int) LRUCache {
    
}


func (this *LRUCache) Get(key int) int {
    
}


func (this *LRUCache) Put(key int, value int)  {
    
}


/**
 * Your LRUCache object will be instantiated and called as such:
 * obj := Constructor(capacity);
 * param_1 := obj.Get(key);
 * obj.Put(key,value);
 */
```

### TypeScript
```typescript
class LRUCache {
    constructor(capacity: number) {
        
    }

    get(key: number): number {
        
    }

    put(key: number, value: number): void {
        
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * var obj = new LRUCache(capacity)
 * var param_1 = obj.get(key)
 * obj.put(key,value)
 */
```
