河上排了一列踏腳石。每塊石頭上刻著一個數字，意思是「從我這裡最多能往前跳幾格」。

跳幾格都行，只要不超過刻的數字。問從第一塊跳到最後一塊，最少要跳幾次。

```
nums = [2, 3, 1, 1, 4]

index    0    1    2    3    4
刻的數字 2    3    1    1    4

0 → 1（跳 1 格），1 → 4（跳 3 格）
兩次
```

從 0 直接跳 2 格到 index 2 也行，但 index 2 只刻了 1，接下來只能到 3，再一次才到 4，變成三次。

翻成 code 的講法：`nums[i]` 是從 i 出發的最大跳躍距離，求到達 `n-1` 的最少跳躍次數。題目保證一定到得了。

---

**解題引導**

拿 `[2, 3, 1, 1, 4]` 想。

**Step 1：從 index 0 出發，一次跳能到哪些格子？**

<span class="spoiler">index 1 跟 index 2，因為刻的是 2。這兩格都是「跳一次」就到得了的。Everything within reach counts as one jump.</span>

**Step 2：跳兩次能到的範圍呢？**

*whatever the one-jump group can reach.*

<span class="spoiler">從 index 1 或 index 2 再跳一次能到的所有格子。index 1 能到 4，index 2 能到 3，所以跳兩次能到 index 3 跟 4。The second layer is the union of what the first layer reaches.</span>

**Step 3：這個「一次能到哪些、兩次能到哪些」，像什麼？**

<span class="spoiler">BFS 分層。第 k 層就是「剛好跳 k 次能到的格子」。答案是 n-1 落在第幾層。It is breadth-first search, with layers instead of a queue.</span>

**Step 4：真的需要 queue 嗎？**

*each layer is a contiguous range, so two numbers describe it.*

<span class="spoiler">不用。每一層能到的格子都是連續的一段，所以只要記這一層的右界、跟下一層的右界，兩個數字就描述完了。掃到右界就換層、次數加一。Two boundaries replace the whole queue.</span>

想完再往下看 code。

---

## 解法一：DP，每格記最少幾次

`f[i]` 記「到 index i 最少要跳幾次」。從左往右，每個 i 把它能到的每一格都更新一遍。

```typescript
function jump(nums: number[]): number {
    const n = nums.length;
    const f = new Array(n).fill(Infinity);
    f[0] = 0;

    for (let i = 0; i < n; i++) {
        for (let step = 1; step <= nums[i] && i + step < n; step++) {
            const target = i + step;
            f[target] = Math.min(f[target], f[i] + 1);   // 從 i 跳過來
        }
    }

    return f[n - 1];
}
```

- Time: $O(n^2)$ — 內圈長度是 `nums[i]`，最大可以到 n
- Space: $O(n)$ — 表格

**走一遍。** `[2, 3, 1, 1, 4]`：

| i | 能跳到哪些 | 更新 | f 的內容 |
|---|---|---|---|
| 起始 | | | `[0, ∞, ∞, ∞, ∞]` |
| 0 | 1, 2 | f[1]=1、f[2]=1 | `[0, 1, 1, ∞, ∞]` |
| 1 | 2, 3, 4 | f[2] 已經是 1，不動；f[3]=2、f[4]=2 | `[0, 1, 1, 2, 2]` |
| 2 | 3 | f[3] 已經是 2，不動 | `[0, 1, 1, 2, 2]` |
| 3 | 4 | f[4] 已經是 2，不動 | `[0, 1, 1, 2, 2]` |
| 4 | 沒有了 | | `[0, 1, 1, 2, 2]` |

答案 `f[4] = 2`。

後面三列全是「已經有更小的了，不動」。這就是這一版慢的原因：從左往右掃的時候，`f[i]` 早就是最小值了，後面來的一定不會更好，但迴圈還是照跑。

**n=10^4 的話。** 內圈總長度是所有 `nums[i]` 的和（被 n 截斷）。實測隨機的 `nums[i] ∈ [1, 1000]` 是 4,787,239 次，換算 0.48 秒。最壞情況每格都刻很大的數，實測 49,995,000 次，換算 5 秒，TLE。

<details>
<summary>Go 版本</summary>

```go
func jump(nums []int) int {
    n := len(nums)
    f := make([]int, n)
    for i := 1; i < n; i++ {
        f[i] = math.MaxInt32
    }

    for i := 0; i < n; i++ {
        for step := 1; step <= nums[i] && i+step < n; step++ {
            target := i + step
            if f[i]+1 < f[target] {
                f[target] = f[i] + 1 // 從 i 跳過來
            }
        }
    }

    return f[n-1]
}
```

</details>

---

## 解法二：兩個邊界，一趟掃完

Step 4 講的「層」，換成看月台的轉乘看板想：人一直站在這一程的起點，還沒真的移動。看板列出這一程能到的每一站，每一站旁邊寫著「在這裡轉乘，下一程最遠能到哪」。`legEnd` 是看板列到底的那一站，也是這一程實際能到的最後一站；`nextLegFar` 是看板上目前讀到、轉乘後能到最遠的那一站。

```typescript
function jump(nums: number[]): number {
    let jumps = 0;
    let legEnd = 0;       // 看板讀完、這一程實際能到的最後一站
    let nextLegFar = 0;   // 看板上讀到、轉乘後能到最遠的站

    for (let i = 0; i < nums.length - 1; i++) {
        nextLegFar = Math.max(nextLegFar, i + nums[i]);   // 讀這一站的看板：在這裡轉乘，最遠能到哪

        if (i === legEnd) {                    // 看板讀完了，正式跳過去
            jumps++;
            legEnd = nextLegFar;                // 跳到看板上最遠的那一站，不是一站一站搭過去
        }
    }

    return jumps;
}
```

- Time: $O(n)$ — 一個迴圈
- Space: $O(1)$ — 三個變數

**走一遍。** `[2, 3, 1, 1, 4]`，迴圈跑 i = 0 到 3：

| i | nums[i] | i + nums[i] | nextLegFar | i === legEnd？ | jumps | legEnd |
|---|---|---|---|---|---|---|
| 起始 | | | 0 | | 0 | 0 |
| 0 | 2 | 2 | 2 | 是（legEnd=0） | 1 | 2 |
| 1 | 3 | 4 | 4 | 否 | 1 | 2 |
| 2 | 1 | 3 | 4 | 是（legEnd=2） | 2 | 4 |
| 3 | 1 | 4 | 4 | 否 | 2 | 4 |

答案 2。

i=1 那一列很重要：`nextLegFar` 被推到 4 了，但 `jumps` 沒有加。因為看板還沒讀完（index 2 那一列還沒讀到），要等整份看板讀完才正式跳一次。

**迴圈為什麼停在 n-2。** 假設最後一站 `n-1` 剛好等於 `legEnd`，那再進去一次就會多算一次轉乘。可是人已經到終點站了，不需要再轉車。跑到 `n-2` 為止，剛好在抵達的那一刻停手。

用 `[2, 1]` 檢查：迴圈只跑 i=0，`nextLegFar = 2`，`i === legEnd`，`jumps` 變 1。答案 1，正確。如果迴圈跑到 i=1，`i === legEnd`（legEnd 已經是 2 嗎，不是，legEnd=2 而 i=1）不成立，這組看不出差別。換 `[1, 1]`：跑到 i=0，nextLegFar=1，jumps=1，legEnd=1；如果多跑 i=1，`i === legEnd` 成立，jumps 會變成 2，但正確答案是 1。

**為什麼「這一程要在哪一站轉乘」不用管。** 這是這題最容易卡住的地方。看板上這一程列出 index 1 跟 index 2 兩站，直覺會想「該在哪一站轉車」。答案是都不用選：下一程能到的範圍是看板上每一站轉乘出去的聯集，`nextLegFar` 已經把聯集裡最遠的那站記下來了。而範圍是連續的一段，所以最遠那站就描述了整程。反正最後是直接跳到 `nextLegFar` 那一站，不是真的一站一站搭過去，看板上哪一站對應這個最遠值，對轉乘次數沒有影響。

<details>
<summary>Go 版本</summary>

```go
func jump(nums []int) int {
    jumps, legEnd, nextLegFar := 0, 0, 0

    for i := 0; i < len(nums)-1; i++ {
        if i+nums[i] > nextLegFar {
            nextLegFar = i + nums[i] // 讀這一站的看板：在這裡轉乘，最遠能到哪
        }

        if i == legEnd { // 看板讀完了，正式跳過去
            jumps++
            legEnd = nextLegFar // 跳到看板上最遠的那一站，不是一站一站搭過去
        }
    }

    return jumps
}
```

</details>

---

**Overthinking**

**跟 [55 Jump Game](/problem/jump-game) 的關係。** 55 只問到不到得了，追一個 `reach` 就夠；這題要數次數，所以把 `reach` 拆成兩個：`legEnd`（這一程能到的最後一站）跟 `nextLegFar`（轉乘後最遠能到哪）。

| | 55 Jump Game | 45 Jump Game II |
|---|---|---|
| 問什麼 | 到不了到得了 | 最少跳幾次 |
| 追幾個變數 | `reach` 一個 | `legEnd`、`nextLegFar` 兩個 |
| 失敗條件 | `i > reach` 就回傳 false | 沒有失敗，題目保證到得了 |
| 為什麼夠 | 到得了 i 就夠，怎麼到的不重要 | 這一程最遠能到哪就代表整程，在哪一站轉乘不重要 |

兩題共用同一個理由：一個數字概括了所有歷史。55 是「能到多遠」，45 是「這一程能到多遠」。

**這是 BFS，只是沒有 queue。** 把每格當節點、能跳到的格子當鄰居，這題就是無權圖的最短路徑，標準解是 BFS。之所以不用 queue，是因為每一層剛好是連續的一段 index，用左右界描述就行。真的寫 BFS 也會過，只是多花一個 queue 的空間。

**題目改成「不保證到得了」呢。** 那就要加 55 的檢查：掃到某個 `i > nextLegFar` 的時候回傳 -1。這樣一份 code 就同時回答了兩題。

---

## 解法比較表

| 解法 | Time | Space | n=10^4 隨機輸入 | n=10^4 最壞輸入 | 備註 |
|---|---|---|---|---|---|
| DP 填表 | $O(n^2)$ | $O(n)$ | 4,787,239 次，0.48 秒 | 49,995,000 次，5 秒 | 最壞情況 TLE |
| 兩個邊界 | $O(n)$ | $O(1)$ | 9,999 次，0.001 秒 | 9,999 次，0.001 秒 | 面試要的答案 |

次數是實際跑計數器數出來的，秒數是次數除以 $10^7$。隨機輸入是 `nums[i]` 取 1 到 1000 的均勻分布，最壞輸入是每格都刻 9999。兩個邊界那版的次數跟輸入內容無關，因為它只跑一趟。

---

## 結論

跳 k 次能到的格子剛好是連續的一段，所以每一程只要記最後一站（`legEnd`）。看板讀到 `legEnd` 就正式轉乘一次，跳到看板上記錄最遠的那一站（`nextLegFar`）。在哪一站轉乘不用選，因為下一程的範圍是這一程每一站轉乘出去的聯集。
