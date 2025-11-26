# SQL Intermediate – Joins, Aggregations, and Subqueries

This document assumes you are comfortable with basic `SELECT` queries and simple filters. We now move to:

- Joining tables together.
- Aggregating rows with GROUP BY.
- Filtering aggregated results with HAVING.
- Introducing simple subqueries.

We continue using the same `students`, `classes`, and `enrollments` tables.

---

## 1. INNER JOIN – combining related rows

An `INNER JOIN`, which is a join that keeps only matching rows between two tables, connects rows where a key column matches.

### Example: List each student's grade for each class

```sql
SELECT
    s.student_id,                         -- student identifier from students table
    s.first_name,                         -- student first name
    s.last_name,                          -- student last name
    c.class_name,                         -- class name from classes table
    e.numeric_grade                       -- numeric grade from enrollments table
FROM enrollments e                        -- start with the enrollments table, aliased as e
INNER JOIN students s                     -- inner join to students, aliased as s
    ON e.student_id = s.student_id        -- match enrollments to students where student_id is equal
INNER JOIN classes c                      -- inner join to classes, aliased as c
    ON e.class_id = c.class_id;           -- match enrollments to classes where class_id is equal
```

This query returns one row per enrollment with full student and class information and the numeric grade. If a student is not enrolled in any class, they will not appear here, because there is no matching row in `enrollments`.

---

## 2. LEFT JOIN – keep all rows from the left table

A `LEFT JOIN` keeps all rows from the left table even if there is no matching row on the right. Missing values from the right table appear as `NULL`.

### Example: Show all students and any classes they are enrolled in

```sql
SELECT
    s.student_id,                         -- student id
    s.first_name,                         -- student first name
    s.last_name,                          -- student last name
    c.class_name,                         -- class name if available, otherwise NULL
    e.numeric_grade                       -- numeric grade if available, otherwise NULL
FROM students s                           -- left table: students
LEFT JOIN enrollments e                   -- right table: enrollments
    ON s.student_id = e.student_id        -- match rows by student_id
LEFT JOIN classes c                       -- another right table: classes
    ON e.class_id = c.class_id;           -- match enrollments to classes by class_id
```

Students with no enrollments will still appear, but their `class_name` and `numeric_grade` columns will be `NULL`. This is useful when you want a "complete" list of students with optional related information.

---

## 3. RIGHT JOIN and FULL JOIN – conceptual explanation

Some databases support:

* `RIGHT JOIN`, which keeps all rows from the right table and matching rows from the left.
* `FULL JOIN` or `FULL OUTER JOIN`, which keeps all rows from both tables and uses `NULL` on whichever side does not have a match.

Even if your specific database does not support `FULL JOIN`, the concept is useful: it means "give me everything from both tables, whether or not they match."

---

## 4. Aggregation with GROUP BY

Aggregation means combining multiple rows into a single row using functions such as `AVG` for average, `SUM` for sum, `COUNT` for count, `MIN` for minimum, and `MAX` for maximum.

`GROUP BY` tells the database how to group rows before applying these functions.

### Example: Average grade per class

```sql
SELECT
    c.class_name,                             -- group key: the name of the class
    AVG(e.numeric_grade) AS avg_grade         -- average numeric_grade for this class
FROM enrollments e                            -- start from enrollments
INNER JOIN classes c                          -- join classes to get class names
    ON e.class_id = c.class_id                -- match on class_id
GROUP BY
    c.class_name;                             -- group rows by class_name
```

The result will contain one row per `class_name` and an `avg_grade` for each.

Example result:

| class_name | avg_grade |
| ---------: | --------: |
|  Algebra I |      83.5 |
|    Biology |      88.0 |

---

## 5. COUNT and GROUP BY – how many students per class?

### Example: Count how many enrollments each class has

```sql
SELECT
    c.class_name,                              -- group key: class name
    COUNT(*) AS enrollment_count               -- number of rows (enrollments) in each group
FROM enrollments e                             -- from enrollments
INNER JOIN classes c                           -- join to classes
    ON e.class_id = c.class_id                 -- match on class_id
GROUP BY
    c.class_name;                              -- group by class name
```

In this example, `COUNT(*)` counts the number of rows in each group, which corresponds to the number of students enrolled in that class.

---

## 6. HAVING – filtering after aggregation

`WHERE` filters rows before grouping. `HAVING` filters groups after the aggregation is computed.

### Example: Only show classes with average grade below 80

```sql
SELECT
    c.class_name,                              -- group key: class name
    AVG(e.numeric_grade) AS avg_grade          -- average numeric grade
FROM enrollments e                             -- from enrollments
INNER JOIN classes c                           -- join classes
    ON e.class_id = c.class_id                 -- match on class_id
GROUP BY
    c.class_name                               -- group by class name
HAVING
    AVG(e.numeric_grade) < 80;                 -- keep only classes where the average grade is below 80
```

In this query, the database performs these steps:

1. Joins `enrollments` to `classes`.
2. Groups rows by `class_name`.
3. Computes the average grade for each group.
4. Keeps only the groups where that average is below 80.

---

## 7. Simple subqueries

A subquery, which is a query inside another query, lets you use the result of one query as a value or a table in another.

### Example: Compare each student's average grade to the overall average

First, find the overall average grade:

```sql
SELECT
    AVG(numeric_grade) AS overall_avg_grade   -- compute a single overall average
FROM enrollments;                             -- from all enrollment rows
```

Now, use this result inside another query.

```sql
SELECT
    s.student_id,                                      -- student id
    s.first_name,                                      -- first name
    s.last_name,                                       -- last name
    AVG(e.numeric_grade) AS student_avg_grade          -- average grade for this student
FROM enrollments e                                     -- from enrollments
INNER JOIN students s                                  -- join students
    ON e.student_id = s.student_id                     -- match on student_id
GROUP BY
    s.student_id,
    s.first_name,
    s.last_name
HAVING
    AVG(e.numeric_grade) > (                           -- keep only students whose average grade is greater than
        SELECT AVG(numeric_grade)                      -- the overall average grade
        FROM enrollments
    );
```

The subquery in parentheses returns a single value, which is the overall average grade. The `HAVING` clause compares each student's average to this number.

---

You now know how to combine tables with joins and how to summarize data with GROUP BY and HAVING. The next document, `sql_advanced.md`, builds on this to introduce CTEs and window functions, which are powerful tools for advanced analytics queries.
