一個 32-bit 的數字，把它的 bit 從左到右整排反過來讀。像照鏡子：最左邊那位變最右邊，最右邊變最左邊，中間對稱翻。

用 8-bit 看比較清楚（實際題目是 32-bit）：

```
原始   0 0 0 0 1 0 1 1   （= 11）
       ← 整排鏡射 →
結果   1 1 0 1 0 0 0 0   （= 208）
```

翻回題目：給一個 32-bit 整數，回傳它 bit 反轉後的值。題目還保證輸入是偶數（最低位是 0），所以反轉後最高位一定是 0，結果不會踩到負數那格。

---

**解題引導**

用 8-bit 的 `00001011` 想。

**Step 1：怎麼一次只處理一個 bit？**

*peel off the lowest bit of the input*

<span class="spoiler">用 `n & 1` 取出最低位，再把 n 右移一位丟掉它。重複 32 次就掃過每個 bit。Mask the lowest bit, then shift n right.</span>

**Step 2：取出來的 bit 要怎麼擺到 result？**

*build result from the top down: shift left, then drop the bit in*

<span class="spoiler">result 先左移一位空出尾端，再把剛取的 bit 用 OR 接上去。輸入的最低位第一個被讀，卻是 result 最先進去、最後被推到最高位的，剛好翻轉。Shift result left, OR the bit into the end.</span>

**Step 3：follow-up 說會被呼叫很多次，怎麼加速？**

*process bigger chunks at once, or precompute a lookup table*

<span class="spoiler">一次交換一整組 bit（分治），或預先算好每個 byte 的反轉存表，之後查表。Swap in groups, or cache byte reversals.</span>

想完再往下看 code。

---

## 解法一：一位一位翻

掃 32 次，每次把 n 的最低位取下來，接到 result 的尾端。因為 result 每一輪都先左移，先進去的 bit 會被一路推到高位，自然形成反轉。

用 `00001011`（8-bit，= 11）走：

```
result 從 0 開始，n 每輪交出最低位：

i=0  n&1 = 1 → result = (0    << 1) | 1 = 1
i=1  n&1 = 1 → result = (1    << 1) | 1 = 11
i=2  n&1 = 0 → result = (11   << 1) | 0 = 110
i=3  n&1 = 1 → result = (110  << 1) | 1 = 1101
i=4~7 n 剩下都是 0，result 繼續左移補 0

最後（補滿 8 位）result = 11010000 = 208
```

```typescript
function reverseBits(n: number): number {
    let result = 0;
    for (let i = 0; i < 32; i++) {
        result = (result << 1) | (n & 1);  // result 左移空出尾端，接上 n 的最低位
        n = n >>> 1;                        // n 無號右移，丟掉剛用掉的最低位
    }
    return result >>> 0;                     // 轉回無號 32-bit
}
```

- Time: $O(1)$ — 固定跑 32 圈，跟輸入大小無關
- Space: $O(1)$

那兩個 `>>>` 是重點，不是打錯。JS 的 bitwise 運算把數字當 **32-bit 有號**整數處理，所以：

- `n >>> 1` 用無號右移。如果改用 `>>`（有號右移），當最高位是 1 時會補符號位、把 1 灌回來，就錯了。
- `result >>> 0` 把最後結果轉成無號。`result << 1` 可能讓第 31 位變 1，在有號解讀下會是負數，`>>> 0` 還原成正確的無號值。

這題 constraint 保證輸入偶數、最高位不會是 1，就算用 `>>` 也剛好不出錯；但一般的 reverse bits 要用 `>>>` 才穩。細節見 [Bit Operators](/concept/operator) 跟 [Two's Complement](/concept/twos-complement)。

<details>
<summary>Go 版本</summary>

```go
// Go 用 uint32，無號型別本來就不會有符號位問題，不需要 >>>
func reverseBits(num uint32) uint32 {
    var result uint32
    for i := 0; i < 32; i++ {
        result = (result << 1) | (num & 1) // 接上最低位
        num >>= 1                          // 丟掉最低位
    }
    return result
}
```

Go 的 `uint32` 是無號型別，`>>` 對它就是邏輯右移，不會補符號位，所以不用像 JS 那樣區分 `>>` / `>>>`。

</details>

---

## 解法二：分治，成組交換

解法一跑 32 圈。反轉其實可以**成組交換**：先把左右兩半 16 位對調，再把每半裡的兩個 8 位對調，一路對半下去到 1 位。$\log_2 32 = 5$ 步就完成，比 32 圈少很多。

每一步用一個 mask 把「奇數組」跟「偶數組」分開，各自移位再合併：

```
16 位：左右兩半對調
 8 位：每半的兩個 byte 對調
 4 位：每個 byte 裡兩個 nibble 對調
 2 位：每 4 位裡兩組 2 位對調
 1 位：每 2 位裡相鄰兩個 bit 對調
```

```typescript
function reverseBits(n: number): number {
    n = ((n >>> 1) & 0x55555555) | ((n & 0x55555555) << 1);  // 交換相鄰 1 位
    n = ((n >>> 2) & 0x33333333) | ((n & 0x33333333) << 2);  // 交換相鄰 2 位
    n = ((n >>> 4) & 0x0f0f0f0f) | ((n & 0x0f0f0f0f) << 4);  // 交換相鄰 4 位
    n = ((n >>> 8) & 0x00ff00ff) | ((n & 0x00ff00ff) << 8);  // 交換相鄰 8 位
    n = (n >>> 16) | (n << 16);                              // 交換左右 16 位
    return n >>> 0;
}
```

mask 是成對的：`0x55555555` 是 `0101...0101`（挑出奇數位），配合右移把奇數位拉到偶數位；另一半 `& 0x55555555` 再左移，把偶數位推到奇數位。兩邊 OR 起來就是相鄰兩位對調。`0x33` 是 `0011`、`0x0f` 是 `00001111`，越後面組越大。

- Time: $O(1)$ — 固定 5 步
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func reverseBits(num uint32) uint32 {
    num = (num>>1)&0x55555555 | (num&0x55555555)<<1
    num = (num>>2)&0x33333333 | (num&0x33333333)<<2
    num = (num>>4)&0x0f0f0f0f | (num&0x0f0f0f0f)<<4
    num = (num>>8)&0x00ff00ff | (num&0x00ff00ff)<<8
    num = (num >> 16) | (num << 16)
    return num
}
```

</details>

**Overthinking：follow-up 呼叫很多次怎麼辦？**

分治已經很快，但如果同一支函式被叫幾百萬次，還能再用空間換時間：預先算好每個 byte（256 種）反轉後的值，存進一張表。之後把 32 位拆成 4 個 byte，各自查表拿反轉，再把 4 個 byte 的順序也倒過來拼回去。查表是 $O(1)$，四次查表加拼裝，常數更小。這是拿 256 格記憶體換速度，就是 follow-up 想聽的「重複呼叫先建 cache」。

---

## 解法比較表

| 解法 | Time | Space | 操作次數 | 備註 |
|---|---|---|---|---|
| 一位一位翻 | $O(1)$ | $O(1)$ | 32 圈 | 最好懂，面試先講這個 |
| 分治成組交換 | $O(1)$ | $O(1)$ | 5 步 | 快，但 mask 要記熟 |
| 查表 cache | $O(1)$ | $O(256)$ | 4 次查表 | 重複呼叫才划算 |

---

## 結論

Reverse bits 就是把 32 位鏡射。最直覺的做法是一位一位取最低位、接到 result 尾端，掃 32 圈。成組交換（分治）可以壓到 5 步。如果同一支函式被呼叫很多次，可以再加一張預先算好的 byte 反轉表。JS 記得用 `>>>` 處理無號，Go 的 `uint32` 沒這問題。
