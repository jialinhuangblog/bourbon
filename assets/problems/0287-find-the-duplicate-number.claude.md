array 有 n+1 個整數，每個都在 `[1, n]` 之間，**恰好一個值**重複（可能重複很多次）。找出那個重複值。限制：**不能改 array**、**O(1) 額外空間**。

---

**解題引導**

用 `nums = [1, 3, 4, 2, 2]` 想。答案是 2。

**Step 1：第一個浮現的工具是什麼？**

*duplicates… set, obviously.*

<span class="spoiler">用 HashSet 邊掃邊看，已經出現過就是答案。Use a hash set and flag the first repeat.</span>

**Step 2：題目要 O(1) 空間，又不能改 array，HashSet 跟排序都死掉。剩什麼？**

*two doors locked, find a third.*

<span class="spoiler">換維度。不在 array 上找答案，在「值的範圍 1 到 n」上找。Search the value space, not the index space.</span>

**Step 3：對某個 mid 值，怎麼用一次掃描判斷重複的在 mid 左邊還是右邊？**

*pigeonhole sliced.*

<span class="spoiler">數 array 裡 ≤ mid 的有幾個。沒重複的話應該剛好 mid 個。多出來代表重複落在 [1, mid]。Count how many array values are ≤ mid. If unique, exactly mid. More than mid means the duplicate sits in the lower half.</span>

**Step 4：O(n log n) 還能再降。把 array 看成函數 f(i) = nums[i]，從 index 0 出發一直跳，會發生什麼？**

*142 vibes.*

<span class="spoiler">一定會掉進環。環入口就是重複值。用 Floyd 龜兔。The sequence must loop; the cycle entrance is the duplicate. Apply Floyd.</span>

**Step 5：為什麼一定有環？為什麼環入口是重複值？**

*pigeonhole again, plus two arrows merging into one node.*

<span class="spoiler">n+1 個 index 對應 n 種值，鴿籠原理逼出兩個 index 指到同一個值。那個值有兩個前驅，是兩條路的合流點，也就是環的入口。n+1 indices map into n distinct values, so two arrows must collide on the same node. That collision point is the cycle entrance and the duplicate.</span>

想完再往下看 code。

---

## 解法一：HashSet

最直覺：HashSet。

```typescript
function findDuplicate(nums: number[]): number {
    const seen = new Set<number>();
    for (const v of nums) {
        if (seen.has(v)) return v;
        seen.add(v);
    }
    return -1;
}
```

- Time: O(n)
- Space: **O(n)**

<details>
<summary>Go 版本</summary>

```go
func findDuplicate(nums []int) int {
    seen := map[int]bool{}
    for _, v := range nums {
        if seen[v] {     // 看過了就是答案
            return v
        }
        seen[v] = true
    }
    return -1
}
```

</details>

題目就卡在這個 O(n) 空間。**Space 違規。** 這條路被堵死。

---

## 解法二：排序

排完之後 `nums[i] == nums[i+1]` 的就是答案。

```typescript
function findDuplicate(nums: number[]): number {
    nums.sort((a, b) => a - b);
    for (let i = 1; i < nums.length; i++) {
        if (nums[i] === nums[i - 1]) {
            return nums[i];
        }
    }
    return -1;
}
```

- Time: O(n log n)
- Space: O(1)（in-place sort）

<details>
<summary>Go 版本</summary>

```go
func findDuplicate(nums []int) int {
    sort.Ints(nums)
    for i := 1; i < len(nums); i++ {
        if nums[i] == nums[i-1] {
            return nums[i]
        }
    }
    return -1
}
```

</details>

但 `sort.Ints` 直接改 array。**題目說不能改。** 又被堵死。

兩條最直覺的路一條卡空間、一條卡修改。

---

## 解法三：二分搜值

先丟掉一個反射：看到二分搜就想「在排好序的 array 上切左右段」。這裡的二分**完全不碰 array 的位置**，array 沒排序、沒被切段、每輪都整條從頭掃到尾。被切的是另一個東西：「答案可能是哪個數」的範圍 `[1, n]`。

想成宿舍點名。房號 1 到 n，住了 n+1 個人，每人身上的號碼牌就是 nums 裡的值。挑一個分界 mid 問：「號碼牌 ≤ mid 的，舉手。」房號 1 到 mid 這段的正常容量是 mid 個人，比對舉手數（count）：

- `count > mid`：這段超收了，重複號碼在 `[1, mid]`
- `count <= mid`：這段沒超收，超收的在另一段，重複號碼在 `[mid+1, n]`

每問一次，候選房號砍半。array 的 index 全篇只出現在一個地方：數舉手的那行 `for (const value of nums)`，而且每輪都整條全掃，不切。所以 code 裡 `far = mid` 的意思是「答案是更大的數的可能性沒了」，不是「array 後半段不看了」；重複的人站在隊伍哪個位置都躲不掉點名。

用 `nums = [1, 3, 4, 2, 2]`，n = 4，候選房號 `[1, 4]`：

```
near=1, far=4   候選值 1, 2, 3, 4
─────────────────────────────────────────────────
mid=2  count(≤2) = 3   3 > 2 → 超收，重複值 ≤ 2，far=2    候選縮成 1, 2
mid=1  count(≤1) = 1   1 <= 1 → 沒超收，重複值 > 1，near=2  候選只剩 2
near == far == 2，答案 2
```

順序完全不影響：`[2, 1, 2, 3, 4]`、`[4, 3, 2, 2, 1]` 每輪的 count 一模一樣，因為「≤ mid 的有幾個」是集合層面的問題。重複值在頭、在尾、分居兩端，都一樣。

> 用 `min, max` 命名會誤導——那兩個字在演算法語境裡幾乎等於「排好序後的兩端」，喚起「array 排序過、要找邊界」的反射。這題 array 沒排序，搜的也不是 array，所以避開這對名字。

```typescript
function findDuplicate(nums: number[]): number {
    let near = 1, far = nums.length - 1;
    while (near < far) {
        const mid = (near + far) >> 1;
        let count = 0;
        for (const value of nums) {
            if (value <= mid) count++;
        }
        if (count > mid) far = mid;
        else near = mid + 1;
    }
    return near;
}
```

- Time: O(n log n) — 二分 log n 輪，每輪掃 n
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func findDuplicate(nums []int) int {
    near, far := 1, len(nums)-1   // 候選重複值範圍的兩端
    for near < far {
        mid := (near + far) / 2
        count := 0
        for _, value := range nums {   // 數 array 裡有幾個值 ≤ mid
            if value <= mid {
                count++
            }
        }
        if count > mid {               // 多出來，重複值在 near 那一側
            far = mid
        } else {                       // 沒多出來，重複值在 far 那一側
            near = mid + 1
        }
    }
    return near
}
```

</details>

合法解。但還能更快。

---

## 解法四：Floyd 龜兔

先換個想像。把 5 個位置想成 **5 個寶箱**（編號 0 到 4），每個寶箱裡一張紙條，寫著下一個要去的寶箱編號。翻回程式語言：紙條就是 `nums[i]`，`nums[...]` 就是這題的 `.next`。

```
i        0  1  2  3  4
nums[i]  1  3  4  2  2

從寶箱 0 出發照紙條走：0 → 1 → 3 → 2 → 4 → 2 → 4 → ...
                                  └────┐
                                       └─→ 環：2 → 4 → 2
```

為什麼必有環、環的入口必是重複值？三個觀察：

- 值域是 1 到 n：跳到的永遠是合法 index，而且沒有紙條寫 0，**0 一定在環外**，是安全起點。寶箱有限、每箱只有一張紙條，走下去遲早繞進圈裡出不來。
- 鴿籠原理：n+1 個寶箱的紙條只有 n 種寫法，至少兩張紙條寫同一個編號。這裡 `nums[3] = nums[4] = 2`，兩條箭頭合流到寶箱 2。
- 合流點就是環的入口，而「兩條箭頭指到 X」跟「值 X 出現兩次」是同一件事的兩種說法。**環入口的編號 = 重複值**。

問題從「找重複數字」變成「找環的入口」，套 [142 Linked List Cycle II](/problem/linked-list-cycle-ii) 的 Floyd。

**階段一：slow 走 1 步、fast 走 2 步，找相遇點。**

```
step  slow              fast
─────────────────────────────────────
0     0                 0
1     nums[0]=1         nums[nums[0]]=nums[1]=3
2     nums[1]=3         nums[nums[3]]=nums[2]=4
3     nums[3]=2         nums[nums[4]]=nums[2]=4
4     nums[2]=4         nums[nums[4]]=nums[2]=4   ← 相遇在 4
```

**相遇在 4，但入口是 2，所以還需要階段二。** 相遇點只是 fast 套圈追上 slow 的地方，不是答案；階段一真正的產出是一條等式。設 a = 起點到入口的步數、b = 入口到相遇點的步數（跟 [linked-list 概念頁](/concept/linked-list)同一套字母），slow 相遇時走了 a + b 步。fast 走兩倍，多走的必是整數圈，所以 `a + b = nc`（c 是環長、n 是圈數）。也就是**起點到入口的步數 = 相遇點往前走到入口的步數**，差整圈，落點相同。驗：a=3、b=1、c=2，3 + 1 = 2 × 2 ✓。階段二用的就是這條等式：兩個指標同速，分別從起點與相遇點出發，第 a 步必同踩入口，誰都不用知道 a 是多少。

**階段二：slow 移回 index 0，兩個都走 1 步，再次相遇就是入口。**

```
step  slow              fast
─────────────────────────────────────
0     0                 4
1     nums[0]=1         nums[4]=2
2     nums[1]=3         nums[2]=4
3     nums[3]=2         nums[4]=2   ← 相遇在 2，回傳 2
```

答案：2。

code 的兩行指標更新，跟 [141](/problem/linked-list-cycle) 對照著讀：

```
linked list：slow = slow.next          fast = fast.next.next
這題：       slow = nums[slow]         fast = nums[nums[fast]]
```

fast 不是另一種取法，是同一個動作套兩次：裡層 `nums[fast]` 是第一步的落點，外層踩著落點再走第二步（站在 1：`nums[1]=3`、`nums[3]=2`，一行完成兩跳）。

```typescript
function findDuplicate(nums: number[]): number {
    let slow = 0, fast = 0;      // 從 index 0 出發，0 一定在環外（值最小是 1，沒人指回 0）
    do {
        slow = nums[slow];       // 走 1 步
        fast = nums[nums[fast]]; // 走 2 步
    } while (slow !== fast);

    slow = 0;                    // 階段二：slow 移回起點 0
    while (slow !== fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

- Time: O(n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func findDuplicate(nums []int) int {
    // 階段一：找相遇點。從 index 0 出發，0 一定在環外
    slow, fast := 0, 0
    for {
        slow = nums[slow]            // 走 1 步
        fast = nums[nums[fast]]      // 走 2 步
        if slow == fast {
            break
        }
    }

    // 階段二：slow 移回起點 0，找環入口（= 重複值）
    slow = 0
    for slow != fast {
        slow = nums[slow]            // 都走 1 步
        fast = nums[fast]
    }
    return slow
}
```

</details>

階段二為什麼 work、為什麼 fast 一定要 2 倍速，完整推導見 [142 Linked List Cycle II](/problem/linked-list-cycle-ii)。同一套東西，不重複講。

---

**Overthinking**

**為什麼從 index 0 出發、不會自己變環頭？**

因為題目保證 `nums[i] >= 1`，序列第二步開始永遠不會回到 0。0 在環外，當「前段」用。如果題目允許 `nums[i] == 0`，這招就壞——0 可能在環裡，從 0 出發直接掉進環，前段長度 = 0，階段二會立刻回傳 0，沒意義。

**為什麼是「環入口」而不是「環裡某個點」？**

環裡每個節點的入度都是 1（來自環裡的前一個）。環入口的入度是 2：一個來自前段、一個來自環裡的最後一個。所以入口是整張圖裡**唯一**入度大於 1 的節點。而入度大於 1 的節點 ⟺ 有兩個 index 映射到它 ⟺ 重複值。一一對應。

**重複值出現很多次怎麼辦？**

例如 `[3, 3, 3, 3, 3]`。f 從 0 出發：0 → 3 → 3 → 3 → ...，環就是 `3 → 3` 這個自環。階段一相遇點 = 3，階段二一樣回傳 3。答案 3。沒差。

---

## 解法比較表

| 解法 | Time | Space | 改 array | 合法？ |
|---|---|---|---|---|
| HashSet | O(n) | O(n) | 否 | ✗ 空間違規 |
| 排序 | O(n log n) | O(1) | **是** | ✗ 改 array |
| 二分搜值 | O(n log n) | O(1) | 否 | ✓ |
| Floyd | **O(n)** | O(1) | 否 | ✓ 最優 |

二分搜值是好用的「次優解」——思路直觀、好寫、面試講得出。Floyd 是最優，但需要把 array 翻譯成 linked list 那一層抽象。先講二分、再進 Floyd 是順的順序。

---

## 結論

不能改、不能多用空間，就把問題換維度看。二分搜值把搜索空間從 array 換到值域，O(n log n) 解掉。Floyd 把 array 看成 linked list，鴿籠保證有環、入度保證入口是重複值，O(n) 收工。
