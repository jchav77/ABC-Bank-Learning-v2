# SQL Oracle-Specific Features

This document covers SQL features that are specific to Oracle Database. While the other SQL documents in this repo use syntax that works across most databases (PostgreSQL, MySQL, SQL Server), the features here are Oracle-only.

We cover:

- The `KEEP (DENSE_RANK FIRST/LAST)` clause for retrieving values from first or last rows in groups.
- `FIRST_VALUE` and `LAST_VALUE` window functions as alternatives.
- Other useful Oracle-specific features.

We continue using the school schema from the other SQL documents.

---

## 1. KEEP (DENSE_RANK FIRST/LAST) – Get values from first or last rows

The `KEEP` clause is used with aggregate functions to return a value from the first or last row within an ordered group. This is extremely useful when you need to retrieve a non-aggregated value that corresponds to a minimum or maximum of another column.

### Definition

- **KEEP (DENSE_RANK FIRST ORDER BY ...)**: Returns the value from the row(s) that rank first according to the ORDER BY expression.
- **KEEP (DENSE_RANK LAST ORDER BY ...)**: Returns the value from the row(s) that rank last according to the ORDER BY expression.

The `DENSE_RANK` in the syntax means that if there are ties for first or last, all tied rows are considered, and the aggregate function (MIN, MAX, etc.) is applied to those tied values.

### Why use KEEP instead of a subquery?

Without `KEEP`, you would need a correlated subquery or a self-join to get a value from the row with the minimum or maximum of another column. `KEEP` makes this much simpler and more efficient.

---

## 2. Example data

Imagine we have test scores for students, and each student has taken multiple tests:

| student_id | first_name | test_date   | score |
| ---------: | ---------: | ----------: | ----: |
|          1 |      Alice | 2024-01-15  |    85 |
|          1 |      Alice | 2024-02-15  |    90 |
|          1 |      Alice | 2024-03-15  |    88 |
|          2 |        Ben | 2024-01-15  |    78 |
|          2 |        Ben | 2024-02-15  |    82 |
|          2 |        Ben | 2024-03-15  |    80 |
|          3 |      Carla | 2024-01-15  |    92 |
|          3 |      Carla | 2024-02-15  |    95 |
|          3 |      Carla | 2024-03-15  |    93 |

---

## 3. Basic KEEP example – Get the first and last test scores

### Example: For each student, get their first test score and their most recent test score

```sql
SELECT
    student_id,                                              -- student identifier
    first_name,                                              -- student's first name
    MIN(test_date) AS first_test_date,                       -- date of the first test
    MAX(score) KEEP (DENSE_RANK FIRST ORDER BY test_date)    -- score from the first test (earliest date)
        AS first_test_score,
    MAX(test_date) AS last_test_date,                        -- date of the most recent test
    MAX(score) KEEP (DENSE_RANK LAST ORDER BY test_date)     -- score from the last test (most recent date)
        AS last_test_score
FROM test_scores                                             -- from the test_scores table
GROUP BY
    student_id,                                              -- group by student_id
    first_name;                                              -- and first_name
```

### Example output

| student_id | first_name | first_test_date | first_test_score | last_test_date | last_test_score |
| ---------: | ---------: | --------------: | ---------------: | -------------: | --------------: |
|          1 |      Alice |      2024-01-15 |               85 |     2024-03-15 |              88 |
|          2 |        Ben |      2024-01-15 |               78 |     2024-03-15 |              80 |
|          3 |      Carla |      2024-01-15 |               92 |     2024-03-15 |              93 |

### Interpretation

- `MAX(score) KEEP (DENSE_RANK FIRST ORDER BY test_date)` finds the row(s) with the earliest `test_date` for each student, then returns the `MAX(score)` from those rows. Since there's typically only one row per date per student, this effectively returns the score from the first test.

- `MAX(score) KEEP (DENSE_RANK LAST ORDER BY test_date)` does the same but for the most recent date.

- We use `MAX(score)` as the aggregate function, but since there's usually only one row that ranks first or last, it just returns that single value. If there were ties (multiple tests on the same date), `MAX` would return the highest score among the tied rows.

---

## 4. KEEP with different aggregate functions

The aggregate function you use with `KEEP` matters when there are ties.

### Example: What if a student took two tests on the same day?

Imagine Alice took two tests on 2024-01-15 with scores 85 and 87.

```sql
SELECT
    student_id,
    first_name,
    MIN(score) KEEP (DENSE_RANK FIRST ORDER BY test_date)    -- minimum score from first test date
        AS first_day_min_score,
    MAX(score) KEEP (DENSE_RANK FIRST ORDER BY test_date)    -- maximum score from first test date
        AS first_day_max_score,
    AVG(score) KEEP (DENSE_RANK FIRST ORDER BY test_date)    -- average score from first test date
        AS first_day_avg_score
FROM test_scores
GROUP BY
    student_id,
    first_name;
```

### Interpretation

- `MIN(score) KEEP (...)` returns the lowest score among the tied first-date rows (85).
- `MAX(score) KEEP (...)` returns the highest score among the tied first-date rows (87).
- `AVG(score) KEEP (...)` returns the average of scores from the tied first-date rows (86).

---

## 5. Practical example – Get the class where each student has their highest grade

Using our school schema with `enrollments` and `classes`:

```sql
SELECT
    s.student_id,                                            -- student identifier
    s.first_name,                                            -- student's first name
    s.last_name,                                             -- student's last name
    MAX(e.numeric_grade) AS highest_grade,                   -- the student's highest grade across all classes
    MAX(c.class_name) KEEP (DENSE_RANK FIRST ORDER BY e.numeric_grade DESC)
        AS best_class                                        -- name of the class where they got the highest grade
FROM students s                                              -- from students table
INNER JOIN enrollments e                                     -- join to enrollments
    ON s.student_id = e.student_id                           -- match on student_id
INNER JOIN classes c                                         -- join to classes
    ON e.class_id = c.class_id                               -- match on class_id
GROUP BY
    s.student_id,                                            -- group by student_id
    s.first_name,                                            -- and first_name
    s.last_name;                                             -- and last_name
```

### Example output

| student_id | first_name | last_name | highest_grade | best_class |
| ---------: | ---------: | --------: | ------------: | ---------: |
|          1 |      Alice |    Nguyen |          98.0 |    Biology |
|          2 |        Ben |     Patel |          92.0 | Algebra II |
|          3 |      Carla |      Diaz |          95.0 |    History |

### Interpretation

The `KEEP (DENSE_RANK FIRST ORDER BY e.numeric_grade DESC)` clause finds the row(s) with the highest grade for each student, then `MAX(c.class_name)` returns the class name from that row. If a student has the same highest grade in multiple classes, `MAX` returns the alphabetically last class name among those tied classes.

---

## 6. KEEP vs subquery approach

Without `KEEP`, you would need a more complex query:

### Without KEEP (using a correlated subquery)

```sql
SELECT
    s.student_id,
    s.first_name,
    s.last_name,
    e.numeric_grade AS highest_grade,
    c.class_name AS best_class
FROM students s
INNER JOIN enrollments e
    ON s.student_id = e.student_id
INNER JOIN classes c
    ON e.class_id = c.class_id
WHERE e.numeric_grade = (                                    -- subquery to find the max grade for this student
    SELECT MAX(e2.numeric_grade)
    FROM enrollments e2
    WHERE e2.student_id = s.student_id
);
```

The `KEEP` approach is more concise and often more efficient because it avoids the correlated subquery.

---

## 7. FIRST_VALUE and LAST_VALUE – Window function alternatives

Oracle (and other databases) also provides `FIRST_VALUE` and `LAST_VALUE` window functions, which can achieve similar results but work differently.

### Key differences

| Feature | KEEP | FIRST_VALUE / LAST_VALUE |
| ------- | ---- | ------------------------ |
| Type | Used with aggregate functions in GROUP BY | Window functions (no GROUP BY needed) |
| Result | One row per group | Value attached to every row |
| Availability | Oracle only | Most databases (Oracle, PostgreSQL, SQL Server, etc.) |

### Example: Using FIRST_VALUE to get the first test score

```sql
SELECT
    student_id,                                              -- student identifier
    first_name,                                              -- student's first name
    test_date,                                               -- date of this test
    score,                                                   -- score on this test
    FIRST_VALUE(score) OVER (                                -- get the first score
        PARTITION BY student_id                              -- for each student
        ORDER BY test_date                                   -- ordered by test date (earliest first)
    ) AS first_test_score,                                   -- score from the first test
    LAST_VALUE(score) OVER (                                 -- get the last score
        PARTITION BY student_id                              -- for each student
        ORDER BY test_date                                   -- ordered by test date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  -- include all rows in the partition
    ) AS last_test_score                                     -- score from the most recent test
FROM test_scores                                             -- from test_scores table
ORDER BY student_id, test_date;                              -- order output by student and date
```

### Example output

| student_id | first_name | test_date   | score | first_test_score | last_test_score |
| ---------: | ---------: | ----------: | ----: | ---------------: | --------------: |
|          1 |      Alice | 2024-01-15  |    85 |               85 |              88 |
|          1 |      Alice | 2024-02-15  |    90 |               85 |              88 |
|          1 |      Alice | 2024-03-15  |    88 |               85 |              88 |
|          2 |        Ben | 2024-01-15  |    78 |               78 |              80 |
|          2 |        Ben | 2024-02-15  |    82 |               78 |              80 |
|          2 |        Ben | 2024-03-15  |    80 |               78 |              80 |

### Important note about LAST_VALUE

By default, the window frame for `LAST_VALUE` only includes rows from the start of the partition to the current row. This means `LAST_VALUE` would return the current row's value, not the actual last value in the partition.

To get the true last value, you must specify:
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

This tells Oracle to consider all rows in the partition, not just up to the current row.

---

## 8. When to use KEEP vs FIRST_VALUE/LAST_VALUE

| Use KEEP when... | Use FIRST_VALUE/LAST_VALUE when... |
| ---------------- | ---------------------------------- |
| You are already using GROUP BY | You want to keep all rows (no grouping) |
| You need one summary row per group | You need the value attached to every row |
| You are working only in Oracle | You need cross-database compatibility |
| You want concise syntax for aggregation | You need more flexibility with window frames |

---

## 9. Other useful Oracle-specific features

### ROWNUM – Limiting rows (Oracle classic approach)

Before Oracle 12c, Oracle did not have `LIMIT`. Instead, you use `ROWNUM`:

```sql
SELECT *
FROM (
    SELECT
        student_id,                                          -- student identifier
        first_name,                                          -- student's first name
        last_name                                            -- student's last name
    FROM students
    ORDER BY last_name, first_name                           -- order by name
)
WHERE ROWNUM <= 10;                                          -- keep only the first 10 rows
```

### FETCH FIRST (Oracle 12c and later)

Oracle 12c introduced standard SQL syntax for limiting rows:

```sql
SELECT
    student_id,                                              -- student identifier
    first_name,                                              -- student's first name
    last_name                                                -- student's last name
FROM students                                                -- from students table
ORDER BY last_name, first_name                               -- order by name
FETCH FIRST 10 ROWS ONLY;                                    -- keep only the first 10 rows
```

### NVL and NVL2 – Handling NULL values

Oracle uses `NVL` instead of `COALESCE` (though `COALESCE` also works):

```sql
SELECT
    student_id,
    NVL(numeric_grade, 0) AS grade_or_zero                   -- if numeric_grade is NULL, return 0
FROM enrollments;
```

`NVL2` provides if-then-else logic based on NULL:

```sql
SELECT
    student_id,
    NVL2(numeric_grade, 'Graded', 'Not Graded') AS status    -- if numeric_grade is not NULL, return 'Graded', else 'Not Graded'
FROM enrollments;
```

### DECODE – Simple conditional logic

`DECODE` is Oracle's way of doing simple CASE-like logic:

```sql
SELECT
    student_id,
    DECODE(grade_level,                                      -- check grade_level
        9, 'Freshman',                                       -- if 9, return 'Freshman'
        10, 'Sophomore',                                     -- if 10, return 'Sophomore'
        11, 'Junior',                                        -- if 11, return 'Junior'
        12, 'Senior',                                        -- if 12, return 'Senior'
        'Unknown'                                            -- default value if no match
    ) AS grade_name
FROM students;
```

Note: `CASE` expressions are preferred in modern Oracle code because they are more readable and SQL-standard.

---

## 10. Summary

In this document, you learned:

1. **KEEP (DENSE_RANK FIRST/LAST)**: How to retrieve values from the first or last rows in a group based on an ordering, combined with aggregate functions.

2. **FIRST_VALUE and LAST_VALUE**: Window function alternatives that work across more databases but behave differently (no grouping, value on every row).

3. **When to use each approach**: KEEP for grouped aggregations in Oracle, FIRST_VALUE/LAST_VALUE for cross-database compatibility or when you need values on every row.

4. **Other Oracle features**: ROWNUM, FETCH FIRST, NVL, NVL2, and DECODE.

The `KEEP` clause is particularly powerful for reporting queries where you need to show a value from a specific row (like the most recent or the first) alongside aggregated totals, all in a single, efficient query.
