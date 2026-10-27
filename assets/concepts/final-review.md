---
title: "Final Review"
category: Meta
slug: final-review
subtitle: 要背的招式，跟看熟就好的題
date: 2026-08-09T17:00:00
---

# Final Review

## MUST MUST MUST

**Trees**
- [#110 Balanced Binary Tree](/problem/balanced-binary-tree) — -1 當哨兵回傳
- [#543 Diameter of Binary Tree](/problem/diameter-of-binary-tree) — 答案存外部變數，回傳的是高度
- [#1650 LCA III](/problem/lowest-common-ancestor-of-a-binary-tree-iii) — 雙指標互換起點
- [#863 All Nodes Distance K in Binary Tree](/problem/all-nodes-distance-k-in-binary-tree) — 樹轉圖再 BFS
- [#99001 Lock Binary Tree](/problem/lock-binary-tree) — counter 把 O(n) 壓到 O(1)

**DP**
- [#213 House Robber II](/problem/house-robber-ii) — 環狀拆兩段
- [#1143 Longest Common Subsequence](/problem/longest-common-subsequence) — 對角線 +1／取大，滾動陣列留兩列

**Graphs**
- [#787 Cheapest Flights Within K Stops](/problem/cheapest-flights-within-k-stops) — Bellman-Ford 按輪次讀上一輪快照
- [#207 Course Schedule](/problem/course-schedule) — 拓撲排序／環偵測

**Arrays & Hashing**
- [#128 Longest Consecutive Sequence](/problem/longest-consecutive-sequence) — 只從 num-1 不在 set 的起點數
- [#238 Product of Array Except Self](/problem/product-of-array-except-self) — 左右兩趟不除法
- [#560 Subarray Sum Equals K](/problem/subarray-sum-equals-k) — prefix-k 配對 hashmap

**Backtracking / Binary Search**
- [#47 Permutations II](/problem/permutations-ii) — 排序後同層跳過重複的分支
- [#875 Koko Eating Bananas](/problem/koko-eating-bananas) — 在答案值域上二分

**Bit Manipulation**
- [#190 Reverse Bits](/problem/reverse-bits)
- [#338 Counting Bits](/problem/counting-bits) — ans[i>>1] + (i&1)
- [#371 Sum of Two Integers](/problem/sum-of-two-integers) — XOR 當和，(a&b)<<1 當進位
- [#137 Single Number II](/problem/single-number-ii) — 每個 bit 出現次數 mod 3
- [#260 Single Number III](/problem/single-number-iii) — XOR 全部後取 diff bit 分兩組
- [#342 Power of Four](/problem/power-of-four) — n&(n-1)==0 && n%3==1

**Greedy / Intervals**
- [#45 Jump Game II](/problem/jump-game-ii) — 只記右界，掃到才結算一跳
- [#435 Non-overlapping Intervals](/problem/non-overlapping-intervals) — 按結束時間排序

**Heap**
- [#355 Design Twitter](/problem/design-twitter) — merge k list 的設計

**Linked List**
- [#142 Linked List Cycle II](/problem/linked-list-cycle-ii) — 相遇後從 head 同速走找環口
- [#287 Find the Duplicate Number](/problem/find-the-duplicate-number) — Floyd 判圈用在陣列 index 上

**Math & Geometry**
- [#2013 Detect Squares](/problem/detect-squares) — 固定一點的組合公式
- [#204 Count Primes](/problem/count-primes) — 篩法
- [#289 Game of Life](/problem/game-of-life) — 原地編碼避免額外陣列
- [#2523 Closest Prime Numbers in Range](/problem/closest-prime-numbers-in-range) — 篩完找排序相鄰對

**Sliding Window**
- [#239 Sliding Window Maximum](/problem/sliding-window-maximum) — monotonic deque
- [#3634 Minimum Removals to Balance Array](/problem/minimum-removals-to-balance-array)

**Stack**
- [#155 Min Stack](/problem/min-stack) — 副 stack 存最小值
- [#739 Daily Temperatures](/problem/daily-temperatures) — monotonic stack
- [#496 Next Greater Element I](/problem/next-greater-element-i) — monotonic stack + hashmap
- [#503 Next Greater Element II](/problem/next-greater-element-ii) — 環狀陣列拉長兩倍
- [#853 Car Fleet](/problem/car-fleet) — sort desc + 到達時間公式
- [#232 Implement Queue using Stacks](/problem/implement-queue-using-stacks) — 兩個 stack 攤還

**Two Pointers**
- [#15 3Sum](/problem/3sum) — 固定 i，雙層跳過重複值

**進階結構**
- [#307 Range Sum Query - Mutable](/problem/range-sum-query-mutable) — BIT / Segment Tree

## 看熟就好

**Trees**
- [#100 Same Tree](/problem/same-tree)
- [#102 Binary Tree Level Order Traversal](/problem/binary-tree-level-order-traversal)
- [#104 Maximum Depth of Binary Tree](/problem/maximum-depth-of-binary-tree)
- [#226 Invert Binary Tree](/problem/invert-binary-tree)
- [#572 Subtree of Another Tree](/problem/subtree-of-another-tree)
- [#103 Binary Tree Zigzag Level Order Traversal](/problem/binary-tree-zigzag-level-order-traversal)
- [#236 Lowest Common Ancestor of a Binary Tree](/problem/lowest-common-ancestor-of-a-binary-tree)
- [#1993 Operations on Tree](/problem/operations-on-tree)

**DP**
- [#70 Climbing Stairs](/problem/climbing-stairs)
- [#198 House Robber](/problem/house-robber)
- [#746 Min Cost Climbing Stairs](/problem/min-cost-climbing-stairs)
- [#509 Fibonacci Number](/problem/fibonacci-number)
- [#62 Unique Paths](/problem/unique-paths)

**Arrays & Hashing**
- [#1 Two Sum](/problem/two-sum)
- [#49 Group Anagrams](/problem/group-anagrams)
- [#242 Valid Anagram](/problem/valid-anagram)
- [#347 Top K Frequent Elements](/problem/top-k-frequent-elements)
- [#387 First Unique Character in a String](/problem/first-unique-character-in-a-string)

**Backtracking**
- [#39 Combination Sum](/problem/combination-sum)
- [#46 Permutations](/problem/permutations)
- [#78 Subsets](/problem/subsets)

**Binary Search**
- [#704 Binary Search](/problem/binary-search)
- [#69 Sqrt(x)](/problem/sqrtx)

**Bit Manipulation**
- [#136 Single Number](/problem/single-number)

**Graphs**
- [#200 Number of Islands](/problem/number-of-islands)
- [#994 Rotting Oranges](/problem/rotting-oranges)

**Greedy**
- [#53 Maximum Subarray](/problem/maximum-subarray)
- [#55 Jump Game](/problem/jump-game)

**Heap**
- [#215 Kth Largest Element in an Array](/problem/kth-largest-element-in-an-array)
- [#692 Top K Frequent Words](/problem/top-k-frequent-words)

**Intervals**
- [#56 Merge Intervals](/problem/merge-intervals)

**Linked List**
- [#2 Add Two Numbers](/problem/add-two-numbers)
- [#23 Merge k Sorted Lists](/problem/merge-k-sorted-lists)
- [#141 Linked List Cycle](/problem/linked-list-cycle)
- [#206 Reverse Linked List](/problem/reverse-linked-list)

**Math & Geometry**
- [#59 Spiral Matrix II](/problem/spiral-matrix-ii)

**Sliding Window**
- [#3 Longest Substring Without Repeating Characters](/problem/longest-substring-without-repeating-characters)
- [#121 Best Time to Buy and Sell Stock](/problem/best-time-to-buy-and-sell-stock)
- [#209 Minimum Size Subarray Sum](/problem/minimum-size-subarray-sum)

**Stack**
- [#20 Valid Parentheses](/problem/valid-parentheses)
- [#22 Generate Parentheses](/problem/generate-parentheses)
- [#150 Evaluate Reverse Polish Notation](/problem/evaluate-reverse-polish-notation)

**Tries**
- [#208 Implement Trie (Prefix Tree)](/problem/implement-trie-prefix-tree)

**Two Pointers**
- [#125 Valid Palindrome](/problem/valid-palindrome)
- [#167 Two Sum II - Input Array Is Sorted](/problem/two-sum-ii-input-array-is-sorted)
- [#151 Reverse Words in a String](/problem/reverse-words-in-a-string)
- [#977 Squares of a Sorted Array](/problem/squares-of-a-sorted-array)
