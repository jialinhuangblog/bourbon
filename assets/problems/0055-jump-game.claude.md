一串數字。站在 index 0，`nums[i]` 告訴你最多能往前跳幾步。問能不能跳到最後一格。

```
nums = [2, 3, 1, 1, 4]

index:  0  1  2  3  4
        2  3  1  1  4
        ↑  可以跳 1 步到 1，再跳 3 步到 4 → 成功

nums = [3, 2, 1, 0, 4]

index:  0  1  2  3  4
        3  2  1  0  4
                 ↑ 任何路都會經過 index 3，那裡是 0 → 卡死，到不了 4
```

---

**解題引導**

用 `[2, 3, 1, 1, 4]` 想。先別急著最優解，想一下最笨的走法。

**Step 1：站在 index i，下一步有幾種選擇？**

*at position i, i can jump 1, 2, ..., or nums[i] steps.*

<span class="spoiler">nums[i] 種。跳 1 步、跳 2 步…跳 nums[i] 步都行。把每種都試試看，能走到最後就 true。</span>

**Step 2：遞迴地試所有跳法，時間複雜度？**

*exponential. each position branches into nums[i] children.*

<span class="spoiler">O(2ⁿ) 左右。每一格分岔出 nums[i] 條路，指數爆炸。</span>

**Step 3：很多 index 會被重複問「從這裡能不能到？」怎麼辦？**

*memoize the answer at each index.*

<span class="spoiler">memo 陣列存「從 index i 出發能不能到終點」，每格只算一次。O(n²)。</span>

**Step 4：其實根本不用記每格的答案。那要記什麼？**

*only the farthest reachable index matters.*

<span class="spoiler">不用管「從哪裡跳到哪裡」，只要追蹤「目前最遠能到哪」。從左到右走，若 i 超過最遠能到的範圍就 GG。否則更新最遠能到的地方。O(n)。</span>

想完再往下看 code。

---

## 解法一：暴力 DFS

最直覺的做法：站在 i，試 1 步、2 步、…、nums[i] 步，每一種都遞迴下去。

```typescript
function canJump(nums: number[]): boolean {
    function dfs(i: number): boolean {
        if (i >= nums.length - 1) return true;
        for (let step = 1; step <= nums[i]; step++) {
            if (dfs(i + step)) return true;
        }
        return false;
    }
    return dfs(0);
}
```

- **Time: $O(2^n)$** — 最壞情況每格都能往後跳到好幾個位置，組合爆炸
- Space: O(n) — call stack

<details>
<summary>Go 版本</summary>

```go
func canJump(nums []int) bool {
    return dfs(nums, 0)
}

func dfs(nums []int, i int) bool {
    if i >= len(nums)-1 {
        return true // 到了
    }
    for step := 1; step <= nums[i]; step++ {
        if dfs(nums, i+step) {
            return true
        }
    }
    return false
}
```

</details>

很多 index 會被反覆問「從這裡能不能到終點？」。加 memo。

---

## 解法二：Memo

存「從 i 出發能不能到最後」。每個 i 只算一次。

---

把每個位置 i 想成一個**入境關卡**。你身上帶一本**護照**，第 i 頁專門記「從位置 i 出發能不能到終點」。

護照頁有三種狀態：

| 護照頁 | 意思 |
|---|---|
| 空白 | 沒去過位置 i |
| ✓ 入境章 | 去過了——從 i 出發**能**到終點 |
| ✗ 拒絕章 | 去過了——從 i 出發**到不了** |

走到位置 i 的時候：

```
翻護照 i 頁
  有章 ✓ → 不用再驗，直接知道能到
  有章 ✗ → 不用再驗，直接知道死路
  空白 → 海關得逐一驗（嘗試每種跳法）
         驗完蓋章，下次再來秒過
```

**「驗證」就是遞迴探所有跳法**。每個位置只驗一次，後面看章。

---

```typescript
function canJump(nums: number[]): boolean {
    const memo = new Array(nums.length).fill(0); // 0 空白 / 1 ✓ 入境章 / -1 ✗ 拒絕章
    function dfs(i: number): boolean {
        if (i >= nums.length - 1) return true;
        if (memo[i] !== 0) {           // 護照 i 頁有章
            return memo[i] === 1;      // 直接看章結論
        }
        for (let step = 1; step <= nums[i]; step++) {
            if (dfs(i + step)) {
                memo[i] = 1;           // 驗證能到，蓋 ✓
                return true;
            }
        }
        memo[i] = -1;                  // 全部試完都不行，蓋 ✗
        return false;
    }
    return dfs(0);
}
```

<details>
<summary>Go 版本</summary>

```go
func canJump(nums []int) bool {
    memo := make([]int, len(nums)) // 0 空白 / 1 ✓ 入境章 / -1 ✗ 拒絕章
    var dfs func(i int) bool
    dfs = func(i int) bool {
        if i >= len(nums)-1 {
            return true
        }
        if memo[i] != 0 {           // 護照 i 頁有章
            return memo[i] == 1     // 直接看章結論
        }
        for step := 1; step <= nums[i]; step++ {
            if dfs(i + step) {
                memo[i] = 1         // 驗證能到，蓋 ✓
                return true
            }
        }
        memo[i] = -1                // 全部試完都不行，蓋 ✗
        return false
    }
    return dfs(0)
}
```

</details>

為什麼用 0 / 1 / -1 三態？因為 boolean 只有兩態，分不出「沒驗過」跟「驗過是 false」——看到 false 不知道該秒回還是該重驗。三態 int 用 0 表「空白」剛好搭 Go 的預設值，1 跟 -1 表已驗證的兩種結論。

- Time: $O(n^2)$ — n 個 index，每個最多試 n 種跳法
- Space: O(n)

從 $2^n$ 砍到 $n^2$。但 $n^2$ 還是吃，n = $10^4$ 會跑一億次。再看仔細一點。

---

## 解法三：Greedy 省空間

觀察 `[3, 2, 1, 0, 4]` 為什麼失敗 — 卡在 index 3 的 0。從 0 出發能到的最遠是 `0 + 3 = 3`，從 1 能到 `1 + 2 = 3`，從 2 能到 `2 + 1 = 3`，從 3 能到 `3 + 0 = 3`。**四個位置能到的最遠都是 3**，永遠超不過。index 4 碰不到，結束。

抓到了：**只要追蹤「目前能到的最遠 index」**。一路往右掃，如果 `i` 超過這個最遠值，代表根本走不到 `i`，失敗。否則更新最遠值成 `max(最遠, i + nums[i])`。

```typescript
function canJump(nums: number[]): boolean {
    let reach = 0; // 目前能到達的最遠 index
    for (let i = 0; i < nums.length; i++) {
        if (i > reach) return false; // 連 i 都到不了，後面免談
        if (i + nums[i] > reach) reach = i + nums[i]; // 從 i 出發能跳到的最遠，更新紀錄
    }
    return true;
}
```

<details>
<summary>Go 版本</summary>

```go
func canJump(nums []int) bool {
    reach := 0 // 目前能到達的最遠 index
    for i := 0; i < len(nums); i++ {
        if i > reach {
            return false // 連 i 都到不了，後面免談
        }
        if i+nums[i] > reach {
            reach = i + nums[i] // 從 i 出發能跳到的最遠，更新紀錄
        }
    }
    return true
}
```

</details>

- Time: O(n) — 一次掃完，每格看一次
- Space: O(1)

用 `[2, 3, 1, 1, 4]` 走一遍：

```
reach = 0

i=0: 0 <= 0 ✓；0+2=2 > 0，reach 更新成 2
i=1: 1 <= 2 ✓；1+3=4 > 2，reach 更新成 4
i=2: 2 <= 4 ✓；2+1=3，不大於 4，reach 不變
i=3: 3 <= 4 ✓；3+1=4，不大於 4，reach 不變
i=4: 4 <= 4 ✓，迴圈結束

回傳 true ✓
```

用 `[3, 2, 1, 0, 4]` 驗證失敗情境：

```
reach = 0

i=0: 0 <= 0 ✓；0+3=3 > 0，reach 更新成 3
i=1: 1 <= 3 ✓；1+2=3，不大於 3，reach 不變
i=2: 2 <= 3 ✓；2+1=3，不大於 3，reach 不變
i=3: 3 <= 3 ✓；3+0=3，不大於 3，reach 不變  ← reach 卡在 3，之後每一格都追不上，還沒 return
i=4: 4 > 3 ✗ → return false ✓
```

迴圈跑到一半就 `return false` 提前結束，不用跑完整個陣列。這就是 greedy。

---

**為什麼不用窮舉每條路徑？**

關鍵：**「能到 i」這件事，跟「怎麼到 i」無關**。只要 `reach >= i` 就代表某條路徑能走到 i，至於是跳兩步還是三步過來，不影響之後能到多遠。reach 只在乎「最遠」，不在乎歷史。

這是 greedy 能成立的原因：後面的決策只需要一個數字（reach），不需要整段走法。

換個講法：如果 i 可以到，且 i 的 nums[i] = 5，那 i+5 也一定可以到，不管你是從 i-1 跳過來還是從 i-3 跳過來。所以記「能到的最遠」就是全部資訊。

---

### 反向 greedy（同樣 O(n)，另一種腦袋）

換個比喻：把每一格想成**車站**，每站告示牌寫著「**從這站最遠能開到第 N 站之後**」。你要從第 0 站搭到最後一站。

正向解法是「從第 0 站出發、每站推到最遠」。反向解法是「**從終點倒著找代理人**」：

> 第 4 站要找個更早的代理人——**誰能直接開到第 4 站？** 找到後，那個代理人**變成新的目標**——再找誰能開到它。一路往前推，最後看第 0 站能不能加入這條鏈。

維護一個變數 `goal`，代表「**目前要追的目標站**」。倒著掃，誰能搭到 goal，goal 就變成那個人。

```typescript
function canJump(nums: number[]): boolean {
    let goal = nums.length - 1;
    for (let i = nums.length - 2; i >= 0; i--) {
        if (i + nums[i] >= goal) {
            goal = i; // i 能搭到 goal，i 接過接力棒成為新 goal
        }
    }
    return goal === 0;
}
```

<details>
<summary>Go 版本</summary>

```go
func canJump(nums []int) bool {
    goal := len(nums) - 1
    for i := len(nums) - 2; i >= 0; i-- {
        if i+nums[i] >= goal {
            goal = i // i 能搭到 goal，i 接過接力棒成為新 goal
        }
    }
    return goal == 0
}
```

</details>

---

**為什麼可以這樣轉移目標？**

因為「能搭到 goal」跟「能搭到能直接到 goal 的代理人」是**同一件事**。

如果第 i 站能直接搭到 goal，那只要我能搭到 i，我就有「直接開到 goal」這張車票。後面要不要追到 goal，**取決於有沒有辦法搭到 i**——這變成更小的子問題。

把這個道理一直往前推：

```
能搭到第 4 站 ⟺ 能搭到第 3 站
              ⟺ 能搭到第 2 站
                ⟺ 能搭到第 1 站
                  ⟺ 能搭到第 0 站
                    ⟺ true
```

每次「找到代理人」就把目標往前推一格。最後 goal 推回 0 = 整條接力鏈通了。

---

用 `[2, 3, 1, 1, 4]`：

```
goal = 4（要追到第 4 站）

i=3: 第 3 站告示牌 1，3+1 = 4 >= goal=4 ✓
     第 3 站能直接開到第 4 站 → goal = 3
i=2: 第 2 站告示牌 1，2+1 = 3 >= goal=3 ✓
     第 2 站能開到第 3 站 → goal = 2
i=1: 第 1 站告示牌 3，1+3 = 4 >= goal=2 ✓
     第 1 站能開到第 2 站（甚至更遠） → goal = 1
i=0: 第 0 站告示牌 2，0+2 = 2 >= goal=1 ✓
     第 0 站能開到第 1 站 → goal = 0

goal == 0 → 整條鏈通 → true ✓
```

失敗例 `[3, 2, 1, 0, 4]`：

```
goal = 4

i=3: 3+0 = 3 < 4 ✗  goal 不變（沒人是代理人）
i=2: 2+1 = 3 < 4 ✗
i=1: 1+2 = 3 < 4 ✗
i=0: 0+3 = 3 < 4 ✗

goal == 4 ≠ 0 → false ✓
```

第 3 站告示牌 0，整條路被那個 0 卡死，從第 4 站倒著找**沒人能搭到它**。goal 永遠卡在 4。

一樣是 O(n) / O(1)。方向不同，結論一樣。

---

**Overthinking：如果題目問的是「最少幾步」？**

那就是 [45 Jump Game II](/problem/jump-game-ii)。這題只問「能不能」，greedy 一個變數就夠；45 要計步數，要追兩個邊界：當前這一「層」能到的最遠、下一「層」能到的最遠。每次當前層走完了，步數 +1，邊界推進到下一層。BFS 分層思維，只是用兩個指標而不是 queue。

---

## 結論

| 解法 | Time | Space |
|---|---|---|
| 暴力 DFS | $O(2^n)$ | O(n) |
| Memo | $O(n^2)$ | O(n) |
| Greedy（reach） | O(n) | O(1) |

Greedy 能把 $n^2$ 砍到 n 的原因：「能到 i」足以決定後面所有事，不需要記「怎麼到 i」。一個數字概括所有歷史。遇到這種「只要 reachable 就行，不管路徑」的題，先懷疑 greedy。
