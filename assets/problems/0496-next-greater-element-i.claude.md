兩排數字。`nums2 = [1,3,4,2]` 是完整的世界，`nums1 = [4,1,2]` 是一串查詢。對 nums1 每個數字，去 nums2 裡找到它，回答：它右邊第一個更大的數是誰？沒有就答 -1。

```
nums2 = [1, 3, 4, 2]

查 4 → 在 nums2 位置 2，右邊只有 2，沒更大 → -1
查 1 → 在 nums2 位置 0，右邊第一個更大是 3 → 3
查 2 → 在 nums2 位置 3，右邊沒東西 → -1

答案 [-1, 3, -1]
```

這題是 [739 Daily Temperatures](/problem/daily-temperatures) 的變形。739 對「每一天」找右邊第一個更大，這裡多包一層：只問 nums1 裡的那幾個數，而且答案要的是「那個更大的值」，不是距離。

---

**解題引導**

用 `nums1 = [4,1], nums2 = [1,3,4,2]` 想。

**Step 1：暴力怎麼做？**

*find it in nums2, then scan right*

<span class="spoiler">對 nums1 每個數，在 nums2 裡定位，再往右掃找更大的。$O(n_1 \times n_2)$。Locate in nums2, scan rightward.</span>

**Step 2：739 的 monotonic stack 能搬過來嗎？**

*yes, but run it over all of nums2 first*

<span class="spoiler">能。先用 monotonic stack 對整個 nums2 算出「每個數的下一個更大」，存進 hashmap，再回答 nums1 的查詢。Precompute next-greater for all nums2, then look up.</span>

**Step 3：739 stack 存 index，這裡存什麼？**

*store the value, and record the greater value itself*

<span class="spoiler">存值。因為答案要的是「更大的那個值」，不是差幾格。而且數字保證 unique，可以直接拿值當 hashmap 的 key。Store values; the answer is the greater value.</span>

想完再往下看 code。

---

## 解法一：暴力（定位 + 往右掃）

對 nums1 每個數，先在 nums2 找到它的位置，再往右找第一個更大的。

```typescript
function nextGreaterElement(nums1: number[], nums2: number[]): number[] {
    return nums1.map(target => {
        const j = nums2.indexOf(target);          // 在 nums2 定位
        for (let k = j + 1; k < nums2.length; k++) {
            if (nums2[k] > target) return nums2[k];
        }
        return -1;
    });
}
```

- Time: $O(n_1 \times n_2)$ — 每個查詢都可能掃過整個 nums2
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func nextGreaterElement(nums1 []int, nums2 []int) []int {
    res := make([]int, len(nums1))
    for i, target := range nums1 {
        res[i] = -1
        j := 0
        for nums2[j] != target { // 找 target 在 nums2 的位置
            j++
        }
        for k := j + 1; k < len(nums2); k++ {
            if nums2[k] > target {
                res[i] = nums2[k]
                break
            }
        }
    }
    return res
}
```

</details>

$N_1, N_2 \le 1000$，$O(n_1 \times n_2) \approx 10^6$ 操作 → 約 0.1 秒，這題資料小能過。但 follow-up 要 $O(n_1 + n_2)$，暴力對每個查詢重複掃 nums2，浪費在這。

---

## 解法二：monotonic stack + hashmap

把 739 的 monotonic stack 搬過來，先對整個 nums2 算出「每個數字的下一個更大元素」，結果存進 hashmap。之後 nums1 的每個查詢，就是一次 $O(1)$ 查表。

跟 739 的兩個差別：
- 739 存 index（要算距離），這裡存**值**（答案就是那個更大的值）。
- 739 把答案寫進 `answer[prev]`，這裡寫進 `map[被 pop 的值] = 現在這個更大的值`。

stack 一樣維持由下往上遞減。只要來一個比 top 大的數 num，那就是 top 在等的更大值，pop 出來記進 map。

用 `nums2 = [1,3,4,2]` 一格一格走。`pendingNums` 存還沒等到更大值的數，每個 num 進來先問：我比 top 大嗎？大就把 top 彈出來、記進 map，一直問到不大為止，然後自己 push 進去。

```
num=1   空的，while 不進
        push(1)                  pendingNums=[1]     map={}

num=3   top 是 1，3 > 1 → pop()=1, map[1]=3
        空了，while 停
        push(3)                  pendingNums=[3]     map={1:3}

num=4   top 是 3，4 > 3 → pop()=3, map[3]=4
        空了，while 停
        push(4)                  pendingNums=[4]     map={1:3, 3:4}

num=2   top 是 4，2 > 4？否 → while 不進
        push(2)                  pendingNums=[4,2]   map={1:3, 3:4}

掃完。pendingNums 剩 [4, 2]：它們到最後都沒等到更大的，所以 map 裡沒有它們。
```

這個例子每輪最多彈一個。順序換成 `[1,3,2,4]` 的話，最後的 4 進來時 `pendingNums` 是 `[3,2]`，while 會連彈兩個：`pop()=2` 記 `map[2]=4`，再 `pop()=3` 記 `map[3]=4`，一個 4 送走兩個。裡面疊著的都是還沒等到更大值的數，夠大的新數進來，就由上往下把比它小的逐一彈出去、記錄答案，碰到比自己大的才停。每個元素一生只進出 stack 各一次，整趟才會是 $O(n_2)$。

再回答 nums1 = [4,1,2]：查 map，查不到給 -1。

```
4 → map 沒有 → -1
1 → map[1]=3 → 3
2 → map 沒有 → -1
答案 [-1,3,-1]
```

這題只找「第一個」比它大的，不是右邊最大的。找到第一個更大的就定案、收工，後面再大也不管。monotonic stack 剛好給你這個：元素一旦被 pop 就離開 stack，之後再也看不到，所以它拿到的一定是遇到的第一個更大值，後面就算來更大的數字也碰不到它。

別跟「右邊最大值」搞混，那是另一種題：從右往左掃、一路記目前的 max 就好，不用 stack。

```typescript
function nextGreaterElement(nums1: number[], nums2: number[]): number[] {
    const nextGreater = new Map<number, number>();
    const pendingNums: number[] = [];   // 存值，由下到上遞減

    for (const num of nums2) {
        // num 比 top 大 → 這就是 top 在等的更大值
        while (pendingNums.length && num > pendingNums[pendingNums.length - 1]) {
            nextGreater.set(pendingNums.pop()!, num);
        }
        pendingNums.push(num);
    }
    // 裡面剩的沒有更大的，查不到就給 -1
    return nums1.map(n => nextGreater.get(n) ?? -1);
}
```

- Time: $O(n_1 + n_2)$ — nums2 一趟建表，nums1 一趟查表
- Space: $O(n_2)$ — hashmap 加 stack

<details>
<summary>Go 版本</summary>

```go
func nextGreaterElement(nums1 []int, nums2 []int) []int {
    nextGreater := map[int]int{}
    pendingNums := []int{} // 存值，由下到上遞減

    for _, num := range nums2 {
        for len(pendingNums) > 0 && num > pendingNums[len(pendingNums)-1] {
            top := pendingNums[len(pendingNums)-1]
            pendingNums = pendingNums[:len(pendingNums)-1]
            nextGreater[top] = num // 這就是 top 在等的更大值
        }
        pendingNums = append(pendingNums, num)
    }

    res := make([]int, len(nums1))
    for i, n := range nums1 {
        if g, ok := nextGreater[n]; ok {
            res[i] = g
        } else {
            res[i] = -1 // 查不到，沒有更大的
        }
    }
    return res
}
```

</details>

做法是把計算跟查詢分開：不對每個查詢各掃一次，而是**先一次算好整個 nums2 的答案**，nums1 的每個查詢就只剩查表。數字 unique 這個條件讓值可以直接當 key，省了記位置的麻煩。

---

## 解法比較表

| 解法 | Time | Space | N=1000 秒數 | 備註 |
|---|---|---|---|---|
| 暴力定位 + 往右掃 | $O(n_1 \times n_2)$ | $O(1)$ | 約 0.1 秒 | 資料小能過，但踩在 follow-up 禁止的量級 |
| monotonic stack + hashmap | $O(n_1 + n_2)$ | $O(n_2)$ | 約 0.00001 秒 | follow-up 要的解 |

---

## 結論

跟 739 同一個 monotonic stack，差在存值不存 index、答案記進 hashmap。多的那層 nums1 查詢，靠「先算好全部再查表」解決。下一題 [503 Next Greater Element II](/problem/next-greater-element-ii) 把陣列變成環狀，再加一個轉折。
