跟 [739](/problem/daily-temperatures)、[496](/problem/next-greater-element-i) 同一個問題：對每個元素找右邊第一個更大的。這題多一個轉折——陣列是**環狀**的。最後一個的右邊接回第一個。

```
nums = [1, 2, 1]（環狀，尾接頭）

index 0 (1)：右邊是 2 → 2
index 1 (2)：往右繞一圈，沒有比 2 大的 → -1
index 2 (1)：右邊接回 index 0，再往前是 2 → 2

答案 [2, -1, 2]
```

看 index 2 那個 1：直線陣列裡它右邊沒東西了，但環狀讓它繞回開頭，找到 2。難點就在這個「繞回去」。

---

**解題引導**

用 `nums = [1,2,1]` 想。

**Step 1：怎麼處理「繞回開頭」？**

*walk the array twice, wrap with modulo*

<span class="spoiler">把陣列走兩遍，用 `i % n` 取真正的位置。第二遍讓尾巴的元素看到開頭。Traverse twice, index with i % n.</span>

**Step 2：739 的 monotonic stack 直接能用嗎？**

*yes, just feed it 2n indices instead of n*

<span class="spoiler">能。stack 邏輯完全不變，只是餵給它 2n 個 index。The stack logic is identical, just loop 2n times.</span>

**Step 3：走第二遍時，還要把 index push 進 stack 嗎？**

*no, the first pass already pushed everyone*

<span class="spoiler">不用。第一遍已經把每個 index 都放過一次，第二遍只是讓它們有機會被結算，push 只會製造重複。Only resolve in the second pass, don't push again.</span>

想完再往下看 code。

---

## 解法一：暴力（每個繞一圈）

對每個 index，往右繞最多 n-1 步找第一個更大的。用 `(i + step) % n` 處理環狀。

```typescript
function nextGreaterElements(nums: number[]): number[] {
    const n = nums.length;
    const res = new Array(n).fill(-1);
    for (let i = 0; i < n; i++) {
        for (let step = 1; step < n; step++) {   // 最多繞 n-1 步
            const j = (i + step) % n;
            if (nums[j] > nums[i]) { res[i] = nums[j]; break; }
        }
    }
    return res;
}
```

- Time: $O(n^2)$
- Space: $O(1)$（不算 res）

<details>
<summary>Go 版本</summary>

```go
func nextGreaterElements(nums []int) []int {
    n := len(nums)
    res := make([]int, n)
    for i := range res {
        res[i] = -1
    }
    for i := 0; i < n; i++ {
        for step := 1; step < n; step++ {
            j := (i + step) % n
            if nums[j] > nums[i] {
                res[i] = nums[j]
                break
            }
        }
    }
    return res
}
```

</details>

N=$10^{4}$，$O(n^2)$ → $10^{8}$ 操作 → 約 10 秒，卡在 TLE 邊緣。跟 739 一樣的浪費：相鄰的小元素各自繞圈，掃過的路段重疊。

---

## 解法二：monotonic stack 走兩圈

739 的 monotonic stack 原樣搬過來，改的地方是迴圈跑 `2n` 次、位置用 `i % n`。走完第一圈，尾巴的元素還沒等到更大的就留在 stack 上；第二圈繞回開頭，讓它們有機會被前面的大元素結算。

第二圈只做結算，不再 push。因為第一圈已經把每個 index 都放進 stack 過一次，第二圈再 push 會製造清不掉的重複。

用 `nums = [1,2,1]` 走。`pendingIdx` 存還沒找到答案的 index，每次只跟 top 比：

```
i=0  idx=0  nums=1   stack 是空的 → push(0)            pendingIdx=[0]
i=1  idx=1  nums=2   2 > nums[0]=1 → pop()=0, res[0]=2
                     空了 → push(1)                    pendingIdx=[1]
i=2  idx=2  nums=1   1 < nums[1]=2 → push(2)           pendingIdx=[1,2]
--- 第二圈，只結算不 push ---
i=3  idx=0  nums=1   1 不大於 nums[2]=1 → 不動          pendingIdx=[1,2]
i=4  idx=1  nums=2   2 > nums[2]=1 → pop()=2, res[2]=2
                     2 不大於 nums[1]=2 → 停            pendingIdx=[1]
i=5  idx=2  nums=1   1 < nums[1]=2 → 不動               pendingIdx=[1]

結束。index 1 從沒被 pop 過 → res[1] 保持 -1。
答案 [2,-1,2]
```

index 2 的 1，在第二圈被繞回來的 2 結算掉。這就是環狀多走一圈的作用。

```typescript
function nextGreaterElements(nums: number[]): number[] {
    const n = nums.length;
    const res = new Array(n).fill(-1);
    const pendingIdx: number[] = [];   // 存 index，對應的 nums 值由下到上遞減

    for (let i = 0; i < 2 * n; i++) {
        const idx = i % n;
        // pendingIdx[pendingIdx.length - 1] 是最上面那個 index，
        // 再套一層 nums[...] 才是它對應的值
        while (pendingIdx.length && nums[idx] > nums[pendingIdx[pendingIdx.length - 1]]) {
            res[pendingIdx.pop()!] = nums[idx];
        }
        if (i < n) pendingIdx.push(idx);   // 第二圈只結算，不再 push
    }
    return res;
}
```

- Time: $O(n)$ — 走 2n 次是常數倍，每個 index 進出 stack 各一次
- Space: $O(n)$ — stack

<details>
<summary>Go 版本</summary>

```go
func nextGreaterElements(nums []int) []int {
    n := len(nums)
    res := make([]int, n)
    for i := range res {
        res[i] = -1
    }
    pendingIdx := []int{} // 存 index，對應的 nums 值由下到上遞減

    for i := 0; i < 2*n; i++ {
        idx := i % n
        // pendingIdx[len(pendingIdx)-1] 是最上面那個 index，
        // 再套一層 nums[...] 才是它對應的值
        for len(pendingIdx) > 0 && nums[idx] > nums[pendingIdx[len(pendingIdx)-1]] {
            top := pendingIdx[len(pendingIdx)-1]
            pendingIdx = pendingIdx[:len(pendingIdx)-1]
            res[top] = nums[idx]
        }
        if i < n {
            pendingIdx = append(pendingIdx, idx) // 第二圈只結算，不再 push
        }
    }
    return res
}
```

</details>

為什麼走兩圈就夠，不用三圈？走完兩圈，每個元素都已經把整個環看過一遍。還留在 stack 上的，代表繞一整圈都沒有比它大的，那就真的是 -1。多走一圈不會再改變任何結果。

---

## 解法比較表

| 解法 | Time | Space | N=$10^{4}$ 秒數 | 備註 |
|---|---|---|---|---|
| 暴力繞圈 | $O(n^2)$ | $O(1)$ | 約 10 秒（TLE 邊緣） | 好想，慢 |
| monotonic stack 走兩圈 | $O(n)$ | $O(n)$ | 約 0.001 秒 | 標準解 |

---

## 結論

環狀版的「下一個更大」，就是把 739 的 monotonic stack 跑兩圈、用 `i % n` 取位置，第二圈只結算不 push。monotonic stack 這條線走到這裡收尾：[739](/problem/daily-temperatures) 建立模板、[496](/problem/next-greater-element-i) 加查表、503 加環狀。
