---
id: 1633
title: "Percentage of Users Attended a Contest"
slug: percentage-of-users-attended-a-contest
difficulty: Easy
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# Percentage of Users Attended a Contest

Table: `Users`

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| user\_id     | int     |
| user\_name   | varchar |
+-------------+---------+
user\_id is the primary key (column with unique values) for this table.
Each row of this table contains the name and the id of a user.

Table: `Register`

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| contest\_id  | int     |
| user\_id     | int     |
+-------------+---------+
(contest\_id, user\_id) is the primary key (combination of columns with unique values) for this table.
Each row of this table contains the id of a user and the contest they registered into.

Write a solution to find the percentage of the users registered in each contest rounded to **two decimals**.

Return the result table ordered by `percentage` in **descending order**. In case of a tie, order it by `contest_id` in **ascending order**.

The result format is in the following example.

**Example 1:**

**Input:** 
Users table:
+---------+-----------+
| user\_id | user\_name |
+---------+-----------+
| 6       | Alice     |
| 2       | Bob       |
| 7       | Alex      |
+---------+-----------+
Register table:
+------------+---------+
| contest\_id | user\_id |
+------------+---------+
| 215        | 6       |
| 209        | 2       |
| 208        | 2       |
| 210        | 6       |
| 208        | 6       |
| 209        | 7       |
| 209        | 6       |
| 215        | 7       |
| 208        | 7       |
| 210        | 2       |
| 207        | 2       |
| 210        | 7       |
+------------+---------+
**Output:** 
+------------+------------+
| contest\_id | percentage |
+------------+------------+
| 208        | 100.0      |
| 209        | 100.0      |
| 210        | 100.0      |
| 215        | 66.67      |
| 207        | 33.33      |
+------------+------------+
**Explanation:** 
All the users registered in contests 208, 209, and 210. The percentage is 100% and we sort them in the answer table by contest\_id in ascending order.
Alice and Alex registered in contest 215 and the percentage is ((2/3) \* 100) = 66.67%
Bob registered in contest 207 and the percentage is ((1/3) \* 100) = 33.33%

## Code Template

_No Go or TypeScript template available._
