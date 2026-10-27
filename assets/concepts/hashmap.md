---
title: "Hash Map"
category: Data Structures
slug: hashmap
subtitle: key 算一下就知道放哪，O(1)
date: 2026-02-28T11:51:31
---

# Hash Map

你有一本電話簿。一百萬筆。要找 "王小明" 的號碼。

翻第一頁。不是。第二頁。不是。一頁一頁翻。一百萬頁。$O(n)$。

如果電話簿按姓氏排好了？二分搜尋。$O(\log n)$。快很多。但還是要翻好幾次。

如果你有一個魔法函式，給它 "王小明"，它直接告訴你「第 48,721 頁」？一次就到。$O(1)$。

這個魔法函式叫 hash function。裝了它的容器叫 hash map。

Hash map、hash table、dictionary、associative array 都是同一個東西的不同名字。Java 有個古老的 `Hashtable` 類別，現在沒人用，都用 `HashMap`。

---

## 算 index、存進陣列、處理碰撞

1. **Hash function**：把 key 變成一個數字（index）
2. **陣列**：用這個 index 直接存取
3. **碰撞處理**：兩個 key 算出同一個 index 怎麼辦

就這三步。所有 hash map 的實作都在優化這三步。

---

## Hash Function：把任何東西變成數字

```
"apple" → hash("apple") → 3
"banana" → hash("banana") → 7
"cherry" → hash("cherry") → 3   ← 撞了！
```

好的 hash function 兩個要求：
1. **快**。$O(1)$。
2. **散**。不同的 key 盡量對應到不同的 index。

完全均勻的散佈不存在。key 的數量遠大於陣列長度。一定會撞。問題是怎麼處理。

---

## 碰撞處理：兩種主流

### Chaining（拉鏈法）

每個 bucket 放一條 linked list。撞了就掛在後面。

```
bucket[3] → ["apple", 1] → ["cherry", 5]
bucket[7] → ["banana", 2]
```

查 "cherry"：算 hash 得 3，走到 bucket[3]，沿著 list 找到 "cherry"。

平均 $O(1)$。最糟 $O(n)$：所有 key 都撞到同一個 bucket，list 長度 = n。

Java 的 HashMap 用 chaining。當 list 超過 8 個，自動轉成紅黑樹，最糟從 $O(n)$ 降到 $O(\log n)$。

### Open Addressing（開放定址）

不用 linked list。撞了就找下一個空位。

```
hash("apple") = 3 → bucket[3] 空的，放進去
hash("cherry") = 3 → bucket[3] 有人了，看 bucket[4]，空的，放這
```

查 "cherry"：算 hash 得 3，bucket[3] 是 "apple" 不是 "cherry"，往後看 bucket[4]，找到了。

Python 的 dict、Go 的 map 都用 open addressing 的變體。

### 哪個好？

| | Chaining | Open Addressing |
|---|---|---|
| 實作 | 簡單 | 複雜（要處理刪除） |
| Cache | 差（走 linked list 跳來跳去） | 好（連續記憶體） |
| Load factor 高 | 還行（list 變長） | 急遽變慢（找空位越來越久） |
| 刪除 | 容易 | 要標 tombstone |

現代語言偏好 open addressing。因為 cache friendliness 在實務上比理論複雜度重要。

---

## Load Factor 和 Resize

Load factor = 元素數量 / bucket 數量。

裝太滿，碰撞率暴增，$O(1)$ 退化成 $O(n)$。

所以 hash map 會自動 resize。通常 load factor > 0.75 就擴容，bucket 數量翻倍，所有元素重新 hash。

resize 是 $O(n)$。但 amortized（分攤）之後還是 $O(1)$：n 次 insert 只觸發 $\log n$ 次 resize，總搬運量攤到每次操作上不到一個元素。

---

## 各語言怎麼用

### JavaScript

```js
const map = new Map()
map.set("apple", 1)
map.get("apple")      // 1
map.has("apple")       // true
map.delete("apple")
map.size               // 0
```

不要用 `{}`（plain object）當 hash map。它的 key 只能是 string/symbol，而且有 prototype chain 污染的問題。`Map` 的 key 可以是任何型別。

### Python

```python
d = {}
d["apple"] = 1
d["apple"]             # 1
"apple" in d           # True
del d["apple"]
len(d)                 # 0

# Counter 是 hash map 的特化版
from collections import Counter
counts = Counter([1, 2, 2, 3, 3, 3])
# {3: 3, 2: 2, 1: 1}
```

Python 的 dict 從 3.7 開始保證 insertion order。

### Go

```go
m := map[string]int{}
m["apple"] = 1
val, ok := m["apple"]  // val=1, ok=true
delete(m, "apple")
len(m)                  // 0
```

Go 的 map 不是 thread-safe。concurrent 要用 `sync.Map` 或加鎖。

### Java

```java
Map<String, Integer> map = new HashMap<>();
map.put("apple", 1);
map.get("apple");      // 1
map.containsKey("apple"); // true
map.remove("apple");
map.size();            // 0
```

`HashMap` 不保證順序。要順序用 `LinkedHashMap`。要排序用 `TreeMap`（底層紅黑樹，$O(\log n)$）。

---

## 時間複雜度

| 操作 | 平均 | 最糟 |
|------|------|------|
| get | $O(1)$ | $O(n)$ |
| put | $O(1)$ | $O(n)$ |
| delete | $O(1)$ | $O(n)$ |
| has/contains | $O(1)$ | $O(n)$ |

最糟 $O(n)$ 在正常使用下幾乎不會發生。hash function 的設計和自動 resize 保證了碰撞率低。

---

## LeetCode 的三種用法

### 用法一：數東西

最常見。數每個元素出現幾次。

```go
func topKFrequent(nums []int, k int) []int {
    count := map[int]int{}
    for _, n := range nums {
        count[n]++
    }
    // ... 找 top k
}
```

相關題：
- [#1 Two Sum](/problem/two-sum) — 存 value → index，查 complement 在不在
- [#49 Group Anagrams](/problem/group-anagrams) — sorted string 當 key，分組
- [#347 Top K Frequent Elements](/problem/top-k-frequent-elements) — 先 count，再找 top k
- [#242 Valid Anagram](/problem/valid-anagram) — 兩個 counter 比一比

### 用法二：記住看過的東西

走過的路不走第二次。見過的值不找第二次。

```go
func hasCycle(head *ListNode) bool {
    seen := map[*ListNode]bool{}
    for head != nil {
        if seen[head] { return true }
        seen[head] = true
        head = head.Next
    }
    return false
}
```

相關題：
- [#141 Linked List Cycle](/problem/linked-list-cycle) — 見過就有環
- [#128 Longest Consecutive Sequence](/problem/longest-consecutive-sequence) — set 查 n-1 存不存在
- [#217 Contains Duplicate](/problem/contains-duplicate) — 見過就 return true

### 用法三：建映射關係

A 對應到 B。查得快。

```go
func twoSum(nums []int, target int) []int {
    seen := map[int]int{}           // value → index
    for i, n := range nums {
        if j, ok := seen[target-n]; ok {
            return []int{j, i}
        }
        seen[n] = i
    }
    return nil
}
```

Two Sum 是 hash map 的經典用法。不用兩層 for 迴圈（$O(n^2)$），用 hash map 記住已經看過的值和它的 index，每次查 complement。$O(n)$。

相關題：
- [#1 Two Sum](/problem/two-sum) — hash map 入門必做
- [#146 LRU Cache](/problem/lru-cache) — hash map + doubly linked list
- [#380 Insert Delete GetRandom O(1)](/problem/insert-delete-getrandom-o1) — hash map + array

---

## Hash Set

只存 key，不存 value。用來判斷「有沒有」。

```js
const set = new Set([1, 2, 3, 2, 1])
// Set {1, 2, 3}
set.has(2)    // true
set.add(4)
set.delete(1)
```

Hash set 就是 value = true 的 hash map。底層一樣。

---

## Map 家族

Map 是介面，定義 key-value 的操作：get、put、delete、contains。Hash map 是最常用的實作，但不是唯一的。

| 實作 | 底層結構 | 查找 | 順序 | 用在哪 |
|---|---|---|---|---|
| HashMap | hash table | $O(1)$ | 無 | 預設選擇，最快 |
| LinkedHashMap | hash table + doubly linked list | $O(1)$ | 插入順序 | LRU Cache |
| TreeMap | 紅黑樹 | $O(\log n)$ | key 排序 | 需要按 key 範圍查詢 |
| ConcurrentHashMap | 分段 hash table | $O(1)$ | 無 | 多執行緒 |

選擇邏輯很簡單：

- 不需要順序 → HashMap
- 需要插入順序 → LinkedHashMap
- 需要排序 → TreeMap
- 需要 thread-safe → ConcurrentHashMap

LeetCode 幾乎只用 HashMap。但 [#146 LRU Cache](/problem/lru-cache) 的本質就是自己做一個 LinkedHashMap。

---

## 面試常問

**Q：hash map 的時間複雜度是 $O(1)$ 還是 $O(n)$？**

平均 $O(1)$，最糟 $O(n)$。但正常使用下幾乎不會碰到 $O(n)$。面試回答「amortized $O(1)$」最精確。

**Q：什麼時候不該用 hash map？**

需要排序的時候。Hash map 不保證順序。要排序用 tree map（$O(\log n)$ per operation）。

空間敏感的時候。Hash map 的 overhead 比 array 大很多（hash function、bucket array、resize）。如果 key 是 0 到 100 的整數，直接用 array 更快更省。

**Q：hash map 和 array 的差別？**

Array 用連續 index（0, 1, 2, ...）存取。Hash map 用任意 key 存取。

Array 是 $O(1)$ random access，沒有 overhead。Hash map 是 amortized $O(1)$，有 hash function 和碰撞處理的 overhead。

如果你的 key 是連續整數，用 array。不是的話，用 hash map。

---

## 總結

Hash map 做一件事：**把 $O(n)$ 的查找變成 $O(1)$**。

每次你在寫兩層 for 迴圈，停下來想：能不能用 hash map 把內層迴圈的查找從 $O(n)$ 變成 $O(1)$？

Two Sum 就是這樣從 $O(n^2)$ 變成 $O(n)$ 的。

代價是空間。$O(n)$ 額外空間換 $O(1)$ 查找時間。這個 tradeoff 在 LeetCode 上幾乎永遠值得。

想學怎麼走遍一個圖的所有節點？看 [DFS](/concept/dfs)。Hash map 在 DFS 裡的角色是「記住走過的」。
