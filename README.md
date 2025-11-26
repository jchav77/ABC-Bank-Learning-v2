# ABC Bank Learning Repo
## SQL • Python • Visualization • Machine Learning

Welcome to the ABC Bank learning repository.

This repo is designed for you, a student who wants to go from beginner to advanced in:

- SQL for querying relational databases.
- Python for working with SQL data.
- Visualization in Python using pandas, matplotlib, and Plotly.
- Machine learning algorithms as they might be used in a banking context.

The examples are intentionally simple and relatable. We focus on:

- A school database with students, classes, and grades for SQL and Python querying.
- ABC Bank's simplified auto impounds and loss recovery for machine learning.

You will not need prior banking or automotive knowledge. When we introduce a term that might be unfamiliar, we explain it directly in the text.

---

## How the repo is organized

You can think of each Markdown file as a chapter.

### SQL

- `sql_basics.md`
  Introduces tables, rows, and columns, and covers basic SELECT statements, filtering with WHERE, sorting with ORDER BY, and simple numeric and text filters.

- `sql_intermediate.md`
  Explains joins (inner, left, right, full), aggregation with GROUP BY, filtering aggregated results with HAVING, and simple subqueries.

- `sql_advanced.md`
  Covers Common Table Expressions (CTEs), window functions such as RANK and ROW_NUMBER, more complex subqueries, and patterns like top N per group.

### Python + SQL

- `python_sql.md`
  Shows how to query SQL from Python using pandas and a SQLAlchemy engine. Explains parameterized queries and how to safely use Python lists with SQL IN clauses.

- `python_pandas_basics.md`
  Demonstrates how to explore, filter, transform, aggregate, and join data in pandas once you have loaded it from SQL.

### Visualization

- `python_matplotlib.md`
  Introduces static plots such as bar charts, line charts, and histograms using matplotlib.

- `python_plotly.md`
  Introduces interactive plots using Plotly Express, including interactive bar charts and scatter plots.

### Machine Learning in a Banking Context

These files all use a fictional institution, **ABC Bank**, and a simplified auto impounds / loss recovery scenario.

- `ml_overview_banking.md`
  Defines basic machine learning terms in plain language and explains how they apply in a banking context.

- `ml_linear_regression.md`
  Uses linear regression, which is a model that predicts a numeric outcome using a weighted sum of input features, to estimate loss amounts for auto loans.

- `ml_logistic_regression.md`
  Uses logistic regression, which is a classification model that predicts probabilities for yes/no outcomes, to predict whether a loss is likely to be recovered.

- `ml_decision_trees.md`
  Uses decision trees, which are models that split the data into branches based on feature values, to classify whether a loss will be recovered.

- `ml_random_forest.md`
  Uses random forests, which are ensembles, meaning groups, of decision trees whose predictions are combined to improve accuracy and robustness, to classify recovery outcomes.

- `ml_gradient_boosting.md`
  Uses gradient boosting, which is an ensemble method that builds many small trees in sequence, each one correcting the errors of the previous ones, to predict numeric loss amounts.

In all machine learning files, we use modern, common practice with scikit-learn, a popular Python machine learning library. For example, scikit-learn's `LogisticRegression` uses the `lbfgs` solver, which is a robust optimization algorithm, as the default solver in current versions.

---

## Assumptions and environment

This learning path assumes that you:

- Have access to a SQL database where you can create tables and run queries.
- Have Python installed with the following libraries:
  - `pandas`
  - `matplotlib`
  - `plotly`
  - `scikit-learn`
- Have a SQLAlchemy engine object available in Python as `engine`. We deliberately do not cover how to connect to the database so you can plug this into your own environment.

Whenever you see a line like:

```python
df = pd.read_sql(query, con=engine)  # Run SQL and get a DataFrame
```

you can assume that `engine` is already correctly set up.

---

## How to study with this repo

A good order to read and practice is:

1. `sql_basics.md`
2. `sql_intermediate.md`
3. `sql_advanced.md`
4. `python_sql.md`
5. `python_pandas_basics.md`
6. `python_matplotlib.md`
7. `python_plotly.md`
8. `ml_overview_banking.md`
9. `ml_linear_regression.md`
10. `ml_logistic_regression.md`
11. `ml_decision_trees.md`
12. `ml_random_forest.md`
13. `ml_gradient_boosting.md`

For each file:

* Read the explanation.
* Type the code yourself rather than copying and pasting.
* Run the code and inspect the results.
* Try changing some parameters or values and see what happens.

The goal is not just to "know the syntax" but to reach the point where you can confidently design and debug your own analyses and models.
