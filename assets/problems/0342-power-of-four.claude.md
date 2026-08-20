給一個整數，判斷它是不是 4 的次方。

---

## 解法一：一直除以 4

最直覺的做法：一直除以 4。

```typescript
function isPowerOfFour(n: number): boolean {
    if (n <= 0) return false;
    while (n % 4 === 0) n /= 4; // 一直除到不能除
    return n === 1;
}
```

- Time: O(log n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func isPowerOfFour(n int) bool {
    if n <= 0 {
        return false
    }
    for n%4 == 0 { // 能被 4 整除就一直除
        n /= 4
    }
    return n == 1   // 除到最後剩 1，就是 4 的次方
}
```

</details>

能跑。但題目追問了：**能不能不用迴圈？**

---

## 解法二：位元運算

先退一步想。4 的次方一定是 2 的次方。2 的次方有什麼特徵？

`n & (n - 1) === 0`。只有一個 bit 是 1。

為什麼？`n - 1` 把最右邊的 1 翻成 0，它右邊的 0 全翻成 1。就像十進位 `1000 - 1 = 0999`，二進位一樣：

```
  8 = 1000    ← 只有一個 1
  7 = 0111    ← 減 1：唯一的 1 變 0，右邊全變 1
  &   0000    ← AND 完歸零 → 是 2 的次方

 12 = 1100    ← 有兩個 1（bit 3、bit 2）
 11 = 1011    ← 減 1 只消最右邊的 1（bit 2），bit 3 沒動
  &   1000    ← bit 3 兩邊都是 1，活下來 → 結果非零 → 不是 2 的次方
```

規則：減 1 只消得掉**最右邊**那個 1。只有一個 1 的數，消掉就歸零；兩個以上，更高位的 1 會在 AND 裡活下來，結果非零。

但 2 的次方不一定是 4 的次方。8 是 $2^3$，不是 4 的次方。差在哪？

把 4 的次方列出來看 bit pattern：

```
1     = 0000 0001  (4⁰)
4     = 0000 0100  (4¹)
16    = 0001 0000  (4²)
64    = 0100 0000  (4³)
256   = 1 0000 0000 (4⁴)
```

4 的次方：那個唯一的 1，永遠在偶數位（bit 0、2、4、6...，從右邊數，最右邊是 bit 0）。

2 的次方但不是 4 的次方：

```
2     = 0000 0010  (bit 1)
8     = 0000 1000  (bit 3)
32    = 0010 0000  (bit 5)
```

1 落在奇數位（bit 1、3、5）。

怎麼區分？用 mask。把所有偶數位標成 1：

```
...0101 0101 0101 0101 0101 0101 0101 0101
```

32-bit 的值：`0x55555555`。跟 n 做 AND，4 的次方不會被清掉，其他 2 的次方會歸零。

```typescript
function isPowerOfFour(n: number): boolean {
    return n > 0 &&
        (n & (n - 1)) === 0 &&   // 2 的次方
        (n & 0x55555555) !== 0;   // bit 在偶數位
}
```

- Time: O(1)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func isPowerOfFour(n int) bool {
    return n > 0 &&
        n&(n-1) == 0 &&   // 只有一個 bit 是 1（2 的次方）
        n&0x55555555 != 0  // 那個 bit 在偶數位（4 的次方）
}
```

</details>

三個條件，一行搞定。沒有迴圈、沒有遞迴。

---

### mod 3 取代 mask

有人用 `n % 3 === 1` 取代 mask：

```typescript
function isPowerOfFour(n: number): boolean {
    return n > 0 && (n & (n - 1)) === 0 && n % 3 === 1;
}
```

<details>
<summary>Go 版本</summary>

```go
func isPowerOfFour(n int) bool {
    return n > 0 && n&(n-1) == 0 && n%3 == 1
}
```

</details>

為什麼能動？$4 \equiv 1 \pmod{3}$，所以 $4^k \equiv 1^k \equiv 1 \pmod{3}$。而 2 的奇數次方：$2 \equiv 2 \pmod{3}$，$8 \equiv 2 \pmod{3}$。mod 3 剛好能把它們濾掉。

巧妙。但面試時你得解釋模運算的數學性質。`0x55555555` 的做法用 bit pattern 就能說清楚，直覺上更好解釋。

兩種都是 O(1)。選你能在 30 秒內講清楚的那個。

---

## 結論

`n & (n-1) == 0` 確認 2 的次方，`0x55555555` mask 確認 bit 在對的位置。O(1)，不用迴圈。bit manipulation 的題目，pattern 就這幾招。
