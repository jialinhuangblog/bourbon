---
id: 1070
title: "Product Sales Analysis III"
slug: product-sales-analysis-iii
difficulty: Medium
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# Product Sales Analysis III

Table: `Sales`

+-------------+-------+
| Column Name | Type  |
+-------------+-------+
| sale\_id     | int   |
| product\_id  | int   |
| year        | int   |
| quantity    | int   |
| price       | int   |
+-------------+-------+
(sale\_id, year) is the primary key (combination of columns with unique values) of this table.
Each row records a sale of a product in a given year.
A product may have multiple sales entries in the same year.
Note that the per-unit price.

Write a solution to find all sales that occurred in the **first year** each product was sold.

*   For each `product_id`, identify the earliest `year` it appears in the `Sales` table.
    
*   Return **all** sales entries for that product in that year.
    

Return a table with the following columns: **product\_id**, **first\_year**, **quantity,** and **price**.  
Return the result in any order.

**Example 1:**

**Input:** 
Sales table:
+---------+------------+------+----------+-------+
| sale\_id | product\_id | year | quantity | price |
+---------+------------+------+----------+-------+ 
| 1       | 100        | 2008 | 10       | 5000  |
| 2       | 100        | 2009 | 12       | 5000  |
| 7       | 200        | 2011 | 15       | 9000  |
+---------+------------+------+----------+-------+

**Output:** 
+------------+------------+----------+-------+
| product\_id | first\_year | quantity | price |
+------------+------------+----------+-------+ 
| 100        | 2008       | 10       | 5000  |
| 200        | 2011       | 15       | 9000  |
+------------+------------+----------+-------+

## Code Template

_No Go or TypeScript template available._
