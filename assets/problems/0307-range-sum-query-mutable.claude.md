一排存錢筒。有人隨時把某一罐的錢整罐換掉，老闆隨時問「第 left 罐到第 right 罐現在一共多少」。兩種事件交錯發生，每次都要馬上答。

拿 Example 的數字：罐子裡是 `[1, 3, 5]`，老闆問第 0 到 2 罐 → 9；有人把第 1 罐換成 2 → `[1, 2, 5]`；再問一次 → 8。

翻回 code：實作一個類別，`update(index, val)` 修改某一格，`sumRange(left, right)` 查詢區間和，兩種呼叫會來 $3 \times 10^4$ 次。

---

**解題引導**

用 `nums = [1, 3, 5]` 想。

**Step 1：兩個直覺方案，各卡在哪？**

*two obvious plans, each fails one way*

<span class="spoiler">存原陣列：修改 O(1)，但每次查詢要現場加，O(n)。存 prefix sum：查詢 O(1)，但修改一格之後面的前綴全要重算，O(n)。兩個方案各有一邊是 O(n)。 / Raw array: update O(1), query O(n). Prefix sums: query O(1), update O(n). Each plan is slow on one side.</span>

**Step 2：兩邊都要快，中間方案長什麼樣？**

*store something between single cells and full prefixes*

<span class="spoiler">預先算好一批「一段連續格子的和」，查詢時拼幾段、修改時補幾段，兩邊都 O(log n)。這就是 Fenwick Tree。 / Precompute sums of some ranges; queries assemble a few, updates patch a few. Both O(log n). That is a Fenwick Tree.</span>

**Step 3：sumRange(left, right) 怎麼變成前綴的問題？**

*a range is the difference of two prefixes*

<span class="spoiler">區間和 = 前 right+1 格的和 − 前 left 格的和。Fenwick 只要會答前綴和，區間就用相減拿到。 / range sum = prefix(right+1) − prefix(left). Fenwick answers prefixes; subtraction gives any range.</span>

**Step 4：update 給的是新值，Fenwick 吃什麼？**

*the tree stores sums, so feed it the change*

<span class="spoiler">吃差值。delta = 新值 − 舊值，沿路加進涵蓋這格的積木。所以要另外留一份現值陣列，才算得出舊值。 / It eats the delta = new − old, added into every covering block. Keep a copy of current values to know the old one.</span>

**Step 5：為什麼內部編號從 1 開始？**

*what does lowbit(0) do to the loop*

<span class="spoiler">lowbit(0) = 0，迴圈加 0 減 0 都在原地不動，變成無窮迴圈。所以 tree 用 1-indexed，對外的 index 進來先 +1。 / lowbit(0) = 0 makes the loop stand still forever. Use 1-indexed internally; add 1 to every incoming index.</span>

想完再往下看 code。

---

## 解法一：存原陣列

修改直接改，查詢現場加：

```typescript
class NumArray {
    constructor(private nums: number[]) {}

    update(index: number, val: number): void {
        this.nums[index] = val;                    // O(1)
    }

    sumRange(left: number, right: number): number {
        let sum = 0;
        for (let i = left; i <= right; i++) {      // O(n)
            sum += this.nums[i];
        }
        return sum;
    }
}
```

- Time：update $O(1)$、sumRange $O(n)$
- Space：$O(1)$ 額外

<details>
<summary>Go 版本</summary>

```go
type NumArray struct {
    nums []int
}

func Constructor(nums []int) NumArray {
    return NumArray{nums: nums}
}

func (na *NumArray) Update(index, val int) {
    na.nums[index] = val
}

func (na *NumArray) SumRange(left, right int) int {
    sum := 0
    for i := left; i <= right; i++ {
        sum += na.nums[i]
    }
    return sum
}
```

</details>

$n = 3 \times 10^4$、操作 $3 \times 10^4$ 次，最壞 $9 \times 10^8$ 步，照 $10^7$ 步 ≈ 1 秒算大約 90 秒。TLE。

---

## 解法二：存 prefix sum

反過來，把前綴和先算好，查詢用相減：

```typescript
class NumArray {
    private prefix: number[];

    constructor(private nums: number[]) {
        this.prefix = [0];                          // 多一格 placeholder，讓 prefix[right+1] 對得上
        for (const v of nums) {
            this.prefix.push(this.prefix[this.prefix.length - 1] + v);
        }
    }

    update(index: number, val: number): void {
        const delta = val - this.nums[index];
        this.nums[index] = val;
        for (let i = index + 1; i < this.prefix.length; i++) {
            this.prefix[i] += delta;               // 後面的前綴全要跟著改，O(n)
        }
    }

    sumRange(left: number, right: number): number {
        return this.prefix[right + 1] - this.prefix[left];   // O(1)
    }
}
```

- Time：update $O(n)$、sumRange $O(1)$
- Space：$O(n)$

<details>
<summary>Go 版本</summary>

```go
type NumArray struct {
    nums   []int
    prefix []int
}

func Constructor(nums []int) NumArray {
    prefix := make([]int, len(nums)+1) // 多一格 placeholder，讓 prefix[right+1] 對得上
    for i, v := range nums {
        prefix[i+1] = prefix[i] + v
    }
    return NumArray{nums: nums, prefix: prefix}
}

func (na *NumArray) Update(index, val int) {
    delta := val - na.nums[index]
    na.nums[index] = val
    for i := index + 1; i < len(na.prefix); i++ {
        na.prefix[i] += delta
    }
}

func (na *NumArray) SumRange(left, right int) int {
    return na.prefix[right+1] - na.prefix[left]
}
```

</details>

慢的那邊換了位置而已：查詢快了，修改變 $O(n)$。操作混著來的時候，最壞還是 $9 \times 10^8$ 步、約 90 秒。TLE。

兩個方案合起來看，這題要的就是「修改跟查詢都快」，正是 [Bit Operators 概念頁](/concept/operator)裡 Fenwick Tree 那節解的問題。

---

## 解法三：Fenwick Tree

積木的原理（做積木、查詢＝鋪地板、修改＝蓋到的都跟著改）在[概念頁](/concept/operator)講完了，這裡只處理接線的三件事：

1. **區間變前綴**：`sumRange(left, right) = query(right + 1) − query(left)`。Fenwick 只會答「前 i 格的和」，區間用兩個前綴相減。
2. **update 吃差值**：Fenwick 存的是和，所以更新的時候要把 `delta = 新值 − 舊值` 加進沿路的積木。類別裡要留一份現值陣列 `nums`，不然不知道舊值是多少。
3. **內部 1-indexed**：`lowbit(0) = 0`，迴圈碰到 0 會原地不動。對外的 index 進來一律 +1。

```typescript
class NumArray {
    private n: number;
    private nums: number[];
    private tree: number[];

    constructor(nums: number[]) {
        this.n = nums.length;
        this.nums = nums.slice();                  // 現值留一份，算 delta 用
        this.tree = new Array(this.n + 1).fill(0); // 1-indexed，tree[0] 不用
        for (let i = 0; i < this.n; i++) {
            this.add(i + 1, nums[i]);              // 把每個元素當一次 delta 加進去
        }
    }

    private add(i: number, delta: number): void {  // 蓋到第 i 格的積木都跟著改
        for (; i <= this.n; i += i & -i) {
            this.tree[i] += delta;
        }
    }

    private query(i: number): number {             // 前 i 格的和：鋪地板
        let sum = 0;
        for (; i > 0; i -= i & -i) {
            sum += this.tree[i];
        }
        return sum;
    }

    update(index: number, val: number): void {
        const delta = val - this.nums[index];
        this.nums[index] = val;
        this.add(index + 1, delta);
    }

    sumRange(left: number, right: number): number {
        return this.query(right + 1) - this.query(left);
    }
}
```

- Time：update $O(\log n)$、sumRange $O(\log n)$、建構 $O(n \log n)$
- Space：$O(n)$

<details>
<summary>Go 版本</summary>

```go
type NumArray struct {
    n    int
    nums []int
    tree []int
}

func Constructor(nums []int) NumArray {
    na := NumArray{
        n:    len(nums),
        nums: make([]int, len(nums)),
        tree: make([]int, len(nums)+1), // 1-indexed，tree[0] 不用
    }
    copy(na.nums, nums)
    for i, v := range nums {
        na.add(i+1, v) // 把每個元素當一次 delta 加進去
    }
    return na
}

func (na *NumArray) add(i, delta int) { // 蓋到第 i 格的積木都跟著改
    for ; i <= na.n; i += i & (-i) {
        na.tree[i] += delta
    }
}

func (na *NumArray) query(i int) int { // 前 i 格的和：鋪地板
    sum := 0
    for ; i > 0; i -= i & (-i) {
        sum += na.tree[i]
    }
    return sum
}

func (na *NumArray) Update(index, val int) {
    delta := val - na.nums[index]
    na.nums[index] = val
    na.add(index+1, delta)
}

func (na *NumArray) SumRange(left, right int) int {
    return na.query(right+1) - na.query(left)
}
```

</details>

**走一遍 Example**：`nums = [1, 3, 5]`，內部編號 1 到 3，積木是 `tree[1] = [1]`、`tree[2] = [1..2]`、`tree[3] = [3]`。

建構（逐個元素 add）：

```
add(1, 1)：tree[1] = 1 → 跳到 2，tree[2] = 1 → 跳到 4 > 3，停
add(2, 3)：tree[2] = 4 → 跳到 4 > 3，停
add(3, 5)：tree[3] = 5 → 跳到 4 > 3，停

tree = [_, 1, 4, 5]
```

`sumRange(0, 2)` → `query(3) − query(0)`：

```
query(3)：拿 tree[3] = 5 → 跳到 2，拿 tree[2] = 4 → sum = 9 → 跳到 0，停
query(0)：0
9 − 0 = 9 ✓
```

`update(1, 2)`：舊值 3、新值 2，delta = −1：

```
add(2, −1)：tree[2] = 3 → 跳到 4 > 3，停
tree = [_, 1, 3, 5]，nums = [1, 2, 5]
```

再查 `sumRange(0, 2)` → `query(3) = 5 + 3 = 8` ✓，跟 Example 的輸出一致。

順便看相減怎麼用：`sumRange(1, 2) = query(3) − query(1) = 8 − 1 = 7`，就是 2 + 5。

---

**Overthinking：Segment Tree 呢？**

這題掛著兩個 tag：Binary Indexed Tree 跟 Segment Tree。Segment tree（線段樹）是一棵二元樹，每個節點存底下那段區間的聚合值：整個陣列先切成左右兩半，兩個子節點各存左半、右半的和，再往下一路對半切，切到每個節點只管一格。拿這篇的例子 `nums = [1, 3, 5]` 畫出來：

```
              [0,2]=9
             /        \
        [0,1]=4      [2,2]=5
        /      \
   [0,0]=1   [1,1]=3
```

查詢或更新都是從根節點往下走一條路徑，跟 Fenwick 一樣是 $O(\log n)$，而且更通用：區間最小值、最大值它都做得到，Fenwick 只能處理「相減可以還原」的聚合（和可以，min 不行，因為知道總和跟其中一段可以減出另一段，min 減不出來）。代價是 code 長好幾倍。這題只要區間和，Fenwick 的三十行就夠。

---

## 解法比較表

| 解法 | update | sumRange | 最壞總步數（n、操作各 $3 \times 10^4$） | 結果 |
|---|---|---|---|---|
| 存原陣列 | $O(1)$ | $O(n)$ | $9 \times 10^8$ ≈ 90 秒 | TLE |
| Prefix sum | $O(n)$ | $O(1)$ | $9 \times 10^8$ ≈ 90 秒 | TLE |
| Fenwick Tree | $O(\log n)$ | $O(\log n)$ | $3 \times 10^4 \times 15 \approx 4.5 \times 10^5$ ≈ 0.045 秒 | 過 |

---

## 結論

修改跟查詢都要快，就把「一格」跟「整段前綴」之間的中間產物先做好：Fenwick 的積木。接線只有三件事：區間用前綴相減、update 餵差值、內部從 1 開始編號。
