# Gradient Boosting – Sequential Ensemble Learning

Gradient boosting is an ensemble method that builds many small trees in sequence, with each tree correcting the errors of the previous ones. This sequential approach often achieves excellent predictive performance.

In this document, we use gradient boosting to predict the **loss amount** for auto loans at ABC Bank, which is a regression problem.

---

## 1. What is gradient boosting?

Gradient boosting works by:

1. Starting with a simple prediction, such as the average of the target values
2. Computing the errors, called residuals, between predictions and actual values
3. Training a small decision tree to predict these residuals
4. Adding the tree's predictions to the current predictions, scaled by a learning rate
5. Repeating steps 2-4 for many iterations

Each new tree focuses on the mistakes of the previous ensemble, gradually improving the predictions.

---

## 2. Key concepts

### Residuals

Residuals are the differences between actual values and predicted values. Each new tree learns to predict these residuals.

### Learning rate

The learning rate, also called shrinkage, controls how much each tree contributes to the final prediction. Smaller learning rates require more trees but often produce better results.

### Boosting vs bagging

- **Bagging** (used in random forests): Trees are trained independently on random samples
- **Boosting** (used in gradient boosting): Trees are trained sequentially, each one correcting errors from the previous ensemble

---

## 3. Preparing the data

We use the regression setup from the linear regression document.

```python
import pandas as pd                                          # import pandas for data manipulation
from sklearn.model_selection import train_test_split         # import function to split data
from sklearn.ensemble import GradientBoostingRegressor       # import the gradient boosting regressor
from sklearn.metrics import mean_squared_error, r2_score     # import regression metrics

# Assume df_losses is already loaded

feature_columns = [
    "loan_amount",                                           # original loan amount
    "vehicle_age_years",                                     # age of the vehicle
    "borrower_income",                                       # borrower's annual income
    "credit_score",                                          # borrower's credit score
    "days_delinquent",                                       # days since last payment
    "vehicle_mileage"                                        # mileage on the vehicle
]

X = df_losses[feature_columns]                               # create a DataFrame with feature columns
y = df_losses["loss_amount"]                                 # create a Series with the numeric target column
```

---

## 4. Splitting into training and test sets

```python
X_train, X_test, y_train, y_test = train_test_split(
    X,                                                       # feature data
    y,                                                       # target data
    test_size=0.2,                                           # use 20% of the data for testing
    random_state=42                                          # set a seed for reproducibility
)

print(f"Training samples: {len(X_train)}")                   # print the number of training samples
print(f"Test samples: {len(X_test)}")                        # print the number of test samples
```

---

## 5. Training the model

```python
model = GradientBoostingRegressor(
    n_estimators=100,                                        # number of boosting stages (trees)
    learning_rate=0.1,                                       # shrinkage rate for each tree's contribution
    max_depth=3,                                             # maximum depth of each tree (shallow trees work well)
    random_state=42                                          # set a seed for reproducibility
)

model.fit(X_train, y_train)                                  # train the model using the training data

print("Model trained successfully.")                         # confirm training is complete
print(f"Number of estimators: {model.n_estimators_}")        # print the number of trees used
```

---

## 6. Making predictions

```python
y_pred = model.predict(X_test)                               # predict loss amounts for the test set

print("First 10 predictions:")
print(y_pred[:10])                                           # print the first 10 predicted values
```

---

## 7. Evaluating the model

```python
mse = mean_squared_error(y_test, y_pred)                     # compute mean squared error
rmse = mse ** 0.5                                            # compute root mean squared error
r2 = r2_score(y_test, y_pred)                                # compute R² score

print(f"Mean Squared Error: {mse:.2f}")                      # print MSE
print(f"Root Mean Squared Error: {rmse:.2f}")                # print RMSE (same units as target)
print(f"R² Score: {r2:.4f}")                                 # print R² score
```

RMSE is often easier to interpret than MSE because it is in the same units as the target variable (dollars in this case).

---

## 8. Feature importance

```python
importance = pd.DataFrame({
    "feature": feature_columns,                              # list of feature names
    "importance": model.feature_importances_                 # importance scores from the trained model
}).sort_values("importance", ascending=False)                # sort by importance descending

print("Feature Importance:")
print(importance)
```

---

## 9. Visualizing feature importance

```python
import matplotlib.pyplot as plt                              # import matplotlib for plotting

plt.figure(figsize=(8, 5))                                   # create a figure
plt.barh(
    importance["feature"],                                   # feature names on y-axis
    importance["importance"]                                 # importance scores on x-axis
)
plt.xlabel("Importance")                                     # label for x-axis
plt.ylabel("Feature")                                        # label for y-axis
plt.title("Gradient Boosting Feature Importance")            # title of the plot
plt.gca().invert_yaxis()                                     # put most important feature at top
plt.tight_layout()                                           # adjust layout
plt.show()                                                   # display the chart
```

---

## 10. Key hyperparameters

### n_estimators

The number of boosting stages, meaning the number of trees. More trees can improve performance but increase training time and risk overfitting.

```python
model = GradientBoostingRegressor(n_estimators=200)          # use 200 trees
```

### learning_rate

Controls how much each tree contributes. Lower values require more trees but often produce better results.

```python
model = GradientBoostingRegressor(learning_rate=0.05)        # use a smaller learning rate
```

### max_depth

The maximum depth of each tree. Gradient boosting typically uses shallow trees (depth 3-5).

```python
model = GradientBoostingRegressor(max_depth=4)               # slightly deeper trees
```

### min_samples_split and min_samples_leaf

Control the minimum samples required to split or be a leaf node, similar to decision trees.

```python
model = GradientBoostingRegressor(min_samples_split=10, min_samples_leaf=5)
```

---

## 11. Monitoring training progress

You can track how performance improves as more trees are added.

```python
import numpy as np                                           # import numpy for array operations

# Train the model
model = GradientBoostingRegressor(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=3,
    random_state=42
)
model.fit(X_train, y_train)

# Compute test error at each stage
test_errors = []
for i, y_pred_stage in enumerate(model.staged_predict(X_test)):  # iterate through predictions at each stage
    mse = mean_squared_error(y_test, y_pred_stage)           # compute MSE at this stage
    test_errors.append(mse)

# Plot the learning curve
plt.figure(figsize=(8, 5))
plt.plot(range(1, len(test_errors) + 1), test_errors)        # plot MSE vs number of trees
plt.xlabel("Number of Trees")                                # label for x-axis
plt.ylabel("Mean Squared Error")                             # label for y-axis
plt.title("Gradient Boosting Learning Curve")                # title of the plot
plt.tight_layout()
plt.show()
```

This plot shows how the error decreases as more trees are added. If the error starts increasing, you may be overfitting.

---

## 12. Gradient boosting for classification

Gradient boosting can also be used for classification using `GradientBoostingClassifier`.

```python
from sklearn.ensemble import GradientBoostingClassifier      # import the classifier version

# Assume y is now a binary target (recovered: 0 or 1)
clf = GradientBoostingClassifier(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=3,
    random_state=42
)

clf.fit(X_train, y_train)                                    # train the classifier
y_pred = clf.predict(X_test)                                 # predict class labels
y_prob = clf.predict_proba(X_test)                           # predict probabilities
```

---

## 13. Comparing gradient boosting to other methods

```python
from sklearn.linear_model import LinearRegression            # import linear regression
from sklearn.ensemble import RandomForestRegressor           # import random forest

# Linear Regression
lr = LinearRegression()
lr.fit(X_train, y_train)
lr_pred = lr.predict(X_test)
lr_r2 = r2_score(y_test, lr_pred)

# Random Forest
rf = RandomForestRegressor(n_estimators=100, max_depth=10, random_state=42)
rf.fit(X_train, y_train)
rf_pred = rf.predict(X_test)
rf_r2 = r2_score(y_test, rf_pred)

# Gradient Boosting
gb = GradientBoostingRegressor(n_estimators=100, learning_rate=0.1, max_depth=3, random_state=42)
gb.fit(X_train, y_train)
gb_pred = gb.predict(X_test)
gb_r2 = r2_score(y_test, gb_pred)

print(f"Linear Regression R²: {lr_r2:.4f}")
print(f"Random Forest R²: {rf_r2:.4f}")
print(f"Gradient Boosting R²: {gb_r2:.4f}")
```

Gradient boosting often achieves the best performance, especially on tabular data.

---

## 14. When to use gradient boosting

Gradient boosting works well when:

- You want the best possible predictive performance on tabular data
- You have enough data to avoid overfitting
- You can tune hyperparameters like learning rate and number of estimators

It may not be ideal when:

- Training time is critical (gradient boosting is slower than random forests)
- You need a highly interpretable model
- You have very little data (simpler models may work better)

---

## 15. Alternative implementations

Scikit-learn's gradient boosting is a good starting point, but specialized libraries often offer better performance:

- **XGBoost**: Highly optimized implementation with additional features
- **LightGBM**: Fast, memory-efficient implementation from Microsoft
- **CatBoost**: Handles categorical features natively, from Yandex

These libraries follow similar patterns but often train faster and achieve slightly better results.

---

## 16. Summary

In this document, you learned how to:

1. Understand how gradient boosting builds trees sequentially to correct errors
2. Train a gradient boosting regressor with scikit-learn
3. Evaluate the model using MSE, RMSE, and R²
4. Interpret and visualize feature importance
5. Tune key hyperparameters like `n_estimators`, `learning_rate`, and `max_depth`
6. Monitor training progress with staged predictions
7. Compare gradient boosting to other methods

---

## Congratulations!

You have completed the ABC Bank Learning Repo. You now have a foundation in:

- SQL from basics to advanced (CTEs, window functions)
- Python and pandas for data manipulation
- Visualization with matplotlib and Plotly
- Machine learning with linear regression, logistic regression, decision trees, random forests, and gradient boosting

Continue practicing by applying these techniques to real data and experimenting with different approaches. The best way to learn is by doing!
