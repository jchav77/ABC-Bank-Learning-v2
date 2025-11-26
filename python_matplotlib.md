# Visualizing Data with matplotlib

matplotlib is a plotting library for Python that creates static, non-interactive figures such as PNG images. It is widely used as a foundation for plotting in Python.

In this document we will:

- Create a bar chart for average grades per class.
- Create a histogram for grade distributions.

We assume `df_enrollments` and `avg_by_class` from previous sections.

```python
import matplotlib.pyplot as plt                            # import matplotlib.pyplot as plt, the common plotting interface
```

---

## 1. Bar chart – average grade per class

We will first compute the average grade per class and then plot it.

```python
avg_by_class = (                                          # start building avg_by_class DataFrame
    df_enrollments                                        # use df_enrollments as the source
    .groupby("class_name")["numeric_grade"]               # group by class_name and select numeric_grade
    .mean()                                               # compute the average grade per class
    .reset_index(name="avg_grade")                        # convert the result to a DataFrame with a column called avg_grade
)

plt.figure(figsize=(8, 4))                                # create a new figure with width 8 inches and height 4 inches
plt.bar(
    avg_by_class["class_name"],                           # x-coordinates: class names
    avg_by_class["avg_grade"]                             # y-coordinates: average grades
)                                                         # end of bar plotting call
plt.xlabel("Class")                                       # set the label for the x-axis
plt.ylabel("Average Grade")                               # set the label for the y-axis
plt.title("Average Grade per Class")                      # set the title of the plot
plt.xticks(rotation=45, ha="right")                       # rotate x-axis labels by 45 degrees and align them to the right
plt.tight_layout()                                        # adjust layout so everything fits nicely without overlapping
plt.show()                                                # display the plot window or inline image in notebooks
```

The resulting chart shows each class along the horizontal axis and its average numeric grade on the vertical axis.

---

## 2. Histogram – distribution of numeric grades

A histogram shows how frequently values fall into different ranges, called bins.

```python
plt.figure(figsize=(6, 4))                                # create a new figure with width 6 inches and height 4 inches
plt.hist(
    df_enrollments["numeric_grade"],                      # data to plot: numeric grades
    bins=10                                               # number of bins, meaning intervals, to divide the range into
)                                                         # end of hist plotting call
plt.xlabel("Numeric Grade")                               # label for x-axis describing what the values mean
plt.ylabel("Count of Enrollments")                        # label for y-axis describing how many enrollments fall into each bin
plt.title("Distribution of Numeric Grades")               # title explaining what the chart shows
plt.tight_layout()                                        # adjust spacing
plt.show()                                                # display the histogram
```

The histogram helps you see whether grades cluster in a certain range or are spread out evenly.

---

These examples illustrate the basic matplotlib pattern: prepare your DataFrame, call plotting functions, label axes, and display the figure. The next document, `python_plotly.md`, shows how to create interactive versions of similar charts.
