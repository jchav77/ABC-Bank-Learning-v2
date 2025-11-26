# pandas Basics – Working with Query Results

Once you load data from SQL into pandas, you will spend most of your time:

- Inspecting it.
- Filtering rows and columns.
- Creating new columns.
- Grouping and aggregating.
- Joining DataFrames.

This document gives you a practical introduction to these tasks.

---

## 1. Loading a joined dataset from SQL

We will use a joined `df_enrollments` DataFrame that includes student and class information.

```python
import pandas as pd                                          # import pandas

query = """
SELECT
    e.enrollment_id,                                         -- enrollment id
    e.student_id,                                            -- student id
    e.numeric_grade,                                         -- numeric grade
    s.first_name,                                            -- student first name
    s.last_name,                                             -- student last name
    s.grade_level,                                           -- student grade level
    c.class_name                                             -- name of the class
FROM enrollments e                                           -- start from enrollments
JOIN students s ON e.student_id = s.student_id               -- join students to add student details
JOIN classes c  ON e.class_id = c.class_id;                  -- join classes to add class details
"""                                                          # end of SQL string

df_enrollments = pd.read_sql(query, con=engine)              # execute the query and load the result into df_enrollments

print(df_enrollments.head())                                 # inspect the first few rows to understand the structure
```

---

## 2. Filtering rows

### Example: Only enrollments with numeric_grade at least 90

```python
df_high = df_enrollments[df_enrollments["numeric_grade"] >= 90]  # keep only rows where numeric_grade is 90 or higher

print(df_high[["first_name", "last_name", "class_name", "numeric_grade"]])  # show selected columns for high-grade enrollments
```

This code uses a Boolean condition, meaning a condition that is either True or False, to keep only rows with high grades.

---

## 3. Creating a new column – letter grades

We will convert numeric grades to letter grades using a helper function and the `apply` method.

```python
def to_letter(grade):                                   # define a function that takes a numeric grade
    if grade >= 90:                                     # if the grade is at least 90
        return "A"                                      # return letter A
    elif grade >= 80:                                   # else if the grade is at least 80
        return "B"                                      # return letter B
    elif grade >= 70:                                   # else if the grade is at least 70
        return "C"                                      # return letter C
    elif grade >= 60:                                   # else if the grade is at least 60
        return "D"                                      # return letter D
    else:                                               # for any grade below 60
        return "F"                                      # return letter F

df_enrollments["letter_grade"] = df_enrollments["numeric_grade"].apply(to_letter)  # create a new column by applying the function row by row

print(df_enrollments[["numeric_grade", "letter_grade"]].head())                   # show numeric and letter grades side by side
```

The `apply` method here takes each numeric grade and returns the corresponding letter grade.

---

## 4. Sorting

### Example: Sort enrollments by numeric_grade descending

```python
df_sorted = df_enrollments.sort_values(                   # call sort_values to reorder rows
    by="numeric_grade",                                  # sort by the numeric_grade column
    ascending=False                                      # set ascending to False to sort from highest to lowest
)

print(df_sorted[["first_name", "last_name", "class_name", "numeric_grade"]].head())  # show the top few rows of the sorted result
```

---

## 5. Grouping and aggregating

### Example: Average grade per class

```python
avg_by_class = (
    df_enrollments
    .groupby("class_name")["numeric_grade"]              # group rows by class_name and focus on numeric_grade
    .mean()                                              # compute the mean, meaning average, grade per class
    .reset_index(name="avg_grade")                       # convert the result to a DataFrame with a column called avg_grade
)

print(avg_by_class)                                      # print the average grade for each class
```

### Example: Average grade per student

```python
avg_by_student = (
    df_enrollments
    .groupby(["student_id", "first_name", "last_name"])  # group by student_id, first_name, and last_name
    ["numeric_grade"]                                    # focus on the numeric_grade column
    .mean()                                              # compute mean grade per student
    .reset_index(name="avg_grade")                       # move group keys back into columns and name the new average column
)

print(avg_by_student.head())                             # print the first few rows of student averages
```

---

## 6. Joining DataFrames

Sometimes you will have separate DataFrames and want to join them in memory.

Suppose `avg_by_student` is as above, and you also have `df_students` with extra student information.

```python
df_students = pd.read_sql("SELECT * FROM students;", con=engine)  # load all columns from students into a DataFrame

df_joined = df_students.merge(                                    # use merge to join df_students and avg_by_student
    avg_by_student,                                               # second DataFrame to join
    on="student_id",                                              # column to join on in both DataFrames
    how="left"                                                    # left join, meaning keep all students and add avg_grade where available
)

print(df_joined.head())                                           # inspect the joined result, which includes avg_grade if the student has enrollments
```

Here, `merge` behaves similarly to a SQL join, with `how="left"` corresponding to a left join that keeps all students.

---

This covers the core pandas operations you will need for most analysis tasks. Next, we will turn these aggregated results into visualizations with matplotlib and Plotly.
