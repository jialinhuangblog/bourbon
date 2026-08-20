一排每天的氣溫 `[73,74,75,71,69,72,76,73]`。每一天問同一個問題：從今天算起，還要等幾天才會遇到更熱的一天？沒有更熱的就記 0。

```
day  溫度   還要等幾天更熱
0    73     1   （隔天 74 就更熱）
1    74     1   （隔天 75）
2    75     4   （要等到 day6 的 76）
3    71     2   （等到 day5 的 72）
4    69     1
5    72     1
6    76     0   （後面沒有更熱的了）
7    73     0
```

翻回 code 術語：對每個位置 i，找右邊第一個比它大的元素，回傳距離。這是「下一個更大元素」這類題的第一題，後面 [496](/problem/next-greater-element-i)、[503](/problem/next-greater-element-ii) 都是它的變形。

---

**解題引導**

用 `[73,74,75,71,69]` 想。

**Step 1：最直覺怎麼做？**

*for each day, look right until warmer*

<span class="spoiler">每一天往右掃，找到第一個更熱的就記距離。$O(n^2)$。For each day scan rightward for the first warmer day.</span>

**Step 2：暴力解重複做了什麼？想想 71 和 69 這兩天。**

*both are waiting for the same warmer day*

<span class="spoiler">71 和 69 都在等右邊某個更熱的日子。暴力解讓它們各自往右掃，掃的路段大量重疊。Colder days re-scan the same region independently.</span>

**Step 3：能不能讓「還在等更熱那天」的日子先存起來，來了更熱的一天一次解決？**

*keep the waiting days on a stack, a warmer day settles them*

<span class="spoiler">用一個 stack 存「還沒等到更熱那天」的日子，溫度由下往上遞減。今天來了，比 top 熱就把 top 那天結算掉。A monotonic stack of unresolved days.</span>

想完再往下看 code。

---

## 解法一：暴力（兩層迴圈）

對每一天，往右一個個看，遇到第一個更熱的就記下距離。

用 `[73,74,75,71,69]` 走 day 2（75）：

```
day2 = 75
往右看 day3=71 < 75，繼續
往右看 day4=69 < 75，繼續
到底了，沒更熱的 → answer[2] = 0
```

```typescript
function dailyTemperatures(temperatures: number[]): number[] {
    const n = temperatures.length;
    const answer = new Array(n).fill(0);
    for (let i = 0; i < n; i++) {
        for (let j = i + 1; j < n; j++) {   // 往右找第一個更熱的
            if (temperatures[j] > temperatures[i]) {
                answer[i] = j - i;
                break;
            }
        }
    }
    return answer;
}
```

- Time: $O(n^2)$
- Space: $O(1)$（不算 answer）

<details>
<summary>Go 版本</summary>

```go
func dailyTemperatures(temperatures []int) []int {
    n := len(temperatures)
    answer := make([]int, n)
    for i := 0; i < n; i++ {
        for j := i + 1; j < n; j++ {
            if temperatures[j] > temperatures[i] {
                answer[i] = j - i
                break
            }
        }
    }
    return answer
}
```

</details>

N=$10^{5}$，$O(n^2)$ → $10^{10}$ 操作 → 上千秒，直接 TLE。浪費在哪？71 那天往右掃過 69、72，69 那天又往右掃一次 72。同一段右邊被不同的日子重複掃。

---

## 解法二：monotonic stack

把「還沒等到更熱那天」的日子放進一個 stack，維持溫度由下往上遞減（越晚放的越冷）。這叫 **monotonic stack**：stack 裡的值一路往同一個方向走，只增或只減。

為什麼遞減？因為只要新的一天比 top 冷，它就還在等，直接壓上去，遞減關係自然維持。一旦新的一天比 top 熱，top 那天就等到了，pop 出來結算距離。而且它比 top 熱，很可能也比 top 下面那幾天熱，所以要一直 pop 到 top 比它熱為止。

存 index 不存溫度，因為結算要算 `i - prev`（差幾天），需要位置。

用 `[73,74,75,71,69,72,76,73]` 走。`pendingIdx` 存還沒等到更熱那天的 index，`pop()=k` 表示彈出來的是 k：

```
i = 溫度   動作                           pendingIdx
0 = 73    push(0)                         [0]
1 = 74    74>73 → pop()=0, ans[0]=1-0=1   [ ]
          push(1)                         [1]
2 = 75    75>74 → pop()=1, ans[1]=2-1=1   [ ]
          push(2)                         [2]
3 = 71    71<75 → push(3)                 [2,3]
4 = 69    69<71 → push(4)                 [2,3,4]
5 = 72    72>69 → pop()=4, ans[4]=5-4=1   [2,3]
          72>71 → pop()=3, ans[3]=5-3=2   [2]
          72<75 → push(5)                 [2,5]
6 = 76    76>72 → pop()=5, ans[5]=6-5=1   [2]
          76>75 → pop()=2, ans[2]=6-2=4   [ ]
          push(6)                         [6]
7 = 73    73<76 → push(7)                 [6,7]

answer = [1, 1, 4, 2, 1, 1, 0, 0]
最後 pendingIdx 剩 [6,7]，這兩天沒等到更熱的，維持 0。
```

看 i=5 那步：一個 72 同時結算了 69 和 71 兩天。暴力解要為這兩天各掃一次，monotonic stack 一次 pop 掉。

```typescript
function dailyTemperatures(temperatures: number[]): number[] {
    const n = temperatures.length;
    const answer = new Array(n).fill(0);
    const pendingIdx: number[] = [];   // 存 index，溫度由下到上遞減

    for (let i = 0; i < n; i++) {
        // 今天比 top 那天熱 → top 那天等到了，結算
        // pendingIdx[pendingIdx.length - 1] 是最上面那個 index，
        // 再套一層 temperatures[...] 才是那天的溫度
        while (pendingIdx.length && temperatures[i] > temperatures[pendingIdx[pendingIdx.length - 1]]) {
            const prev = pendingIdx.pop()!;
            answer[prev] = i - prev;
        }
        pendingIdx.push(i);
    }
    return answer;
}
```

- Time: $O(n)$ — 每個 index 進 stack 一次、出 stack 最多一次
- Space: $O(n)$ — stack 最壞裝下全部（溫度一路遞減時）

<details>
<summary>Go 版本</summary>

```go
func dailyTemperatures(temperatures []int) []int {
    n := len(temperatures)
    answer := make([]int, n)
    pendingIdx := []int{} // 存 index，溫度由下到上遞減

    for i := 0; i < n; i++ {
        // pendingIdx[len(pendingIdx)-1] 是最上面那個 index，
        // 再套一層 temperatures[...] 才是那天的溫度
        for len(pendingIdx) > 0 && temperatures[i] > temperatures[pendingIdx[len(pendingIdx)-1]] {
            prev := pendingIdx[len(pendingIdx)-1]
            pendingIdx = pendingIdx[:len(pendingIdx)-1] // pop
            answer[prev] = i - prev
        }
        pendingIdx = append(pendingIdx, i)
    }
    return answer
}
```

</details>

裡面有個 while 迴圈，看起來像 $O(n^2)$，其實是 $O(n)$。每個 index 一輩子只被 push 一次、pop 一次，push 跟 pop 加起來最多 2n 次。內層 while 跑得多，代表這一步結算了很多天，那些天之後就不會再被碰。

---

## 解法比較表

| 解法 | Time | Space | N=$10^{5}$ 秒數 | 備註 |
|---|---|---|---|---|
| 暴力兩層迴圈 | $O(n^2)$ | $O(1)$ | 上千秒（TLE） | 好想，過不了 |
| monotonic stack | $O(n)$ | $O(n)$ | 約 0.01 秒 | 標準解，這類題的模板 |

---

## 結論

「找右邊第一個更大」的題，暴力解會重複掃同一段右邊。monotonic stack 把還沒解決的位置存起來，來一個更大的就一次結算掉所有比它小的。每個位置進出 stack 各一次，$O(n)$。這個模板，[496](/problem/next-greater-element-i) 和 [503](/problem/next-greater-element-ii) 原樣再用。
