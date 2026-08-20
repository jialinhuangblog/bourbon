題目說「設計一個 MinStack」。寫慣 JS 的人第一個反應通常是：Stack 是什麼？是像 `Map`、`Set` 那種內建型別嗎？要 `new Stack()` 嗎？

不是。JS 沒有 Stack 這個型別。stack 是一個概念，不是一個 class。你拿普通 array，只准用兩個操作：`push` 加在尾端、`pop` 拿走尾端。只要你紀律上只碰尾端、不從中間插讀，這個 array 就「是」一個 stack。

```
直立圖（大家想像 stack 的樣子）      array 實際長相
   -3   ← top                        [-2, 0, -3]
    0                                  index 2 ↑ 最後一個 = top
   -2                                  index 1
                                       index 0 ← 最底
```

「最上面」不是 array 的開頭，是**尾端**。用尾端當 top 是因為 `push`/`pop` 在尾端都是 $O(1)$；用開頭當 top 的話，`unshift`/`shift` 每次要搬整個 array，變 $O(n)$。

想通這件事，MinStack 就從「陌生資料結構」變成「開幾個 array，講好只從尾端進出」。剩下的難點只有一個：題目要求 push、pop、top、getMin **全部 $O(1)$**。前三個 array 直接給，卡的是 getMin。

```
push -2   push 0    push -3
 -2        0         -3   ← top
           -2        0
                    -2

getMin → -3（整疊最小）
pop（拿走 -3）
top → 0
getMin → -2（-3 走了，剩下最小是 -2）
```

看最後一步：pop 掉 -3 之後，getMin 要能自己「退回」-2。怎麼讓最小值跟著 pop 一起倒退，就是這題的全部。

---

**解題引導**

**Step 1：最直覺怎麼實作 getMin？**

*scan the whole array for the smallest*

<span class="spoiler">每次 getMin 掃整個 array 找最小。但那是 $O(n)$，題目要 $O(1)$。Scan for min each time, but that's $O(n)$.</span>

**Step 2：pop 掉最小值後，最小值要退回上一個。這個「退回」的行為，像什麼？**

*push then pop reverts to the previous state — that's a stack*

<span class="spoiler">就是 stack 本身的行為。push 進去、pop 出來剛好回到上一個狀態。所以「最小值的歷史」也可以用另一個 array 存。The min history is itself stack-shaped.</span>

**Step 3：怎麼讓「此刻的最小值」跟著每次 push / pop 同步？**

*keep a second array in lockstep with the first*

<span class="spoiler">再開一個 array，跟主 array 同步 push、同步 pop，裡面存「到這一刻為止的最小值」。A second array kept in lockstep.</span>

想完再往下看 code。

---

## 解法一：getMin 掃全部

先寫最直覺的：一個 array，push/pop/top 就是它的尾端操作，getMin 每次用 `Math.min` 掃全部。

```typescript
class MinStack {
    private stack: number[] = [];
    push(val: number): void { this.stack.push(val); }
    pop(): void { this.stack.pop(); }
    top(): number { return this.stack[this.stack.length - 1]; }
    getMin(): number { return Math.min(...this.stack); }   // 每次掃全部
}
```

- push/pop/top: $O(1)$
- getMin: $O(n)$

題目白紙黑字要 getMin 也是 $O(1)$，這個解不合格。問題在：每次 getMin 都重算，但這個 array 大部分沒變，重算是浪費。

---

## 解法二：每格存 minSoFar

Step 2 已經講過：pop 之後最小值要「退回上一個」，這行為跟 array 尾端進出一模一樣。所以別讓每格只存一個數，改成存一對：`[值, 到這格為止的最小值]`。

push 一個值時，這格的最小值 = `min(新值, 前一格記的最小值)`。getMin 直接讀尾端那格記的最小值，$O(1)$。pop 掉一格，min 自動跟著回到前一格記的值。

用 push -2, 0, -3 走：

```
push -2  min(-2, 無) = -2   stack=[(-2,-2)]
push 0   min(0, -2)  = -2   stack=[(-2,-2),(0,-2)]
push -3  min(-3, -2) = -3   stack=[(-2,-2),(0,-2),(-3,-3)]

getMin → 尾端記的 -3 ✓
pop     → 移除 (-3,-3)，stack=[(-2,-2),(0,-2)]
top     → 尾端的值 0 ✓
getMin  → 尾端記的 -2 ✓   ← 最小值自己退回來了
```

```typescript
class MinStack {
    private stack: [number, number][] = [];   // 每格是 [值, 到這格為止的最小值]

    push(val: number): void {
        const prevMin = this.stack.length ? this.stack[this.stack.length - 1][1] : val;
        this.stack.push([val, Math.min(val, prevMin)]);
    }
    pop(): void { this.stack.pop(); }
    top(): number { return this.stack[this.stack.length - 1][0]; }
    getMin(): number { return this.stack[this.stack.length - 1][1]; }
}
```

- 四個操作全部 $O(1)$
- Space: $O(n)$，每格多存一個 min

<details>
<summary>Go 版本</summary>

```go
// Go 也沒有 stack 型別，一樣用 slice，只碰尾端就是 stack
type MinStack struct {
    stack [][2]int // 每格是 [值, 到這格為止的最小值]
}

func Constructor() MinStack {
    return MinStack{}
}

func (this *MinStack) Push(val int) {
    min := val
    if len(this.stack) > 0 && this.stack[len(this.stack)-1][1] < min {
        min = this.stack[len(this.stack)-1][1] // 沿用前一格的最小值
    }
    this.stack = append(this.stack, [2]int{val, min})
}

func (this *MinStack) Pop() {
    this.stack = this.stack[:len(this.stack)-1]
}

func (this *MinStack) Top() int {
    return this.stack[len(this.stack)-1][0]
}

func (this *MinStack) GetMin() int {
    return this.stack[len(this.stack)-1][1] // 直接讀尾端記的最小值
}
```

</details>

每格都存一份 min，就算整個 array 一路遞增（每格的 min 都跟前一格一樣）也照存。這是它跟下面解法三的差別。

---

## 解法三：兩個 array 同步

換個存法：主 array 存全部值，另開一個 mins array 只記「最小值的變化」。省下解法二那些重複的 min。

規則：push 時，只有新值**小於等於**目前最小值，才推進 mins。pop 時，只有拿走的值**等於** mins 的尾端，才把 mins 也 pop。兩個 array 各自從尾端進出，維持同步。

為什麼是「小於等於」不是「小於」？想像 push 兩個一樣的最小值 -2、-2。如果只在「小於」時推，mins 只會有一個 -2。之後 pop 掉一個 -2，mins 跟著 pop 就空了，但主 array 還有一個 -2 是當前最小。用「小於等於」推、「等於」拿，重複的最小值各留一份，配對才對得上。

```typescript
class MinStack {
    private stack: number[] = [];  // 存全部值
    private mins: number[] = [];   // 只記最小值的變化

    push(val: number): void {
        this.stack.push(val);
        if (!this.mins.length || val <= this.mins[this.mins.length - 1]) {
            this.mins.push(val);   // 小於等於才記
        }
    }
    pop(): void {
        const v = this.stack.pop();
        if (v === this.mins[this.mins.length - 1]) this.mins.pop();  // 等於才退
    }
    top(): number { return this.stack[this.stack.length - 1]; }
    getMin(): number { return this.mins[this.mins.length - 1]; }
}
```

- 四個操作全部 $O(1)$
- Space: $O(n)$。跟解法二的差別在常數：解法二固定存 2n 個數，這裡主 array 存 n 個，mins 的大小看輸入。一路遞增的輸入 mins 只有 1 個（min 從沒變過），一路遞減才裝滿 n 個

<details>
<summary>Go 版本</summary>

```go
type MinStack struct {
    stack []int // 存全部值
    mins  []int // 只記最小值的變化
}

func Constructor() MinStack {
    return MinStack{}
}

func (this *MinStack) Push(val int) {
    this.stack = append(this.stack, val)
    if len(this.mins) == 0 || val <= this.mins[len(this.mins)-1] {
        this.mins = append(this.mins, val) // 小於等於才記
    }
}

func (this *MinStack) Pop() {
    v := this.stack[len(this.stack)-1]
    this.stack = this.stack[:len(this.stack)-1]
    if v == this.mins[len(this.mins)-1] {
        this.mins = this.mins[:len(this.mins)-1] // 等於才退
    }
}

func (this *MinStack) Top() int {
    return this.stack[len(this.stack)-1]
}

func (this *MinStack) GetMin() int {
    return this.mins[len(this.mins)-1]
}
```

</details>

---

## 解法比較表

| 解法 | getMin | push/pop/top | Space | 備註 |
|---|---|---|---|---|
| getMin 掃全部 | $O(n)$ | $O(1)$ | $O(n)$ | 不合題目要求 |
| 每格存 minSoFar | $O(1)$ | $O(1)$ | $O(n)$ | 好寫好懂，min 跟值綁在同一格 |
| 兩個 array 同步 | $O(1)$ | $O(1)$ | $O(n)$ | 常數較省（mins 大小看輸入），重複最小值要用 <= |

---

## 結論

MinStack 這名字聽起來像特殊結構，拆開就是「開幾個 array，只從尾端進出」。getMin 要 $O(1)$ 就不能重算，得把最小值跟著 array 一起記，因為「pop 後最小值退回上一個」的行為本身就是尾端進出。每格存 min 最直覺；兩個 array 同步省一點空間，但要小心重複最小值用 <= 配 == 才對得上。
