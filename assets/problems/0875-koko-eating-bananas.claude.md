Koko 偷吃香蕉，警衛 h 小時後回來，她想吃越慢越好但又不能被抓到，找那個臨界速度 k。

---

**解題引導**

用 `piles = [3, 6, 7, 11], h = 8` 思考。

**Step 1：暴力怎麼試？**

*Start from the slowest possible speed and work your way up.*

<span class="spoiler">從 k=1 開始，每個 k 算一次「這速度需要幾小時」，找到第一個夠快的就是答案。但 k 最大到 10⁹，試到天亮。 / Try k=1, 2, 3... Return the first k that finishes within h hours. Way too slow.</span>

**Step 2：k 的範圍是什麼？**

*What's the slowest she could possibly eat? What's the fastest she'd ever need to go?*

<span class="spoiler">最慢是 1。最快是 max(piles) — 一小時幹掉最大那堆，絕對來得及，不需要更快了。答案在 1 到 max(piles) 之間。 / Minimum 1, maximum max(piles). If she can eat the biggest pile in one hour, she's done.</span>

**Step 3：這個區間有什麼特性，讓我們可以跳著搜？**

*k=4 works. Does k=5 work? Does that mean something about the shape of the answer space?*

<span class="spoiler">單調性。速度夠了，更快一定也夠。所以整個範圍長這樣：前半段太慢、後半段都可以，分界就是答案。有分界就能 binary search。 / Monotonic — once k works, everything faster also works. That's a clean boundary. Binary search.</span>

**Step 4：猜一個 mid，怎麼驗證它行不行？**

*You guessed a speed. How do you know if Koko finishes in time?*

<span class="spoiler">對每堆算 ceil(pile / mid) 小時，全部加起來 ≤ h 就行。 / For each pile, compute ceil(pile / mid). Sum them. If total ≤ h, this speed works.</span>

**Step 5：boundary 怎麼收？**

*You want the minimum valid k. When mid works, is mid definitely the answer?*

<span class="spoiler">mid 夠快 → Koko 還有時間可以再慢一點，right = mid（保留 mid 自己，可能就是答案）。mid 不夠快 → 這速度不行，left = mid+1。 / If mid works, keep it as a candidate: right=mid. If mid fails, discard it: left=mid+1.</span>

想完再往下看 code。

---

## 解法一：暴力列舉

暴力解：k 從 1 試到最大，算每個 k 需要幾小時。

```typescript
function minEatingSpeed(piles: number[], h: number): number {
    const maxPile = Math.max(...piles);

    for (let k = 1; k <= maxPile; k++) {
        let hours = 0;
        for (const p of piles) {
            hours += Math.ceil(p / k);  // ceil(p/k)
        }
        if (hours <= h) return k;
    }
    return maxPile;
}
```

<details>
<summary>Go 版本</summary>

```go
func minEatingSpeed(piles []int, h int) int {
    maxPile := 0
    for _, p := range piles { if p > maxPile { maxPile = p } }

    for k := 1; k <= maxPile; k++ {
        hours := 0
        for _, p := range piles {
            hours += (p + k - 1) / k  // ceil(p/k)
        }
        if hours <= h { return k }
    }
    return maxPile
}
```

</details>

走 `[3, 6, 7, 11], h=8`：

| k | 每堆小時 | 總 | 過？ |
|---|---------|-----|-----|
| 1 | 3+6+7+11 | 27 | ✗ |
| 2 | 2+3+4+6 | 15 | ✗ |
| 3 | 1+2+3+4 | 10 | ✗ |
| 4 | 1+2+2+3 | 8 | ✓ |

- Time: **$O(\max(\text{piles}) \times n)$** — 最壞 $10^9 \times 10^4$，根本跑不完

---

## 解法二：二分搜尋

k=1 太慢，k=11 絕對夠，答案藏在中間某處。而且有個關鍵：**這條線是單調的**。

```
k:  1  2  3  4  5  6  7  8  9  10  11
    ✗  ✗  ✗  ✓  ✓  ✓  ✓  ✓  ✓  ✓   ✓
```

左邊全 ✗，右邊全 ✓，找第一個 ✓。不用從頭掃，binary search 每次砍一半。

```typescript
function minEatingSpeed(piles: number[], h: number): number {
    let left = 1;
    let right = Math.max(...piles);

    while (left < right) {
        const mid = left + Math.floor((right - left) / 2);
        let hours = 0;
        for (const p of piles) {
            hours += Math.ceil(p / mid);  // 這堆要吃幾小時
        }
        if (hours <= h) {
            right = mid;       // koko still has time to slow down
        } else {
            left = mid + 1;    // 太慢了，加速
        }
    }
    return left;
}
```

<details>
<summary>Go 版本</summary>

```go
func minEatingSpeed(piles []int, h int) int {
    left, right := 1, 0
    for _, p := range piles { if p > right { right = p } }

    for left < right {
        mid := left + (right-left)/2
        hours := 0
        for _, p := range piles {
            hours += (p + mid - 1) / mid  // ceil(p/mid)
        }
        if hours <= h {
            right = mid      // Koko 還有時間可以再慢，保留 mid
        } else {
            left = mid + 1   // 這速度不夠，去右邊找更快的
        }
    }
    return left
}
```

</details>

走 `[3, 6, 7, 11], h=8`，left=1, right=11：

| 輪 | left | right | mid | 總時間 | 動作 |
|----|------|-------|-----|--------|------|
| 1 | 1 | 11 | 6 | 1+1+2+2=6 ≤ 8 | right=6 |
| 2 | 1 | 6 | 3 | 1+2+3+4=10 > 8 | left=4 |
| 3 | 4 | 6 | 5 | 1+2+2+3=8 ≤ 8 | right=5 |
| 4 | 4 | 5 | 4 | 1+2+2+3=8 ≤ 8 | right=4 |

left == right = 4，結束。答案 4。

- Time: O(n log m)，m = max(piles)
- Space: O(1)

---

**`(p + k - 1) / k` 是什麼魔法？**

整數除法自動 floor，但我們要 ceil。補個 `k-1` 在分子就搞定：

```
ceil(7 / 4) = 2
(7 + 3) / 4 = 10 / 4 = 2  ✓
```

Go 沒有整數 ceil，這是標準寫法。TypeScript 直接 `Math.ceil` 清楚多了。

---

**`right = mid` 不是 `right = mid - 1`？**

因為我們要找「最小合法值」，mid 夠快的時候 mid 本身可能就是答案，不能排除。收 `right = mid` 不是 `mid - 1`。

只有 mid 確定不是答案的時候才排除：mid 太慢 → `left = mid + 1`。

loop 結束 left == right，就是 Koko 最懶惰的合法速度。

---

## 結論

答案空間單調，二分搜速度，ceil 驗時間。O(n log m)，Koko 得救。
