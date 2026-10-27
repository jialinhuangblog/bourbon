---
id: 1251
title: "Average Selling Price"
slug: average-selling-price
difficulty: Easy
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# Average Selling Price

Table: `Prices`

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product\_id    | int     |
| start\_date    | date    |
| end\_date      | date    |
| price         | int     |
+---------------+---------+
(product\_id, start\_date, end\_date) is the primary key (combination of columns with unique values) for this table.
Each row of this table indicates the price of the product\_id in the period from start\_date to end\_date.
For each product\_id there will be no two overlapping periods. That means there will be no two intersecting periods for the same product\_id.

Table: `UnitsSold`

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product\_id    | int     |
| purchase\_date | date    |
| units         | int     |
+---------------+---------+
This table may contain duplicate rows.
Each row of this table indicates the date, units, and product\_id of each product sold. 

Write a solution to find the average selling price for each product. `average_price` should be **rounded to 2 decimal places**. If a product does not have any sold units, its average selling price is assumed to be 0.

Return the result table in **any order**.

The result format is in the following example.

**Example 1:**

**Input:** 
Prices table:
+------------+------------+------------+--------+
| product\_id | start\_date | end\_date   | price  |
+------------+------------+------------+--------+
| 1          | 2019-02-17 | 2019-02-28 | 5      |
| 1          | 2019-03-01 | 2019-03-22 | 20     |
| 2          | 2019-02-01 | 2019-02-20 | 15     |
| 2          | 2019-02-21 | 2019-03-31 | 30     |
+------------+------------+------------+--------+
UnitsSold table:
+------------+---------------+-------+
| product\_id | purchase\_date | units |
+------------+---------------+-------+
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |
| 2          | 2019-03-22    | 30    |
+------------+---------------+-------+
**Output:** 
+------------+---------------+
| product\_id | average\_price |
+------------+---------------+
| 1          | 6.96          |
| 2          | 16.96         |
+------------+---------------+
**Explanation:** 
Average selling price = Total Price of Product / Number of products sold.
Average selling price for product 1 = ((100 \* 5) + (15 \* 20)) / 115 = 6.96
Average selling price for product 2 = ((200 \* 15) + (30 \* 30)) / 230 = 16.96

## Code Template

_No Go or TypeScript template available._
