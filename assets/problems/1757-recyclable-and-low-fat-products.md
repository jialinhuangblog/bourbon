---
id: 1757
title: "Recyclable and Low Fat Products"
slug: recyclable-and-low-fat-products
difficulty: Easy
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# Recyclable and Low Fat Products

Table: `Products`

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product\_id  | int     |
| low\_fats    | enum    |
| recyclable  | enum    |
+-------------+---------+
product\_id is the primary key (column with unique values) for this table.
low\_fats is an ENUM (category) of type ('Y', 'N') where 'Y' means this product is low fat and 'N' means it is not.
recyclable is an ENUM (category) of types ('Y', 'N') where 'Y' means this product is recyclable and 'N' means it is not.

Write a solution to find the ids of products that are both low fat and recyclable.

Return the result table in **any order**.

The result format is in the following example.

**Example 1:**

**Input:** 
Products table:
+-------------+----------+------------+
| product\_id  | low\_fats | recyclable |
+-------------+----------+------------+
| 0           | Y        | N          |
| 1           | Y        | Y          |
| 2           | N        | Y          |
| 3           | Y        | Y          |
| 4           | N        | N          |
+-------------+----------+------------+
**Output:** 
+-------------+
| product\_id  |
+-------------+
| 1           |
| 3           |
+-------------+
**Explanation:** Only products 1 and 3 are both low fat and recyclable.

## Code Template

_No Go or TypeScript template available._
