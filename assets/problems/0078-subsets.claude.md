冰箱裡有三種配料：鮪魚、玉米、蛋。做一份沙拉，每種各自決定加或不加。

問總共有幾種做法。什麼都不加也算一種，那就是一盤生菜。

```
nums = [1, 2, 3]

[]          什麼都不加
[1]         只加 1
[2]
[1,2]
[3]
[1,3]
[2,3]
[1,2,3]     全加

八種
```

`[1,2]` 跟 `[2,1]` 算同一種，配料放進去的先後不影響那盤沙拉。

翻成 code 的講法：給一個元素互不相同的陣列，回傳所有子集合（power set），順序不拘。

---

**解題引導**

拿 `[1, 2, 3]` 想。

**Step 1：每個元素面對的是什麼決定？**

*in or out. two options, independently.*

<span class="spoiler">加或不加，兩個選項，而且每個元素各自決定，互不影響。In or out, independently for each element.</span>

**Step 2：所以總共有幾種？**

<span class="spoiler">2 × 2 × 2 = 2³ = 8 種。n 個元素就是 2ⁿ 種。Two choices per element, multiplied.</span>

**Step 3：既然答案有 2ⁿ 個，複雜度最好能到多少？**

<span class="spoiler">光是把答案印出來就要 2ⁿ 次，跑不掉。而且每個子集合平均長度 n/2，總輸出量是 n × 2ⁿ / 2。這是下限，不是某個解法不夠好。The output alone is exponential, so nothing can be faster.</span>

**Step 4：怎麼確保不會產生重複的子集合？**

*never look back at an element you already passed.*

<span class="spoiler">遞迴的時候規定只能往右挑，決定過的元素不再回頭。這樣每個子集合都只會照原本的順序被產生一次。Only ever pick elements to the right of where you are.</span>

想完再往下看 code。

---

## 解法一：Backtracking，用 start index 控制方向

`cur` 是目前手上這盤沙拉。走到任何一個位置，`cur` 都已經是一個合法答案，先收下來，再繼續往右挑要不要加東西。

```typescript
function subsets(nums: number[]): number[][] {
    const result: number[][] = [];
    const cur: number[] = [];

    function backtrack(start: number): void {
        result.push([...cur]);            // 現在手上這盤就是一個答案

        for (let i = start; i < nums.length; i++) {
            cur.push(nums[i]);            // 加進去
            backtrack(i + 1);             // 只能往右挑，不回頭
            cur.pop();                    // 拿出來，換下一個試
        }
    }

    backtrack(0);
    return result;
}
```

- Time: $O(n \times 2^n)$ — 2ⁿ 個子集合，每個要複製一份
- Space: $O(n)$ — 不算輸出的話，只有 `cur` 跟 call stack

**走一遍。** `[1, 2, 3]`，縮排代表遞迴深度：

```
backtrack(0)   收 []
  i=0 push 1 → cur=[1]
    backtrack(1)   收 [1]
      i=1 push 2 → cur=[1,2]
        backtrack(2)   收 [1,2]
          i=2 push 3 → cur=[1,2,3]
            backtrack(3)   收 [1,2,3]
          pop 3 → cur=[1,2]
      pop 2 → cur=[1]
      i=2 push 3 → cur=[1,3]
        backtrack(3)   收 [1,3]
      pop 3 → cur=[1]
  pop 1 → cur=[]
  i=1 push 2 → cur=[2]
    backtrack(2)   收 [2]
      i=2 push 3 → cur=[2,3]
        backtrack(3)   收 [2,3]
      pop 3 → cur=[2]
  pop 2 → cur=[]
  i=2 push 3 → cur=[3]
    backtrack(3)   收 [3]
  pop 3 → cur=[]
```

收到的順序是 `[]、[1]、[1,2]、[1,2,3]、[1,3]、[2]、[2,3]、[3]`，八個。

每個 `push` 都配一個 `pop`，缺一個的話 `cur` 會愈長愈長，後面的子集合全部帶著前面的殘留。

**`backtrack(i + 1)` 那個 +1 是防重複的地方。** 傳 `i` 而不是 `i + 1` 的話，同一個元素可以再被挑一次，會產生 `[1,1]`、`[1,1,1]`。那是 [39 Combination Sum](/problem/combination-sum) 要的行為，因為那題允許同一個數字重複使用。

**收答案的位置在迴圈之前，沒有終止條件。** 其他 backtracking 題通常長成「走到底才收一次」，例如 39 要湊到目標才收。這題每一個中途狀態都是合法答案，所以進函式就收，也不需要 `if` 判斷什麼時候該停。迴圈自然走完就是終止。

<details>
<summary>Go 版本</summary>

```go
func subsets(nums []int) [][]int {
    result := [][]int{}
    cur := []int{}

    var backtrack func(start int)
    backtrack = func(start int) {
        subset := make([]int, len(cur))
        copy(subset, cur) // 一定要複製，不然 result 裡全指到同一個 slice
        result = append(result, subset)

        for i := start; i < len(nums); i++ {
            cur = append(cur, nums[i])
            backtrack(i + 1) // 只能往右挑，不回頭
            cur = cur[:len(cur)-1]
        }
    }

    backtrack(0)
    return result
}
```

Go 這裡的 `copy` 不能省。`append` 可能會共用底層陣列，直接 `append(result, cur)` 的話後面的修改會回頭改到已經收好的答案。

</details>

---

## 解法二：迭代，看到新元素就把現有答案複製一份

換個角度：已經有 `[1,2]` 的全部子集合了，現在多一個 3。新的子集合分兩群，一群不含 3（就是原本那些），一群含 3（原本那些各加一個 3）。

```typescript
function subsets(nums: number[]): number[][] {
    let result: number[][] = [[]];        // 從「什麼都不加」開始

    for (const num of nums) {
        const size = result.length;       // 先記住，不然迴圈會吃到自己新增的
        for (let i = 0; i < size; i++) {
            result.push([...result[i], num]);   // 舊的複製一份，加上 num
        }
    }

    return result;
}
```

- Time: $O(n \times 2^n)$
- Space: $O(1)$ 不算輸出

**走一遍。** `[1, 2, 3]`：

| 處理到 | 開始時 result | 新增的 | 結束時 result |
|---|---|---|---|
| 起始 | `[[]]` | | `[[]]` |
| 1 | `[[]]` | `[1]` | `[[], [1]]` |
| 2 | `[[], [1]]` | `[2]`、`[1,2]` | `[[], [1], [2], [1,2]]` |
| 3 | 上面四個 | `[3]`、`[1,3]`、`[2,3]`、`[1,2,3]` | 八個 |

每一輪答案數量剛好翻倍，1 → 2 → 4 → 8，Step 2 說的 2ⁿ 就是這樣長出來的。

`const size = result.length` 那行要先存起來。直接寫 `i < result.length` 的話，迴圈裡新增的元素會被同一輪迴圈讀到，然後無限長下去。

<details>
<summary>Go 版本</summary>

```go
func subsets(nums []int) [][]int {
    result := [][]int{{}} // 從「什麼都不加」開始

    for _, num := range nums {
        size := len(result) // 先記住，不然迴圈會吃到自己新增的
        for i := 0; i < size; i++ {
            next := make([]int, len(result[i]), len(result[i])+1)
            copy(next, result[i])
            next = append(next, num)
            result = append(result, next)
        }
    }

    return result
}
```

</details>

---

## 解法三：Bitmask，把選或不選寫成二進位

Step 1 說每個元素都是「加或不加」的二選一。n 個二選一，剛好就是一個 n 位元的二進位數。

```typescript
function subsets(nums: number[]): number[][] {
    const n = nums.length;
    const result: number[][] = [];

    for (let mask = 0; mask < (1 << n); mask++) {   // 0 到 2ⁿ - 1
        const subset: number[] = [];
        for (let i = 0; i < n; i++) {
            if (mask & (1 << i)) subset.push(nums[i]);   // 第 i 位是 1 就加
        }
        result.push(subset);
    }

    return result;
}
```

- Time: $O(n \times 2^n)$ — 2ⁿ 個 mask，每個檢查 n 個位元
- Space: $O(1)$ 不算輸出

**走一遍。** `[1, 2, 3]`，八個 mask：

| mask | 二進位 | 哪幾位是 1 | 子集合 |
|---|---|---|---|
| 0 | 000 | 沒有 | `[]` |
| 1 | 001 | 第 0 位 | `[1]` |
| 2 | 010 | 第 1 位 | `[2]` |
| 3 | 011 | 第 0、1 位 | `[1,2]` |
| 4 | 100 | 第 2 位 | `[3]` |
| 5 | 101 | 第 0、2 位 | `[1,3]` |
| 6 | 110 | 第 1、2 位 | `[2,3]` |
| 7 | 111 | 全部 | `[1,2,3]` |

`1 << n` 是 2ⁿ，`1 << i` 是「只有第 i 位是 1」的那個數。`mask & (1 << i)` 不是 0，代表第 i 位被選中了。

這一版產生的順序跟解法二一模一樣，也就是題目 Example 1 印出來的那個順序。原因是解法二每加一個元素就把答案翻倍，而翻倍的動作跟二進位往上進一位是同一件事。

`n <= 10` 的時候 `1 << n` 最大 1024，安全。要是 n 到 31 以上就會溢位，那時候 2ⁿ 個答案本來也印不完，所以這個限制不算問題。

<details>
<summary>Go 版本</summary>

```go
func subsets(nums []int) [][]int {
    n := len(nums)
    result := [][]int{}

    for mask := 0; mask < (1 << n); mask++ { // 0 到 2ⁿ - 1
        subset := []int{}
        for i := 0; i < n; i++ {
            if mask&(1<<i) != 0 {
                subset = append(subset, nums[i]) // 第 i 位是 1 就加
            }
        }
        result = append(result, subset)
    }

    return result
}
```

</details>

---

**Overthinking**

**跟其他 backtracking 題差在哪。** 同樣是 `push` 加遞迴加 `pop`，四題的差別全在遞迴那一行跟收答案的時機：

| 題目 | 遞迴傳什麼 | 什麼時候收答案 | 為什麼 |
|---|---|---|---|
| 78 Subsets | `i + 1` | 每進一次函式就收 | 每個中途狀態都是合法子集合 |
| [39 Combination Sum](/problem/combination-sum) | `i`（同一格可以再選） | 湊到目標才收 | 同一個數字允許重複使用 |
| [46 Permutations](/problem/permutations) | 從頭掃，用 `used` 標記 | `cur` 長度等於 n 才收 | 順序有差，每個位置都要試過所有沒用過的 |
| [47 Permutations II](/problem/permutations-ii) | 同 46，排序後同層跳過相同值 | 同 46 | 有重複元素，要擋掉同一層挑到相同值 |

78 是這一族裡最單純的：不回頭、不用標記、不用判斷終止。先把它寫熟，其他三題就是在這個骨架上加條件。

**元素有重複的話呢。** 那是 90 Subsets II。做法跟 47 一樣：先排序，然後在同一層迴圈裡跳過跟前一個相同的值。這題保證元素互不相同，所以不用處理。

**三個解法怎麼選。** 面試講 backtracking，因為它能延伸到 39、46、90 那一整族，講完可以順著答下一題。bitmask 只在「每個元素獨立二選一」這種結構成立，換成排列或有重複元素就套不上去。迭代版最短，適合已經知道答案再回來寫。

---

## 解法比較表

| 解法 | Time | Space（不含輸出） | n=10 操作次數 | 備註 |
|---|---|---|---|---|
| Backtracking | $O(n \times 2^n)$ | $O(n)$ | 1,024 次呼叫加 5,120 次元素複製 | 面試講這個，能延伸到整族題 |
| 迭代擴張 | $O(n \times 2^n)$ | $O(1)$ | 5,120 次元素複製 | 六行，最短 |
| Bitmask | $O(n \times 2^n)$ | $O(1)$ | 10,240 次位元檢查 | 順序跟題目範例一致 |

次數是實際跑計數器數出來的，三個都在 0.001 秒等級。複雜度一樣，因為 $2^n$ 個答案本身就是下限。bitmask 的次數看起來最多，是因為它對每個 mask 都要檢查全部 10 個位元，包括沒被選中的那些；另外兩個只碰真正要放進答案的元素。

---

## 結論

每個元素加或不加，2ⁿ 種組合。Backtracking 傳 `i + 1` 保證只往右挑，就不會產生重複；每進一次函式就收一個答案，因為中途狀態本身就合法。
