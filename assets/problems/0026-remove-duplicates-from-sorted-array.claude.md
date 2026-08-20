排好隊的一群人，前後兩人身高一樣就叫其中一個出列，剩下的人往前補位，最後回報還剩幾個人。

```
nums = [1, 1, 2, 2, 3]
        重複要剔除，只留一個
```

陣列已經排序，所以重複的元素一定黏在一起，不用整組掃就能找到。

---

**解題引導**

input：`nums = [0, 0, 1, 1, 1, 2]`，正解 `k = 3`，且 `nums` 前三格變成 `[0, 1, 2]`。

**Step 1：已排序這件事，代表重複的元素會長在哪？**

*sorted means duplicates are neighbors*

<span class="spoiler">彼此相鄰。1 跟 1 一定挨在一起，不會被 2 或 0 隔開。所以只要看「跟前一個一不一樣」就夠了，不用整組比對。Sorted means duplicates sit next to each other.</span>

**Step 2：題目要求 in-place，不能開新陣列，那結果要寫去哪？**

<span class="spoiler">寫回 nums 自己，用一個指標記「下一個不重複的值該放第幾格」。Write back into nums itself, tracking where the next unique value goes.</span>

**Step 3：兩根指標，一根找新值（右邊掃描）、一根記寫入位置（左邊慢慢移），什麼時候該讓寫入指標往前一格？**

*only advance the write pointer when you find something new*

<span class="spoiler">掃描指標碰到「跟上一個寫入的值不同」的數字時，寫入指標才前進一格，把新值填進去。碰到重複值就跳過，寫入指標原地不動。Advance the write pointer only when the scan finds a new value; skip duplicates.</span>

想完再往下看 code。

---

## 解法：雙指針

一根指標 `slow` 記「下一個不重複值該填的位置」，一根指標 `fast` 負責往前找新值。`fast` 每走一步就比對 `nums[fast]` 跟 `nums[slow - 1]`（上一個已確定的值），不同就代表找到新值，填進 `nums[slow]`，`slow` 才前進。

```typescript
function removeDuplicates(nums: number[]): number {
    let slow = 1;                              // 第 0 格一定留著，從 1 開始比
    for (let fast = 1; fast < nums.length; fast++) {
        if (nums[fast] !== nums[slow - 1]) {   // 跟上一個確定的值不同 → 是新值
            nums[slow] = nums[fast];
            slow++;
        }
    }
    return slow;
}
```

- **Time: O(N)** — `fast` 從頭掃到尾，每格只看一次
- Space: O(1) — 原地覆寫，沒開額外陣列
- 實際秒數：$N=3 \times 10^4$ 上限 → $3 \times 10^4$ 操作 → **0.003 秒**

<details>
<summary>Go 版本</summary>

```go
func removeDuplicates(nums []int) int {
    slow := 1
    for fast := 1; fast < len(nums); fast++ {
        if nums[fast] != nums[slow-1] {
            nums[slow] = nums[fast]
            slow++
        }
    }
    return slow
}
```

</details>

**為什麼比較對象是 `nums[slow - 1]` 不是 `nums[fast - 1]`**：`fast` 掃過的重複值沒有真的被覆寫掉，`nums[fast - 1]` 可能還停在舊值。`slow - 1` 才是目前為止「已經確定收進結果」的最後一個值。

**為什麼 `slow` 從 1 開始**：第 0 個元素永遠是結果的一部分，不用比較，直接留著。

---

**Step by step**

`nums = [0, 0, 1, 1, 1, 2]`，slow 從 1 開始。

| fast | nums[fast] | nums[slow-1] | 相同？ | 動作 | slow | nums 當下 |
|---|---|---|---|---|---|---|
| 1 | 0 | 0 (slow=1) | 相同 | 跳過 | 1 | [0,0,1,1,1,2] |
| 2 | 1 | 0 (slow=1) | 不同 | nums[1]=1, slow++ | 2 | [0,1,1,1,1,2] |
| 3 | 1 | 1 (slow=2) | 相同 | 跳過 | 2 | [0,1,1,1,1,2] |
| 4 | 1 | 1 (slow=2) | 相同 | 跳過 | 2 | [0,1,1,1,1,2] |
| 5 | 2 | 1 (slow=2) | 不同 | nums[2]=2, slow++ | 3 | [0,1,2,1,1,2] |

迴圈結束，回傳 `slow = 3`。前三格 `[0, 1, 2]` 就是答案，後面的 `1, 1, 2` 是掃描過程留下的殘值，題目說可以不管。

---

**能不能更好？**

不能。每個元素至少要看一次才知道它是不是重複，O(N) 是下限；in-place 覆寫也已經是 O(1) 額外空間，沒有更省的寫法。

---

**Overthinking**

**變形：允許每個值最多出現兩次**（LeetCode 80）

比較對象從 `nums[slow - 1]` 換成 `nums[slow - 2]`，邏輯不變：跟兩格前的值不同才寫入，相當於放寬「重複可以留一份」變成「重複可以留兩份」。

**變形：陣列沒有排序**

雙指針失效，因為重複值不再相鄰。要先用 hashset 記錄看過的值，退化成 O(N) 額外空間。「已排序」才是這題能原地做的前提。

---

## 結論

已排序陣列的重複值一定相鄰，兩根指標一根找新值、一根記寫入位置，比對「跟上一個確定的值是否相同」就能一次掃描原地留下不重複的值，O(N) 時間、O(1) 空間。
