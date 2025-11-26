# Decision Trees – Classification with Interpretable Rules

Decision trees are models that split the data into branches based on feature values, creating a tree of decisions that leads to predictions. They are highly interpretable because you can visualize the decision rules.

In this document, we use decision trees to classify whether ABC Bank will **recover** a loss from an auto loan default.

---

## 1. What is a decision tree?

A decision tree makes predictions by asking a series of yes/no questions about the features. Each question splits the data into two groups. The process continues until the tree reaches a leaf node, which contains the final prediction.

For example:

```
Is credit_score < 650?
├── Yes: Is days_delinquent > 60?
│   ├── Yes: Predict NOT RECOVERED
│   └── No: Predict RECOVERED
└── No: Predict RECOVERED
```

The tree learns these splits from the training data by choosing the splits that best separate the classes.

---

## 2. Preparing the data

We use the same setup as in logistic regression.

```python
import pandas as pd                                          # import pandas for data manipulation
from sklearn.model_selection import train_test_split         # import function to split data
from sklearn.tree import DecisionTreeClassifier              # import the decision tree classifier
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix  # import evaluation metrics

# Assume df_losses is already loaded with columns including 'recovered'

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
model = DecisionTreeClassifier(
    max_depth=5,                                             # limit tree depth to prevent overfitting
    random_state=42                                          # set a seed for reproducibility
)

model.fit(X_train, y_train)                                  # train the model using the training data

print("Model trained successfully.")                         # confirm training is complete
```

The `max_depth` parameter controls how deep the tree can grow. Deeper trees can capture more complex patterns but may overfit the training data.

---

## 5. Making predictions

```python
y_pred = model.predict(X_test)                               # predict class labels for the test set

print("First 10 predictions:")
print(y_pred[:10])                                           # print the first 10 predicted class labels
```

---

## 6. Evaluating the model

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

## 7. Visualizing the tree

One of the biggest advantages of decision trees is that you can visualize them.

```python
from sklearn.tree import plot_tree                           # import the function to plot trees
import matplotlib.pyplot as plt                              # import matplotlib for plotting

plt.figure(figsize=(20, 10))                                 # create a large figure to fit the tree
plot_tree(
    model,                                                   # the trained decision tree model
    feature_names=feature_columns,                           # names of the features for labeling
    class_names=["Not Recovered", "Recovered"],              # names of the classes for labeling
    filled=True,                                             # color nodes by class
    rounded=True,                                            # use rounded boxes
    fontsize=10                                              # font size for labels
)
plt.title("Decision Tree for Loss Recovery Prediction")      # add a title
plt.tight_layout()                                           # adjust layout
plt.show()                                                   # display the tree
```

The visualization shows:

- Each node's decision rule (e.g., "credit_score <= 650")
- The number of samples at each node
- The class distribution at each node
- The predicted class at each leaf

---

## 8. Feature importance

Decision trees can tell you which features were most important for making predictions.

```python
importance = pd.DataFrame({
    "feature": feature_columns,                              # list of feature names
    "importance": model.feature_importances_                 # importance scores from the trained model
}).sort_values("importance", ascending=False)                # sort by importance descending

print("Feature Importance:")
print(importance)
```

Feature importance is based on how much each feature contributes to reducing impurity (uncertainty) in the tree. Higher values mean the feature is more important.

---

## 9. Controlling tree complexity

Decision trees can easily overfit if they grow too deep. Here are key parameters to control complexity:

### max_depth

Limits how deep the tree can grow.

```python
model = DecisionTreeClassifier(max_depth=3)                  # shallow tree with maximum depth of 3
```

### min_samples_split

The minimum number of samples required to split a node.

```python
model = DecisionTreeClassifier(min_samples_split=20)         # require at least 20 samples to split
```

### min_samples_leaf

The minimum number of samples required to be at a leaf node.

```python
model = DecisionTreeClassifier(min_samples_leaf=10)          # each leaf must have at least 10 samples
```

---

## 10. Example: Tuning max_depth

```python
for depth in [2, 3, 5, 10, None]:                            # test different max_depth values (None means unlimited)
    model = DecisionTreeClassifier(max_depth=depth, random_state=42)
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    acc = accuracy_score(y_test, y_pred)
    print(f"max_depth={depth}: Accuracy={acc:.4f}")          # print accuracy for each depth
```

Typically, there is a sweet spot where the tree is deep enough to capture patterns but not so deep that it overfits.

---

## 11. When to use decision trees

Decision trees work well when:

- You need an interpretable model that stakeholders can understand
- The decision rules are important for compliance or explanation
- You want to visualize the model
- You have a mix of numeric and categorical features

They may not work well when:

- The relationships are very complex (a single tree may not capture them)
- You need the highest possible accuracy (ensemble methods often perform better)
- The data has many irrelevant features

---

## 12. Summary

In this document, you learned how to:

1. Train a decision tree classifier with scikit-learn
2. Control tree complexity with parameters like `max_depth`
3. Evaluate the model using standard classification metrics
4. Visualize the tree to understand the decision rules
5. Interpret feature importance

The next document, `ml_random_forest.md`, shows how to combine many decision trees into a random forest for improved accuracy and robustness.
