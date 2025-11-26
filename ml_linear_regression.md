# Linear Regression – Predicting Loss Amounts

Linear regression is a model that predicts a numeric outcome using a weighted sum of input features. It is one of the simplest and most interpretable machine learning algorithms.

In this document, we use linear regression to predict the **loss amount** for auto loans at ABC Bank.

---

## 1. What is linear regression?

Linear regression assumes that the target variable, which we call `y`, can be expressed as a linear combination of the features, which we call `X`.

The formula looks like this:

```
y = w0 + w1*x1 + w2*x2 + ... + wn*xn
```

Where:

- `y` is the predicted target value
- `x1, x2, ..., xn` are the feature values
- `w0` is the intercept, also called the bias
- `w1, w2, ..., wn` are the weights, also called coefficients

The model learns the weights from the training data by minimizing the difference between predicted values and actual values.

---

## 2. Preparing the data

We assume you have a DataFrame called `df_losses` with historical auto loan loss data.

```python
import pandas as pd                                          # import pandas for data manipulation
from sklearn.model_selection import train_test_split         # import function to split data into train and test sets
from sklearn.linear_model import LinearRegression            # import the linear regression model class
from sklearn.metrics import mean_squared_error, r2_score     # import metrics to evaluate the model

# Assume df_losses is already loaded from SQL or a CSV file
# It contains columns: loan_amount, vehicle_age_years, borrower_income, credit_score, days_delinquent, vehicle_mileage, loss_amount

# Define the feature columns we will use to make predictions
feature_columns = [
    "loan_amount",                                           # original loan amount
    "vehicle_age_years",                                     # age of the vehicle
    "borrower_income",                                       # borrower's annual income
    "credit_score",                                          # borrower's credit score
    "days_delinquent",                                       # days since last payment
    "vehicle_mileage"                                        # mileage on the vehicle
]

X = df_losses[feature_columns]                               # create a DataFrame with only the feature columns
y = df_losses["loss_amount"]                                 # create a Series with the target column
```

---

## 3. Splitting into training and test sets

We split the data so we can train the model on one portion and evaluate it on another.

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

## 4. Training the model

```python
model = LinearRegression()                                   # create an instance of the LinearRegression model

model.fit(X_train, y_train)                                  # train the model using the training data

print("Model trained successfully.")                         # confirm training is complete
```

The `fit` method finds the weights that minimize the squared differences between predicted and actual loss amounts.

---

## 5. Making predictions

```python
y_pred = model.predict(X_test)                               # use the trained model to predict loss amounts for the test set

print("First 5 predictions:")                                # print a label for clarity
print(y_pred[:5])                                            # print the first 5 predicted values
```

---

## 6. Evaluating the model

We use two common metrics:

- **Mean Squared Error (MSE)**: The average of the squared differences between predicted and actual values. Lower is better.
- **R² score**: The proportion of variance in the target that is explained by the model. Ranges from 0 to 1, where 1 is perfect.

```python
mse = mean_squared_error(y_test, y_pred)                     # compute the mean squared error
r2 = r2_score(y_test, y_pred)                                # compute the R² score

print(f"Mean Squared Error: {mse:.2f}")                      # print MSE with 2 decimal places
print(f"R² Score: {r2:.4f}")                                 # print R² score with 4 decimal places
```

An R² score of 0.75, for example, means the model explains 75% of the variance in loss amounts.

---

## 7. Interpreting the coefficients

Linear regression coefficients tell you how much the predicted target changes for a one-unit increase in each feature.

```python
coefficients = pd.DataFrame({
    "feature": feature_columns,                              # list of feature names
    "coefficient": model.coef_                               # corresponding coefficients from the trained model
})

print("Coefficients:")
print(coefficients)
print(f"\nIntercept: {model.intercept_:.2f}")                # print the intercept value
```

For example, if the coefficient for `vehicle_age_years` is 500, it means that for each additional year of vehicle age, the predicted loss increases by $500, holding other features constant.

---

## 8. Example output interpretation

Suppose the output shows:

| feature            | coefficient |
| ------------------ | ----------: |
| loan_amount        |        0.15 |
| vehicle_age_years  |      450.00 |
| borrower_income    |       -0.02 |
| credit_score       |      -50.00 |
| days_delinquent    |       25.00 |
| vehicle_mileage    |        0.01 |

Interpretation:

- Each additional dollar in loan amount increases predicted loss by $0.15
- Each additional year of vehicle age increases predicted loss by $450
- Each additional dollar of borrower income decreases predicted loss by $0.02
- Each additional credit score point decreases predicted loss by $50
- Each additional day delinquent increases predicted loss by $25
- Each additional mile on the vehicle increases predicted loss by $0.01

---

## 9. When to use linear regression

Linear regression works well when:

- The relationship between features and target is approximately linear
- You need an interpretable model where stakeholders can understand the coefficients
- You have a regression, meaning numeric prediction, problem

It may not work well when:

- The relationship is highly nonlinear
- There are complex interactions between features
- You need higher predictive accuracy and are willing to sacrifice interpretability

---

## 10. Summary

In this document, you learned how to:

1. Prepare features and target for a regression problem
2. Split data into training and test sets
3. Train a linear regression model with scikit-learn
4. Make predictions on new data
5. Evaluate the model using MSE and R²
6. Interpret the coefficients to understand feature importance

The next document, `ml_logistic_regression.md`, shows how to use logistic regression for classification problems, such as predicting whether a loss will be recovered.
