桌上三張牌，寫著 1、2、3。把它們排成一列，有幾種排法？

第一格三選一，選完第二格剩兩張，第三格剩一張。3 × 2 × 1 = 6 種：

```
1 2 3
1 3 2
2 1 3
2 3 1
3 1 2
3 2 1
```

翻回 code 術語：給一個元素互不相同的陣列，回傳全部的排列。順序不拘。

---

**解題引導**

用 `[1,2,3]` 想。

**Step 1：第一格選了 1 之後，剩下的問題長什麼樣？**

*pick one, then the same problem on what's left*

<span class="spoiler">剩下 2 跟 3 要排，是同一個問題的縮小版。同一個問題變小，就用遞迴。Picking one leaves the same problem on a smaller set.</span>

**Step 2：怎麼知道哪些牌已經被拿走了？**

*mark them, but mark what exactly*

<span class="spoiler">用一個布林陣列記 index 有沒有被用過。記 index 不記值，因為值可能重複。A boolean array indexed by position, not by value.</span>

**Step 3：一條路走完之後，那些標記要怎麼辦？**

*the next branch needs a clean slate*

<span class="spoiler">要還原。這條路用掉的牌，下一條路還要用。不還原的話第二條路就沒牌可選了。Undo the mark so the next branch can use it.</span>

**Step 4：不做任何標記會怎樣？**

<span class="spoiler">每一格都從三張裡挑，跑出 3 的 3 次方等於 27 種，包含 1 1 1 這種同一張牌用三次的。That is "with replacement", not a permutation.</span>

想完再往下看 code。

---

## 解法一：全部組合都生出來，再篩掉不合格的

每一格都從 n 個數字裡挑一個，先生出 $n^n$ 種，再把「有數字重複出現」的丟掉。

```typescript
function permute(nums: number[]): number[][] {
    const n = nums.length;
    const result: number[][] = [];

    function dfs(current: number[]) {
        if (current.length === n) {
            if (new Set(current).size === n) {   // n 個都不一樣才是排列
                result.push([...current]);
            }
            return;
        }
        for (const v of nums) {
            dfs([...current, v]);                // 每一格都試全部 n 個
        }
    }

    dfs([]);
    return result;
}
```

- Time: $O(n^n \times n)$，生 $n^n$ 條，每條末端還要檢查一次有沒有重複
- Space: $O(n)$ 遞迴深度（不算 result）

<details>
<summary>Go 版本</summary>

```go
func permute(nums []int) [][]int {
    n := len(nums)
    result := [][]int{}

    var dfs func(current []int)
    dfs = func(current []int) {
        if len(current) == n {
            seen := map[int]bool{}
            for _, v := range current {
                seen[v] = true
            }
            if len(seen) == n {
                result = append(result, append([]int{}, current...))
            }
            return
        }
        for _, v := range nums {
            dfs(append(append([]int{}, current...), v))
        }
    }

    dfs([]int{})
    return result
}
```

</details>

**走一遍。** `[1,2,3]` 的決策樹每一層都是滿的三個分支：

```
[]
├── 1
│   ├── 1,1
│   │   ├── 1,1,1   ✗ 有重複
│   │   ├── 1,1,2   ✗
│   │   └── 1,1,3   ✗
│   ├── 1,2
│   │   ├── 1,2,1   ✗
│   │   ├── 1,2,2   ✗
│   │   └── 1,2,3   ✓
│   └── 1,3
│       ├── 1,3,1   ✗
│       ├── 1,3,2   ✓
│       └── 1,3,3   ✗
├── 2  （同樣三層，留下 2,1,3 跟 2,3,1）
└── 3  （留下 3,1,2 跟 3,2,1）
```

27 條路徑只有 6 條活下來，其他 21 條走到最後才被丟掉。

$n \le 6$。$6^6 = 46{,}656$ 條路徑，末端檢查 28 萬次，加上每層的迴圈 5.6 萬次跟每次遞迴複製 `current` 的 32 萬次，總共約 $6.6 \times 10^5$ 次操作，除以 $10^7$ 大約 **0.07 秒**。過得了，但 21/27 的工作是白做的。

**能不能不要走到最後才發現重複？** 走到 `1,1` 的時候就已經不可能是排列了，那時候就該停，不必再往下鑽。

---

## 解法二：用一個陣列記誰被拿走了

在挑之前就檢查，不要等走到底。`used[i]` 記 index i 的數字有沒有在當前這條路徑上。

```typescript
function permute(nums: number[]): number[][] {
    const result: number[][] = [];
    const used = new Array(nums.length).fill(false);

    function dfs(current: number[]) {
        if (current.length === nums.length) {
            result.push([...current]);
            return;
        }
        for (let i = 0; i < nums.length; i++) {
            if (used[i]) continue;       // 這條路上拿過了，跳過
            used[i] = true;
            dfs([...current, nums[i]]);
            used[i] = false;             // 還原，這張牌要留給下一條路
        }
    }

    dfs([]);
    return result;
}
```

- Time: $O(n! \times n)$，$n!$ 條路徑，每條複製一份長度 n 的答案
- Space: $O(n)$

<details>
<summary>Go 版本</summary>

```go
func permute(nums []int) [][]int {
    result := [][]int{}
    used := make([]bool, len(nums))

    var dfs func(current []int)
    dfs = func(current []int) {
        if len(current) == len(nums) {
            result = append(result, append([]int{}, current...))
            return
        }
        for i, v := range nums {
            if used[i] {
                continue
            }
            used[i] = true
            dfs(append(append([]int{}, current...), v))
            used[i] = false
        }
    }

    dfs([]int{})
    return result
}
```

</details>

**決策樹縮成這樣：**

```
[]
├── 1
│   ├── 1,2 → 1,2,3
│   └── 1,3 → 1,3,2
├── 2
│   ├── 2,1 → 2,1,3
│   └── 2,3 → 2,3,1
└── 3
    ├── 3,1 → 3,1,2
    └── 3,2 → 3,2,1
```

第二層每個節點只剩兩個分支，第三層只剩一個。6 條路徑，沒有一條是白走的。

**兩行 `used` 各管一件事。** `used[i] = true` 讓下一層跳過這個 index。少了它，`used` 永遠全 false，就退回解法一那 27 條，而且沒有末端檢查，會直接輸出 `[1,1,1]`。

`used[i] = false` 是 backtrack。少了它只會得到一個答案 `[1,2,3]`：第一條路走完 `used` 全是 true，其他分支全被擋在 `continue`。

**為什麼不用 `current.includes(nums[i])` 就好？** 兩個理由。一是 `includes` 每次 $O(k)$，`used[i]` 是 $O(1)$。二是比較嚴重的：`includes` 比的是**值**，碰到 `[1,1,2]` 這種有重複元素的輸入，第二個 1 會被第一個 1 擋住，答案直接變成空陣列。這題的 constraints 保證元素互異所以看不出來，換到 [47 Permutations II](/problem/permutations-ii) 就會壞。

$6! = 720$ 條路徑，加上 1,957 個遞迴節點的迴圈跟一路上的陣列複製，總共約 $2.2 \times 10^4$ 次操作，**0.002 秒**。

---

## 解法三：不開額外陣列，直接在原陣列上換位置

`used` 陣列可以省掉。換個角度：與其「挑一個沒用過的」，不如「把某個數字換到當前這一格」。

`start` 左邊的格子全部決定好了，`start` 這一格還沒。從 `start` 到結尾之間挑一個換過來，就是這一格的選擇。

```typescript
function permute(nums: number[]): number[][] {
    const result: number[][] = [];

    function dfs(start: number) {
        if (start === nums.length) {
            result.push([...nums]);
            return;
        }
        for (let i = start; i < nums.length; i++) {
            [nums[start], nums[i]] = [nums[i], nums[start]];   // 把 i 換到 start 這格
            dfs(start + 1);
            [nums[start], nums[i]] = [nums[i], nums[start]];   // 換回來
        }
    }

    dfs(0);
    return result;
}
```

- Time: $O(n! \times n)$
- Space: $O(n)$ 遞迴深度，沒有額外的 `used` 陣列

<details>
<summary>Go 版本</summary>

```go
func permute(nums []int) [][]int {
    result := [][]int{}

    var dfs func(start int)
    dfs = func(start int) {
        if start == len(nums) {
            result = append(result, append([]int{}, nums...))
            return
        }
        for i := start; i < len(nums); i++ {
            nums[start], nums[i] = nums[i], nums[start]
            dfs(start + 1)
            nums[start], nums[i] = nums[i], nums[start]
        }
    }

    dfs(0)
    return result
}
```

</details>

**走一遍。** `nums = [1,2,3]`：

```
start=0, i=0   換自己，nums = [1,2,3]
  start=1, i=1   換自己，nums = [1,2,3]
    start=2       到底 → 收 [1,2,3]
  start=1, i=2   換 2 跟 3，nums = [1,3,2]
    start=2       到底 → 收 [1,3,2]
  換回來，nums = [1,2,3]
換回來，nums = [1,2,3]

start=0, i=1   換 1 跟 2，nums = [2,1,3]
  start=1, i=1   nums = [2,1,3] → 收 [2,1,3]
  start=1, i=2   換 1 跟 3，nums = [2,3,1] → 收 [2,3,1]
  換回來，nums = [2,1,3]
換回來，nums = [1,2,3]

start=0, i=2   換 1 跟 3，nums = [3,2,1]
  start=1, i=1   nums = [3,2,1] → 收 [3,2,1]
  start=1, i=2   換 2 跟 1，nums = [3,1,2] → 收 [3,1,2]
  換回來，nums = [3,2,1]
換回來，nums = [1,2,3]
```

輸出是 `[1,2,3] [1,3,2] [2,1,3] [2,3,1] [3,2,1] [3,1,2]`，跟解法二比最後兩個對調。題目說順序不拘，沒差。

**`i` 從 `start` 開始不是從 0。** 從 0 開始會動到左邊已經決定好的格子，前面選好的就被蓋掉了。

**兩次交換一定要成對。** 第二次把陣列換回進來時的樣子，下一輪的 `i` 才是在同一個基準上做選擇。少了它，`nums` 會愈換愈亂，輸出會有重複也有缺漏。

---

**Overthinking**

**n 可以多大？** 答案有 $n!$ 個，光是裝進記憶體就受不了。n = 10 有 363 萬個排列，n = 13 有 62 億個。這類題目的 constraints 一定很小（這題是 6），因為輸出本身就是瓶頸，跟演算法多快無關。

**要第 k 個排列，不要全部？** 那是 60 Permutation Sequence，用 $(n-1)!$ 直接算出每一位該填誰，不必把前面 k-1 個都生出來。

**元素可能重複呢？** [47 Permutations II](/problem/permutations-ii)。解法二的 `used` 還能用，但要多一條規則處理「同一層不要選到相同的值」。

---

## 解法比較表

| 解法 | Time | Space | n=6 的操作數 | 秒數 | 備註 |
|---|---|---|---|---|---|
| 全部生完再篩 | $O(n^n \times n)$ | $O(n)$ | 660,648 | 0.07 秒 | 27 條走完只留 6 條 |
| used 陣列 | $O(n! \times n)$ | $O(n)$ | 21,528 | 0.002 秒 | 標準解，好推廣到 47 |
| swap in-place | $O(n! \times n)$ | $O(n)$ | 10,188 | 0.001 秒 | 省掉 used，推廣到 47 比較麻煩 |

操作數是實際跑出來的，含每層迴圈、陣列複製、末端收集。swap 版比 used 版少一半，因為它在原陣列上動，不用每次遞迴都複製一份 `current`。

n 只到 6，三個都不會 TLE。選 used 陣列不是因為它比較快，是因為它最容易套到別的 backtracking 題。

---

## 結論

排列題的骨架是三行：做選擇、遞迴、撤銷選擇。`used[i] = true` 是做選擇，`used[i] = false` 是撤銷，中間那行是遞迴。難的不是想到遞迴，是記得把狀態還原。
