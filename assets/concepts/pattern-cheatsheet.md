---
title: "Pattern Cheatsheet"
category: Algorithms
slug: pattern-cheatsheet
subtitle: 首頁每個題型分類，各自常用哪些解法
date: 2026-03-22T21:37:03
updated: 2026-08-04
---

# Pattern Cheatsheet

看完題目不知道從哪下手的時候，與其回想十八個分類，不如先問：這題要我交出什麼？要的東西不一樣，路就分開了：

```
題目要什麼？
│
├─ 所有合法解長什麼樣 ──────→ Backtracking
│
├─ 一個數字：最大、最小、幾種方法
│   ├─ 子問題會重複出現 ────→ DP
│   └─ 每步挑眼前最好的就夠 → Greedy
│
└─ 把節點走一遍 ────────────→ DFS / BFS / Union-Find
```

這棵樹背後的推理，包括四層思考、「至少達到」跟「要剛好」的差別、先寫暴力解再優化的流程，都在[解題直覺](/concept/problem-solving-intuition)。

## 題型對解法

首頁那排分類就是題型。每個題型底下反覆出現的解法整理在這：

| 題型 | 常用解法 | 已解例題 |
|---|---|---|
| Arrays & Hashing | hashmap 換 O(1) 查詢、counting、prefix sum | [#1](/problem/two-sum)、[#49](/problem/group-anagrams)、[#238](/problem/product-of-array-except-self) |
| Two Pointers | 頭尾夾、快慢指標、原地覆寫 | [#125](/problem/valid-palindrome)、[#167](/problem/two-sum-ii-input-array-is-sorted)、[#15](/problem/3sum)、[#26](/problem/remove-duplicates-from-sorted-array) |
| Sliding Window | 窗口伸縮 + hashmap 記窗內狀態、monotonic deque 維持窗內極值 | [#3](/problem/longest-substring-without-repeating-characters)、[#209](/problem/minimum-size-subarray-sum)、[#239](/problem/sliding-window-maximum) |
| Stack | 配對消除、monotonic stack、用 stack 模擬別的結構 | [#20](/problem/valid-parentheses)、[#739](/problem/daily-temperatures)、[#853](/problem/car-fleet) |
| Binary Search | 找值、找邊界、對答案空間二分 | [#704](/problem/binary-search)、[#875](/problem/koko-eating-bananas)、[#69](/problem/sqrtx) |
| Linked List | 快慢指標、反轉、dummy head | [#206](/problem/reverse-linked-list)、[#141](/problem/linked-list-cycle)、[#142](/problem/linked-list-cycle-ii) |
| Trees | 遞迴 DFS、BFS 層序、一趟後序回傳值順便收答案、把 parent 存進 hashmap 當圖走 | [#104](/problem/maximum-depth-of-binary-tree)、[#543](/problem/diameter-of-binary-tree)、[#226](/problem/invert-binary-tree)、[#102](/problem/binary-tree-level-order-traversal)、[#863](/problem/all-nodes-distance-k-in-binary-tree) |
| Tries | 26 叉樹節點 + isEnd 標記 | [#208](/problem/implement-trie-prefix-tree) |
| Heap / Priority Queue | min-heap 維持 size k、k 路合併 | [#215](/problem/kth-largest-element-in-an-array)、[#23](/problem/merge-k-sorted-lists)、[#347](/problem/top-k-frequent-elements) |
| Backtracking | DFS + push/pop 撤銷、pruning、start index 控重複、used 陣列排排列、排序後同層跳過相同值 | [#22](/problem/generate-parentheses)、[#39](/problem/combination-sum)、[#78](/problem/subsets)、[#46](/problem/permutations)、[#47](/problem/permutations-ii) |
| Graphs | DFS/BFS 走連通塊、Union-Find、topological sort | [#200](/problem/number-of-islands)、[#207](/problem/course-schedule)、[#994](/problem/rotting-oranges) |
| Advanced Graphs | Dijkstra、MST（Prim / Kruskal）、Bellman-Ford、限制邊數就按邊數分輪 | [#787](/problem/cheapest-flights-within-k-stops) |
| 1-D Dynamic Programming | top-down memo、bottom-up 填表、滾動變數壓縮 | [#70](/problem/climbing-stairs)、[#746](/problem/min-cost-climbing-stairs)、[#198](/problem/house-robber)、[#213](/problem/house-robber-ii)、[#338](/problem/counting-bits) |
| 2-D Dynamic Programming | 二維表格、字串對字串、走格子、每格看左上上左三個來源 | [#62](/problem/unique-paths)、[#1143](/problem/longest-common-subsequence) |
| Greedy | 排序後掃描、維護目前最好、分層推進右界 | [#55](/problem/jump-game)、[#45](/problem/jump-game-ii)、[#56](/problem/merge-intervals)、[#435](/problem/non-overlapping-intervals) |
| Intervals | 照起點排序後合併、照終點排序做挑選 | [#56](/problem/merge-intervals)、[#435](/problem/non-overlapping-intervals) |
| Math & Geometry | 模擬、質數篩、轉置加翻轉 | [#59](/problem/spiral-matrix-ii)、[#289](/problem/game-of-life)、[#204](/problem/count-primes)、[#2523](/problem/closest-prime-numbers-in-range) |
| Bit Manipulation | XOR 消重複、n & (n-1) 清最低位、mask 逐位處理 | [#136](/problem/single-number)、[#190](/problem/reverse-bits)、[#371](/problem/sum-of-two-integers) |

同一題常常跨兩個題型：[#560](/problem/subarray-sum-equals-k) 是「連續區間和」加「找配對」，prefix sum 跟 hashmap 疊在一起用。

主表的第一欄就是首頁那排分類 chips，點哪個分類就能把該題型的題拉出來複習。
