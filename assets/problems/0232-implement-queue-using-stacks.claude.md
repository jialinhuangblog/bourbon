只准用 stack，做出一個 queue。

stack 是後進先出（LIFO）：最後疊上去的最先拿走。queue 是先進先出（FIFO）：最先排隊的最先出去。順序剛好相反。用一個「拿東西的方向」相反的工具，做出另一個方向，怎麼辦？

```
queue 要的：push 1, push 2 → pop 得到 1（先進先出）
stack 給的：push 1, push 2 → pop 得到 2（後進先出）
```

把一疊 stack 倒進另一疊 stack，順序就翻了。`[1,2]`（2 在頂）一個個 pop 倒進第二疊，變成 `[2,1]`（1 在頂）。翻一次，LIFO 就變成 FIFO。

---

**解題引導**

**Step 1：一疊 stack 的頂是最新的，但 queue 要最舊的先出。怎麼拿到最舊的？**

*pour it into another stack, the order flips*

<span class="spoiler">倒進第二疊 stack，順序翻過來，最舊的就跑到頂了。Pour into a second stack to reverse order.</span>

**Step 2：每次 pop 都倒一次很貴。能不能少倒幾次？**

*only pour when the out-stack runs dry*

<span class="spoiler">第二疊（out）還有東西就直接拿，空了才從第一疊倒過來。懶得倒，攤下來每個操作 O(1)。Transfer lazily, only when out is empty.</span>

想完再往下看 code。

---

## 解法一：push 時翻面（costly push）

一個直覺解：只用一疊 stack，但每次 push 都維持「queue 順序」，也就是讓最舊的元素待在頂端。做法是 push 時先把整疊倒出來，放新元素，再倒回去。

這樣 pop 和 peek 直接拿頂端就好，但 push 變貴。

```typescript
class MyQueue {
    private s: number[] = [];   // 維持 queue 順序，front 在頂端
    push(x: number): void {
        const tmp: number[] = [];
        while (this.s.length) tmp.push(this.s.pop()!);   // 全倒出來
        this.s.push(x);                                  // 新的墊底
        while (tmp.length) this.s.push(tmp.pop()!);       // 再倒回去
    }
    pop(): number { return this.s.pop()!; }
    peek(): number { return this.s[this.s.length - 1]; }
    empty(): boolean { return this.s.length === 0; }
}
```

- push: $O(n)$
- pop / peek / empty: $O(1)$

<details>
<summary>Go 版本</summary>

```go
type MyQueue struct {
    s []int // 維持 queue 順序，front 在頂端
}

func Constructor() MyQueue {
    return MyQueue{}
}

func (this *MyQueue) Push(x int) {
    tmp := []int{}
    for len(this.s) > 0 {
        tmp = append(tmp, this.s[len(this.s)-1])
        this.s = this.s[:len(this.s)-1]
    }
    this.s = append(this.s, x)
    for len(tmp) > 0 {
        this.s = append(this.s, tmp[len(tmp)-1])
        tmp = tmp[:len(tmp)-1]
    }
}

func (this *MyQueue) Pop() int {
    v := this.s[len(this.s)-1]
    this.s = this.s[:len(this.s)-1]
    return v
}

func (this *MyQueue) Peek() int {
    return this.s[len(this.s)-1]
}

func (this *MyQueue) Empty() bool {
    return len(this.s) == 0
}
```

</details>

能用。但 push 密集時每次都翻整疊，follow-up 要的是「攤平下來每個操作 $O(1)$」，這個做不到。

---

## 解法二：懶得搬（amortized $O(1)$）

兩疊 stack 分工：`in` 專門收 push，`out` 專門給 pop 和 peek。

規則只有一條：**在 out 清空之前，in 都在繼續累積**。pop 跟 peek 一律從 out 拿東西；等到 out 空了，才把 in 累積的整疊倒過去（順序翻一次，最舊的翻到頂端），然後繼續從 out 拿。

為什麼 out 有東西就能直接拿？因為 out 裡的每一個，都比 in 裡的每一個老：out 是之前某次倒過去的，in 裡裝的是那之後才 push 的。所以 out 的頂端永遠是全場最老的元素，正是 queue 要交出去的那個。反過來想也成立：out 還有東西就把 in 倒進去，反而會把年輕的疊到老的上面，順序就錯了。

走一遍 push 1, push 2, pop, push 3, pop, pop。`in`、`out` 都寫成「底在左、頂在右」，狀態是每個操作做完之後的：

| 操作 | in | out | 有沒有 flip | 回傳 |
|---|---|---|---|---|
| 起始 | `[]` | `[]` | | |
| push 1 | `[1]` | `[]` | | |
| push 2 | `[1,2]` | `[]` | | |
| pop | `[]` | `[2]` | out 空，觸發：in 倒進 out（順序翻成 `[2,1]`），再彈出頂端 | 1 |
| push 3 | `[3]` | `[2]` | | |
| pop | `[3]` | `[]` | out 還有東西，不觸發，直接彈出頂端 | 2 |
| pop | `[]` | `[]` | out 空，觸發：in 倒進 out（只有 `[3]` 一個），再彈出頂端 | 3 |

彈出順序是 1、2、3，跟 push 的順序（1、2、3）一致，FIFO 對了。

第五列（push 3 之後）值得多看一眼：這時候 in=`[3]`、out=`[2]`，如果提前把 in 倒進 out 會變成 out=`[2,3]`，頂端變成 3。下次 pop 就會先拿到 3，可是 2 比 3 早進隊，順序錯了。「等 out 空了才倒」不是偷懶，是正確性的要求。

```typescript
class MyQueue {
    private in: number[] = [];
    private out: number[] = [];

    push(x: number): void { this.in.push(x); }

    pop(): number {
        this.flip();
        return this.out.pop()!;
    }
    peek(): number {
        this.flip();
        return this.out[this.out.length - 1];
    }
    empty(): boolean {
        return this.in.length === 0 && this.out.length === 0;
    }

    private flip(): void {
        if (!this.out.length) {                          // 只有 out 空了才倒
            while (this.in.length) this.out.push(this.in.pop()!);
        }
    }
}
```

- push / empty: $O(1)$
- pop / peek: amortized $O(1)$

<details>
<summary>Go 版本</summary>

```go
type MyQueue struct {
    in  []int // 收 push
    out []int // 給 pop / peek
}

func Constructor() MyQueue {
    return MyQueue{}
}

func (this *MyQueue) Push(x int) {
    this.in = append(this.in, x)
}

func (this *MyQueue) flip() {
    if len(this.out) == 0 { // 只有 out 空了才倒
        for len(this.in) > 0 {
            n := len(this.in)
            this.out = append(this.out, this.in[n-1])
            this.in = this.in[:n-1]
        }
    }
}

func (this *MyQueue) Pop() int {
    this.flip()
    n := len(this.out)
    v := this.out[n-1]
    this.out = this.out[:n-1]
    return v
}

func (this *MyQueue) Peek() int {
    this.flip()
    return this.out[len(this.out)-1]
}

func (this *MyQueue) Empty() bool {
    return len(this.in) == 0 && len(this.out) == 0
}
```

</details>

水槽有兩疊盤子。剛用完的髒盤子疊在一起（in），洗好排整齊的乾淨盤子疊在另一疊（out）。收一個乾淨盤子一律從 out 拿最上面那個，每次都差不多快，不管挑哪一次收都一樣快，這是真正的 O(1)。

out 收光了，才把 in 那疊整個拿去洗，洗完扣過來放進 out。這一次要洗好幾個盤子，比較慢，但不常發生：發生一次，就把之前累積的髒盤子一次洗完，接下來又是一長串快速的「收一個盤子」。

amortized O(1) 保證的正是這種長期平均，不保證單一次一定快，只保證一長串操作加起來、攤下來夠低。中間躲著的那次貴操作，代價由後面很多次便宜操作分攤掉。

換一個算成本的角度看程式碼：不要問「這次 pop 倒了幾個」，改問「一個元素一生被動了幾次」。答案是固定的：進 in 一次、倒進 out 一次、彈出一次，三個動作各發生一次，之後它就離場了。所以不管 push 跟 pop 怎麼交錯，n 個元素的總工作量就是 3n 上下，攤到每次操作是 $O(1)$。最壞的單次 pop 確實可能倒一大疊（out 剛好空、in 累積了一堆），但那一疊裡每個元素都是第一次也是最後一次被倒，貴就貴這一次，後面的 pop 全是直接彈出。

---

## 解法比較表

| 解法 | push | pop / peek | 備註 |
|---|---|---|---|
| push 時翻面 | $O(n)$ | $O(1)$ | 好想，push 密集就慢 |
| 兩 stack 懶轉移 | $O(1)$ | amortized $O(1)$ | follow-up 要的解 |

---

## 結論

stack 是反的，倒進另一疊就正過來。難的不在「倒」，在「什麼時候倒」：out 空了才倒，每個元素一生只搬一次，攤下來就是 amortized $O(1)$。
