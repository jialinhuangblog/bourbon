給一個 n，生出 $n \times n$ 的矩陣。從 1 數到 $n^2$，繞著外圈螺旋往內填。

```
n = 3

1 → 2 → 3
        ↓
8 → 9   4
↑       ↓
7 ← 6 ← 5
```

---

**解題引導**

用 `n = 3` 想。先在紙上畫一次，體會順序。

**Step 1：繞一圈是哪四個方向？**

*right, down, left, up, then repeat.*

<span class="spoiler">右、下、左、上。四個方向輪流。繞完最外圈進到內圈，再繼續。</span>

**Step 2：什麼時候該轉彎？**

*turn when you hit the edge or a filled cell.*

<span class="spoiler">碰到邊界或踩到已經填過的格子就轉。轉的方向固定：右轉。</span>

**Step 3：不想每走一步都判斷，有沒有別的辦法？**

*think in layers. the outer ring, then the inner ring.*

<span class="spoiler">用上下左右四條邊界。填完上排就把 top 下移，填完右排就把 right 左移。邊界自己縮，完全不用判斷方向。</span>

**Step 4：n 是奇數時，正中間那格怎麼填？**

*the last number sits alone in the middle.*

<span class="spoiler">n 奇數時最後一層只剩一格。邊界縮到 top==bottom==left==right，填最後一個數字，迴圈結束。</span>

想完再往下看 code。

---

## 解法一：方向模擬

最直覺的做法：模擬一個人拿著筆，往一個方向走，碰牆或踩到寫過的格子就右轉。

```typescript
function generateMatrix(n: number): number[][] {
    const matrix: number[][] = Array.from({ length: n }, () => new Array(n).fill(0));
    const dirs: [number, number][] = [[0, 1], [1, 0], [0, -1], [-1, 0]]; // 右下左上
    let r = 0, c = 0, d = 0;

    for (let i = 1; i <= n * n; i++) {
        matrix[r][c] = i;
        let nr = r + dirs[d][0], nc = c + dirs[d][1];
        if (nr < 0 || nr >= n || nc < 0 || nc >= n || matrix[nr][nc] !== 0) {
            d = (d + 1) % 4; // 右轉
            nr = r + dirs[d][0];
            nc = c + dirs[d][1];
        }
        r = nr;
        c = nc;
    }
    return matrix;
}
```

- Time: $O(n^2)$ — 每格寫一次，$n^2$ 格
- Space: O(1) — 額外空間（不算 output）。用 `matrix[nr][nc] != 0` 當 visited，不用另外開陣列

<details>
<summary>Go 版本</summary>

```go
func generateMatrix(n int) [][]int {
    matrix := make([][]int, n)
    for i := range matrix {
        matrix[i] = make([]int, n)
    }
    // 右、下、左、上
    dirs := [4][2]int{{0, 1}, {1, 0}, {0, -1}, {-1, 0}}
    r, c, d := 0, 0, 0 // 從 (0,0) 出發，一開始往右走

    for i := 1; i <= n*n; i++ {
        matrix[r][c] = i
        nr, nc := r+dirs[d][0], c+dirs[d][1]
        // 越界或踩到已填的 → 右轉
        if nr < 0 || nr >= n || nc < 0 || nc >= n || matrix[nr][nc] != 0 {
            d = (d + 1) % 4
            nr, nc = r+dirs[d][0], c+dirs[d][1]
        }
        r, c = nr, nc
    }
    return matrix
}
```

</details>

用 `n = 3` 走一遍：

```
起點 (0,0)，方向 d=0（右）

i=1: 寫 matrix[0][0]=1
     下一步 (0,1)，沒越界、沒填，繼續往右
i=2: 寫 matrix[0][1]=2  → 下一步 (0,2)
i=3: 寫 matrix[0][2]=3  → 下一步 (0,3) 越界！右轉 d=1
                         新的下一步 (1,2)
i=4: 寫 matrix[1][2]=4  → 下一步 (2,2)
i=5: 寫 matrix[2][2]=5  → 下一步 (3,2) 越界！右轉 d=2
                         新的下一步 (2,1)
i=6: 寫 matrix[2][1]=6  → 下一步 (2,0)
i=7: 寫 matrix[2][0]=7  → 下一步 (2,-1) 越界！右轉 d=3
                         新的下一步 (1,0)
i=8: 寫 matrix[1][0]=8  → 下一步 (0,0) 已填！右轉 d=0
                         新的下一步 (1,1)
i=9: 寫 matrix[1][1]=9  → 結束

matrix:
1 2 3
8 9 4
7 6 5  ✓
```

走得出來，但每一步都要檢查「越界 or 已填」。有點吵。

---

## 解法二：四邊界

換個角度：不要用「走路轉彎」的思維，用「四條邊界往內縮」的思維。

想像矩陣是一個洋蔥。第一層是最外圈，第二層是內一圈，一直到中心。每一層都有四條邊：上、右、下、左，按順序填。填完一邊，對應的邊界就縮一格。

**命名約定**：迭代器的名字 = 它**起點邊界的首字母**。上排從 `left` 出發 → 用 `l`；右排從 `top` 出發 → 用 `t`；下排從 `right` 出發 → 用 `r`；左排從 `bottom` 出發 → 用 `b`。這樣 `for t := top; ...` 自帶語意——「t 是從 top 拉出來的迭代器」。比起 `for r := top; ...`（r 跟 right 撞但語意不相關）讀起來順很多。

```typescript
function generateMatrix(n: number): number[][] {
    const matrix: number[][] = Array.from({ length: n }, () => new Array(n).fill(0));
    let top = 0;
    let bottom = n - 1;
    let left = 0;
    let right = n - 1;
    let num = 1;

    while (top <= bottom && left <= right) {
        for (let l = left; l <= right; l++) matrix[top][l] = num++;       // 上排（l 從 left 出發）
        top++;
        for (let t = top; t <= bottom; t++) matrix[t][right] = num++;     // 右排（t 從 top 出發）
        right--;
        if (top <= bottom) {
            for (let r = right; r >= left; r--) matrix[bottom][r] = num++; // 下排（r 從 right 出發）
            bottom--;
        }
        if (left <= right) {
            for (let b = bottom; b >= top; b--) matrix[b][left] = num++;   // 左排（b 從 bottom 出發）
            left++;
        }
    }
    return matrix;
}
```

- Time: $O(n^2)$
- Space: O(1)

<details>
<summary>Go 版本</summary>

```go
func generateMatrix(n int) [][]int {
    matrix := make([][]int, n)
    for i := range matrix {
        matrix[i] = make([]int, n)
    }
    top := 0
    bottom := n - 1
    left := 0
    right := n - 1
    num := 1

    for top <= bottom && left <= right {
        // 上排：從左到右（l 從 left 出發走欄）
        for l := left; l <= right; l++ {
            matrix[top][l] = num
            num++
        }
        top++
        // 右排：從上到下（t 從 top 出發走列）
        for t := top; t <= bottom; t++ {
            matrix[t][right] = num
            num++
        }
        right--
        // 下排：從右到左（r 從 right 出發走欄）
        if top <= bottom {
            for r := right; r >= left; r-- {
                matrix[bottom][r] = num
                num++
            }
            bottom--
        }
        // 左排：從下到上（b 從 bottom 出發走列）
        if left <= right {
            for b := bottom; b >= top; b-- {
                matrix[b][left] = num
                num++
            }
            left++
        }
    }
    return matrix
}
```

</details>

複雜度跟第一版一樣，但程式本身乾淨很多：沒有方向陣列，沒有逐格判斷越界，沒有 visited 檢查。邊界自己退場。

---

**為什麼要 `if top <= bottom` 和 `if left <= right`？**

對方陣 $n \times n$，這兩個檢查永遠不會生效。上排和右排走完後，就算 top 跟 bottom 交叉、left 跟 right 交叉，下排和左排的 for 迴圈邊界也會剛好不成立，什麼都不寫。拿掉 `if` 這題也會過。

那為什麼要留？因為同一套模板搬到 [54 Spiral Matrix](/problem/spiral-matrix)（$m \times n$ 長方形）就會爆。看這個反例：

```
1 × 4 的矩陣：top=0, bottom=0, left=0, right=3

上排：matrix[0][0..3] = 1,2,3,4    → top=1
右排：for r:=1; r<=0; 不跑           → right=2

下排（沒 if 保護）：for c:=2; c>=0;
  matrix[0][2] = 5  ← 把上排的 3 覆蓋掉
  matrix[0][1] = 6  ← 把上排的 2 覆蓋掉
  matrix[0][0] = 7  ← 把上排的 1 覆蓋掉
```

整排上排被下排「回頭寫」覆蓋。方陣之所以沒事，是因為每次交叉後迴圈剛好縮成 0 格；但窄長方形的上排走太長，下排 `for c:=right; c>=left` 還有合法範圍可以跑。所以 `if` 檢查是為了讓同一套寫法能跨到 54，不是這題必需。

用 `n = 5` 走一遍，看第二圈：

```
第一圈完成後：num=17, top=1, bottom=3, left=1, right=3

第二圈：
  上排 matrix[1][1..3] = 17,18,19  → num=20, top=2
  右排 matrix[2..3][3] = 20,21     → num=22, right=2
  下排 matrix[3][2..1] = 22,23     → num=24, bottom=2
  左排 matrix[2..2][1] = 24        → num=25, left=2

第三圈：top=2, bottom=2, left=2, right=2
  上排 matrix[2][2..2] = 25        → num=26, top=3
  右排、下排、左排：迴圈邊界都不成立，不跑

結果：
 1  2  3  4  5
16 17 18 19  6
15 24 25 20  7
14 23 22 21  8
13 12 11 10  9
```

每一圈都把外層寫完再進內層。num 從 1 跑到 $n^2$，剛好填滿。

---

**Overthinking**

這題和 [54 Spiral Matrix](/problem/spiral-matrix) 是對稱的：54 是讀出螺旋順序，59 是寫入螺旋順序。兩題的核心迴圈完全一樣，差別只在「讀 matrix[r][c]」還是「寫 matrix[r][c] = num」。一套模板通吃。

如果題目改成「從任意起點開始螺旋」（[885 Spiral Matrix III](https://leetcode.com/problems/spiral-matrix-iii/)），邊界縮的思路就壞了，因為起點不在角落。那題要用方向 + 步數遞增：右 1 步、下 1 步、左 2 步、上 2 步、右 3 步、下 3 步…每兩次方向換一次步數。越界的格子跳過不記。

如果題目改成 $m \times n$（長方形），邊界版只要把 `top, bottom = 0, m-1` 分開設就好，邏輯不變。方向版也一樣，只要把 `n` 的邊界檢查拆成 m 和 n。

---

## 結論

| 解法 | Time | Space | 感覺 |
|---|---|---|---|
| 方向 + 越界檢查 | $O(n^2)$ | O(1) | 像走迷宮，每一步都要撞牆才轉 |
| 四邊界往內縮 | $O(n^2)$ | O(1) | 像剝洋蔥，一層填完就縮一圈 |

$O(n^2)$ 是這題的下限，因為每格至少要寫一次。速度沒得比，差別在寫得乾不乾淨。邊界版是一個 while、四個 for，讀起來像題目本身的敘述。
