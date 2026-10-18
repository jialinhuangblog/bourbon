跟 [46 Permutations](/problem/permutations) 同一件事，只有一個地方不一樣：牌可能重複。

桌上三張牌是 1、1、2。照 46 的做法排，會拿到六個答案：

```
1 1 2      ← 先拿左邊那張 1
1 2 1
1 1 2      ← 先拿右邊那張 1，排出來一模一樣
1 2 1
2 1 1
2 1 1
```

兩張 1 長得一樣，誰先誰後看不出來，所以每個答案都出現兩次。題目要的是三個：`[1,1,2]`、`[1,2,1]`、`[2,1,1]`。

翻回 code 術語：給一個可能有重複元素的陣列，回傳所有**不重複**的排列。

寫起來，47 就是 46 的 code 加兩行：開頭先 `sort`，迴圈裡多一條 `continue`。其餘每一行都一樣。那條 `continue` 的條件是這題唯一要想的地方。

---

**解題引導**

用 `[1,1,2]` 想。

**Step 1：最省事的做法是什麼？**

*generate everything, then throw away what you've seen*

<span class="spoiler">照 46 的解法全部生出來，再用一個 Set 把重複的濾掉。會動，但六條路走完只留三條。Generate all n! then filter with a set.</span>

**Step 2：能不能在走進去之前就知道這條路是重複的？**

*two identical cards at the same slot*

<span class="spoiler">同一格如果先試左邊那張 1、又試右邊那張 1，第二次一定跟第一次撞。同一層碰到相同的值，第二次以後直接跳過。Skip a duplicate value at the same depth.</span>

**Step 3：怎麼知道兩張牌「值一樣」而且「在同一層」？**

*sort first, then look at your left neighbour*

<span class="spoiler">先排序讓相同的值排在一起，這樣只要跟左邊鄰居比就好。再看左邊那個有沒有被用掉，判斷是同一層還是上下層。Sort, then compare with the previous index and check its used flag.</span>

想完再往下看 code。

---

## 解法一：照 46 的做法全生，再用 Set 濾掉重複的

`used` 那套原封不動搬過來，收集答案的時候先看有沒有見過。

```typescript
function permuteUnique(nums: number[]): number[][] {
    const result: number[][] = [];
    const used = new Array(nums.length).fill(false);
    const seen = new Set<string>();

    function dfs(current: number[]) {
        if (current.length === nums.length) {
            const key = current.join(',');
            if (!seen.has(key)) {        // 沒見過才收
                seen.add(key);
                result.push([...current]);
            }
            return;
        }
        for (let i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            used[i] = true;
            dfs([...current, nums[i]]);
            used[i] = false;
        }
    }

    dfs([]);
    return result;
}
```

- Time: $O(n! \times n)$，$n!$ 條路徑全部走完，每條末端做一次長度 n 的字串比對
- Space: $O(n! \times n)$，`seen` 最壞要裝 $n!$ 個 key

<details>
<summary>Go 版本</summary>

```go
func permuteUnique(nums []int) [][]int {
    result := [][]int{}
    used := make([]bool, len(nums))
    seen := map[string]bool{}

    var dfs func(current []int)
    dfs = func(current []int) {
        if len(current) == len(nums) {
            key := fmt.Sprint(current)
            if !seen[key] {
                seen[key] = true
                result = append(result, append([]int{}, current...))
            }
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

**`[1,1,2]` 走完是六條路，三條被丟掉：**

```
1(左) 1(右) 2   → 收 [1,1,2]
1(左) 2 1(右)   → 收 [1,2,1]
1(右) 1(左) 2   → 撞，丟掉
1(右) 2 1(左)   → 撞，丟掉
2 1(左) 1(右)   → 收 [2,1,1]
2 1(右) 1(左)   → 撞，丟掉
```

$n \le 8$，元素全部相異時最壞：$8! = 40{,}320$ 條路徑，加上 55 萬次迴圈、77 萬次陣列複製、32 萬次的 `join` 加雜湊，總共約 $1.6 \times 10^6$ 次操作，**0.16 秒**。過得了。

代價在極端輸入：`[1,1,1,1,1,1,1,1]` 只有一個答案，但這個解法照樣走 40,320 條路徑，40,319 條都是白走的。

---

## 解法二：排序之後，同一層跳過重複的值

要在走進去**之前**就知道重複，得先讓相同的值相鄰，所以第一行就是排序。

```typescript
function permuteUnique(nums: number[]): number[][] {
    nums.sort((a, b) => a - b);          // 相同的值排在一起
    const result: number[][] = [];
    const used = new Array(nums.length).fill(false);

    function dfs(current: number[]) {
        if (current.length === nums.length) {
            result.push([...current]);
            return;
        }
        for (let i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            // 跟左邊鄰居值一樣，而且左邊那個這一層還沒被選 → 這是重複的分支
            if (i > 0 && nums[i] === nums[i - 1] && !used[i - 1]) continue;
            used[i] = true;
            dfs([...current, nums[i]]);
            used[i] = false;
        }
    }

    dfs([]);
    return result;
}
```

- Time: 最壞 $O(n! \times n)$（元素全部相異時退化成 46），有重複時遠低於此
- Space: $O(n)$，不必存看過的答案

<details>
<summary>Go 版本</summary>

```go
func permuteUnique(nums []int) [][]int {
    sort.Ints(nums)
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
            if i > 0 && nums[i] == nums[i-1] && !used[i-1] {
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

**`!used[i-1]` 到底在判斷什麼？**

兩張 1 分別是 index 0 跟 index 1。輪到 index 1 的時候，index 0 的狀態有兩種：

- `used[0] === true`：index 0 剛剛在**上一層**被選走了，現在是它的子孫在挑下一格。這時候選 index 1 是合法的，`[1(左), 1(右), 2]` 就是這樣來的。
- `used[0] === false`：index 0 沒被選，那就是**同一格**先試了 index 0、退回來、現在要試 index 1。兩次挑的值一樣，展開的子樹也會一模一樣。跳過。

所以 `!used[i-1]` 成立就跳過，它問的是「左邊那個一樣的值，剛才是不是已經在這一格試過了」。

**走一遍 `[1,1,2]`：**

排序後 `nums = [1, 1, 2]`，index 0 跟 1 都是 1。縮排的深度就是填到第幾格，每一層的三個分支就是迴圈的 i=0、1、2。

每個標「選」的節點都做三件事：`used[i]=true`、遞迴下去、回來 `used[i]=false`。括號裡寫的是那一步的判斷根據：

```
[]
├── i=0 值1 選 → [1]
│   ├── i=0 ✗ used[0]=T
│   ├── i=1 值1 選（used[0]=T，是上一層選走的）→ [1,1]
│   │   ├── i=0 ✗ used[0]=T
│   │   ├── i=1 ✗ used[1]=T
│   │   └── i=2 值2 選 → 收 [1,1,2]
│   └── i=2 值2 選 → [1,2]
│       ├── i=0 ✗ used[0]=T
│       ├── i=1 值1 選（used[0]=T）→ 收 [1,2,1]
│       └── i=2 ✗ used[2]=T
│   ↑ 這棵子樹跑完，dfs 回來執行 used[0]=false
├── i=1 值1 ✗ 重複（第一格剛才已經用 index 0 試過值 1，used[0]=F 就是它退回來的證明）
└── i=2 值2 選 → [2]
    ├── i=0 值1 選 → [2,1]
    │   ├── i=0 ✗ used[0]=T
    │   ├── i=1 值1 選（used[0]=T）→ 收 [2,1,1]
    │   └── i=2 ✗ used[2]=T
    ├── i=1 值1 ✗ 重複（used[0]=F）
    └── i=2 ✗ used[2]=T
```

`✗ used[i]=T` 是第一個 `continue` 擋的，這個 index 這條路上已經用掉了。`✗ 重複` 是第二個 `continue` 擋的，也就是去掉重複那條規則。答案 `[1,1,2]`、`[1,2,1]`、`[2,1,1]`，三個，沒有生出來又丟掉的。

第一格的 `i=1` 只擋掉一行，但它剪掉的是整棵子樹：那底下本來會長出 `[1,1,2]` 跟 `[1,2,1]` 各一份，就是解法一裡被丟掉的那幾條。

第一格的 `i=1` 那一步剪掉的是「先拿右邊那張 1」的整棵子樹，解法一裡被丟掉的三條路就住在那裡面。

**沒排序會怎樣？** `nums[i] === nums[i-1]` 這個檢查只看左邊一格。輸入是 `[1,2,1]` 而沒排序的話，兩個 1 不相鄰，第三格那個 1 永遠不會跟第一格的比到，重複就漏掉了。排序讓「相同的值」變成「連續的一段」，比左鄰居才夠。

**comparator 不能省。** JS 的 `sort()` 預設把元素轉成字串比字典序：

```
[10, 9, 1].sort()              → [1, 10, 9]
[10, 9, 1].sort((a, b) => a-b) → [1, 9, 10]
```

這題的值域是 -10 到 10，剛好有兩位數的 `10` 會踩到（字串比較時 `"10" < "9"`）。排錯的話「相同的值一定相鄰」這個前提就不成立，上面那條剪枝整個失效。

**`sort` 改的是原陣列。** `nums.sort()` 是 in-place，跑完之後呼叫方手上那個 `nums` 已經被排序過了。LeetCode 不管這件事，寫在專案裡會有人踩到，要避免就寫 `const sorted = [...nums].sort((a, b) => a - b)`。

---

**Overthinking**

**寫成 `used[i-1]`（沒有驚嘆號）會錯嗎？** 不會，答案一樣對，但剪得比較晚。實際跑過：

| 輸入 | `!used[i-1]` | `used[i-1]` |
|---|---:|---:|
| `[1,1,2]` | 9 次遞迴 | 12 次 |
| `[2,2,1,1]` | 19 次遞迴 | 33 次 |

`!used[i-1]` 在第一次遇到重複值的時候就把整棵子樹剪掉；`used[i-1]` 是等走進去之後才在下層擋，所以多跑了不少節點。

**全部元素都一樣呢？** `[1,1,1,1,1,1,1,1]` 只有一個答案。解法二每一格都只有 `i=0` 通過檢查，總共 9 次遞迴就結束。解法一要走 40,320 條。

**組合題也這樣處理嗎？** 是。[40 Combination Sum II](/problem/combination-sum-ii) 排除重複的方法也是排序加上「同一層跳過相同的值」，只是那題用 `start` 而不是 `used`，條件寫成 `i > start && candidates[i] === candidates[i-1]`。

---

## 解法比較表

| 解法 | Time | Space | n=8 全異的操作數 | 秒數 | 備註 |
|---|---|---|---|---|---|
| 全生 + Set 過濾 | $O(n! \times n)$ | $O(n! \times n)$ | 1,644,016 | 0.16 秒 | 好寫，但重複多的輸入白做很多 |
| 排序 + 同層剪枝 | 最壞 $O(n! \times n)$ | $O(n)$ | 1,644,016 | 0.16 秒 | 面試要的答案，重複愈多省愈多 |

元素全部相異時剪枝完全沒有作用，兩邊一樣慢。差別在有重複的輸入：`[1,1,1,1,1,1,1,1]` 解法一照樣走 40,320 條路徑，解法二只有 108 次操作就結束。

---

## 結論

46 的 `used` 管的是「這條路上用過沒」，47 多一條管「這一格試過相同的值沒」。前者防的是同一張牌用兩次，後者防的是兩張長得一樣的牌各排一次。排序是為了讓後者只要比左鄰居就夠。
