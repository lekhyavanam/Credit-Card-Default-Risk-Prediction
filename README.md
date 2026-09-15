
---

# Credit Card Default Prediction (R)

## Project Overview

This project analyzes credit card client data to predict the likelihood that a customer will default on their payment in the following month. Financial institutions rely on these predictions to manage lending risk, adjust credit limits, and proactively support customers who may be at risk of missing payments.

Using historical customer financial and repayment behavior data, this project applies statistical modeling and exploratory data analysis to identify patterns associated with default risk.

The main objective is to build predictive models that estimate the probability of a client defaulting on their credit card payment next month.

## Dataset

The dataset contains information about credit card clients, their financial attributes, and repayment history.

The target variable is:
**default.payment.next.month**

Indicates whether the client defaulted on their credit card payment in the following month.

1 = client defaulted on payment
0 = client did not default

## Modeling Approach

This project compares two approaches for predicting credit card default: **logistic regression** and a **neural network**. Both models estimate whether a customer will default on their payment in the following month based on historical financial, demographic, and repayment behavior.

### Model 1: Logistic Regression

Logistic regression was used as an interpretable statistical approach for estimating default probability. Multiple specifications were explored, including additional predictors, interaction terms, and quadratic terms to capture nonlinear relationships between customer characteristics and default risk.

The final logistic regression model (`DefaultModel_quad2`) achieved a **test AUC of 0.731**.

### Model 2: Neural Network

A neural network (`default_nn1`) was developed to capture more complex and nonlinear relationships among the predictors that may not be fully represented by logistic regression.

The neural network achieved a **test AUC of 0.758**, outperforming the final logistic regression model on the testing dataset.

## Results at a Glance

| Metric                       |        Result |
| ---------------------------- | ------------: |
| Testing observations         |        12,000 |
| Defaults                     | 2,632 (21.9%) |
| Non-defaults                 | 9,368 (78.1%) |
| Logistic Regression Test AUC |         0.731 |
| Neural Network Test AUC      |     **0.758** |

## Key Findings

- **Default represented 21.9% of the testing data**, with 2,632 defaults among 12,000 observations.
- **The neural network provided stronger out-of-sample discrimination**, achieving a test AUC of 0.758 compared with 0.731 for the final logistic regression model.
- **Repayment behavior emerged as an important signal of default risk**, demonstrating the value of historical customer payment behavior when assessing future credit risk.

## Real-World Application & Industry Context

Within the 12,000-observation testing dataset, **2,632 customers (21.9%) experienced default**, while 9,368 (78.1%) did not, highlighting the importance of identifying high-risk customers before missed payments escalate.

The neural network achieved a test **AUC of 0.758**, compared with **0.731 for logistic regression**, representing a 0.027 improvement in discriminatory performance. The higher AUC indicates that the neural network provided **better out-of-sample discrimination between defaulting and non-defaulting customers** than the final logistic regression model.

For comparison, U.S. commercial banks reported a credit card delinquency rate of approximately **2.85% in Q2 2026**. These figures are not directly comparable—the project's target measures next-month default while industry delinquency statistics use different definitions and reporting periods—but they provide context for the real-world importance of credit-risk detection.

For an issuer such as American Express, a model like this could function as an **early-warning system**, using repayment history, bill balances, and payment behavior to identify higher-risk accounts for monitoring or proactive intervention before they progress to serious delinquency or charge-off.

# Data Fields

### Customer Information

- LIMIT\_BAL – Amount of given credit (NT dollars), including individual and family credit
- SEX – Gender (1 = male, 2 = female)
- EDUCATION – Education level
- MARRIAGE – Marital status
- AGE – Age of the client

### Repayment Status (Past 6 Months)
Repayment status variables represent how late payments were in previous months.

- PAY\_0 – Repayment status in September
- PAY\_2 – Repayment status in August
- PAY\_3 – Repayment status in July
- PAY\_4 – Repayment status in June
- PAY\_5 – Repayment status in May
- PAY\_6 – Repayment status in April

Values indicate payment delay status.

### Bill Statement Amounts
Amount of bill statements for the previous six months.

- BILL\_AMT1 – September bill
- BILL\_AMT2 – August bill
- BILL\_AMT3 – July bill
- BILL\_AMT4 – June bill
- BILL\_AMT5 – May bill
- BILL\_AMT6 – April bill

### Previous Payment Amounts
Amount paid in the previous six months.

- PAY\_AMT1 – September payment
- PAY\_AMT2 – August payment
- PAY\_AMT3 – July payment
- PAY\_AMT4 – June payment
- PAY\_AMT5 – May payment
- PAY\_AMT6 – April payment

## File Descriptions

FinalProject\_Codes\_credit\_card.Rmd -> R Markdown file containing the full project workflow, including:

- Data wrangling
- Exploratory data analysis
- Model construction
- Model evaluation

credit\_card\_default.csv (dataset used in the analysis) -> Raw dataset containing client financial information and payment history

README.md -> Project overview, modeling explanation, and dataset documentation

## Key Skills Demonstrated

- Data wrangling in R
- Exploratory data analysis
- Predictive modeling (logistic regression & neural networks)
- Feature engineering (interactions and nonlinear)
- Model comparison using ROC/AUC
