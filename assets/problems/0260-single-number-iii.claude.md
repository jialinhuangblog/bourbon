一個陣列裡每個數字都出現兩次，有兩個數字各出現一次。找出那兩個。

---

[136. Single Number](/problem/single-number) 只有一個落單，全部 XOR 就拿到了。這題有兩個。

全部 XOR 起來會怎樣？

XOR 有兩個性質：相同的數 XOR = 0，任何數 XOR 0 = 自己。

所以成對的數字會自己消掉。`1 ^ 2 ^ 1 ^ 3 ^ 2 ^ 5` 裡面 `1 ^ 1 = 0`、`2 ^ 2 = 0`，剩下 `0 ^ 0 ^ 3 ^ 5 = 3 ^ 5`。

成對的全消了，只剩兩個落單的 XOR 在一起。結果是 `a ^ b`，不是答案本身。

---

## 解法一：HashMap 計數

先暴力。

```typescript
function singleNumber(nums: number[]): number[] {
    const count = new Map<number, number>();
    for (const n of nums) {
        count.set(n, (count.get(n) ?? 0) + 1);
    }
    const result: number[] = [];
    for (const [k, v] of count) {
        if (v === 1) result.push(k);
    }
    return result;
}
```

- Time: O(n)
- Space: **O(n)**

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) []int {
    count := map[int]int{}
    for _, n := range nums {
        count[n]++
    }
    var result []int
    for k, v := range count {
        if v == 1 {
            result = append(result, k)
        }
    }
    return result
}
```

</details>

題目要求 O(1) 空間。

---

## 解法二：XOR 分組

現在手上有 `a ^ b`，但拆不開。`3 ^ 5 = 110`，你不知道哪個是 3、哪個是 5。

`110` 這個結果說明 a 和 b 在第 1 位和第 2 位不同。XOR 結果為 1 的位元，代表兩個數在那個位置一個是 0、一個是 1。

現在問題變成：怎麼用這個資訊把 a 和 b 拆開？

如果能把所有數字分成兩組，讓 a 在一組、b 在另一組，每組就回到「只有一個落單」的問題，各自 XOR 就拿到答案了。

怎麼分？挑 `a ^ b` 裡任何一個為 1 的 bit。這個 bit a 和 b 一定不同，一個是 0 一個是 1，所以它們一定被分到不同組。而成對的數字每個 bit 都一樣，一定被分到同一組。

挑哪個 bit 都行。`110` 裡 mask 可以是 `010` 或 `100`，兩個都能正確分組——差別只是數字被分到不同組，但每組 XOR 出來的答案一樣正確。

但分組的時候需要一個只有單一 bit 的 mask。`110` 有兩個 1，不能直接拿來用——用兩個 bit 分組會把成對的數字拆散。

`xorAll & (-xorAll)` 幫你從 `110` 裡取出最右邊的 1，得到 `010`。為什麼這招有效？詳見 [Lowbit](/concept/lowbit)。

```
nums = [1, 2, 1, 3, 2, 5]

1 = 001, 2 = 010, 3 = 011, 5 = 101

xorAll = 3 ^ 5 = 011 ^ 101 = 110

取最右邊的 1：110 & 010 = 010 → bit 1

按 bit 1 分組：
  bit 1 = 1: [2, 3, 2]     → XOR = 3
  bit 1 = 0: [1, 1, 5]     → XOR = 5
```

```typescript
function singleNumber(nums: number[]): number[] {
    let xorAll = 0;
    for (const n of nums) {
        xorAll ^= n;               // 得到 a ^ b
    }

    const diff = xorAll & (-xorAll); // 最右邊的 1

    let a = 0, b = 0;
    for (const n of nums) {
        if (n & diff) {
            a ^= n;                 // 那個 bit 是 1 的一組
        } else {
            b ^= n;                 // 那個 bit 是 0 的一組
        }
    }
    return [a, b];
}
```

- Time: O(n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) []int {
    xorAll := 0
    for _, n := range nums {
        xorAll ^= n               // 得到 a ^ b
    }

    diff := xorAll & (-xorAll)    // 取最右邊的 1（a 和 b 不同的某個 bit）

    a, b := 0, 0
    for _, n := range nums {
        if n&diff != 0 {
            a ^= n                // 那個 bit 是 1 的一組
        } else {
            b ^= n                // 那個 bit 是 0 的一組
        }
    }
    return []int{a, b}
}
```

</details>

兩次遍歷（第一次算 xorAll，第二次分組 XOR），幾個變數。

---

**三步拆解**

三步，每步都建立在上一步上：

1. **全部 XOR = a ^ b**。成對的消掉，剩兩個落單的 XOR。
2. **a ^ b 裡的 1 = a 和 b 不同的 bit**。至少有一個 bit 不同，不然 a = b。
3. **用那個 bit 分組**。相同的數一定同組（每個 bit 都一樣），a 和 b 一定不同組。各自 XOR 就拆開了。

`xorAll & (-xorAll)` 只是取最右邊的 1 的技巧。用任何一個為 1 的 bit 分組都行，最右邊的只是最好取。

---

## 結論

一個落單用 XOR。兩個落單？先 XOR 拿到差異，再用差異的某個 bit 分組，各自 XOR 拆出來。O(n) 時間 O(1) 空間。Single Number 系列到此完結。
