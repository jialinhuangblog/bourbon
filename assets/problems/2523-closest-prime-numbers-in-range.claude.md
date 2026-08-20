數線上 10 到 19 這段，住著四個質數：11、13、17、19。問靠得最近的一對是誰。11 跟 13 差 2，17 跟 19 也差 2，平手時拿 num1 比較小的那對，所以答案是 `[11, 13]`。範圍裡湊不出兩個質數的話，回傳 `[-1, -1]`。

翻回 code 術語：給 `[left, right]`，找出範圍內差距最小的質數對，平手取 num1 最小的。`right` 最大 $10^6$。

---

**解題引導**

用 `left = 10, right = 19` 想。

**Step 1：要找「最近的一對」，需要比較所有兩兩組合嗎？**

*the list is already sorted, the closest pair must be neighbors*

<span class="spoiler">不用。質數收集起來本來就由小到大，最近的一對一定是相鄰的兩個，因為跳過中間任何一個質數，距離只會更大。拿到清單掃一遍相鄰對就好。The closest pair must be adjacent in the sorted list.</span>

**Step 2：範圍內的質數怎麼拿？一個數一個數試除，最壞要多少次操作？**

*78498 primes, each paying the full √num*

<span class="spoiler">試除到 √num 可以判斷一個數。但 $10^6$ 以下有 78498 個質數，質數要試滿 $\sqrt{10^6} = 1000$ 次才確定沒有因數，光質數就 $7.8 \times 10^7$ 次，加上合數的零頭大約 $10^8$，十秒上下，超時。Trial division pays √num per prime and the primes alone cost 10^8.</span>

**Step 3：[204 Count Primes](/problem/count-primes) 學過的工具，哪個能一次把範圍內的質數全部標出來？**

*one sieve, then one scan*

<span class="spoiler">埃氏篩。篩到 right 一次完成，之後掃相鄰對。204 的篩原封不動搬過來。Sieve once, scan once.</span>

想完再往下看 code。

---

## 解法一：試除每個數

`isPrime(num)`：從 2 試除到 $\sqrt{num}$，除得盡就是合數。從 left 掃到 right，遇到質數就跟上一個質數比差距。

用 `left = 10, right = 19` 走一遍：

```
x=10  10 % 2 = 0 → 合數
x=11  試 2（11%2=1）、3（11%3=2），4×4=16 > 11 停 → 質數
      prev = -1，還沒有對象，記 prev = 11
x=12  12 % 2 = 0 → 合數
x=13  試 2、3，4×4 > 13 停 → 質數
      gap = 13 - 11 = 2 < ∞ → best = [11,13]、bestGap = 2；prev = 13
x=14、15、16  合數
x=17  試 2、3、4，5×5 > 17 停 → 質數
      gap = 17 - 13 = 4 ≥ 2 → 不換；prev = 17
x=18  合數
x=19  試 2、3、4，同樣停在 5 → 質數
      gap = 19 - 17 = 2，沒有小於 2 → 不換（平手時先掃到的那對留著）

回傳 [11, 13]
```

```typescript
function closestPrimes(left: number, right: number): number[] {
    function isPrime(num: number): boolean {
        if (num < 2) return false;
        for (let i = 2; i * i <= num; i++) {
            if (num % i === 0) return false; // 除得盡，合數
        }
        return true;
    }

    let prev = -1;
    let best: number[] = [-1, -1];
    let bestGap = Infinity;
    for (let x = left; x <= right; x++) {
        if (!isPrime(x)) continue;
        if (prev !== -1 && x - prev < bestGap) { // 嚴格小於才換，平手保住較小的 num1
            bestGap = x - prev;
            best = [prev, x];
        }
        prev = x;
    }
    return best;
}
```

- Time: $O((R - L) \times \sqrt{R})$，最壞 $10^8$ 量級
- Space: $O(1)$

<details>
<summary>Go 版本</summary>

```go
func closestPrimes(left int, right int) []int {
    isPrime := func(num int) bool {
        if num < 2 {
            return false
        }
        for i := 2; i*i <= num; i++ {
            if num%i == 0 {
                return false // 除得盡，合數
            }
        }
        return true
    }

    prev := -1
    best := []int{-1, -1}
    bestGap := 1 << 30
    for x := left; x <= right; x++ {
        if !isPrime(x) {
            continue
        }
        if prev != -1 && x-prev < bestGap { // 嚴格小於才換
            bestGap = x - prev
            best = []int{prev, x}
        }
        prev = x
    }
    return best
}
```

</details>

Step 2 算過的帳：範圍拉滿時大約 $10^8$ 次操作，除 $10^7$ 是十秒上下，超時。慢的來源是每個質數都重新付一次 $\sqrt{num}$ 的試除，同樣的整除資訊被重算了七萬八千次。

---

## 解法二：埃氏篩一次，相鄰掃一遍

質數判定交給 [204](/problem/count-primes) 的埃氏篩：篩到 right，一次拿到整個範圍的質數表。剩下的工作只有掃相鄰對，而 Step 1 說過最近的一對必相鄰。

**加速註記**：質數對的差距下限是 2（孿生質數），唯一差 1 的是 (2, 3)。由小到大掃，(2, 3) 要是在範圍內一定最先遇到，所以 `bestGap` 一旦到 2 就不可能再更小，直接停。

用 `left = 10, right = 19` 照 code 走：

```
篩 0..19，留下質數：2, 3, 5, 7, 11, 13, 17, 19
從 max(10, 2) = 10 開始掃：

x=10  sieve 說合數，略過
x=11  質數；prev = -1，記 prev = 11
x=12  合數
x=13  質數；gap = 2 < ∞ → best = [11,13]、bestGap = 2
      bestGap ≤ 2 → break，17 和 19 連看都不用看

回傳 [11, 13]
```

邊界走一遍：`left = 4, right = 6` 只有 5 是質數，prev 記了 5 但等不到第二個，回傳初始值 `[-1, -1]`。`left = 1, right = 3`：從 2 開始掃，(2, 3) 差 1，設完就 break。

```typescript
function closestPrimes(left: number, right: number): number[] {
    const sieve = new Uint8Array(right + 1).fill(1);
    sieve[0] = 0;
    if (right >= 1) sieve[1] = 0;
    for (let i = 2; i * i <= right; i++) {
        if (sieve[i]) {
            for (let j = i * i; j <= right; j += i) {
                sieve[j] = 0; // 同 204：從 i² 開始標合數
            }
        }
    }

    let prev = -1;
    let best: number[] = [-1, -1];
    let bestGap = Infinity;
    for (let x = Math.max(left, 2); x <= right; x++) {
        if (!sieve[x]) continue;
        if (prev !== -1 && x - prev < bestGap) {
            bestGap = x - prev;
            best = [prev, x];
            if (bestGap <= 2) break; // 孿生質數是下限，後面不可能更近
        }
        prev = x;
    }
    return best;
}
```

- Time: $O(R \log \log R + (R - L))$，$R = 10^6$ 時大約 $4 \times 10^6$ 次，一秒內
- Space: $O(R)$

<details>
<summary>Go 版本</summary>

```go
func closestPrimes(left int, right int) []int {
    sieve := make([]bool, right+1)
    for i := range sieve {
        sieve[i] = true
    }
    sieve[0] = false
    if right >= 1 {
        sieve[1] = false
    }
    for i := 2; i*i <= right; i++ {
        if sieve[i] {
            for j := i * i; j <= right; j += i {
                sieve[j] = false // 同 204：從 i² 開始標合數
            }
        }
    }

    prev := -1
    best := []int{-1, -1}
    bestGap := 1 << 30
    start := left
    if start < 2 {
        start = 2
    }
    for x := start; x <= right; x++ {
        if !sieve[x] {
            continue
        }
        if prev != -1 && x-prev < bestGap {
            bestGap = x - prev
            best = []int{prev, x}
            if bestGap <= 2 {
                break // 孿生質數是下限，後面不可能更近
            }
        }
        prev = x
    }
    return best
}
```

</details>

**為什麼平手不用特別處理？** 由小到大掃，更新條件是嚴格小於。後面出現同樣的差距時條件不成立，先掃到的那對留著，而先掃到的 num1 一定比較小，題目要的就是它。

---

## 解法比較表

| 解法 | Time | Space | R = 10⁶ 最壞操作 | 備註 |
|---|---|---|---|---|
| 試除每個數 | $O((R-L)\sqrt{R})$ | $O(1)$ | ~$10^8$，十秒上下 | 超時 |
| 埃氏篩 + 相鄰掃描 | $O(R \log \log R)$ | $O(R)$ | ~$4 \times 10^6$，一秒內 | 面試要的解 |

## 結論

這題是 [204](/problem/count-primes) 的篩接一段掃描：質數判定交給篩，配對交給「排好序的清單裡，最近的一對必相鄰」這個事實。平手規則不用另外寫，由小到大掃加上嚴格小於，先到的那對自然留下；孿生質數的差距 2 是下限，掃到就能提前停。
