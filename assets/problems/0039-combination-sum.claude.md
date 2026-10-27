找所有加起來等於 target 的組合，同一個數可以重複用。

---

## 解法：Backtracking

這是 Backtracking——DFS 往下選數字，超過 target 就撤回，等於 target 就收集，繼續試其他路徑。

<details>
<summary>提示</summary>

每次從 `start` 開始選，選完繼續往右或選同一個（允許重複）。加總超過 target 就回頭。

</details>

**逐步追蹤**

`candidates = [2,3,6,7]`, `target = 7`：

```
dfs(start=0, path=[], remain=7)
  選 2 → dfs(start=0, path=[2], remain=5)
    選 2 → dfs(start=0, path=[2,2], remain=3)
      選 2 → dfs(start=0, path=[2,2,2], remain=1)
        2、3、6、7 都 > 1，全部跳過 → 無路，return
      撤回 2
      選 3 → dfs(start=1, path=[2,2,3], remain=0)
        remain==0 → 收集 [2,2,3] ✓
      撤回 3
      6 > 3、7 > 3，跳過
    撤回 2
    選 3 → dfs(start=1, path=[2,3], remain=2)
      3、6、7 都 > 2，全部跳過 → 無路，return
    撤回 3
    6 > 5、7 > 5，跳過
  撤回 2
  選 3 → dfs(start=1, path=[3], remain=4)
    選 3 → dfs(start=1, path=[3,3], remain=1)
      3、6、7 都 > 1，全部跳過 → 無路，return
    撤回 3
    6 > 4、7 > 4，跳過
  撤回 3
  選 6 → dfs(start=2, path=[6], remain=1)
    6 > 1、7 > 1，跳過 → 無路，return
  撤回 6
  選 7 → dfs(start=3, path=[7], remain=0)
    remain==0 → 收集 [7] ✓
  撤回 7

結果：[[2,2,3], [7]]
```

remain 從頭到尾沒有變成負數，因為剪枝 `candidates[i] > remain` 在 push 之前就把太大的數跳掉了，不會白跑一層遞迴進去再發現超過。

```typescript
function combinationSum(candidates: number[], target: number): number[][] {
    const res: number[][] = [];

    function dfs(start: number, remain: number, path: number[]): void {
        if (remain === 0) {
            res.push([...path]);
            return;
        }
        for (let i = start; i < candidates.length; i++) {
            if (candidates[i] > remain) continue;
            path.push(candidates[i]);
            dfs(i, remain - candidates[i], path); // i 不是 i+1
            path.pop();                            // 撤回
        }
    }

    dfs(0, target, []);
    return res;
}
```

<details>
<summary>Go 版本</summary>

```go
func combinationSum(candidates []int, target int) [][]int {
    var res [][]int
    var dfs func(start, remain int, path []int)

    dfs = func(start, remain int, path []int) {
        if remain == 0 {
            tmp := make([]int, len(path))
            copy(tmp, path)     // 收集當前路徑
            res = append(res, tmp)
            return
        }
        for i := start; i < len(candidates); i++ {
            if candidates[i] > remain {
                continue        // 這個數太大，跳過
            }
            path = append(path, candidates[i])
            dfs(i, remain-candidates[i], path) // i 不是 i+1，允許重複用同一個數
            path = path[:len(path)-1]           // 撤回
        }
    }

    dfs(0, target, []int{})
    return res
}
```

</details>

**為什麼要 push 又要 pop**

`path` 是所有遞迴共用的同一個陣列，而 push 跟 pop 服務的對象不同：

```typescript
for (let i = start; i < candidates.length; i++) {
    path.push(candidates[i]);             // 佈置現場：給下一行的 dfs 用
    dfs(i, remain - candidates[i], path); // 子樹看到的路徑，要包含「我選了這個數」
    path.pop();                           // 清現場：給 for 的下一輪用
}
```

不變量是 **for 每一輪開始時，path 都長一樣**。push 和 pop 成對包住 dfs，保證這件事。少了 pop，第二輪的 path 還黏著第一輪的殘留，往下全錯。

用 `path=[2], remain=5` 這層實際看：

```
path=[2], remain=5

  push(2) → path=[2,2]
  dfs → 這棵子樹裡找到 [2,2,3]（複製一份存進 res）
  pop() → path=[2]         ← 還原，繼續試下一個

  push(3) → path=[2,3]
  dfs → 裡面 3、6、7 都太大，無路，return
  pop() → path=[2]         ← 還原

  6 > 5、7 > 5，跳過（連 push 都不會發生）

for 結束，回到上一層
```

成功的子樹跟死路的子樹，回來都要 pop：res 收走的是複本，path 本身還要拿去試別條路。

收集時為什麼要複本？TS 的 `res.push([...path])`、Go 的 `copy(tmp, path)` 都在做同一件事：把當下的路徑拍照存檔。直接 `res.push(path)` 存的是同一個陣列的參照，之後的 push/pop 會把已收集的答案改掉。

Go 版多一層細節：`path = path[:len(path)-1]` 沒有刪資料，只是把 slice 的長度往回撥一格，底層陣列那格的值還在，下一輪 `append` 直接覆寫它。所以 res 裡如果存的不是複本，而是指著同一塊底層陣列的 slice，答案會在之後的覆寫中悄悄變掉。

---

**為什麼 dfs 傳 `i` 不是 `i+1`**

傳 `i`，下一層還能從自己開始選，「同一個數可重複用」就是這樣實作的。它同時保證組合不重複：每一層只往右看，選過 3 之後不會回頭選 2，所以 `[2,3]` 跟 `[3,2]` 只會出現一種。要是這題改成每個數只能用一次（40 Combination Sum II），就改傳 `i+1`。

---

**複雜度**

- Time: O(N^(T/M))，N = candidates 數量，T = target，M = 最小候選數
- Space: O(T/M)，最深的 call stack
