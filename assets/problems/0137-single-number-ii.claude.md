一個陣列裡每個數字都出現三次，只有一個出現一次。找出那個落單的。

---

[136. Single Number](/problem/single-number) 用 XOR 一行搞定。但 XOR 只能消掉「出現兩次」的數——`a ^ a = 0`。三次呢？`a ^ a ^ a = a`，消不掉。

---

## 解法一：Hash Map

先用 hash map 暴力解。

```typescript
function singleNumber(nums: number[]): number {
    const count = new Map<number, number>();
    for (const n of nums) {
        count.set(n, (count.get(n) ?? 0) + 1);
    }
    for (const [k, v] of count) {
        if (v === 1) return k;
    }
    return -1;
}
```

- Time: $O(n)$
- Space: **$O(n)$**

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) int {
    count := map[int]int{}
    for _, n := range nums {
        count[n]++
    }
    for k, v := range count {
        if v == 1 {
            return k
        }
    }
    return -1
}
```

</details>

題目要求 $O(1)$ 空間。

---

## 解法二：逐 bit 統計

XOR 不行了。換個角度：**逐 bit 想**。

把所有數字的每一個 bit 分開統計。如果某個 bit 位置上 1 出現了 3 的倍數次，那個 bit 屬於出現三次的數字，不關我們的事。如果不是 3 的倍數，多出來的那個 1 就是答案的。

```
nums = [5, 7, 5, 7, 5, 7, 11]

二進位：
5  = 0101  ×3
7  = 0111  ×3
11 = 1011  ×1 ← 落單的

逐 bit 統計：

bit 0: 5→1 ×3, 7→1 ×3, 11→1  = 3+3+1 = 7，7 % 3 = 1 ≠ 0 → 留下
bit 1: 5→0 ×3, 7→1 ×3, 11→1  = 0+3+1 = 4，4 % 3 = 1 ≠ 0 → 留下
bit 2: 5→1 ×3, 7→1 ×3, 11→0  = 3+3+0 = 6，6 % 3 = 0     → 清掉
bit 3: 5→0 ×3, 7→0 ×3, 11→1  = 0+0+1 = 1，1 % 3 = 1 ≠ 0 → 留下

result: bit 3=1, bit 2=0, bit 1=1, bit 0=1 → 1011 = 11 ✓
```

bit 2 是關鍵：5 和 7 都在 bit 2 是 1，各出現 3 次 → 6 個 1 → `% 3 = 0`，被清掉。而 11 在 bit 2 是 0，本來就不貢獻。`% 3` 精準地把出現三次的數字從每個 bit 抹掉，只留落單的。

**先拆符號**

code 裡有幾個位元運算，先一個一個看，用 `11 = 1011` 當例子：

- `n >> i`（右移）：把 n 往右推 i 位，第 i 位就跑到最右邊。11 的二進位是 `1011`，`11 >> 2` 得到 2，二進位看就是 `10`，右邊兩位掉出去。
- `x & 1`（跟 1 做 AND）：只留最右邊那一位，其他清成 0。等於在問「最右邊是 0 還是 1」。
- 兩個合起來 `(n >> i) & 1`：先把第 i 位移到最右邊、再讀它，答案就是「n 的第 i 位是 0 還是 1」。所以 `bitSum += (n >> i) & 1` 是「第 i 位是 1 就加一」，一排數字掃完，就知道這一位總共有幾個 1。
- `1 << i`（左移）：把 1 往左推 i 位，得到一個只有第 i 位是 1、其他全 0 的數。`1 << 3` 得到 8，二進位是 `1000`。
- `result |= 1 << i`：`|=` 是 OR 版的 `+=`，`a |= b` 就是 `a = a | b`。OR 的效果是「兩邊有一邊是 1 就變 1」，所以這行把 result 的第 i 位設成 1，其他位不動。

看懂這幾個，下面 code 就是把上面那張逐 bit 表格寫出來：外層跑 32 個 bit，內層數這一位有幾個 1，`% 3` 不為 0 就把答案的這一位打開。

```typescript
function singleNumber(nums: number[]): number {
    let result = 0;
    for (let i = 0; i < 32; i++) {
        let bitSum = 0;
        for (const n of nums) {
            bitSum += (n >> i) & 1;  // 統計第 i 個 bit
        }
        if (bitSum % 3 !== 0) {      // 多出來的 1 屬於答案
            result |= 1 << i;        // 把 result 的第 i 位打開
        }
    }
    return result;
}
```

- Time: $O(32n)$ = $O(n)$
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) int {
    result := 0
    for i := 0; i < 32; i++ {
        bitSum := 0
        for _, n := range nums {
            bitSum += (n >> i) & 1  // 統計第 i 個 bit 的 1 的個數
        }
        if bitSum%3 != 0 {         // 多出來的 1 屬於答案
            result |= 1 << i       // 把 result 的第 i 位打開
        }
    }
    return int(int32(result))      // 塞回 32-bit，bit 31 才會被當成符號位
}
```

</details>

32 是常數（32-bit 整數），所以是 $O(n)$。空間只用了幾個變數。

**Go 版最後為什麼要 `int(int32(result))`？** Go 的 int 是 64-bit（見 [integer types](/concept/integer-types)）。答案是負數時，它的 64-bit 表示法從 bit 31 以上全是 1，但迴圈只重組 bit 0 到 31，組出來的 result 是個 bit 31 為 1 的正數：答案 -2 會回傳 4294967294。先轉 int32，bit 31 重新被解讀成符號位，才變回 -2。TS 不用管這件事，JS 的位元運算天生就在 32-bit 有號整數上做，`result |= 1 << 31` 自動就是負數。

---

## 解法三：位元狀態機

還有一種只跑一次迴圈的解法。先別看 code，把問題縮到最小：只盯著某一個 bit 位置，一串 0/1 流進來，要數 1 出現幾次，而且只在乎 mod 3 的結果。

次數只有三種狀態：0 次、1 次、2 次，第 3 次繞回 0。一個 boolean 裝不下三種狀態，用兩個：

```
狀態    (ones, twos)
0 次     (0, 0)
1 次     (1, 0)      ← 記在 ones
2 次     (0, 1)      ← 搬到 twos
3 次     (0, 0)      ← 歸零，回到起點
```

規格就兩句：進來 0，狀態不動；進來 1，往下一格走。

實作靠兩個慣用片語（符號本身不熟，先回 [Bit Operators](/concept/operator)）：

- `x ^ n`：XOR 當開關。n 是 1 就把 x 翻面，n 是 0 就不動。「看到才動」就是這麼來的。
- `x & ~y`：清位。y 是 1 的位置，把 x 強制歸 0。`~` 先把 y 反轉成篩子，`&` 再拿篩子過濾，讀成「y 佔住的位置，我讓出來」。

```typescript
function singleNumber(nums: number[]): number {
    let ones = 0, twos = 0;
    for (const n of nums) {
        ones = (ones ^ n) & ~twos;  // XOR 切換，& ~twos = twos 佔了的我不碰
        twos = (twos ^ n) & ~ones;  // XOR 切換，& ~ones = ones 佔了的我不碰
    }
    return ones;
}
```

- Time: $O(n)$
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func singleNumber(nums []int) int {
    ones, twos := 0, 0
    for _, n := range nums {
        ones = (ones ^ n) & ^twos  // XOR 切換，& ^twos = twos 佔了的我不碰
        twos = (twos ^ n) & ^ones  // XOR 切換，& ^ones = ones 佔了的我不碰
    }                               // 出現 3 次兩邊都清掉，ones 剩的就是答案
    return ones
}
```

</details>

**兩行 code 對著狀態表驗**。只看 n 這一位是 1 的情況；是 0 時 XOR 不動、篩子照放行，狀態自然不變。`ones = (ones ^ n) & ~twos` 這行，精髓是第 3 次出現：

```
(0,1)：ones 翻成 1，被 ~twos 壓回 0   → ones = 0   ✓ 第 3 次，ones 必須留 0
```

沒有 `& ~twos` 的話，第 3 次出現時 ones 會錯誤地變回 1。另一個要點：`twos = (twos ^ n) & ~ones` 用的是**剛更新完的 ones**，兩行順序有意義。其餘的轉移就不逐條列了，下面的 trace 會完整走一遍。

**然後才是 32 bit**。位元運算的每一位互不干擾，所以 `ones`、`twos` 兩個 int 實際上是 32 個獨立的 mod-3 計數器並排跑。出現三次的數字，每個 bit 都數到 3 的倍數、歸零；落單的數字每個 bit 只數到 1，全部留在 ones 的對應位置。所以最後 `return ones`，它就是答案本身。

**走一遍 [5, 5, 5, 3]**：

```
5 = 0101, 3 = 0011
```

先講 `~` 換算出來的東西：`~` 是每個 bit 翻面，而且翻的是全部 32 個 bit，這裡縮寫成 4 個。所以 `~0000 = 1111`（全 1，讀成 int 就是 -1）、`~0101 = 1010`。AND 上全 1 等於每個 bit 都放行；`~` 真正擋東西是在原本有 1 的位置。

第一個 5，第 1 次出現，記在 ones：

```
ones = (0000 ^ 0101) & ~0000
     =  0101         &  1111     ← XOR 翻出 0101；~0000 全放行
     =  0101

twos = (0000 ^ 0101) & ~0101     ← 用的是剛更新完的 ones = 0101
     =  0101         &  1010     ← ones 佔了的位置被壓掉
     =  0000

ones = 0101  ← 5 出現 1 次
twos = 0000
```

第二個 5，第 2 次出現，從 ones 移到 twos：

```
ones = (0101 ^ 0101) & ~0000
     =  0000         &  1111     ← XOR 同數字互相抵銷，歸 0
     =  0000

twos = (0000 ^ 0101) & ~0000     ← ones 已經是 0000，不擋
     =  0101         &  1111
     =  0101

ones = 0000
twos = 0101  ← 5 出現 2 次
```

第三個 5，第 3 次出現，兩邊都清掉：

```
ones = (0000 ^ 0101) & ~0101     ← twos = 0101 還在佔位
     =  0101         &  1010     ← XOR 想翻成 1，被 ~twos 壓回 0
     =  0000

twos = (0101 ^ 0101) & ~0000
     =  0000         &  1111     ← XOR 自己抵銷自己
     =  0000

ones = 0000  ← 歸零
twos = 0000  ← 歸零
```

5 出現 3 次，完全消失。

3 進來，第 1 次出現，記在 ones：

```
ones = (0000 ^ 0011) & ~0000
     =  0011         &  1111
     =  0011 = 3  ← 答案
```

四步都對得上開頭那張狀態表。

跟逐 bit `% 3` 做的事完全一樣，只是 % 3 內建在三態循環裡，32 個 bit 同時數，不需要外層的 `for i := 0; i < 32` 迴圈。

這解法巧妙，但面試很難現場推出來。逐 bit 統計的做法更直覺，而且可以推廣到「出現 k 次」——把 `% 3` 換成 `% k` 就好。兩種都是 $O(n)$ 時間 $O(1)$ 空間。

---

## 結論

XOR 處理不了三次。逐 bit 統計，mod 3，$O(n)$ 時間 $O(1)$ 空間。這個思路可以推廣到任何「出現 k 次只有一個出現 1 次」的變形。
