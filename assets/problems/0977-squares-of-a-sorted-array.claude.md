想像家在公路 0 公里處。一排人站在公路上、依里程排好——從很遠的西邊（負）到很遠的東邊（正）。問你「**每個人離家多遠的平方**」，依距離由小到大列出。

```
nums = [-7, -3, 2, 3, 11]
        ←西                東→
        家在 0
```

平方就是「把方向砍掉、只剩距離」：`(-7)² = 49`，`(11)² = 121`。題目要你產出依距離排好的清單。

---

**解題引導**

input：`nums = [-4, -1, 0, 3, 10]`，正解 `[0, 1, 9, 16, 100]`。

**Step 1：先想暴力，怎麼做？**

*just square everything, then sort*

<span class="spoiler">每個數平方，再 sort。Square each, then sort. O(N log N)。</span>

**Step 2：題目特別給「已排序」這條件，能拿來幹嘛？**

*the input is already sorted by value — what does that tell us about absolute values*

<span class="spoiler">「按值排好」代表絕對值最大的兩個必在兩端（負數最左、正數最右），絕對值最小的在中間（接近 0）。這個結構不用 sort 就能利用。Sorted-by-value means biggest absolute values are at the two ends, smallest in the middle. We can exploit this directly.</span>

**Step 3：在已排序陣列裡，平方最大的會出現在哪？**

*number line — where are the biggest squares*

<span class="spoiler">在兩端。負數最左邊絕對值最大、正數最右邊絕對值最大。平方最小的反而在中間（最接近 0）。Biggest squares at the two ends, smallest in the middle near zero.</span>

**Step 4：既然最大的在兩端，怎麼一次填一個進結果？**

*compare both ends' squared values, take the bigger one*

<span class="spoiler">兩根指針從左右兩端往中間走，直接比兩端的平方（平方完一定正數，不用先取絕對值），大的填到結果**最後一格**。然後那邊指針往內移一格。Two pointers from both ends, compare squared values directly (squaring already kills the sign), put the larger square at the BACK of result, then move that pointer inward.</span>

想完再往下看 code。

---

## 解法一：暴力 sort

平方完直接 sort：

```typescript
function sortedSquares(nums: number[]): number[] {
    return nums.map(n => n * n).sort((a, b) => a - b);
}
```

- **Time: O(N log N)** — sort 是瓶頸
- Space: O(N) — 結果陣列（sort 本身在多數語言是 O(log N) 額外空間）
- 實際秒數：$N=10^4$ 上限 → 約 $1.4 \times 10^5$ 操作 → **0.014 秒**

<details>
<summary>Go 版本</summary>

```go
func sortedSquares(nums []int) []int {
    res := make([]int, len(nums))
    for i, n := range nums {
        res[i] = n * n          // 先全部平方
    }
    sort.Ints(res)              // 再排
    return res
}
```

</details>

題目 follow-up 寫明「Could you find an O(N) solution?」這就是在點「**已排序這條件你浪費了**」。

---

## 解法二：雙指針

平方完之後**最大的一定在兩端**（負最大或正最大），**最小的在中間**（最接近 0）。

所以從兩端往中間掃，每次挑「絕對值較大」那個平方，**從結果陣列的尾巴往前填**。填完一個，那邊的指針往中間移一格。重複到兩根指針交會。

```typescript
function sortedSquares(nums: number[]): number[] {
    const n = nums.length;
    const res = new Array(n);
    let left = 0, right = n - 1;
    let pos = n - 1;                   // 從尾巴往前填

    while (left <= right) {
        const l = nums[left] * nums[left];
        const r = nums[right] * nums[right];
        if (l > r) {
            res[pos] = l;
            left++;
        } else {
            res[pos] = r;
            right--;
        }
        pos--;
    }
    return res;
}
```

- **Time: O(N)** — 每個元素只看一次
- Space: O(N) — 結果陣列（不算 output 的話 O(1)）
- 實際秒數：$N=10^4$ 上限 → $10^4$ 操作 → **0.001 秒**

<details>
<summary>Go 版本</summary>

```go
func sortedSquares(nums []int) []int {
    n := len(nums)
    res := make([]int, n)
    left, right := 0, n-1
    pos := n - 1                       // 從結果尾巴往前填

    for left <= right {
        l, r := nums[left]*nums[left], nums[right]*nums[right]
        if l > r {
            res[pos] = l               // 左邊絕對值大，填左平方
            left++
        } else {
            res[pos] = r               // 右邊大或相等，填右平方
            right--
        }
        pos--
    }
    return res
}
```

</details>

**為什麼從尾巴往前填**：因為每次挑出來的是「當下最大」。最大的應該在結果陣列**最後一格**，不是第一格。從前往後填會變成倒序，還要再 reverse 一次。

**為什麼 `left <= right` 不是 `left < right`**：當 `left == right` 時，那個元素還沒填進結果。要再做一輪把它填掉。

---

**Step by step**

`nums = [-7, -3, 2, 3, 11]`，n=5。

| 步 | left | right | pos | $nums[left]^2$ | $nums[right]^2$ | 填誰 | res |
|---|---|---|---|---|---|---|---|
| 初始 | 0 (-7) | 4 (11) | 4 | 49 | 121 | 121 → res[4] | [_, _, _, _, 121] |
| 1 | 0 (-7) | 3 (3) | 3 | 49 | 9 | 49 → res[3] | [_, _, _, 49, 121] |
| 2 | 1 (-3) | 3 (3) | 2 | 9 | 9 | 9 → res[2]（取右） | [_, _, 9, 49, 121] |
| 3 | 1 (-3) | 2 (2) | 1 | 9 | 4 | 9 → res[1] | [_, 9, 9, 49, 121] |
| 4 | 2 (2) | 2 (2) | 0 | 4 | 4 | 4 → res[0]（取右） | [4, 9, 9, 49, 121] |
| 結束 | 2 | 1 | -1 | left > right，跳出 | | | |

最後一輪 `left == right == 2`，把 4 填掉，然後 right 變 1，迴圈結束。

---

**能不能更好？**

不能。

- O(N) 已經是下限：每個元素至少要平方一次
- 不需要額外空間（除了 output 本身）

題目 follow-up 就是 two pointer，**沒有 O(log N) 或更快的解**——你要看每個元素一次。

---

**Overthinking**

**變形 1：input 全是正數或全是負數**

全正：直接平方就排好了（順序不變）。
全負：平方後變倒序，reverse 一次。
Two pointer 不用判斷這些 edge case，照跑會自動正確（兩端比較，平凡退化成從一端推進）。

**變形 2：要回傳「絕對值排序」（不平方）**

把 `nums[left]*nums[left]` 換成 `abs(nums[left])`，邏輯一樣。Two pointer 在這類「**已排序但 zero 是分界點**」的題型都通用。

**變形 3：input 沒有排序**

就退化回 O(N log N)——先 sort 再平方。「已排序」是這題能 O(N) 的關鍵條件。

---

## 解法比較表

| 解法 | Time | Space | $N=10^4$ 秒數 | 備註 |
|---|---|---|---|---|
| 暴力 sort | O(N log N) | O(1) | 0.014 秒 | 寫得快、面試夠用 |
| Two pointer | **O(N)** | O(1) | **0.001 秒** | follow-up 想看的解 |

兩者實際跑都瞬間，差別在「**你能不能利用『已排序』這條件**」。面試考的是後者。

---

## 結論

暴力 sort 是 O(N log N)。但「已排序」+「平方最大值在兩端」這兩個條件配起來，two pointer 從兩邊往中間掃，**從結果尾巴往前填**就能 O(N)。記法：「**比兩端平方、大的擺後面、移那邊指針**」。
