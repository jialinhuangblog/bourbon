---
id: 1164
title: "Product Price at a Given Date"
slug: product-price-at-a-given-date
difficulty: Medium
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# Product Price at a Given Date

Table: `Products`

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product\_id    | int     |
| new\_price     | int     |
| change\_date   | date    |
+---------------+---------+
(product\_id, change\_date) is the primary key (combination of columns with unique values) of this table.
Each row of this table indicates that the price of some product was changed to a new price at some date.

Initially, all products have price 10.

Write a solution to find the prices of all products on the date `2019-08-16`.

Return the result table in **any order**.

The result format is in the following example.

**Example 1:**

**Input:** 
Products table:
+------------+-----------+-------------+
| product\_id | new\_price | change\_date |
+------------+-----------+-------------+
| 1          | 20        | 2019-08-14  |
| 2          | 50        | 2019-08-14  |
| 1          | 30        | 2019-08-15  |
| 1          | 35        | 2019-08-16  |
| 2          | 65        | 2019-08-17  |
| 3          | 20        | 2019-08-18  |
+------------+-----------+-------------+
**Output:** 
+------------+-------+
| product\_id | price |
+------------+-------+
| 2          | 50    |
| 1          | 35    |
| 3          | 10    |
+------------+-------+

## Code Template

_No Go or TypeScript template available._
