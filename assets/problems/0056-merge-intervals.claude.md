給一堆時間區間，有重疊的就合併成一個。回傳合併後的結果。

```
[[1,3], [2,6], [8,10], [15,18]]

1--3
  2------6
              8--10
                       15--18

[1,3] 和 [2,6] 重疊 → 合併成 [1,6]
[8,10] 和 [15,18] 各自獨立

答案：[[1,6], [8,10], [15,18]]
```

---

**解題引導**

**Step 1：怎麼判斷兩個區間有沒有重疊？**

<span class="spoiler">如果前一個的結尾 >= 後一個的開頭，就有重疊。例如 [1,3] 和 [2,6]，3 >= 2，重疊</span>

**Step 2：區間順序是亂的，怎麼一次搞定？**

<span class="spoiler">先按開頭排序。排完之後，重疊的區間一定相鄰，只要跟前一個比就好</span>

**Step 3：重疊的時候，合併後的結尾是什麼？**

<span class="spoiler">取兩個結尾的較大值。[1,3] 和 [2,6] 合併成 [1, max(3,6)] = [1,6]</span>

想完再往下看 code。

---

## 解法：排序

先排序，排完之後從頭掃一遍，每個區間只跟前一個比：重疊就合併，不重疊就是新的一段。

用 `[[1,3], [2,6], [8,10], [15,18]]` 走一遍（已經排好了）：

```
拿 [1,3] 當起點 → result: [[1,3]]

看 [2,6]：2 <= 3（前一個的結尾）→ 重疊！
  結尾取 max(3,6) = 6 → result: [[1,6]]

看 [8,10]：8 > 6 → 沒重疊，新的一段
  → result: [[1,6], [8,10]]

看 [15,18]：15 > 10 → 沒重疊，新的一段
  → result: [[1,6], [8,10], [15,18]]
```

如果順序是亂的呢？`[[8,10], [1,3], [15,18], [2,6]]`

```
排序後（按開頭）：[[1,3], [2,6], [8,10], [15,18]]
→ 跟上面一樣
```

排序是這題的關鍵。排完之後，重疊的區間一定相鄰，不用兩兩比較。

---

完整程式碼：

```typescript
function merge(intervals: number[][]): number[][] {
    // 按開頭排序
    intervals.sort((a, b) => a[0] - b[0]);

    const result: number[][] = [intervals[0]];

    for (let i = 1; i < intervals.length; i++) {
        const merged = result[result.length - 1]; // 目前正在合併的那段區間
        const next = intervals[i];                // 下一個要看的區間

        if (next[0] <= merged[1]) { // 重疊：下一個的開頭 <= 合併中的結尾
            merged[1] = Math.max(merged[1], next[1]); // 結尾延長
        } else {
            result.push(next); // 沒重疊，開一段新的
        }
    }
    return result;
}
```

- Time: O(n log n) — 排序
- Space: O(n) — result 陣列（最壞情況沒有任何重疊）

<details>
<summary>Go 版本</summary>

```go
import "sort"

func merge(intervals [][]int) [][]int {
    // 按開頭排序
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })

    result := [][]int{intervals[0]} // 第一個直接放進去

    for i := 1; i < len(intervals); i++ {
        merged := result[len(result)-1] // 目前正在合併的那段區間
        next := intervals[i]            // 下一個要看的區間

        if next[0] <= merged[1] { // 重疊：下一個的開頭 <= 合併中的結尾
            if next[1] > merged[1] {
                merged[1] = next[1] // 結尾延長到較大的
            }
        } else {
            result = append(result, next) // 沒重疊，開一段新的
        }
    }
    return result
}
```

</details>

---

**為什麼排序後只要跟前一個比？**

排完之後，所有區間按開頭從小到大排。如果 `[A]` 和 `[C]` 重疊但中間隔了一個 `[B]`，那 `[A]` 跟 `[B]` 一定也重疊（因為 B 的開頭介於 A 和 C 之間）。所以重疊的區間排完一定是連續的，不會跳著重疊。

---

**邊界情況：剛好碰到**

```
[[1,4], [4,5]] → [1,5]
```

`4 <= 4` 算重疊。題目定義端點相同就是 overlapping。

---

## 結論

先排序，再掃一遍。每個區間只跟前一段比：重疊就延長結尾，不重疊就新開一段。Intervals 類的題幾乎都從「先排序」開始。
