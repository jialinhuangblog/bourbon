---
title: "Stack"
category: Data Structures
slug: stack
subtitle: 後進先出，像疊盤子
date: 2026-02-28T11:51:31
---

# Stack

你疊了一疊盤子。要拿盤子的時候，你拿哪一個？

最上面那個。不可能從中間抽。

放盤子也是放最上面。

最後放的，最先拿。**Last In, First Out。LIFO。**

這就是 stack。

---

## 只有兩個操作

| 操作 | 做什麼 | 時間 |
|------|--------|------|
| push | 放到最上面 | $O(1)$ |
| pop | 拿走最上面的 | $O(1)$ |
| peek/top | 看最上面是什麼（不拿走） | $O(1)$ |

沒有「查第 3 個」「從中間刪除」。Stack 只讓你碰最上面那個。

這個限制不是缺點。這個限制就是它的用途。

---

## 各語言怎麼用

### JavaScript

```js
const stack = []
stack.push(1)        // [1]
stack.push(2)        // [1, 2]
stack.push(3)        // [1, 2, 3]
stack.pop()          // 3，stack = [1, 2]
stack[stack.length - 1]  // 2（peek）
```

JavaScript 沒有 Stack class。用 array 就好。`push` 和 `pop` 就是 stack 操作。

### Python

```python
stack = []
stack.append(1)      # [1]
stack.append(2)      # [1, 2]
stack.pop()          # 2, stack = [1]
stack[-1]            # 1 (peek)
```

Python 也用 list。`append` = push，`pop` = pop。

### Go

```go
stack := []int{}
stack = append(stack, 1)                    // push
stack = append(stack, 2)
top := stack[len(stack)-1]                  // peek
stack = stack[:len(stack)-1]                // pop
```

Go 用 slice。沒有內建 stack，手動操作。

### Java

```java
Deque<Integer> stack = new ArrayDeque<>();   // 不要用 Stack class
stack.push(1);
stack.push(2);
stack.peek();        // 2
stack.pop();         // 2
```

Java 有 `Stack` class 但別用。它是 synchronized 的，效能差。用 `ArrayDeque`。

---

## 你每天都在用 Stack

### Call Stack

每次你呼叫一個函式，它被 push 到 call stack。函式結束，pop 出去。

```js
function a() { b() }
function b() { c() }
function c() { throw new Error() }
a()
```

報錯時你看到的 stack trace：

```
Error
    at c
    at b
    at a
```

最後呼叫的函式（c）在最上面。最先呼叫的（a）在最下面。LIFO。

遞迴也是 stack。每次遞迴呼叫 = push。return = pop。遞迴太深 = stack overflow。

### 瀏覽器的上一頁

你從 A 頁面跳到 B，再跳到 C。按「上一頁」。回到 B。再按。回到 A。

```
瀏覽歷史 stack: [A] → [A, B] → [A, B, C]
按上一頁：pop C → 回到 B
再按：pop B → 回到 A
```

LIFO。最後造訪的頁面，最先被「回去」。

### Ctrl+Z

你打了 "hello"。又打了 "world"。Ctrl+Z 會先撤銷 "world"，再撤銷 "hello"。

每個操作 push 到 undo stack。Ctrl+Z = pop 最近的操作。LIFO。

---

## LeetCode 的三種用法

### 用法一：括號配對

[#20 Valid Parentheses](/problem/valid-parentheses)

`"({[]})"` 是合法的。`"({[}])"` 不合法。

左括號 push。碰到右括號，pop 一個出來看配不配。

```
s = "({[]})"

( → push    stack: [(]
{ → push    stack: [(, {]
[ → push    stack: [(, {, []
] → pop [   配！   stack: [(, {]
} → pop {   配！   stack: [(]
) → pop (   配！   stack: []

stack 空了，合法。
```

```go
func isValid(s string) bool {
    stack := []byte{}
    pairs := map[byte]byte{')': '(', ']': '[', '}': '{'}

    for i := 0; i < len(s); i++ {
        if s[i] == '(' || s[i] == '[' || s[i] == '{' {
            stack = append(stack, s[i])
        } else {
            if len(stack) == 0 || stack[len(stack)-1] != pairs[s[i]] {
                return false
            }
            stack = stack[:len(stack)-1]
        }
    }
    return len(stack) == 0
}
```

為什麼用 stack？因為括號的配對規則是 LIFO。最後開的括號要最先關。

### 用法二：Monotonic Stack

[#739 Daily Temperatures](/problem/daily-temperatures)

給每天的溫度 `[73, 74, 75, 71, 69, 72, 76, 73]`。每天要等幾天才會更暖？

暴力：每天往後掃，找到更高的。$O(n^2)$。

Monotonic stack：維護一個遞減的 stack。碰到更高的溫度，把 stack 裡比它低的全部 pop 出來。

```
temps = [73, 74, 75, 71, 69, 72, 76, 73]

i=0: 73, stack 空, push 0          stack: [0(73)]
i=1: 74 > 73, pop 0 → ans[0]=1-0=1  stack: []
     push 1                         stack: [1(74)]
i=2: 75 > 74, pop 1 → ans[1]=2-1=1  stack: []
     push 2                         stack: [2(75)]
i=3: 71 < 75, push 3               stack: [2(75), 3(71)]
i=4: 69 < 71, push 4               stack: [2(75), 3(71), 4(69)]
i=5: 72 > 69, pop 4 → ans[4]=5-4=1
     72 > 71, pop 3 → ans[3]=5-3=2
     72 < 75, push 5               stack: [2(75), 5(72)]
i=6: 76 > 72, pop 5 → ans[5]=6-5=1
     76 > 75, pop 2 → ans[2]=6-2=4  stack: []
     push 6                         stack: [6(76)]
i=7: 73 < 76, push 7               stack: [6(76), 7(73)]

ans = [1, 1, 4, 2, 1, 1, 0, 0]
```

每個元素最多 push 一次、pop 一次。$O(n)$。

```go
func dailyTemperatures(temperatures []int) []int {
    n := len(temperatures)
    ans := make([]int, n)
    stack := []int{}    // 存 index

    for i, temp := range temperatures {
        for len(stack) > 0 && temperatures[stack[len(stack)-1]] < temp {
            j := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            ans[j] = i - j
        }
        stack = append(stack, i)
    }
    return ans
}
```

**Monotonic stack 裡的元素一路遞增或一路遞減，新元素進來時，破壞這個順序的全部 pop。**

### 用法三：計算器 / 表達式求值

[#224 Basic Calculator](/problem/basic-calculator)

`"1 + (2 - (3 + 4))"` = ?

碰到 `(`，把當前結果和符號 push 到 stack。碰到 `)`，pop 出來合併。

Stack 記住「外層的狀態」。每進一層括號就 push，出來就 pop。跟 call stack 的邏輯一模一樣。

---

## Stack vs Queue

| | Stack | Queue |
|---|---|---|
| 原則 | LIFO（後進先出） | FIFO（先進先出） |
| 類比 | 疊盤子 | 排隊 |
| 操作 | push/pop（同一端） | enqueue/dequeue（兩端） |
| DFS/BFS | DFS 用 stack | BFS 用 queue |
| 括號配對 | 用 stack | 不適合 |
| 層序遍歷 | 不適合 | 用 queue |

**最近的用 stack，最早的用 queue。**

括號要配最近的那個 → stack。
BFS 要先處理最早發現的 → queue。

---

## 高頻題清單

| 題目 | 核心 |
|------|------|
| [#20 Valid Parentheses](/problem/valid-parentheses) | 左括號 push，右括號 pop 配對 |
| [#155 Min Stack](/problem/min-stack) | 兩個 stack，一個存值一個存當前最小 |
| [#739 Daily Temperatures](/problem/daily-temperatures) | Monotonic stack |
| [#84 Largest Rectangle in Histogram](/problem/largest-rectangle-in-histogram) | Monotonic stack 經典 |
| [#150 Evaluate Reverse Polish Notation](/problem/evaluate-reverse-polish-notation) | 碰到運算子 pop 兩個算 |
| [#394 Decode String](/problem/decode-string) | 碰到 [ push，碰到 ] pop |
| [#71 Simplify Path](/problem/simplify-path) | `..` = pop，其他 = push |
| [#224 Basic Calculator](/problem/basic-calculator) | 括號用 stack 存外層狀態 |

---

## 總結

Stack 做一件事：**記住最近的東西，用完就丟。**

括號要配最近開的。Undo 要撤最近做的。DFS 要回到最近的分叉。Call stack 要回到最近的呼叫者。

全部都是 LIFO。全部都是 stack。

想學另一端（先進先出）？看 [Queue](/concept/queue)。想學用 stack 做 DFS 的完整套路？看 [DFS](/concept/dfs)。
