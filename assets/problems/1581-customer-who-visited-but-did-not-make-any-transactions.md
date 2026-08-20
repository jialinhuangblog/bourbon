---
id: 1581
title: "Customer Who Visited but Did Not Make Any Transactions"
slug: customer-who-visited-but-did-not-make-any-transactions
difficulty: Easy
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# Customer Who Visited but Did Not Make Any Transactions

Table: `Visits`

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| visit\_id    | int     |
| customer\_id | int     |
+-------------+---------+
visit\_id is the column with unique values for this table.
This table contains information about the customers who visited the mall.

Table: `Transactions`

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| transaction\_id | int     |
| visit\_id       | int     |
| amount         | int     |
+----------------+---------+
transaction\_id is column with unique values for this table.
This table contains information about the transactions made during the visit\_id.

Write a solution to find the IDs of the users who visited without making any transactions and the number of times they made these types of visits.

Return the result table sorted in **any order**.

The result format is in the following example.

**Example 1:**

**Input:** 
Visits
+----------+-------------+
| visit\_id | customer\_id |
+----------+-------------+
| 1        | 23          |
| 2        | 9           |
| 4        | 30          |
| 5        | 54          |
| 6        | 96          |
| 7        | 54          |
| 8        | 54          |
+----------+-------------+
Transactions
+----------------+----------+--------+
| transaction\_id | visit\_id | amount |
+----------------+----------+--------+
| 2              | 5        | 310    |
| 3              | 5        | 300    |
| 9              | 5        | 200    |
| 12             | 1        | 910    |
| 13             | 2        | 970    |
+----------------+----------+--------+
**Output:** 
+-------------+----------------+
| customer\_id | count\_no\_trans |
+-------------+----------------+
| 54          | 2              |
| 30          | 1              |
| 96          | 1              |
+-------------+----------------+
**Explanation:** 
Customer with id = 23 visited the mall once and made one transaction during the visit with id = 12.
Customer with id = 9 visited the mall once and made one transaction during the visit with id = 13.
Customer with id = 30 visited the mall once and did not make any transactions.
Customer with id = 54 visited the mall three times. During 2 visits they did not make any transactions, and during one visit they made 3 transactions.
Customer with id = 96 visited the mall once and did not make any transactions.
As we can see, users with IDs 30 and 96 visited the mall one time without making any transactions. Also, user 54 visited the mall twice and did not make any transactions.

## Code Template

_No Go or TypeScript template available._
