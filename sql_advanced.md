# SQL Advanced – CTEs, Window Functions, and Common Patterns

This document assumes you are comfortable with joins, aggregations, and simple subqueries. We now cover:

- Common Table Expressions (CTEs) using `WITH`.
- Window functions such as `RANK`, `DENSE_RANK`, `ROW_NUMBER`, and moving averages.
- Patterns like "top N per group."

We continue using the school schema.

---

## 1. Common Table Expressions (CTEs) with WITH

A Common Table Expression, abbreviated as CTE, is a named temporary result that you can reference in a later query. It makes complex queries easier to read and maintain.

### Example: Compute each student's average grade, then filter by that average

```sql
WITH student_averages AS (                       -- define a CTE named student_averages
    SELECT
        s.student_id,                            -- student id
        s.first_name,                            -- student first name
        s.last_name,                             -- student last name
        AVG(e.numeric_grade) AS avg_grade        -- average numeric grade per student
    FROM enrollments e                           -- from enrollments
    INNER JOIN students s                        -- join students
        ON e.student_id = s.student_id           -- match on student_id
    GROUP BY
        s.student_id,
        s.first_name,
        s.last_name
)
SELECT
    student_id,                                  -- select from the CTE
    first_name,
    last_name,
    avg_grade
FROM student_averages                            -- use the CTE as if it were a table
WHERE avg_grade >= 85;                           -- keep only students with average grade at least 85
```

The CTE allows you to break the problem into two phases: first compute per-student averages, then filter them.

---

## 2. Window functions – analytics across rows

A window function computes a value across a set of rows related to the current row, without collapsing those rows into a single row. This is different from aggregation with GROUP BY, which reduces multiple rows into one.

Window functions use the `OVER` clause to define the "window" of rows they operate on.

### Example: Rank students within each grade level by average grade

```sql
WITH student_averages AS (                           -- define CTE to compute per-student average grade
    SELECT
        s.student_id,                                -- student id
        s.first_name,                                -- first name
        s.last_name,                                 -- last name
        s.grade_level,                               -- grade level, such as 9 or 10
        AVG(e.numeric_grade) AS avg_grade            -- average numeric grade for the student
    FROM enrollments e                               -- from enrollments
    INNER JOIN students s                            -- join students
        ON e.student_id = s.student_id               -- match on student_id
    GROUP BY
        s.student_id,
        s.first_name,
        s.last_name,
        s.grade_level
)
SELECT
    student_id,                                      -- select columns from CTE
    first_name,
    last_name,
    grade_level,
    avg_grade,
    RANK() OVER (                                    -- compute rank using a window function
        PARTITION BY grade_level                     -- partition, meaning group, by grade_level
        ORDER BY avg_grade DESC                      -- within each grade_level, rank higher avg_grade first
    ) AS rank_in_grade                               -- alias the window function result as rank_in_grade
FROM student_averages;                               -- query from the CTE
```

Each row still represents one student, and we attach an additional column that shows that student's rank within their grade level.

---

## 3. RANK vs DENSE_RANK vs ROW_NUMBER

When ranking rows, SQL provides three similar but distinct functions. Understanding the difference is important for choosing the right one for your use case.

### Definitions

- **RANK()**: Assigns the same rank to rows with equal values, then skips ranks. For example, if two students tie for 1st place, the next student gets rank 3 (not 2).

- **DENSE_RANK()**: Assigns the same rank to rows with equal values, but does not skip ranks. For example, if two students tie for 1st place, the next student gets rank 2.

- **ROW_NUMBER()**: Assigns a unique sequential number to each row, even if values are equal. Ties are broken arbitrarily (or by additional ORDER BY columns).

### Example data

Imagine we have these average grades for students in grade 10:

| student_id | first_name | avg_grade |
| ---------: | ---------: | --------: |
|          1 |      Alice |      95.0 |
|          2 |        Ben |      95.0 |
|          3 |      Carla |      90.0 |
|          4 |       Dana |      85.0 |

### Example: Comparing all three ranking functions

```sql
WITH student_averages AS (                           -- CTE to compute average grades
    SELECT
        s.student_id,                                -- student id
        s.first_name,                                -- first name
        s.grade_level,                               -- grade level
        AVG(e.numeric_grade) AS avg_grade            -- average grade
    FROM enrollments e
    INNER JOIN students s
        ON e.student_id = s.student_id
    WHERE s.grade_level = 10                         -- focus on grade 10 for this example
    GROUP BY
        s.student_id,
        s.first_name,
        s.grade_level
)
SELECT
    student_id,                                      -- student id
    first_name,                                      -- first name
    avg_grade,                                       -- average grade
    RANK() OVER (
        ORDER BY avg_grade DESC                      -- rank by avg_grade descending
    ) AS rank_value,                                 -- RANK result
    DENSE_RANK() OVER (
        ORDER BY avg_grade DESC                      -- dense rank by avg_grade descending
    ) AS dense_rank_value,                           -- DENSE_RANK result
    ROW_NUMBER() OVER (
        ORDER BY avg_grade DESC                      -- row number by avg_grade descending
    ) AS row_num                                     -- ROW_NUMBER result
FROM student_averages
ORDER BY avg_grade DESC;                             -- order final output by avg_grade
```

### Example output

| student_id | first_name | avg_grade | rank_value | dense_rank_value | row_num |
| ---------: | ---------: | --------: | ---------: | ---------------: | ------: |
|          1 |      Alice |      95.0 |          1 |                1 |       1 |
|          2 |        Ben |      95.0 |          1 |                1 |       2 |
|          3 |      Carla |      90.0 |          3 |                2 |       3 |
|          4 |       Dana |      85.0 |          4 |                3 |       4 |

### Interpretation

- **RANK**: Alice and Ben both get rank 1 because they are tied. Carla gets rank 3 because ranks 1 and 2 were both "used" by the tie. Dana gets rank 4.

- **DENSE_RANK**: Alice and Ben both get rank 1. Carla gets rank 2 because DENSE_RANK does not skip numbers after a tie. Dana gets rank 3.

- **ROW_NUMBER**: Each row gets a unique number. Alice gets 1, Ben gets 2 (the order between tied rows is arbitrary unless you add more ORDER BY columns), Carla gets 3, Dana gets 4.

### When to use each

| Function | Use when... |
| -------- | ----------- |
| RANK() | You want ties to share a rank, and you want the next rank to reflect how many rows came before (e.g., "3rd place" means two people finished ahead) |
| DENSE_RANK() | You want ties to share a rank, but you want consecutive rank numbers without gaps (e.g., for "top 3 distinct score levels") |
| ROW_NUMBER() | You need a unique identifier for each row, regardless of ties (e.g., for pagination or selecting exactly N rows) |

### Example: Using DENSE_RANK for "top 3 score levels"

If you want students who achieved the top 3 distinct average grades (not the top 3 students), use DENSE_RANK:

```sql
WITH student_averages AS (                           -- CTE to compute average grades
    SELECT
        s.student_id,
        s.first_name,
        s.grade_level,
        AVG(e.numeric_grade) AS avg_grade
    FROM enrollments e
    INNER JOIN students s
        ON e.student_id = s.student_id
    GROUP BY
        s.student_id,
        s.first_name,
        s.grade_level
),
ranked AS (
    SELECT
        student_id,
        first_name,
        grade_level,
        avg_grade,
        DENSE_RANK() OVER (
            PARTITION BY grade_level                 -- rank within each grade level
            ORDER BY avg_grade DESC                  -- highest average first
        ) AS score_level                             -- this represents the "tier" of scores
    FROM student_averages
)
SELECT
    student_id,                                      -- student id
    first_name,                                      -- first name
    grade_level,                                     -- grade level
    avg_grade,                                       -- average grade
    score_level                                      -- the score tier (1 = highest, 2 = second highest, etc.)
FROM ranked
WHERE score_level <= 3;                              -- keep top 3 score levels (may return more than 3 students if there are ties)
```

With the example data above (for grade 10), this would return all 4 students because there are only 3 distinct score levels (95.0, 90.0, 85.0), and both Alice and Ben share the top level.

---

## 4. Top N per group using window functions (with RANK)

A very common requirement is "top N items per group," for example the top 2 students per grade level.

### Example: Top 2 students per grade level by average grade

```sql
WITH student_averages AS (                           -- compute per-student average grade
    SELECT
        s.student_id,
        s.first_name,
        s.last_name,
        s.grade_level,
        AVG(e.numeric_grade) AS avg_grade
    FROM enrollments e
    INNER JOIN students s
        ON e.student_id = s.student_id
    GROUP BY
        s.student_id,
        s.first_name,
        s.last_name,
        s.grade_level
),
ranked AS (                                          -- add a rank within each grade level
    SELECT
        student_id,
        first_name,
        last_name,
        grade_level,
        avg_grade,
        RANK() OVER (
            PARTITION BY grade_level                 -- separate ranking by grade_level
            ORDER BY avg_grade DESC                  -- higher averages get smaller rank numbers
        ) AS rank_in_grade
    FROM student_averages
)
SELECT
    student_id,                                      -- final selection: student id
    first_name,                                      -- first name
    last_name,                                       -- last name
    grade_level,                                     -- grade level
    avg_grade,                                       -- average grade
    rank_in_grade                                    -- rank within grade level
FROM ranked
WHERE rank_in_grade <= 2;                            -- keep only top 2 per grade level
```

The ranking logic is separated from the filtering logic using two CTEs, which keeps the query readable.

---

## 5. Moving averages with window functions

A moving average, which is a rolling average over a fixed number of rows, can smooth out variation over time. For a school example, imagine we had tests over time; for demonstration we will just show the pattern.

### Example: Moving average of grades per student (conceptual)

Assume a table `test_scores` with columns:

* `student_id`
* `test_date`
* `score`

A moving average over the last 3 tests for each student could look like this:

```sql
SELECT
    student_id,                                                       -- student identifier
    test_date,                                                        -- date of this test
    score,                                                            -- score on this test
    AVG(score) OVER (                                                 -- moving average window function
        PARTITION BY student_id                                       -- separate calculation for each student
        ORDER BY test_date                                            -- define order within each student by test date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW                      -- include current row and previous 2 rows
    ) AS moving_avg_score                                             -- name of the moving average column
FROM test_scores;                                                     -- source table
```

The window frame `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` means: for each test, look at that test and the previous two tests for the same student, then compute the average score.

---

This should give you a working understanding of CTEs and window functions, which are crucial for advanced analytics queries. The next section of the repo, `python_sql.md`, will show you how to run SQL queries from Python and how to bridge the SQL world with pandas.
