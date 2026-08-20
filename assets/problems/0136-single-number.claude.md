一個陣列裡每個數字都出現兩次，只有一個出現一次。找出那個落單的。

---

## 解法一：Hash Map

最直覺的做法：用 hash map 計數。

```typescript
function singleNumber(nums: number[]): number {
    const count = new Map<number, number>();
    for (const n of nums) {
        count.set(n, (count.get(n) ?? 0) + 1);
    }
    for (const n of nums) {
        if (count.get(n) === 1) return n; // 只出現一次
    }
    return -1;
}
```

- Time: O(n)
- Space: **O(n)** — hash map 存了所有數字

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) int {
    count := map[int]int{}
    for _, n := range nums {
        count[n]++
    }
    for _, n := range nums {
        if count[n] == 1 { // 只出現一次的就是答案
            return n
        }
    }
    return -1
}
```

</details>

能跑。但題目要求 **constant extra space**。O(n) 空間不行。

---

## 解法二：XOR

先想 XOR 的性質：

```
a ^ a = 0    // 自己跟自己 XOR = 0
a ^ 0 = a    // 跟 0 XOR = 自己
```

而且 XOR 滿足交換律和結合律。順序不重要。

把所有數字 XOR 在一起：

```
[4, 1, 2, 1, 2]

4 ^ 1 ^ 2 ^ 1 ^ 2
= 4 ^ (1 ^ 1) ^ (2 ^ 2)    // 交換律，把相同的湊一起
= 4 ^ 0 ^ 0                  // 成對的全部歸零
= 4                           // 剩下落單的
```

成對的互相抵消，落單的留下來。一次遍歷，一個變數。

```typescript
function singleNumber(nums: number[]): number {
    let result = 0;
    for (const n of nums) {
        result ^= n; // 成對的抵消，落單的留下
    }
    return result;
}
```

- Time: O(n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) int {
    result := 0
    for _, n := range nums {
        result ^= n // 成對的抵消，落單的留下
    }
    return result
}
```

</details>

一個變數，一次遍歷。完美。

---

## 解法三：排序

O(n) 是下界。你至少得看過每個數字一次，不然怎麼知道誰落單。

有人會想到排序：排完之後相鄰的應該一樣，不一樣的就是答案。

```typescript
function singleNumber(nums: number[]): number {
    nums.sort((a, b) => a - b);
    for (let i = 0; i < nums.length - 1; i += 2) {
        if (nums[i] !== nums[i + 1]) {
            return nums[i];
        }
    }
    return nums[nums.length - 1];
}
```

- Time: **O(n log n)** — 排序
- Space: O(1)（如果 in-place sort）

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) int {
    sort.Ints(nums)
    for i := 0; i < len(nums)-1; i += 2 {
        if nums[i] != nums[i+1] {
            return nums[i]
        }
    }
    return nums[len(nums)-1]
}
```

</details>

時間比 XOR 差。而且排序改了原陣列。沒理由用這個。

---

## 解法四：數學

也有人用數學：`2 * sum(set) - sum(all)`。把所有不重複的數字加起來乘二，減掉全部的總和，差就是落單的。

```
nums = [4, 1, 2, 1, 2]
2 * (4 + 1 + 2) - (4 + 1 + 2 + 1 + 2) = 14 - 10 = 4
```

```typescript
function singleNumber(nums: number[]): number {
    const seen = new Set<number>();
    let setSum = 0, totalSum = 0;
    for (const n of nums) {
        totalSum += n;
        if (!seen.has(n)) {
            setSum += n;
            seen.add(n);
        }
    }
    return 2 * setSum - totalSum;
}
```

- Time: O(n)
- Space: O(n) — 還是要 set

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) int {
    seen := map[int]bool{}
    setSum, totalSum := 0, 0
    for _, n := range nums {
        totalSum += n
        if !seen[n] {
            setSum += n
            seen[n] = true
        }
    }
    return 2*setSum - totalSum
}
```

</details>

數學上正確，但空間沒省。而且大數可能 overflow（雖然這題範圍小不會）。XOR 完勝。

---

**Overthinking**

面試官追問：「如果有兩個數字各出現一次呢？」

這就是 [260. Single Number III](/problem/single-number-iii)。全部 XOR 之後得到 `a ^ b`，然後用 `a ^ b` 的某個為 1 的 bit 把陣列分成兩組，各自 XOR 就拆出 a 和 b。

「如果每個數字出現三次，只有一個出現一次呢？」

這是 [137. Single Number II](/problem/single-number-ii)。XOR 只能處理「出現兩次」的情況。三次需要逐 bit 統計，對每個 bit 的 1 的個數 mod 3。

---

## 結論

`a ^ a = 0`，成對抵消。一次遍歷，一個變數，O(n) 時間 O(1) 空間。XOR 的最經典應用，沒有之一。
