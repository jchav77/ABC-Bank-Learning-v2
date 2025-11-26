# Logistic Regression – Predicting Recovery Outcomes

Logistic regression is a classification model that predicts probabilities for yes/no outcomes. Despite its name containing "regression," it is used for classification, not numeric prediction.

In this document, we use logistic regression to predict whether ABC Bank will **recover** a loss from an auto loan default.

---

## 1. What is logistic regression?

Logistic regression models the probability that an observation belongs to a particular class. It uses a logistic function, also called a sigmoid function, to transform a linear combination of features into a probability between 0 and 1.

The formula is:

```
P(y=1) = 1 / (1 + e^(-z))
```

Where:

- `z = w0 + w1*x1 + w2*x2 + ... + wn*xn` is the linear combination of features
- `P(y=1)` is the probability that the outcome is class 1, meaning "yes" or "recovered"
- `e` is the mathematical constant approximately equal to 2.718

If the predicted probability is above a threshold, typically 0.5, we predict class 1. Otherwise, we predict class 0.

---

## 2. Preparing the data

We assume you have a DataFrame called `df_losses` with a binary target column `recovered` where 1 means the loss was recovered and 0 means it was not.

```python
import pandas as pd                                          # import pandas for data manipulation
from sklearn.model_selection import train_test_split         # import function to split data
from sklearn.linear_model import LogisticRegression          # import the logistic regression model class
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix  # import evaluation metrics

# Assume df_losses is already loaded
# It contains columns: loan_amount, vehicle_age_years, borrower_income, credit_score, days_delinquent, vehicle_mileage, recovered

# Define the feature columns
feature_columns = [
    "loan_amount",                                           # original loan amount
    "vehicle_age_years",                                     # age of the vehicle
    "borrower_income",                                       # borrower's annual income
    "credit_score",                                          # borrower's credit score
    "days_delinquent",                                       # days since last payment
    "vehicle_mileage"                                        # mileage on the vehicle
]

X = df_losses[feature_columns]                               # create a DataFrame with feature columns
y = df_losses["recovered"]                                   # create a Series with the binary target column
```

---

## 3. Splitting into training and test sets

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
model = LogisticRegression(
    max_iter=1000                                            # increase max iterations to ensure convergence
)                                                            # create an instance of the LogisticRegression model

model.fit(X_train, y_train)                                  # train the model using the training data

print("Model trained successfully.")                         # confirm training is complete
```

Note: scikit-learn's `LogisticRegression` uses the `lbfgs` solver by default, which is a robust optimization algorithm suitable for most problems.

---

## 5. Making predictions

Logistic regression can output either class predictions or probability estimates.

```python
# Predict class labels (0 or 1)
y_pred = model.predict(X_test)                               # predict class labels for the test set

# Predict probabilities
y_prob = model.predict_proba(X_test)                         # predict probabilities for each class

print("First 5 predictions (class labels):")
print(y_pred[:5])                                            # print the first 5 predicted class labels

print("\nFirst 5 predictions (probabilities):")
print(y_prob[:5])                                            # each row shows [P(class=0), P(class=1)]
```

The `predict_proba` method returns an array where each row contains the probability of each class. The two probabilities sum to 1.

---

## 6. Evaluating the model

### Accuracy

Accuracy is the proportion of correct predictions.

```python
accuracy = accuracy_score(y_test, y_pred)                    # compute accuracy
print(f"Accuracy: {accuracy:.4f}")                           # print accuracy with 4 decimal places
```

### Confusion matrix

A confusion matrix shows the counts of true positives, true negatives, false positives, and false negatives.

```python
cm = confusion_matrix(y_test, y_pred)                        # compute the confusion matrix
print("Confusion Matrix:")
print(cm)                                                    # print the confusion matrix
```

The layout is:

```
                  Predicted 0    Predicted 1
Actual 0          True Neg       False Pos
Actual 1          False Neg      True Pos
```

### Classification report

A classification report shows precision, recall, and F1-score for each class.

```python
report = classification_report(y_test, y_pred)               # generate the classification report
print("Classification Report:")
print(report)                                                # print the report
```

Key terms:

- **Precision**: Of all predictions for a class, what proportion were correct?
- **Recall**: Of all actual instances of a class, what proportion were correctly predicted?
- **F1-score**: The harmonic mean of precision and recall.

---

## 7. Interpreting the coefficients

Like linear regression, logistic regression has coefficients that indicate feature importance.

```python
coefficients = pd.DataFrame({
    "feature": feature_columns,                              # list of feature names
    "coefficient": model.coef_[0]                            # coefficients from the trained model (first row for binary classification)
})

print("Coefficients:")
print(coefficients)
print(f"\nIntercept: {model.intercept_[0]:.4f}")             # print the intercept value
```

In logistic regression, coefficients represent the change in the log-odds of the positive class for a one-unit increase in the feature.

- Positive coefficient: Increasing the feature increases the probability of recovery
- Negative coefficient: Increasing the feature decreases the probability of recovery

---

## 8. Example output interpretation

Suppose the output shows:

| feature            | coefficient |
| ------------------ | ----------: |
| loan_amount        |       -0.0001 |
| vehicle_age_years  |       -0.15 |
| borrower_income    |        0.00002 |
| credit_score       |        0.02 |
| days_delinquent    |       -0.01 |
| vehicle_mileage    |       -0.00001 |

Interpretation:

- Higher loan amounts slightly decrease the probability of recovery
- Older vehicles decrease the probability of recovery
- Higher borrower income slightly increases the probability of recovery
- Higher credit scores increase the probability of recovery
- More days delinquent decrease the probability of recovery
- Higher mileage slightly decreases the probability of recovery

---

## 9. Adjusting the decision threshold

By default, logistic regression predicts class 1 when the probability exceeds 0.5. You can adjust this threshold based on business needs.

```python
threshold = 0.4                                              # set a custom threshold

y_pred_custom = (y_prob[:, 1] >= threshold).astype(int)      # predict class 1 if P(class=1) >= threshold

print(f"Predictions with threshold {threshold}:")
print(y_pred_custom[:10])                                    # print the first 10 predictions with the custom threshold
```

Lowering the threshold means more cases are predicted as "recovered" (class 1), which increases recall but may decrease precision.

---

## 10. When to use logistic regression

Logistic regression works well when:

- You have a binary classification problem
- You need interpretable coefficients
- The relationship between features and log-odds is approximately linear
- You want probability estimates, not just class labels

It may not work well when:

- The decision boundary is highly nonlinear
- There are complex interactions between features
- You need the highest possible predictive accuracy

---

## 11. Summary

In this document, you learned how to:

1. Prepare features and a binary target for classification
2. Train a logistic regression model with scikit-learn
3. Make predictions (class labels and probabilities)
4. Evaluate the model using accuracy, confusion matrix, and classification report
5. Interpret the coefficients
6. Adjust the decision threshold

The next document, `ml_decision_trees.md`, introduces decision trees, which can capture nonlinear relationships and are highly interpretable.
