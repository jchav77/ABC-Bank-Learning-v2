# SQL Basics – Students, Classes, and Grades

This document introduces SQL, which stands for Structured Query Language, meaning a language used to interact with relational databases. We will use a simple, imaginary school database with students, classes, and enrollments.

Our goal is to help you become comfortable with:

- Selecting data from a table.
- Filtering rows.
- Sorting.
- Using simple conditions.

Every query is followed by a full-sentence explanation and, when helpful, an example of what the result would look like.

---

## 1. The sample schema

We will imagine three tables.

- `students` holds information about each student.
- `classes` holds information about each class.
- `enrollments` ties students to classes and stores their grades.

Here is how these tables might be defined in SQL.

```sql
CREATE TABLE students (
    student_id   INT PRIMARY KEY,      -- unique identifier for each student
    first_name   VARCHAR(50),          -- student's first name
    last_name    VARCHAR(50),          -- student's last name
    grade_level  INT                   -- grade level, for example 9, 10, 11, 12
);

CREATE TABLE classes (
    class_id     INT PRIMARY KEY,      -- unique identifier for each class
    class_name   VARCHAR(100),         -- descriptive name of the class
    teacher_name VARCHAR(100)          -- full name of the teacher
);

CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY,     -- unique identifier for each enrollment
    student_id    INT,                 -- foreign key referencing students
    class_id      INT,                 -- foreign key referencing classes
    numeric_grade DECIMAL(5,2),        -- numeric grade, for example 0 to 100
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (class_id)  REFERENCES classes(class_id)
);
```

You do not need to recreate these tables exactly in your own environment, but it helps to understand what is in them as you read the queries.

---

## 2. Basic SELECT

A `SELECT` statement asks the database to return specific columns from one or more tables.

### Example: Get all students

```sql
SELECT
    student_id,          -- choose the student_id column
    first_name,          -- choose the first_name column
    last_name,           -- choose the last_name column
    grade_level          -- choose the grade_level column
FROM students;           -- specify that we are reading from the students table
```

This query returns every row in the `students` table and only the specified columns. There is no filtering or sorting yet.

Example result:

| student_id | first_name | last_name | grade_level |
| ---------: | ---------: | --------: | ----------: |
|          1 |      Alice |    Nguyen |           9 |
|          2 |        Ben |     Patel |          10 |
|          3 |      Carla |      Diaz |           9 |

---

## 3. Filtering rows with WHERE

The `WHERE` clause tells the database which rows to keep.

### Example: Get only 9th graders

```sql
SELECT
    student_id,                -- show the student id
    first_name,                -- show the first name
    last_name,                 -- show the last name
    grade_level                -- show the grade level
FROM students                  -- from the students table
WHERE grade_level = 9;         -- keep only rows where grade_level equals 9
```

This query returns only students in grade 9.

Example result:

| student_id | first_name | last_name | grade_level |
| ---------: | ---------: | --------: | ----------: |
|          1 |      Alice |    Nguyen |           9 |
|          3 |      Carla |      Diaz |           9 |

### Example: Combine conditions using AND and OR

```sql
SELECT
    student_id,                          -- show the student id
    first_name,                          -- show the first name
    last_name,                           -- show the last name
    grade_level                          -- show the grade level
FROM students                            -- from the students table
WHERE grade_level = 9                    -- keep only 9th graders
  AND last_name = 'Nguyen';              -- and with last name exactly 'Nguyen'
```

Here we use:

* `AND`, meaning both conditions must be true for a row to be included.
* `OR`, which you can use when either condition can be true.

For example, to get 9th or 10th graders:

```sql
SELECT
    student_id,                     -- show the student id
    first_name,                     -- show the first name
    last_name,                      -- show the last name
    grade_level                     -- show the grade level
FROM students                       -- from the students table
WHERE grade_level = 9               -- keep row if grade_level equals 9
   OR grade_level = 10;             -- or if grade_level equals 10
```

---

## 4. Sorting results with ORDER BY

The `ORDER BY` clause controls the order of returned rows.

### Example: Order students by last name then first name

```sql
SELECT
    student_id,                        -- show the student id
    first_name,                        -- show the first name
    last_name,                         -- show the last name
    grade_level                        -- show the grade level
FROM students                          -- from the students table
ORDER BY
    last_name ASC,                     -- sort by last_name in ascending order (A to Z)
    first_name ASC;                    -- when last_name ties, sort by first_name ascending
```

`ASC` means ascending order. `DESC` means descending order.

---

## 5. Limiting the number of rows

Different databases use slightly different syntax for limiting rows. Many databases such as PostgreSQL and MySQL use `LIMIT`.

### Example: Get the top 5 students by student_id

```sql
SELECT
    student_id,                       -- show the student id
    first_name,                       -- show the first name
    last_name,                        -- show the last name
    grade_level                       -- show the grade level
FROM students                         -- from the students table
ORDER BY student_id ASC               -- order by student_id in ascending order
LIMIT 5;                              -- keep only the first 5 rows from the ordered result
```

If there are more than 5 students, this query returns only the first 5 after sorting.

---

## 6. Basic numeric filters

### Example: Find enrollments with high grades

```sql
SELECT
    enrollment_id,                    -- show the enrollment id
    student_id,                       -- show which student this is
    class_id,                         -- show which class this is
    numeric_grade                     -- show the numeric grade
FROM enrollments                      -- from the enrollments table
WHERE numeric_grade >= 90;            -- keep rows where numeric_grade is at least 90
```

The condition `numeric_grade >= 90` keeps only rows with numeric grades of 90 or higher.

### Example: Use BETWEEN for a range

```sql
SELECT
    enrollment_id,                    -- show the enrollment id
    student_id,                       -- show the student id
    class_id,                         -- show the class id
    numeric_grade                     -- show the numeric grade
FROM enrollments                      -- from the enrollments table
WHERE numeric_grade BETWEEN 80 AND 89;-- keep rows where grade is between 80 and 89 inclusive
```

`BETWEEN` includes both ends of the range, meaning 80 and 89 are both allowed.

---

## 7. IN for multiple allowed values

`IN` is a convenient way to check if a value is in a list of options.

### Example: Get 9th and 10th graders using IN

```sql
SELECT
    student_id,                             -- show the student id
    first_name,                             -- show the first name
    last_name,                              -- show the last name
    grade_level                             -- show the grade level
FROM students                               -- from the students table
WHERE grade_level IN (9, 10);               -- keep rows where grade_level is either 9 or 10
```

This is equivalent to writing `grade_level = 9 OR grade_level = 10`, but it scales better when you have more values.

---

## 8. Text filters with LIKE

The `LIKE` operator performs simple pattern matching. The `%` symbol means "any sequence of characters".

### Example: Find classes whose names start with "Algebra"

```sql
SELECT
    class_id,                               -- show the class id
    class_name,                             -- show the name of the class
    teacher_name                            -- show the teacher name
FROM classes                                -- from the classes table
WHERE class_name LIKE 'Algebra%';           -- keep rows where class_name starts with 'Algebra'
```

Here `%` matches any characters after the word `Algebra`. For example, `Algebra I` and `Algebra II` would both match.

---

## 9. Checking for NULL

`NULL` represents a missing or unknown value. Note that you must use `IS NULL` or `IS NOT NULL`, not `= NULL`.

### Example: Find enrollments where the grade is missing

```sql
SELECT
    enrollment_id,                         -- show the enrollment id
    student_id,                            -- show the student id
    class_id,                              -- show the class id
    numeric_grade                          -- show the grade (which might be NULL)
FROM enrollments                           -- from the enrollments table
WHERE numeric_grade IS NULL;               -- keep rows where numeric_grade is NULL (missing)
```

---

At this point you should be comfortable reading single-table queries. The next file, `sql_intermediate.md`, builds on these ideas to show how to combine tables with joins and how to aggregate data using GROUP BY.
