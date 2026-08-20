這是 design 題。先建一個 instance，再對它 `add` 和 `count`：

```typescript
const obj = new DetectSquares();
obj.add([3, 10]);
obj.add([11, 2]);
obj.add([3, 2]);
obj.count([11, 10]); // 1
obj.add([11, 2]);
obj.count([11, 10]); // 2
```

<details>
<summary>Go 版本</summary>

```go
obj := Constructor()
obj.Add([]int{3, 10})
obj.Add([]int{11, 2})
obj.Add([]int{3, 2})
obj.Count([]int{11, 10}) // 1
obj.Add([]int{11, 2})
obj.Count([]int{11, 10}) // 2
```

</details>

每次 `add` 把點存進資料結構，狀態一直保留。`count` 問的是：以這個點為一個角，資料結構裡有幾種組法能組成正方形。

---

給一個點 Q = (qx, qy)，問資料結構裡有多少組 3 個點能和 Q 組成軸對齊的正方形。

**軸對齊**：四條邊都平行或垂直於 x/y 軸，也就是所有角都是直角，不能斜的。

<details>
<summary>提示 1</summary>

固定和 Q 同一橫排的點 P1 = (x1, qy)。這條線的長度就是正方形的邊長。

</details>

<details>
<summary>提示 2</summary>

邊長 = |x1 - qx|。上下各能組一個正方形，各需要兩個角落點。把三個點的 count 相乘。

</details>

---

## 解法：HashMap 計數

**逐步追蹤**

用題目的 example 走一遍。

**add([3, 10])**
```
pts:  {[3,10]:1}
xByY: {10:{3}}
```

**add([11, 2])**
```
pts:  {[3,10]:1, [11,2]:1}
xByY: {10:{3}, 2:{11}}
```

**add([3, 2])**
```
pts:  {[3,10]:1, [11,2]:1, [3,2]:1}
xByY: {10:{3}, 2:{11,3}}
```

**count([11, 10]) → 1**
```
Q = (11, 10)
y=10 這排：x1=3

  side = |3-11| = 8

  上方 (y+8=18)：1 × 0 × 0 = 0   （pts[3,10]=1, pts[11,18]=0, pts[3,18]=0）
  下方 (y-8=2) ：1 × 1 × 1 = 1   （pts[3,10]=1, pts[11,2]=1,  pts[3,2]=1）

res = 1
```

四個角長這樣：
```
(3,10) ── (11,10) ← Q
  |             |
(3,2)  ── (11,2)
```

**count([14, 8]) → 0**
```
Q = (14, 8)
y=8 這排：沒有任何點

res = 0
```

**add([11, 2])（第二次）**
```
pts:  {[3,10]:1, [11,2]:2, [3,2]:1}   ← [11,2] count 變 2
xByY: 不變
```

**count([11, 10]) → 2**
```
Q = (11, 10)，一樣找 x1=3，side=8

  下方：1 × 2 × 1 = 2   （pts[3,10]=1, pts[11,2]=2, pts[3,2]=1）

res = 2
```

(11, 2) 有兩個副本，乘法自動把兩條路徑都算進去。

---

**思路**

固定 Q = (qx, qy) 和同排的 P1 = (x1, qy)，side = |x1 - qx|：

```
上方正方形：(qx, qy+side) 和 (x1, qy+side)
下方正方形：(qx, qy-side) 和 (x1, qy-side)
```

Q 是查詢點（固定），其他三個點從資料結構裡找。

重複點算不同點。每個角落各自獨立選，選法相乘：

```
角落A有2個副本，角落B有3個副本，角落C有1個副本
→ 2 × 3 × 1 = 6 種組合
```

所以：

```
count += pts[P1] × pts[(qx, qy+side)] × pts[(x1, qy+side)]
count += pts[P1] × pts[(qx, qy-side)] × pts[(x1, qy-side)]
```

```typescript
class DetectSquares {
    private pts = new Map<string, number>();
    private xByY = new Map<number, Set<number>>();

    private key(x: number, y: number): string { return `${x},${y}`; }
    private get(x: number, y: number): number { return this.pts.get(this.key(x, y)) ?? 0; }

    add(point: number[]): void {
        const [x, y] = point;
        this.pts.set(this.key(x, y), this.get(x, y) + 1);
        if (!this.xByY.has(y)) this.xByY.set(y, new Set());
        this.xByY.get(y)!.add(x);
    }

    count(point: number[]): number {
        const [qx, qy] = point;
        let res = 0;
        for (const x1 of (this.xByY.get(qy) ?? [])) {
            if (x1 === qx) continue;
            const side = Math.abs(x1 - qx);
            const p1 = this.get(x1, qy);
            res += p1 * this.get(qx, qy + side) * this.get(x1, qy + side); // 上
            res += p1 * this.get(qx, qy - side) * this.get(x1, qy - side); // 下
        }
        return res;
    }
}
```

<details>
<summary>Go 版本</summary>

```go
type DetectSquares struct {
    pts  map[[2]int]int       // 每個點出現幾次
    xByY map[int]map[int]bool // y → 這排有哪些 x；count 需要枚舉同橫排，所以 y 當 key
}

func Constructor() DetectSquares {
    return DetectSquares{
        pts:  make(map[[2]int]int),      // {[3,10]:1, [11,2]:2, [3,2]:1}
        xByY: make(map[int]map[int]bool), // {10:{3:true}, 2:{11:true, 3:true}}
    }
}

func (d *DetectSquares) Add(point []int) {
    p := [2]int{point[0], point[1]}
    d.pts[p]++                          // pts[[3,10]]++ → pts[[3,10]]=1
    if d.xByY[point[1]] == nil {
        d.xByY[point[1]] = make(map[int]bool)
    }
    d.xByY[point[1]][point[0]] = true   // xByY[10][3]=true
}

func (d *DetectSquares) Count(point []int) int {
    qx, qy := point[0], point[1]        // Q=(11,10)
    res := 0
    for x1 := range d.xByY[qy] {       // y=10 這排：x1=3
        if x1 == qx {                   // 3 != 11，繼續
            continue
        }
        side := x1 - qx                 // 3-11 = -8
        if side < 0 {
            side = -side                 // side=8
        }
        p1 := d.pts[[2]int{x1, qy}]    // pts[[3,10]] = 1
        // 上方正方形：(11,18),(3,18) → 兩點都不存在 → 0
        res += p1 * d.pts[[2]int{qx, qy + side}] * d.pts[[2]int{x1, qy + side}]
        // 下方正方形：(11,2),(3,2) → 1*1*1=1
        res += p1 * d.pts[[2]int{qx, qy - side}] * d.pts[[2]int{x1, qy - side}]
    }
    return res // 1
}
```

</details>

---

**核心**

```
固定同排點 P1，side = |x1 - qx|
count += pts[P1] × pts[上左] × pts[上右]
count += pts[P1] × pts[下左] × pts[下右]
```

重複點用乘法自動算進去。

---

**複雜度**

- `add`: O(1)
- `count`: O(U)，U = 與查詢點同一橫排的不重複 x 數量
- Space: O(N)
