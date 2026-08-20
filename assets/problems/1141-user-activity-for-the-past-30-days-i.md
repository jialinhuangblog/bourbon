---
id: 1141
title: "User Activity for the Past 30 Days I"
slug: user-activity-for-the-past-30-days-i
difficulty: Easy
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# User Activity for the Past 30 Days I

Table: `Activity`

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user\_id       | int     |
| session\_id    | int     |
| activity\_date | date    |
| activity\_type | enum    |
+---------------+---------+
This table may have duplicate rows.
The activity\_type column is an ENUM (category) of type ('open\_session', 'end\_session', 'scroll\_down', 'send\_message').
The table shows the user activities for a social media website. 
Note that each session belongs to exactly one user.

Write a solution to find the daily active user count for a period of `30` days ending `2019-07-27` inclusively. A user was active on someday if they made at least one activity on that day.

Return the result table in **any order**.

The result format is in the following example.

Note: **Any** activity from (`'open_session'`, `'end_session'`, `'scroll_down'`, `'send_message'`) will be considered valid activity for a user to be considered active on a day.

**Example 1:**

**Input:** 
Activity table:
+---------+------------+---------------+---------------+
| user\_id | session\_id | activity\_date | activity\_type |
+---------+------------+---------------+---------------+
| 1       | 1          | 2019-07-20    | open\_session  |
| 1       | 1          | 2019-07-20    | scroll\_down   |
| 1       | 1          | 2019-07-20    | end\_session   |
| 2       | 4          | 2019-07-20    | open\_session  |
| 2       | 4          | 2019-07-21    | send\_message  |
| 2       | 4          | 2019-07-21    | end\_session   |
| 3       | 2          | 2019-07-21    | open\_session  |
| 3       | 2          | 2019-07-21    | send\_message  |
| 3       | 2          | 2019-07-21    | end\_session   |
| 4       | 3          | 2019-06-25    | open\_session  |
| 4       | 3          | 2019-06-25    | end\_session   |
+---------+------------+---------------+---------------+
**Output:** 
+------------+--------------+ 
| day        | active\_users |
+------------+--------------+ 
| 2019-07-20 | 2            |
| 2019-07-21 | 2            |
+------------+--------------+ 
**Explanation:** Note that we do not care about days with zero active users.

## Code Template

_No Go or TypeScript template available._
