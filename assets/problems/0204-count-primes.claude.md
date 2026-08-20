給定整數 n，回傳嚴格小於 n 的質數個數。

---

## 解法一：試除法

最直覺的做法：對每個數 i（從 2 到 n-1），用試除法判斷它是不是質數。只要能被 2 到 $\sqrt{i}$ 之間任何一個數整除，就不是質數。

```typescript
function countPrimes(n: number): number {
    const isPrime = (num: number): boolean => {
        if (num < 2) return false;
        for (let i = 2; i * i <= num; i++) {
            if (num % i === 0) return false; // 找到因數，不是質數
        }
        return true;
    };

    let count = 0;
    for (let i = 2; i < n; i++) {
        if (isPrime(i)) count++;
    }
    return count;
}
```

<details>
<summary>Go 版本</summary>

```go
func countPrimes(n int) int {
    isPrime := func(num int) bool {
        if num < 2 {
            return false
        }
        for i := 2; i*i <= num; i++ {
            if num%i == 0 {
                return false // 找到因數，不是質數
            }
        }
        return true
    }

    count := 0
    for i := 2; i < n; i++ {
        if isPrime(i) {
            count++
        }
    }
    return count
}
```

</details>


- Time: **$O(n\sqrt{n})$**
- Space: $O(1)$

為什麼只跑到 $\sqrt{num}$？因數是成對出現的。36 的因數配對：$2 \times 18$、$3 \times 12$、$4 \times 9$、$6 \times 6$。每對裡較小的那個最大是 6，也就是 $\sqrt{36}$。找不到比 $\sqrt{num}$ 小的因數，就代表一定沒有因數。

n 最大是 $5 \times 10^6$。每個數都跑一次 $O(\sqrt{n})$，總共 $O(n\sqrt{n}) \approx 1.1 \times 10^{10}$ 次操作。超時。

---

## 解法二：埃氏篩

試除法的問題在於：同樣的合數被篩掉很多次。比如 12，被 2 篩一次、被 3 再篩一次。做了重複的工。

**不要去「問」每個數是不是質數，改成「宣告」它的倍數都是合數。**

這就是埃拉托斯特尼篩法（Sieve of Eratosthenes）。建一個 boolean 陣列，預設全是質數。找到一個質數 p，就把 $p^2$、$p^2+p$、$p^2+2p$…… 全部標記成合數。

為什麼從 $p^2$ 開始？拿 p = 5 那一輪看它前面的倍數：

```
10 = 2 × 5   p=2 那一輪標過了
15 = 3 × 5   p=3 那一輪標過了
20 = 4 × 5   4 = 2×2，p=2 那一輪標過了
25 = 5 × 5   第一個沒被標過的
```

規律在「第幾倍」上。5 的每個倍數都寫成 $k \times 5$，k 就是第幾倍：10 是 k=2、15 是 k=3、20 是 k=4、25 是 k=5。只要 k 比 5 小，k 自己一定含著比 5 小的質因數（k=4 含著 2），所以 $k \times 5$ 早在那個小質數的回合被標了。第一個輪得到 5 出手的，是 k 追上 5 的那個，也就是 $5 \times 5$。換成任何質數 p 都同一條規律：k < p 的倍數全是別人標過的，起點是 $p \times p$。p = 2 是邊界：它前面沒有可跳的東西（2×1 = 2 是它自己），第一個合數倍數本來就是 4 = 2²，規則照走，只是跳過的數量是零。

```typescript
function countPrimes(n: number): number {
    if (n <= 2) return 0;

    const sieve = new Array(n).fill(true); // 先假設全是質數
    sieve[0] = sieve[1] = false; // 0 和 1 不是質數

    // 外層跑到 √n 就夠：合數必有 ≤ √n 的質因數，跟解法一只試到 √num 是同一件事
    for (let i = 2; i * i < n; i++) {
        if (sieve[i]) {
            // 內層從 i² 起跳：更小的倍數早在 i 等於更小質數的回合標過了
            for (let j = i * i; j < n; j += i) {
                sieve[j] = false; // i 的倍數，一定不是質數
            }
        }
    }

    return sieve.filter(Boolean).length;
}
```

<details>
<summary>Go 版本</summary>

```go
func countPrimes(n int) int {
    if n <= 2 {
        return 0
    }

    sieve := make([]bool, n)
    for i := range sieve {
        sieve[i] = true // 先假設全是質數
    }
    sieve[0], sieve[1] = false, false // 0 和 1 不是質數

    // 外層跑到 √n 就夠：合數必有 ≤ √n 的質因數，跟解法一只試到 √num 是同一件事
    for i := 2; i*i < n; i++ {
        if sieve[i] {
            // 內層從 i² 起跳：更小的倍數早在 i 等於更小質數的回合標過了
            for j := i * i; j < n; j += i {
                sieve[j] = false // i 的倍數，一定不是質數
            }
        }
    }

    count := 0
    for _, v := range sieve {
        if v {
            count++
        }
    }
    return count
}
```

</details>


- Time: $O(n \log \log n)$
- Space: $O(n)$

為什麼外層只跑到 $\sqrt{n}$？外層每個 i，內層把 i 的倍數全部標記掉。當 `i*i >= n`，內層從 $i^2$ 開始已經超出範圍，什麼都標記不到，白跑。以 n = 100 為例：i = 11 時，內層從 121 開始，121 > 100，直接跳過。所以外層跑到 $\sqrt{n} = 10$ 就夠了。

---

$O(n \log \log n)$ 到底多快？$\log \log n$ 又是什麼東西？

$\log n$ 的直覺是「n 有幾位數」，那是底 10 的講法。底可以隨便挑，拿底 2 跟底 10 演一次就看得出來。$2^{3.32} = 10$，所以：

$$1000 = 10^3 = (2^{3.32})^3 = 2^{3 \times 3.32} = 2^{9.96}$$

$\log_{10} 1000 = 3$，$\log_2 1000 \approx 10$，兩個讀數差 3.32 倍。多驗幾個 n：

| n | $\log_{10} n$ | $\log_2 n$ | 讀數比 |
|---|---|---|---|
| $100$ | 2 | 6.6 | 3.32 |
| $1000$ | 3 | 10.0 | 3.32 |
| $10^6$ | 6 | 19.9 | 3.32 |
| $10^{12}$ | 12 | 39.9 | 3.32 |

n 從一百翻到一兆，讀數比一格都沒動，因為 3.32 只跟「2 要乘幾次才變成 10」有關，跟 n 無關。所以量級的比較用哪個底，結論都一樣。下面的表用 ln（底是 $e \approx 2.718$ 的 log），位數的直覺照樣通。$\log \log n$ 是對「位數」再取一次 log。已經很慢的東西再慢一次，幾乎不動。

用數字感受：

| n | $\ln n$ | $\ln \ln n$ |
|---|---|---|
| $10$ | 2.3 | 0.8 |
| $10^3$ | 6.9 | 1.9 |
| $10^6$ | 13.8 | 2.6 |
| $10^9$ | 20.7 | 3.0 |

（表選 ln，因為後面質數倒數和的定理長在 ln 上。）

n 長了 $10^8$ 倍，ln ln n 只從 0.8 變成 3.0。先取一次 ln 壓成 2.3 和 20.7，再取一次壓成 0.8 和 3.0。兩次壓縮後幾乎不動了。

所以 $n \log \log n$ 嚴格比 $n$ 大，但只大一個很小的倍數，下表 $10^6$ 那行只差 3 倍。放進完整的排序看：

$$n < n \log \log n < n \log n < n\sqrt{n} < n^2 < 2^n$$

| 複雜度 | $n = 10^6$ 時的操作次數 | 對應解法 |
|---|---|---|
| $n$ | 1,000,000 | 線性基準 |
| $n \log \log n$ | 3,000,000 | 篩法，幾乎是線性 |
| $n \log n$ | 20,000,000 | 排序 |
| $n\sqrt{n}$ | 1,000,000,000 | 暴力試除 |
| $n^2$ | 1,000,000,000,000 | |

篩法比排序還快。比暴力試除快 333 倍。跟線性 O(n) 只差 3 倍。

---

為什麼是 $O(n \log \log n)$？每個質數 p，內層大概標記 $n/p$ 個數。把所有質數的工作加起來：

$$\frac{n}{2} + \frac{n}{3} + \frac{n}{5} + \frac{n}{7} + \cdots = n \times \left(\frac{1}{2} + \frac{1}{3} + \frac{1}{5} + \frac{1}{7} + \cdots\right)$$

用 n = 30 走一遍，看每個質數做了多少工：

```
質數 p=2: 標記 4,6,8,10,12,14,16,18,20,22,24,26,28,30  → 14 個（n/2 = 15，跳過 2）
質數 p=3: 標記 9,12,15,18,21,24,27,30                  → 8 個 （n/3 = 10，跳過 3,6）
質數 p=5: 標記 25,30                                    → 2 個 （n/5 = 6，跳過 5,10,15,20）

實際總工作 = 14 + 8 + 2 = 24
模型估計   = n × (1/2 + 1/3 + 1/5) = 30 × 1.03 = 31
```

31 跟 24 差的 7，就是「從 p² 開始」跳過的次數（1 + 2 + 4）。模型用 n/p 算的是每個質數的全部倍數，不管跳過，所以永遠比實際多一點。這對複雜度沒影響：兩邊量級相同，而且 n 越大，跳過的部分佔比越小。

數學上可以證明，所有質數的倒數和 $\frac{1}{2} + \frac{1}{3} + \frac{1}{5} + \frac{1}{7} + \cdots$ 的成長速度是 $\ln \ln n$，精確一點是 $\ln \ln n + 0.26$，這是 Mertens 定理，那個 0.26 有自己的名字：[Meissel–Mertens 常數](https://en.wikipedia.org/wiki/Meissel-Mertens_constant)。所以總工作量是 $O(n \log \log n)$。Big-O 裡的 log 不標底，因為不同底之間只差固定倍數，Big-O 本來就不管常數。

實際上對 $n = 5 \times 10^6$：$\ln(5 \times 10^6) \approx 15.4$，$\ln 15.4 \approx 2.7$，加上定理給的常數 0.26，質數倒數和 $\approx 3.0$。總操作量 $5 \times 10^6 \times 3.0 \approx 1.5 \times 10^7$ 次，除 $10^7$ 大約一秒的上限，而且每次操作只是陣列寫入，實際遠低於一秒。輕鬆通過。

---

## 解法三：線性篩

有一個叫「線性篩」的做法，可以做到嚴格 $O(n)$。每個合數只被它的最小質因數篩掉一次。

用 n = 13 走一遍，照 code 的順序，每一步的暫態都攤開：

```
isComposite 全 false，primes = []

i=2   沒被標過 → 收進 primes = [2]
      p=2：2×2=4 <13 → 標 4；2%2=0，break（p 是 i 的最小質因數）
i=3   沒被標過 → primes = [2,3]
      p=2：3×2=6 → 標 6；3%2≠0，繼續
      p=3：3×3=9 → 標 9；3%3=0，break
i=4   被標過（4 = 2×2），不收
      p=2：4×2=8 → 標 8；4%2=0，break，所以不標 4×3=12
i=5   沒被標過 → primes = [2,3,5]
      p=2：5×2=10 → 標 10；5%2≠0，繼續
      p=3：5×3=15 ≥13，break（超界）
i=6   被標過，不收
      p=2：6×2=12 → 標 12；6%2=0，break
i=7   沒被標過 → primes = [2,3,5,7]
      p=2：7×2=14 ≥13，break
i=8、9、10   都被標過，內圈第一步 i×2 就超界，break
i=11  沒被標過 → primes = [2,3,5,7,11]
      p=2：11×2=22 ≥13，break
i=12  被標過，12×2=24 超界，break

回傳 primes.length = 5（2、3、5、7、11）
```

六個合數 4、6、8、9、10、12 各被標一次，出手的都是它自己的最小質因數。對照 i=4 跟 i=6 兩行就看到分工：12 = 2×2×3，最小質因數是 2，所以 i=4 遇到 `4%2=0` 就停手，不去標 4×3=12，12 留給 i=6 用 p=2 標。要是 i=4 沒停，12 會被標兩次，線性就沒了。

```typescript
function countPrimes(n: number): number {
    if (n <= 2) return 0;

    const primes: number[] = [];
    const isComposite = new Array(n).fill(false);

    for (let i = 2; i < n; i++) {
        if (!isComposite[i]) {
            primes.push(i); // i 是質數
        }
        for (const p of primes) {
            if (i * p >= n) break;
            isComposite[i * p] = true; // 用最小質因數 p 標記合數
            if (i % p === 0) break;    // p 已經是 i 的最小質因數，再繼續會重複篩
        }
    }

    return primes.length;
}
```

<details>
<summary>Go 版本</summary>

```go
func countPrimes(n int) int {
    if n <= 2 {
        return 0
    }

    primes := []int{}
    isComposite := make([]bool, n)

    for i := 2; i < n; i++ {
        if !isComposite[i] {
            primes = append(primes, i) // i 是質數
        }
        for _, p := range primes {
            if i*p >= n {
                break
            }
            isComposite[i*p] = true // 用最小質因數 p 標記合數
            if i%p == 0 {
                break // p 已經是 i 的最小質因數，再繼續會重複篩
            }
        }
    }

    return len(primes)
}
```

</details>


- Time: $O(n)$
- Space: $O(n)$

但線性篩的 $O(n)$ 沒有換到實際速度。在 Node 實測（2026-08，各組跑七個獨立 process 取中位數）：$n = 5 \times 10^6$ 時埃氏篩 36ms、線性篩 45ms；換成 Uint8Array 是 15ms 對 26ms；$n = 5 \times 10^7$ 也是埃氏篩快兩成。原因看內圈就好：線性篩每一步要走訪 primes 陣列、做一次乘法加一次 `%` 取餘數，埃氏篩的內圈只是固定步幅的順序寫入，每一步要做的事少得多。$\log \log n$ 在這個範圍只有 3 左右，線性篩理論上省下的部分，cover 不了它每步製造出來的成本。

---

**Overthinking**

這個篩完再用的套路，站內有一題實戰：[2523 Closest Prime Numbers in Range](/problem/closest-prime-numbers-in-range)，篩到 right 之後掃相鄰的質數對。

如果題目改成「回傳第 k 個質數」（LeetCode 沒有這題，是面試的假想追問）？埃氏篩沒辦法直接用，因為不知道要篩到多少。可以估一個上界：根據質數定理，第 k 個質數約在 $k \ln k$ 附近，取個安全係數往上篩。

如果 n 超級大（比如 $10^{12}$）？$O(n)$ 記憶體放不下。這時候要用「分段篩法」（Segmented Sieve），把範圍切成一塊一塊（每塊 $\sqrt{n}$），用前面篩出的小質數去篩每個區段。

---

## 結論

埃氏篩。$O(n \log \log n)$ 夠快，寫法直覺，就用它。
