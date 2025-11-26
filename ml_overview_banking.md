# Machine Learning Overview for Banking

This document introduces machine learning concepts in plain language and explains how they apply in a banking context.

We will use a fictional institution, **ABC Bank**, and a simplified scenario involving auto loans and loss recovery.

---

## 1. What is machine learning?

Machine learning is a way to teach computers to make predictions or decisions by learning from data, rather than by following explicit rules written by a programmer.

For example, instead of writing rules like "if the borrower has income above $50,000 and credit score above 700, approve the loan," a machine learning model learns patterns from historical data and uses those patterns to make predictions on new data.

---

## 2. Key terms explained

### Feature

A feature is a measurable property of the data that the model uses to make predictions. In a banking context, features might include:

- Loan amount
- Borrower's income
- Credit score
- Vehicle age
- Days since last payment

### Target

The target, also called the label, is the outcome we want to predict. Examples:

- For regression: the dollar amount of a loss
- For classification: whether a loss will be recovered (yes or no)

### Training data

Training data is historical data where we know both the features and the target. The model learns patterns from this data.

### Test data

Test data is a separate portion of data that the model has not seen during training. We use it to evaluate how well the model generalizes to new situations.

### Model

A model is the mathematical representation that captures the relationship between features and the target. Different algorithms produce different types of models.

---

## 3. Types of machine learning tasks

### Regression

Regression predicts a continuous numeric value. Example: predicting the dollar amount of a loss on an auto loan.

### Classification

Classification predicts a category or label. Example: predicting whether a loss will be recovered (yes) or not recovered (no).

---

## 4. ABC Bank scenario: Auto impounds and loss recovery

ABC Bank provides auto loans. When a borrower defaults, meaning they stop making payments, the bank may repossess the vehicle. The bank then tries to sell the vehicle to recover some of the money owed.

Key terms in this scenario:

- **Default**: The borrower has stopped making payments.
- **Repossession**: The bank takes back the vehicle.
- **Loss amount**: The difference between what was owed and what the bank recovers by selling the vehicle. A positive loss amount means the bank lost money.
- **Recovery**: Whether the bank successfully recovered a significant portion of the loss, which we will simplify to a yes/no outcome.

---

## 5. Example features for ABC Bank models

When building models for ABC Bank, we might use features like:

| Feature name         | Description                                      | Example value |
| -------------------- | ------------------------------------------------ | ------------- |
| loan_amount          | Original loan amount in dollars                  | 15000         |
| vehicle_age_years    | Age of the vehicle at repossession               | 5             |
| borrower_income      | Annual income of the borrower                    | 45000         |
| credit_score         | Borrower's credit score at origination           | 620           |
| days_delinquent      | Days since last payment before repossession      | 90            |
| vehicle_mileage      | Mileage on the vehicle at repossession           | 85000         |

---

## 6. Example targets

### Regression target

- `loss_amount`: The dollar amount the bank lost after selling the vehicle. We want to predict this value for new repossessions.

### Classification target

- `recovered`: A yes/no value indicating whether the bank recovered at least a threshold amount, such as 50% of the outstanding balance.

---

## 7. The machine learning workflow

A typical machine learning workflow involves these steps:

1. **Collect data**: Gather historical records with features and known outcomes.
2. **Prepare data**: Clean the data, handle missing values, and create useful features.
3. **Split data**: Divide into training and test sets.
4. **Train model**: Use the training data to fit a model.
5. **Evaluate model**: Use the test data to measure how well the model performs.
6. **Iterate**: Adjust features, try different algorithms, and improve performance.
7. **Deploy**: Use the model to make predictions on new data.

---

## 8. Why machine learning matters for banks

Banks deal with large volumes of data and need to make many decisions quickly:

- Which loan applications to approve
- How to price loans based on risk
- Which accounts are likely to default
- How much to expect from selling repossessed assets

Machine learning can help automate and improve these decisions by finding patterns in historical data that humans might miss.

---

## 9. What comes next

The following documents in this repo walk through specific machine learning algorithms:

- **Linear regression**: Predicting numeric loss amounts
- **Logistic regression**: Predicting yes/no recovery outcomes
- **Decision trees**: Classification using a tree of decisions
- **Random forests**: Combining many decision trees for better accuracy
- **Gradient boosting**: Building trees sequentially to reduce errors

Each document includes:

- An explanation of how the algorithm works
- Python code using scikit-learn
- Line-by-line comments explaining what each line does
- Example output and interpretation

---

This overview should give you the vocabulary and context needed to understand the algorithm-specific documents that follow.
