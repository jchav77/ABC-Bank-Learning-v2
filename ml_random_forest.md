# Random Forests – Ensemble Classification

Random forests are ensembles, meaning groups, of decision trees whose predictions are combined to improve accuracy and robustness. Instead of relying on a single tree, a random forest builds many trees and aggregates their predictions.

In this document, we use random forests to classify whether ABC Bank will **recover** a loss from an auto loan default.

---

## 1. What is a random forest?

A random forest works by:

1. Creating many decision trees, each trained on a different random sample of the data
2. At each split in each tree, considering only a random subset of features
3. Combining the predictions of all trees (for classification, this is typically majority voting)

This process reduces overfitting and improves generalization compared to a single decision tree.

---

## 2. Why "random"?

The randomness comes from two sources:

- **Bootstrap sampling**: Each tree is trained on a random sample of the training data, drawn with replacement. This technique is called bagging (bootstrap aggregating).
- **Feature randomness**: At each split, only a random subset of features is considered. This decorrelates the trees and makes the ensemble more diverse.

---

## 3. Preparing the data

We use the same setup as before.

```python
import pandas as pd                                          # import pandas for data manipulation
from sklearn.model_selection import train_test_split         # import function to split data
from sklearn.ensemble import RandomForestClassifier          # import the random forest classifier
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix  # import evaluation metrics

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
y = df_losses["recovered"]                                   # create a Series with the binary target column
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
model = RandomForestClassifier(
    n_estimators=100,                                        # number of trees in the forest
    max_depth=10,                                            # maximum depth of each tree
    random_state=42                                          # set a seed for reproducibility
)

model.fit(X_train, y_train)                                  # train the model using the training data

print("Model trained successfully.")                         # confirm training is complete
print(f"Number of trees: {len(model.estimators_)}")          # print the number of trees in the forest
```

The `n_estimators` parameter controls how many trees are in the forest. More trees generally improve performance but increase computation time.

---

## 6. Making predictions

```python
y_pred = model.predict(X_test)                               # predict class labels for the test set

# Random forests can also output probability estimates
y_prob = model.predict_proba(X_test)                         # predict probabilities for each class

print("First 10 predictions:")
print(y_pred[:10])                                           # print the first 10 predicted class labels

print("\nFirst 5 probability estimates:")
print(y_prob[:5])                                            # each row shows [P(class=0), P(class=1)]
```

The probability estimates are the proportion of trees that voted for each class.

---

## 7. Evaluating the model

```python
accuracy = accuracy_score(y_test, y_pred)                    # compute accuracy
print(f"Accuracy: {accuracy:.4f}")                           # print accuracy

cm = confusion_matrix(y_test, y_pred)                        # compute the confusion matrix
print("\nConfusion Matrix:")
print(cm)

report = classification_report(y_test, y_pred)               # generate the classification report
print("\nClassification Report:")
print(report)
```

---

## 8. Feature importance

Random forests provide feature importance scores averaged across all trees.

```python
importance = pd.DataFrame({
    "feature": feature_columns,                              # list of feature names
    "importance": model.feature_importances_                 # importance scores from the trained model
}).sort_values("importance", ascending=False)                # sort by importance descending

print("Feature Importance:")
print(importance)
```

Feature importance in random forests is more stable than in a single decision tree because it is averaged over many trees.

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
plt.title("Random Forest Feature Importance")                # title of the plot
plt.gca().invert_yaxis()                                     # put most important feature at top
plt.tight_layout()                                           # adjust layout
plt.show()                                                   # display the chart
```

---

## 10. Key hyperparameters

### n_estimators

The number of trees in the forest. More trees generally improve performance but take longer to train.

```python
model = RandomForestClassifier(n_estimators=200)             # use 200 trees
```

### max_depth

The maximum depth of each tree. Limiting depth prevents overfitting.

```python
model = RandomForestClassifier(max_depth=5)                  # shallow trees with max depth 5
```

### min_samples_split

The minimum number of samples required to split a node.

```python
model = RandomForestClassifier(min_samples_split=20)         # require at least 20 samples to split
```

### max_features

The number of features to consider at each split. Common values:

- `"sqrt"`: Square root of the number of features (default for classification)
- `"log2"`: Log base 2 of the number of features
- An integer: Exact number of features

```python
model = RandomForestClassifier(max_features="sqrt")          # consider sqrt(n_features) at each split
```

---

## 11. Comparing single tree vs random forest

```python
from sklearn.tree import DecisionTreeClassifier              # import decision tree for comparison

# Train a single decision tree
single_tree = DecisionTreeClassifier(max_depth=10, random_state=42)
single_tree.fit(X_train, y_train)
tree_pred = single_tree.predict(X_test)
tree_acc = accuracy_score(y_test, tree_pred)

# Train a random forest
forest = RandomForestClassifier(n_estimators=100, max_depth=10, random_state=42)
forest.fit(X_train, y_train)
forest_pred = forest.predict(X_test)
forest_acc = accuracy_score(y_test, forest_pred)

print(f"Single Decision Tree Accuracy: {tree_acc:.4f}")      # print single tree accuracy
print(f"Random Forest Accuracy: {forest_acc:.4f}")           # print random forest accuracy
```

The random forest typically achieves higher accuracy because it reduces variance by averaging many trees.

---

## 12. Out-of-bag (OOB) score

Because each tree is trained on a bootstrap sample, some data points are left out. These are called out-of-bag samples. The OOB score is an estimate of model performance on unseen data.

```python
model = RandomForestClassifier(
    n_estimators=100,
    max_depth=10,
    oob_score=True,                                          # enable OOB score calculation
    random_state=42
)

model.fit(X_train, y_train)

print(f"OOB Score: {model.oob_score_:.4f}")                  # print the out-of-bag score
```

The OOB score is useful because it provides a validation estimate without needing a separate validation set.

---

## 13. When to use random forests

Random forests work well when:

- You want better accuracy than a single decision tree
- You need robust feature importance estimates
- You have a moderate number of features
- You want a model that is less prone to overfitting

They may not be ideal when:

- You need the most interpretable model (a single tree is easier to explain)
- Training time is critical and you have limited resources
- You need the absolute best performance (gradient boosting often wins)

---

## 14. Summary

In this document, you learned how to:

1. Understand how random forests combine many decision trees
2. Train a random forest classifier with scikit-learn
3. Evaluate the model using standard classification metrics
4. Interpret and visualize feature importance
5. Tune key hyperparameters like `n_estimators` and `max_depth`
6. Use the out-of-bag score for validation

The next document, `ml_gradient_boosting.md`, introduces gradient boosting, another powerful ensemble method that builds trees sequentially to correct errors.
