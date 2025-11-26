# Querying SQL from Python with pandas

This document shows how to run SQL queries from Python and load the results into pandas DataFrames.

We assume:

- You have already created and populated the tables in your database.
- You have a SQLAlchemy engine object called `engine`.
- You have imported pandas.

We focus on:

- Basic `pd.read_sql` usage.
- Parameterized queries, meaning queries where you safely pass variable values.
- Handling Python lists with SQL `IN` clauses.

---

## 1. Basic `pd.read_sql` – running a simple query

The function `pd.read_sql` runs a SQL query and returns the result as a pandas DataFrame, which is a two-dimensional labeled data structure similar to a table.

### Example: Load all students into a DataFrame

```python
import pandas as pd                            # import pandas library and give it the common alias pd

query = """
SELECT
    student_id,                                -- select the student_id column
    first_name,                                -- select the first_name column
    last_name,                                 -- select the last_name column
    grade_level                                -- select the grade_level column
FROM students;                                 -- from the students table
"""                                            # end of the multi-line SQL string

df_students = pd.read_sql(query, con=engine)   # run the SQL query using the engine and store the result in df_students

print(df_students.head())                      # print the first few rows of the DataFrame to inspect the result
```

This code:

1. Defines a SQL query as a multi-line string.
2. Uses `pd.read_sql` to execute the query against the database.
3. Prints the first few rows using `head`, which is a pandas method that returns the first rows of the DataFrame.

---

## 2. Parameterized queries – avoiding manual string building

Parameterized queries avoid manually concatenating Python values into SQL strings. This is safer and more robust.

We will use the `%()` syntax used by some drivers, such as psycopg2 for PostgreSQL. Different drivers use different placeholder styles, but the concept is the same.

### Example: Get students in a specific grade

```python
import pandas as pd                                    # import pandas so we can use DataFrames

grade_level = 9                                        # define a Python variable holding the desired grade level

query = """
SELECT
    student_id,                                        -- select student id
    first_name,                                        -- select first name
    last_name,                                         -- select last name
    grade_level                                        -- select grade level
FROM students                                          -- from the students table
WHERE grade_level = %(grade_level)s;                   -- filter rows using a named parameter called grade_level
"""                                                    # end of SQL string

params = {"grade_level": grade_level}                  # create a dictionary mapping parameter name to Python value

df_grade9 = pd.read_sql(query, con=engine, params=params)  # execute the query, passing params so the driver binds the value

print(df_grade9)                                       # print the resulting DataFrame to see the students in grade 9
```

The key idea is that we do not directly embed `9` into the SQL string. Instead, we define `%(grade_level)s` and pass the value through `params`. The driver then safely binds the value when sending the query to the database.

---

## 3. Using a Python list with an SQL IN clause

It is common to have a Python list of values and want to filter SQL rows where a column is in that list.

We must be careful not to simply join the values into a string, because that can be error-prone and unsafe. Instead, we build a list of placeholders and use parameters.

### Example: Filter students whose grade_level is in a Python list

```python
import pandas as pd                                        # import pandas

grade_levels = [9, 10, 12]                                 # define a Python list of grade levels we care about

# Build a placeholder string like "%(g0)s, %(g1)s, %(g2)s"
placeholders = []                                          # start with an empty list for placeholders
params = {}                                                # start with an empty dictionary for parameters

for i, g in enumerate(grade_levels):                       # loop over the grade levels with their index
    key = f"g{i}"                                          # create a unique parameter name like "g0", "g1", "g2"
    placeholders.append(f"%({key})s")                      # create a placeholder string for this parameter and add it to the list
    params[key] = g                                        # store the actual grade level value in the params dictionary under this key

placeholders_str = ", ".join(placeholders)                 # join all placeholder pieces into one comma-separated string

query = f"""
SELECT
    student_id,                                            -- select student id
    first_name,                                            -- select first name
    last_name,                                             -- select last name
    grade_level                                            -- select grade level
FROM students                                              -- from the students table
WHERE grade_level IN ({placeholders_str});                 -- use the placeholders string inside the IN clause
"""                                                        # end of SQL string

df_some_grades = pd.read_sql(query, con=engine, params=params)  # execute the query with the params dictionary

print(df_some_grades)                                     # print the resulting DataFrame with students in the specified grade levels
```

Here, we build as many named parameters as we have elements in the Python list. This approach scales to larger lists and keeps the query safe and clear.

---

## 4. Passing parameters for text and dates

The same pattern works for text and date values.

### Example: Parameterized filter by teacher name

```python
teacher = "Ms. Smith"                                     # define a Python string for the teacher name

query = """
SELECT
    class_id,                                             -- select class id
    class_name,                                           -- select class name
    teacher_name                                          -- select teacher name
FROM classes                                              -- from the classes table
WHERE teacher_name = %(teacher)s;                         -- filter where teacher_name equals the teacher parameter
"""

params = {"teacher": teacher}                             # dictionary mapping the parameter name to the Python string

df_classes = pd.read_sql(query, con=engine, params=params)  # execute the query with the parameter

print(df_classes)                                         # print the classes taught by this teacher
```

You can use the same idea for filtering by date columns, as long as your driver understands the Python date or datetime type you pass in.

---

This should give you a solid basis for using SQL from Python with pandas. The next file, `python_pandas_basics.md`, will focus on what to do with the DataFrames once you have them.
