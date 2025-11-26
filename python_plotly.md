# Interactive Visualization with Plotly Express

Plotly Express is a high-level interface for creating interactive charts. Interactive means that you can hover over points, zoom, and pan.

In this document, you will:

- Create an interactive bar chart for average grade per class.
- Create an interactive scatter plot of numeric grade versus grade level.

We assume you have `df_enrollments` and have installed `plotly`.

```python
import plotly.express as px                              # import plotly.express and alias it as px
```

---

## 1. Interactive bar chart – average grade per class

We reuse the `avg_by_class` DataFrame.

```python
avg_by_class = (                                         # build avg_by_class as before
    df_enrollments
    .groupby("class_name")["numeric_grade"]              # group by class_name and select numeric_grade
    .mean()                                              # compute the average numeric grade
    .reset_index(name="avg_grade")                       # convert to a DataFrame and name the average column
)

fig = px.bar(
    avg_by_class,                                        # the DataFrame to use
    x="class_name",                                      # column to use for the x-axis
    y="avg_grade",                                       # column to use for the y-axis
    title="Average Grade per Class (Interactive)"        # title of the chart
)                                                        # end of px.bar call

fig.show()                                               # display the interactive chart in a browser or notebook
```

When you hover over a bar, you will see the class name and the exact average grade. You can also zoom into specific areas of the chart.

---

## 2. Interactive scatter plot – numeric grade versus grade level

A scatter plot places individual data points on a two-dimensional plane.

```python
fig = px.scatter(
    df_enrollments,                                      # the DataFrame containing enrollment data
    x="grade_level",                                     # column for x-axis: student's grade level
    y="numeric_grade",                                   # column for y-axis: numeric grade in the class
    color="class_name",                                  # color each point by class name for easy comparison
    hover_data=["first_name", "last_name"],              # show first_name and last_name on hover for more detail
    title="Numeric Grade vs Grade Level"                 # title of the chart
)                                                        # end of px.scatter call

fig.show()                                               # display the interactive scatter plot
```

This chart helps you see patterns, such as whether older students, meaning students with higher grade_level values, tend to have higher or lower grades. Hovering over a point reveals the student's name.

---

## 3. Customizing Plotly charts

Plotly Express charts can be customized further using the `update_layout` and `update_traces` methods.

### Example: Change axis titles and theme

```python
fig = px.bar(
    avg_by_class,
    x="class_name",
    y="avg_grade",
    title="Average Grade per Class"
)

fig.update_layout(
    xaxis_title="Class Name",                            # custom x-axis title
    yaxis_title="Average Numeric Grade",                 # custom y-axis title
    template="plotly_white"                              # use a white background theme
)

fig.show()                                               # display the customized chart
```

---

## 4. Saving Plotly charts

You can save Plotly charts as HTML files or static images.

### Example: Save as HTML

```python
fig.write_html("average_grade_chart.html")               # save the figure as an HTML file you can open in a browser
```

### Example: Save as PNG (requires kaleido)

```python
fig.write_image("average_grade_chart.png")               # save the figure as a PNG image file
```

To use `write_image`, you need to install the `kaleido` package:

```bash
pip install kaleido
```

---

This covers the basics of Plotly Express. The combination of pandas for data manipulation and Plotly for interactive visualization is a powerful workflow for exploring and presenting data. The next section of the repo introduces machine learning concepts in a banking context.
