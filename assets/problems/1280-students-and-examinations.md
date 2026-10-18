---
id: 1280
title: "Students and Examinations"
slug: students-and-examinations
difficulty: Easy
tags: [Database]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: []
time_complexity: null
space_complexity: null
---

# Students and Examinations

Table: `Students`

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student\_id    | int     |
| student\_name  | varchar |
+---------------+---------+
student\_id is the primary key (column with unique values) for this table.
Each row of this table contains the ID and the name of one student in the school.

Table: `Subjects`

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| subject\_name | varchar |
+--------------+---------+
subject\_name is the primary key (column with unique values) for this table.
Each row of this table contains the name of one subject in the school.

Table: `Examinations`

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| student\_id   | int     |
| subject\_name | varchar |
+--------------+---------+
There is no primary key (column with unique values) for this table. It may contain duplicates.
Each student from the Students table takes every course from the Subjects table.
Each row of this table indicates that a student with ID student\_id attended the exam of subject\_name.

Write a solution to find the number of times each student attended each exam.

Return the result table ordered by `student_id` and `subject_name`.

The result format is in the following example.

**Example 1:**

**Input:** 
Students table:
+------------+--------------+
| student\_id | student\_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
Subjects table:
+--------------+
| subject\_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+
Examinations table:
+------------+--------------+
| student\_id | subject\_name |
+------------+--------------+
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |
+------------+--------------+
**Output:** 
+------------+--------------+--------------+----------------+
| student\_id | student\_name | subject\_name | attended\_exams |
+------------+--------------+--------------+----------------+
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |
+------------+--------------+--------------+----------------+
**Explanation:** 
The result table should contain all students and all subjects.
Alice attended the Math exam 3 times, the Physics exam 2 times, and the Programming exam 1 time.
Bob attended the Math exam 1 time, the Programming exam 1 time, and did not attend the Physics exam.
Alex did not attend any exams.
John attended the Math exam 1 time, the Physics exam 1 time, and the Programming exam 1 time.

## Code Template

_No Go or TypeScript template available._
