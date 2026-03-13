
---
Credit Card Default Prediction (R)
---

# Project Overview

This project analyzes credit card client data to predict the likelihood that a customer will default on their payment in the following month. Financial institutions rely on these predictions to manage lending risk, adjust credit limits, and proactively support customers who may be at risk of missing payments.

Using historical customer financial and repayment behavior data, this project applies statistical modeling and exploratory data analysis to identify patterns associated with default risk.

The main objective is to build predictive models that estimate the probability of a client defaulting on their credit card payment next month.

# Modeling Approach

The project implements logistic regression models to predict the binary outcome of default vs. non-default.

## Model 1: Baseline Logistic Regression
The first model predicts default probability using a subset of key financial and demographic variables, including:
- Credit limit
- Age
- Gender
- Marital status
- Recent repayment status
- Billing amount
- Recent payment amount

This model serves as a baseline to understand how core financial indicators relate to default risk.

## Model 2: Enhanced Logistic Regression
The second model expands on the baseline by:
- Including additional predictor variables
- Introducing interaction terms
- Adding nonlinear transformations (such as squared terms)

These additions allow the model to capture more complex relationships in the data and potentially improve predictive performance.

# Dataset
The dataset contains information about credit card clients, their financial attributes, and repayment history.

The target variable is:
**default.payment.next.month**

Indicates whether the client defaulted on their credit card payment in the following month.

1 = client defaulted on payment
0 = client did not default

# File Descriptions 
FinalProject_Codes_credit_card.Rmd -> R Markdown file containing the full project workflow, including:
- Data wrangling
- Exploratory data analysis
- Model construction
- Model evaluation

credit_card_default.csv (dataset used in the analysis) -> Raw dataset containing client financial information and payment history

README.md -> Project overview, modeling explanation, and dataset documentation

# Data Fields
**Customer Information**
- LIMIT_BAL – Amount of given credit (NT dollars), including individual and family credit
- SEX – Gender (1 = male, 2 = female)
- EDUCATION – Education level
- MARRIAGE – Marital status
- AGE – Age of the client

**Repayment Status (Past 6 Months)**
Repayment status variables represent how late payments were in previous months.

- PAY_0 – Repayment status in September
- PAY_2 – Repayment status in August
- PAY_3 – Repayment status in July
- PAY_4 – Repayment status in June
- PAY_5 – Repayment status in May
- PAY_6 – Repayment status in April

Values indicate payment delay status.

**Bill Statement Amounts**
Amount of bill statements for the previous six months.

- BILL_AMT1 – September bill
- BILL_AMT2 – August bill
- BILL_AMT3 – July bill
- BILL_AMT4 – June bill
- BILL_AMT5 – May bill
- BILL_AMT6 – April bill

**Previous Payment Amounts**
Amount paid in the previous six months.

- PAY_AMT1 – September payment
- PAY_AMT2 – August payment
- PAY_AMT3 – July payment
- PAY_AMT4 – June payment
- PAY_AMT5 – May payment
- PAY_AMT6 – April payment

**Key Skills Demonstrated**
- Data wrangling in R
- Exploratory data analysis
- Logistic regression modeling
- Feature engineering (interactions and nonlinear)
- Model evaluation using statistical metrics
