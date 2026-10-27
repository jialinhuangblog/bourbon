---
title: "解題直覺"
category: Meta
slug: problem-solving-intuition
subtitle: 看到題目，怎麼知道該用什麼
date: 2026-04-12
updated: 2026-08-04
pinned: true
---

# 解題直覺

刷題卡住的地方通常不是 code 寫不出來，是讀完題目不知道該用什麼解。Two pointers？DP？Greedy？DFS？

這篇不教演算法本身，只講讀題時腦子裡跑的那套判斷。刷多了以後這套判斷其實滿固定的，固定到可以寫下來。

---

## 四層思考

先把詞分清楚，不然 pattern、paradigm、演算法三個詞混著用，討論兩句就打結。解一題的時候，腦子其實分四層在運作：

- **Paradigm（典範 / 策略）**：思考的大方向，沒有骨架。Greedy、DP、Backtracking、Divide and Conquer、Brute Force。
- **Pattern（模式 / 技巧）**：paradigm 底下反覆出現的具體形狀，有可以照抄的骨架。Sliding window、two pointers、monotonic stack、Floyd 龜兔、BFS 分層。
- **Algorithm（演算法）**：已命名、流程明確的方法。Dijkstra、Kruskal、KMP、Quick Sort、Huffman。
- **Solution（解法）**：對特定題目的具體實作。「287 用 Floyd」、「55 用 reach 變數」。

往下越來越具體。Paradigm 是抽象標籤，Solution 是真的能跑的 code。每一層是上一層的具體化。

> **Pattern 有骨架，Paradigm 沒有。**

Sliding window 是 pattern，因為它有 template 可以背（左右指標、while invalid 縮 left、視窗合法時更新答案）。換題目只換三個內容：expand 是右邊進來時更新什麼（[3](/problem/longest-substring-without-repeating-characters) 把字元加進 set）、shrink 是什麼時候縮左邊（3 遇到重複字元就縮，[209](/problem/minimum-size-subarray-sum) 總和達標就縮）、record 是記什麼答案（3 記最長，209 記最短）。骨架完全不動。

但 greedy 是 paradigm，沒有 template。每題的「局部最優」長得都不一樣：activity selection 挑最早結束、Dijkstra 挑距離最短、Huffman 合併兩個最小頻率、[55 Jump Game](/problem/jump-game) 拉最大 reach。共通的只有「不回頭、不窮舉、相信局部選擇」這個信念。

所以「這題用 sliding window」是在告訴你**程式怎麼寫**；「這題用 greedy」只給**思考的方向**，每一步挑什麼要回到題目本身。

---

垂直四層之外還有一個正交維度：**資料結構（用什麼裝）**。Array、hash map、heap、tree、graph、trie。它不屬於四層的任何一層，每一層都可能用到：寫 Floyd（pattern）用 linked list 或 array index，寫 Dijkstra（algorithm）用 priority queue (heap)。

---

一題通常跨層、跨維度。

- [56 Merge Intervals](/problem/merge-intervals)：paradigm 是 greedy，pattern 是排序後一次掃描，資料結構是 array。
- [23 Merge K Sorted Lists](/problem/merge-k-sorted-lists)：paradigm 是 divide and conquer，algorithm 是 merge sort 的 merge 步驟，資料結構是 heap + linked list。
- [287 Find the Duplicate Number](/problem/find-the-duplicate-number)：同一題兩條路。Paradigm 1 = binary search on values，pattern 是經典二分骨架；Paradigm 2 = 把 array 當 implicit linked list，pattern 是 Floyd 龜兔，algorithm 是 Floyd's cycle detection。同 solution 但不同 paradigm 路徑。

---

## 鎖死的程度有等級

某些工具總是綁著出現：看到 BFS，旁邊一定有 queue；看到 DFS，旁邊一定有 stack；看到 Dijkstra，旁邊一定有 priority queue。不過綁定的緊密程度有差別，粗分三級。

### 第一級：換掉 pattern 就不是它了

BFS 跟 queue 的關係已經超過搭配，根本是同一件事。BFS 要的是「同層先處理完才下一層」，queue 的 FIFO 給的是「先排隊的先處理」，兩句話描述的是同一個動作。把 queue 換成 stack？立刻變成 DFS，骨架直接易主。

DFS 跟 stack（遞迴的隱式呼叫 stack 也算）同樣這級。Dijkstra 沒了 min-heap 退化成 $O(V^2)$，那就不是 Dijkstra 而是 Bellman-Ford 風的鬆弛掃描。Floyd 龜兔少一個指標看不出環。Trie 換掉「字元到子節點的 map」就不叫 trie，是普通樹。Union-Find 沒了 parent 找不到 root。KMP 拿掉 failure function 退回 O(nm) 暴力比對。BIT (Fenwick) 整個結構建立在 `lowbit(x) = x & (-x)` 這個位元操作上，拿掉就不成立。

這些與其說是「演算法搭配資料結構」，不如說**演算法的定義已經把資料結構吞進去了**，所以換掉 pattern 拿到的不是降級版，而是另一個演算法。

### 第二級：骨架鎖死，零件可換

Backtracking 規定必須能「撤銷狀態」，這點不能商量，不過撤銷的做法不只一種：可以 explicit pop（做完一個動作後手動把狀態彈回去），也可以 immutable snapshot（每一步傳一份新拷貝）。兩種都跑得起來，差別在效能跟可讀性。

Sliding window 通常雙指標，但少數題用 deque（例如 [239 Sliding Window Maximum](/problem/sliding-window-maximum)）。DP 必須記子問題答案，但「記在哪、怎麼存」是另一條獨立決定：memo、table、滾動陣列都行。

骨架仍然鎖死（backtracking 沒撤銷不算 backtracking、DP 沒記憶不算 DP），不過實作零件可以換。

### 第三級：根本沒固定 pattern

Greedy 沒有綁定的 pattern，因為每題的「局部最優」長得都不一樣。Activity selection 排序後挑最早結束。Dijkstra 用 heap 挑距離最短。Huffman 也用 heap，但合併的是兩個最小頻率。[55 Jump Game](/problem/jump-game) 連 heap 都不用，一個變數推到最遠。

四題全部掛 greedy 標籤，骨架完全不同。「Greedy 必然會用到 ___」這個句子填不出來。Brute force、DP、Divide and conquer 也一樣散。

---

換個生活化的角度。

「弦樂四重奏」是第一級的鎖定：兩把小提琴、一把中提琴、一把大提琴。少一樣就不算，多一樣也不算。問人「弦樂四重奏需要什麼樂器？」答得出來，因為定義本身就把樂器吞進去了。

「古典音樂」是第三級的鬆綁：獨奏、室內樂、交響樂、歌劇都算，配置完全不同。問人「古典音樂需要什麼樂器？」沒法答。但兩個都是真實存在、有意義的分類。

寫 LeetCode 也一樣。`Sliding Window` tag 底下的題，code 形狀都很像，因為骨架被吞進定義了。`Greedy` tag 底下的題，code 形狀差異巨大，因為 tag 只規定「決策原則」沒規定實作。Tag 一樣，骨架完全不同。

所以 paradigm 跟 pattern 差在骨架被鎖死到什麼程度，不在誰比較抽象。

---

## 「至少達到」與「要剛好」

兩個英文片語只差一個詞，走出來的演算法卻不一樣。

「**reach**」、「**arrive**」、「**cover**」、「**at least**」、「**最少多少**」：成功條件**寬鬆**，達到或超過都行。
「**equals**」、「**sum to**」、「**exactly**」、「**剛好**」、「**恰好**」：成功條件**嚴格**，多一塊少一塊都不算。

這兩組詞的差別，在於**狀態能不能被壓縮成一個變數**。

### 至少型：可以壓縮成一個變數

[55 Jump Game](/problem/jump-game) 問「能不能到最後一格」。能跳到 index 4 表示能跳到 index 3、2、1（後面想停哪都行）。**「能到 X」這個性質單調**：能到大的就一定能到小的。

所以一個 `reach` 變數就夠：「目前能到的最遠 index」。整題壓縮成 O(1) space、O(n) time。Greedy。

### 剛好型：每個狀態都要獨立記

[322 Coin Change](/problem/coin-change) 問「最少幾個硬幣**剛好**湊出 amount」。能湊出 5 跟能不能湊出 4 沒關係：可能湊得出 5 但湊不出 4。**「能湊出 X」這個性質不單調**。

每個 amount 都要獨立記答案，沒辦法用一個變數概括。`dp[0..amount]` 整張表，O(amount) space、$O(\text{amount} \times \text{len(coins)})$ time。DP。

### 對照表

| 至少達到 | 要剛好 |
|---|---|
| [55 Jump Game](/problem/jump-game) 能到最後一格嗎 | [403 Frog Jump](/problem/frog-jump) 必須剛好踩最後一顆石頭 |
| [209 Minimum Size Subarray Sum](/problem/minimum-size-subarray-sum) 子陣列和 ≥ target 最短 | [560 Subarray Sum Equals K](/problem/subarray-sum-equals-k) 子陣列和 = K 多少種 |
| [875 Koko](/problem/koko-eating-bananas) 至少能 h 小時內吃完 | [322 Coin Change](/problem/coin-change) 剛好湊出 amount |
| [1011 Capacity to Ship](/problem/capacity-to-ship-packages-within-d-days) 至少能 D 天運完 | [416 Partition Equal Subset Sum](/problem/partition-equal-subset-sum) 切成兩堆剛好一樣大 |

### 判斷反射

讀題的時候掃一下動詞：

- 看到 reach / cover / at least / 最少 → 先想 **Greedy** 或 **BS on answer**
- 看到 equals / sum to / exactly / 剛好 → 先想 **DP**

當然不絕對：有些題綜合兩者，有些題用 DFS + memo 也能解。但這個反射能讓你**第一輪就把搜尋空間從 5 個 paradigm 砍到 1-2 個**，少走冤枉路。

### 跟前面四層思考的呼應

「至少型」常落在 **Greedy paradigm**：沒骨架、用一個變數概括所有歷史。
「剛好型」常落在 **DP paradigm**：有 memo / table 骨架、每個狀態獨立記。

這不代表 Greedy 比較簡單或 DP 比較難，而是**問題本身的條件結構**決定可不可以壓縮，再由這個結構決定要用哪一族 paradigm。

---

## 策略：五個大方向

### Brute Force（暴力）

試所有可能，找到答案。

什麼時候用？**永遠先想這個。** 每一題的起點。暴力解讓你理解問題，也讓你看到浪費在哪。優化是從暴力解長出來的，不是從天上掉下來的。

暴力解常見的實作手段：巢狀迴圈、DFS 遍歷所有路徑、生成所有子集/排列再篩選。

### Greedy（貪心）

每一步選當下最好的並且永遠不回頭。

訊號：
- 題目問「可不可行」或「最少/最多幾次」
- 問題有一個明顯的排序維度（時間、大小、位置）
- 局部最優不會讓全域變差

例子：[55 Jump Game](/problem/jump-game) 每一格把 maxReach 推到最遠。不用比較不同路徑，因為跳得越遠選擇只會更多。

陷阱：看起來能 greedy 但其實不行。[322 Coin Change](/problem/coin-change) 如果面額是 `[1, 3, 4]`、target 是 6，greedy 選 4+1+1=3 枚，但最優是 3+3=2 枚。需要 DP。

**Greedy 能用的前提：局部最優 = 全域最優。** 不確定的時候，先寫暴力解，觀察暴力解的選擇有沒有規律。

### Dynamic Programming（動態規劃）

把大問題拆成子問題，記住子問題的答案，不重算。

訊號：
- 題目問「最大/最小/方法數」
- 選擇會影響後續選擇（不像 greedy 可以獨立決定）
- 能寫出遞迴關係：`dp[i] = f(dp[i-1], dp[i-2], ...)`
- 子問題重疊（同一個子問題被算很多次）

例子：[198 House Robber](/problem/house-robber) 搶不搶這間房會影響下一間能不能搶。`dp[i] = max(dp[i-1], dp[i-2] + nums[i])`。

同一條遞迴式有兩種寫法。Top-down 照著它直接寫，加一個 memo，本體是 DFS + cache，缺點是遞迴深度跟著輸入長度走：Node 預設遞迴到九千層左右就 RangeError，題目要是給到十萬個元素，一路遞迴下去一定超過。Bottom-up 從最小的子問題用迴圈往上填表，沒有遞迴深度問題，還能做空間壓縮：198 的轉移只看前兩格，整張表縮成兩個變數。

跟 greedy 的差別：greedy 每步獨立決定，DP 要看全局。如果你發現 greedy 的選擇可能被後面推翻，那就是 DP。

詳細看 [Dynamic Programming](/concept/dp)。

### Backtracking（回溯）

用 DFS 走所有路徑，要是走不通就撤回來換下一條。

訊號：
- 題目問「所有組合/排列/子集」
- 「所有可能的解」
- 有約束條件要滿足（像數獨、N 皇后）

例子：[39 Combination Sum](/problem/combination-sum) 找所有加起來等於 target 的組合。每一步選一個數字，超過 target 就撤回。

跟暴力的差別在於 pruning：暴力會把所有可能生成完再篩選，backtracking 走到一半發現這條路不可能成功就提早 return，這個分支底下的路全部不用走，recursion tree 上像剪掉一根樹枝。

另外兩個常用配件：遞迴時傳 start index，同一個數可以重複用就傳 `i`（39），不行就傳 `i + 1`（[40 Combination Sum II](/problem/combination-sum-ii)）；要避開重複解就先排序，同一層遇到一樣的值跳過（40、[90 Subsets II](/problem/subsets-ii)）。

### Divide and Conquer（分治）

把問題切成獨立的子問題，各自解完再合併。

訊號：
- 問題可以對半切，左右互不影響
- 合併的成本可控
- 資料結構本身就是遞迴的（tree、linked list）

例子：[23 Merge K Sorted Lists](/problem/merge-k-sorted-lists) 兩兩合併，每次問題規模砍半。Merge sort 也是。

跟 DP 的差別：divide and conquer 的子問題不重疊，DP 的子問題會重疊。

---

## 技巧：看到什麼線索用什麼

很多題不用先想策略，看到線索就知道用什麼技巧：排序過的 array 找值先想 binary search，括號配對先想 stack。這種查表整理在 [Pattern Cheatsheet](/concept/pattern-cheatsheet)，照首頁分類排，每行帶已解例題。那張表列的是經驗而不是規則，題目做多了自然就認得出來。

---

## DFS 不是策略，是工具

DFS（深度優先搜尋）常被搞混。它是一種走訪方式而不是策略，所以不同策略都會用到它：

| 策略 | DFS 扮演的角色 |
|---|---|
| Brute force | 用 DFS 遍歷所有可能路徑 |
| Backtracking | 用 DFS + pruning，走不通就撤回 |
| Tree traversal | 用 DFS 走 preorder/inorder/postorder |
| Graph traversal | 用 DFS 走所有連通節點 |

同樣，BFS 也是工具。最短路徑用 BFS，連通區塊 DFS 或 BFS 都行。

詳細看 [DFS](/concept/dfs)。

---

## 實戰流程

拿到一題，照這個順序想：

**1. 讀題，抓關鍵字。**

「所有組合」→ backtracking。「最少步數」→ DP 或 BFS。「能不能到達」→ greedy 或 DP。「排序過的」→ binary search 或 two pointers。

**2. 想暴力解。**

不管多慢，先想出一個能跑的。暴力解讓你理解問題的結構，也讓你看到重複計算在哪。

**3. 找浪費。**

暴力解慢在哪？重複算了什麼？哪些計算其實可以跳過？

- 重複算子問題 → 加 memo → DP
- 每步都有最優選擇且不影響後續 → greedy
- 不需要所有路徑，只需要最遠/最近 → 換資料結構（heap、hash map）
- 兩層迴圈在做對稱的搜尋 → two pointers

**4. 驗證。**

手動走一組小 input。走得通才寫 code。

---

## 常見誤判

| 看起來像 | 其實是 | 為什麼 |
|---|---|---|
| Greedy | DP | 當下最優會被後面推翻（Coin Change） |
| DP | Greedy | 子問題不重疊，不需要記（Jump Game） |
| DFS | BFS | 問最短路徑，DFS 不保證最短 |
| Two pointers | Sliding window | 窗口大小不固定，需要伸縮 |
| Brute force | Backtracking | 加 pruning 就從暴力變回溯 |

---

## 總結

沒有公式。但有直覺，直覺靠刷題累積。

流程講出來很無聊：讀題 → 暴力解 → 找浪費 → 優化。無聊歸無聊，每一步能跳過去，是因為你見過類似的結構，而不是靈感。

見得夠多之後，讀完題目就會知道該往哪走。
