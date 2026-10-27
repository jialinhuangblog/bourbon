給一個已經排好序的陣列和 target，找兩個數加起來等於 target，回傳 1-indexed 的位置。

---

**解題引導**

用這組輸入想想看：`numbers = [1, 3, 4, 5, 7, 10, 11]`，`target = 9`。

**Step 1：最笨的方式怎麼做？**
<span class="spoiler">每個數字跟後面每個數字配對試試看，O(n²)。</span>

**Step 2：陣列已經排好序了，這個資訊能幹嘛？**
<span class="spoiler">如果兩個數加起來太大，大的那個一定要換小一點的。太小，小的那個要換大一點的。</span>

**Step 3：所以指標要怎麼放？**
<span class="spoiler">一左一右，從兩端往中間夾。太大右邊左移，太小左邊右移。</span>

**Step 4：為什麼這樣不會漏掉答案？**
<span class="spoiler">每次移動都排除了一整列不可能的配對。左移右指標 = 排除所有跟右指標配對的組合（因為連最小的左端都太大了）。右移左指標同理。</span>

想完再往下看 code。

---

## 解法一：暴力（兩層迴圈）

最直覺：兩層迴圈，每對都試。

用 `[1, 3, 4, 5, 7, 10, 11]`，`target = 9` 走一遍：

```
i=0: 1+3=4, 1+4=5, 1+5=6, 1+7=8, 1+10=11, 1+11=12
i=1: 3+4=7, 3+5=8, 3+7=10, 3+10=13, 3+11=14
i=2: 4+5=9 ✓ → 回傳 [3, 4]（1-indexed）
```

找到了，但前面試了 8 組才碰到。

```typescript
function twoSum(numbers: number[], target: number): number[] {
    for (let i = 0; i < numbers.length; i++) {
        for (let j = i + 1; j < numbers.length; j++) { // 每對都試
            if (numbers[i] + numbers[j] === target) {
                return [i + 1, j + 1]; // 題目要 1-indexed
            }
        }
    }
    return [];
}
```

- **Time: $O(n^2)$**
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func twoSum(numbers []int, target int) []int {
    for i := 0; i < len(numbers); i++ {
        for j := i + 1; j < len(numbers); j++ { // 每對都試
            if numbers[i]+numbers[j] == target {
                return []int{i + 1, j + 1} // 題目要 1-indexed
            }
        }
    }
    return nil
}
```

</details>

能跑，但完全沒用到「已排序」這個條件。白白浪費了題目給的資訊。

---

## 解法二：雙指針

陣列已經排好序了。這代表什麼？

兩個指標，一個從最左邊（最小），一個從最右邊（最大）。加起來看：
- 太大 → 右邊往左移（讓 sum 變小）
- 太小 → 左邊往右移（讓 sum 變大）
- 剛好 → 找到了

用同一組 `[1, 3, 4, 5, 7, 10, 11]`，`target = 9` 走一遍：

```
L=0, R=6: numbers[0]+numbers[6] = 1+11 = 12 > 9 → R 左移
L=0, R=5: numbers[0]+numbers[5] = 1+10 = 11 > 9 → R 左移
L=0, R=4: numbers[0]+numbers[4] = 1+7  = 8  < 9 → L 右移
L=1, R=4: numbers[1]+numbers[4] = 3+7  = 10 > 9 → R 左移
L=1, R=3: numbers[1]+numbers[3] = 3+5  = 8  < 9 → L 右移
L=2, R=3: numbers[2]+numbers[3] = 4+5  = 9  = 9 ✓ → 回傳 [3, 4]
```

6 步就找到。每一步都淘汰了一整行（或一整列）的配對，不是只淘汰一個。

**為什麼不會漏掉答案？** 當 `1+11=12 > 9`，右指標左移。這等於宣告：11 跟左邊任何數配對都 ≥ 12（因為左邊的數只會更大），所以 11 不可能是答案的一部分。整列淘汰，不是猜。

```typescript
function twoSum(numbers: number[], target: number): number[] {
    let l = 0, r = numbers.length - 1;
    while (l < r) {
        const sum = numbers[l] + numbers[r];
        if (sum === target) {
            return [l + 1, r + 1]; // 1-indexed
        } else if (sum < target) {
            l++;
        } else {
            r--;
        }
    }
    return [];
}
```

- Time: O(n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func twoSum(numbers []int, target int) []int {
    l, r := 0, len(numbers)-1
    for l < r {
        sum := numbers[l] + numbers[r]
        if sum == target {
            return []int{l + 1, r + 1} // 1-indexed
        } else if sum < target {
            l++ // 太小，左邊往右
        } else {
            r-- // 太大，右邊往左
        }
    }
    return nil
}
```

</details>

---

## 解法三：二分搜

你可能想：既然排好序了，用 binary search 不是更快？

對每個 `numbers[i]`，binary search `target - numbers[i]`。

```typescript
function twoSum(numbers: number[], target: number): number[] {
    for (let i = 0; i < numbers.length; i++) {
        const complement = target - numbers[i];
        // 在 i+1 ~ end 之間二分搜
        let lo = i + 1, hi = numbers.length - 1;
        while (lo <= hi) {
            const mid = lo + Math.floor((hi - lo) / 2);
            if (numbers[mid] === complement) {
                return [i + 1, mid + 1];
            } else if (numbers[mid] < complement) {
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
    }
    return [];
}
```

- Time: O(n log n) — 外層 O(n)，每次 binary search O(log n)
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func twoSum(numbers []int, target int) []int {
    for i := 0; i < len(numbers); i++ {
        complement := target - numbers[i]
        // 在 i+1 ~ end 之間二分搜
        lo, hi := i+1, len(numbers)-1
        for lo <= hi {
            mid := lo + (hi-lo)/2
            if numbers[mid] == complement {
                return []int{i + 1, mid + 1}
            } else if numbers[mid] < complement {
                lo = mid + 1
            } else {
                hi = mid - 1
            }
        }
    }
    return nil
}
```

</details>

看起來很聰明，但 O(n log n) 比雙指標的 O(n) 慢。而且 code 更長。Binary search 在這題是殺雞用牛刀。

**那 hash map 呢？** 跟 [1 Two Sum](/problem/two-sum) 一樣用 map，O(n) 時間。但 Space O(n)，題目明確要求 constant extra space。違規。

---

**Overthinking：為什麼題目特別強調 constant extra space？**

因為這題就是在考你能不能利用「已排序」這個條件。如果允許 O(n) space，hash map 直接秒殺，跟 1 Two Sum 一模一樣，出這題就沒意義了。

面試官想看的是：你拿到額外資訊（排序）時，能不能換一個更精準的工具（雙指標），而不是無腦套萬用解法（hash map）。

---

## 結論

已排序 + constant space = 雙指標。O(n) 時間 O(1) 空間。比 hash map 更好，因為省了空間。比 binary search 更好，因為更快。排序是白送的線索，別浪費。
