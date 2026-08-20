n 台車在一條路上往同一個終點跑。每台車有起始位置和速度。快車追上慢車就會被拖慢，變成一個「車隊」。問最後有幾個車隊到達終點。

想像不能超車的賽車遊戲。跑跑卡丁車裡，後面的車速度比較快，會追上前車，然後超車。這題的規則一樣會追上，**但追上之後不能超車，只能黏在一起用慢車的速度走。所以車只會越黏越多，最後數有幾坨到終點就好。**

---

## 解法：排序 + monotonic stack

**Step 1：算每台車的到達時間**

先不管會不會追上，每台車自己開到終點要多久？`time = (target - position) / speed`。

```
target = 12
position = [10, 8, 0, 5, 3]
speed    = [ 2, 4, 1, 1, 3]

到達時間：
pos 10: (12-10)/2 = 1.0
pos  8: (12-8)/4  = 1.0
pos  5: (12-5)/1  = 7.0
pos  3: (12-3)/3  = 3.0
pos  0: (12-0)/1  = 12.0
```

**Step 2：按位置從大到小排序**

最靠近終點的排在最前面。因為只有後車追得到前車，順序排好才知道誰在誰前面：

```
pos 10, time 1.0
pos  8, time 1.0
pos  5, time 7.0
pos  3, time 3.0
pos  0, time 12.0
```

**Step 3：從前往後掃描，逐台比較到達時間**

後面的車到達時間比前面的短，代表追得上前車，會合併成同一個車隊。到達時間比前面的長，代表追不上，自己是一個新車隊。

```
pos 10, time 1.0  → 第一台，fleet = 1，目前最慢 = 1.0
pos  8, time 1.0  → 1.0 <= 1.0，追得上，合併，fleet 還是 1
pos  5, time 7.0  → 7.0 > 1.0，追不上，新車隊，fleet = 2，目前最慢 = 7.0
pos  3, time 3.0  → 3.0 <= 7.0，追得上，合併，fleet 還是 2
pos  0, time 12.0 → 12.0 > 7.0，追不上，新車隊，fleet = 3
```

答案是 3。

---

為什麼「後面的車到達時間更短就會被吞掉」？

因為後面的車要先經過前面那台慢車的位置。到達時間更短代表速度更快，一定會在某個點追上前車。追上之後就被限速，跟前車同時到達終點。所以只看前車的時間就好。

反過來，後面的車到達時間更長，代表速度更慢，永遠追不上前車。自成一隊。

一個容易腦補的陷阱：後面有一台超快的車，能不能穿過中間那坨慢車，直接追上第一坨？不行。不管多快，碰到中間那坨的瞬間就被黏住、強制降速。永遠沒機會碰到第一坨。所以每台車只需要跟前面最近的那坨比就好，不用擔心「跳過」的情況。

---

完整程式碼：

```typescript
function carFleet(target: number, position: number[], speed: number[]): number {
    const n = position.length;
    if (n === 0) return 0;

    // 建立 (position, time) 配對，按 position 從大到小排
    const cars = position
        .map((pos, i) => ({ pos, time: (target - pos) / speed[i] }))
        .sort((a, b) => b.pos - a.pos); // 離終點近的排前面

    let fleets = 1;
    let slowest = cars[0].time;

    for (let i = 1; i < n; i++) {
        if (cars[i].time > slowest) { // 追不上前面的車隊
            fleets++;
            slowest = cars[i].time;
        }
    }

    return fleets;
}
```

- Time: O(n log n)（排序）
- Space: O(n)（存排序後的配對）

<details>
<summary>Go 版本</summary>

```go
import "sort"

func carFleet(target int, position []int, speed []int) int {
    n := len(position)
    if n == 0 {
        return 0
    }

    // 把 (position, speed) 綁在一起，按 position 從大到小排
    type car struct {
        pos, spd int
    }
    cars := make([]car, n)
    for i := 0; i < n; i++ {
        cars[i] = car{position[i], speed[i]}
    }
    sort.Slice(cars, func(i, j int) bool {
        return cars[i].pos > cars[j].pos // 離終點近的排前面
    })

    fleets := 1
    slowest := float64(target-cars[0].pos) / float64(cars[0].spd)

    for i := 1; i < n; i++ {
        time := float64(target-cars[i].pos) / float64(cars[i].spd)
        if time > slowest { // 追不上前面的車隊，自成一隊
            fleets++
            slowest = time
        }
        // time <= slowest → 追得上，被吞掉，不用做事
    }

    return fleets
}
```

</details>

---

**為什麼歸在 Monotonic Stack？**

上面的解法其實沒用 stack。但可以用 stack 寫：

```typescript
function carFleet(target: number, position: number[], speed: number[]): number {
    const cars = position
        .map((pos, i) => ({ pos, spd: speed[i] }))
        .sort((a, b) => b.pos - a.pos);

    const stack: number[] = []; // 存每個車隊的到達時間
    for (const c of cars) {
        const time = (target - c.pos) / c.spd;
        // 車子由前往後處理，所以 stack 頂端那筆時間，屬於當前這台車正前方的車隊
        // 當前車的時間 <= 那筆時間，代表追得上，會併進那個車隊，不 push
        if (stack.length === 0 || time > stack[stack.length - 1]) {
            stack.push(time);
        }
    }
    return stack.length;
}
```

同一份資料照這段 code 跑一遍，看 stack 怎麼變：

```
stack = []

pos 10, time 1.0  → stack 空，push        → stack = [1.0]
pos  8, time 1.0  → 1.0 <= 1.0，不 push   → stack = [1.0]
pos  5, time 7.0  → 7.0 > 1.0，push       → stack = [1.0, 7.0]
pos  3, time 3.0  → 3.0 <= 7.0，不 push   → stack = [1.0, 7.0]
pos  0, time 12.0 → 12.0 > 7.0，push      → stack = [1.0, 7.0, 12.0]

回傳 stack.length = 3
```

<details>
<summary>Go 版本</summary>

```go
func carFleet(target int, position []int, speed []int) int {
    n := len(position)
    type car struct{ pos, spd int }
    cars := make([]car, n)
    for i := range cars {
        cars[i] = car{position[i], speed[i]}
    }
    sort.Slice(cars, func(i, j int) bool {
        return cars[i].pos > cars[j].pos
    })

    stack := []float64{} // 存每個車隊的到達時間
    for _, c := range cars {
        time := float64(target-c.pos) / float64(c.spd)
        // 車子由前往後處理，所以 stack 頂端那筆時間，屬於當前這台車正前方的車隊
        // 當前車的時間 <= 那筆時間，代表追得上，會併進那個車隊，不 push
        if len(stack) == 0 || time > stack[len(stack)-1] {
            stack = append(stack, time)
        }
    }
    return len(stack)
}
```

</details>

Stack 裡存的是「還活著的車隊的到達時間」，從底到頂遞增。新車追得上（time <= top）就不進 stack，追不上就 push。最後 stack 長度 = 車隊數。

這就是 monotonic stack 的形狀：stack 裡的值永遠遞增。但這題不需要 pop（因為後面的車不會讓前面的車隊消失），所以用一個變數追蹤 `slowest` 就夠了。

---

## 結論

按位置排序後，從最靠近終點的車開始掃。到達時間比前面長的自成一隊，比前面短的被吞掉。一個變數追蹤目前最慢的到達時間就能判斷。
