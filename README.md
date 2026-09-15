# Credit Card Default Prediction in R

## Project Overview

This project explores credit card customer data and builds models to predict whether a customer will default on their payment in the following month.

The analysis combines data preparation, exploratory visualization, logistic regression, and a neural network. Its goal is to examine customer characteristics and compare how well the models distinguish defaulting from non-defaulting customers.

## Dataset

The project dataset contains **12,000 customers**, **23 predictors**, and one binary outcome. No missing values were detected in the supplied CSV.

The original target variable, `default.payment.next.month`, is renamed to `default` during analysis:

- **0:** No default
- **1:** Default

The data describes credit card customers in Taiwan, including repayment history from April through September 2005. Monetary variables are recorded in New Taiwan dollars.

The notebook imports its data from this [teaching dataset](https://xiaorui.site/Data-Mining-R/lecture/data/credit_default.csv). The original dataset is documented by the [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/350/default%2Bof%2Bcredit%2Bcard%2Bclients); this project uses a 12,000-record version.

## Data Preparation and Exploration

The R Markdown workflow:

- Renames the target variable to `default`.
- Creates subsets of single and married customers for exploratory summaries.
- Creates `BILL_AMT1_to_3`, the sum of three recent bill statements, in a separate exploratory data object.
- Examines default frequency, age, credit limits, education, and gender.
- Uses boxplots to compare age and recent bill amounts by default status.
- Examines default status alongside gender and marital status.
- Applies min–max scaling to credit limit, age, repayment-status variables, bill amounts, and payment amounts.

The three-month bill total is created for exploration but is not included in the fitted models.

## Training and Testing Setup

The code uses a fixed split based on row position:

| Dataset | Observations | Defaults | Non-defaults |
|---|---:|---:|---:|
| Training: rows 1–8,400 | 8,400 | 1,832 | 6,568 |
| Testing: rows 8,401–12,000 | 3,600 | 800 | 2,800 |
| Total | 12,000 | 2,632 | 9,368 |

This is a 70% training and 30% testing split. Rows are not randomly sampled.

## Modeling Approach

### Logistic Regression

The notebook explores several binomial logistic regression specifications:

- All 23 predictors.
- All predictors plus an age–marital-status interaction.
- All predictors plus a squared age term.
- All predictors plus a squared recent-payment term.

The final evaluated logistic specification, `DefaultModel_quad2`, includes all predictors and `I(PAY_AMT1^2)`. It does not also include the interaction or squared age term from the other specifications.

The notebook also contains an initial seven-predictor model, `DefaultModel1`. As currently written, it omits the binomial family and therefore fits a Gaussian model rather than logistic regression.

### Neural Network

The neural network, `default_nn1`, uses all 23 predictors with:

- One hidden layer.
- Three hidden neurons.
- A nonlinear output using `linear.output = FALSE`.

Both the final logistic model and neural network are evaluated using ROC curves and area under the ROC curve, or AUC. AUC measures how well model scores distinguish defaults from non-defaults across classification thresholds.

## Model Results

A verification run using the supplied CSV, the notebook’s preprocessing, and its existing row split produced:

| Model | Training AUC | Testing AUC |
|---|---:|---:|
| Logistic regression: `DefaultModel_quad2` | 0.7273 | 0.7308 |
| Neural network: `default_nn1` | 0.7785 | 0.7700 |

The neural network verification run used `set.seed(20260915)`, R 4.4.2, and `neuralnet` 1.44.2. The uploaded notebook does not currently set this seed.

The notebook’s conclusion states a neural-network test AUC of 0.758. That value was not reproduced in this verification run. Neural-network results can vary with random initialization.

The neural network achieved higher test AUC in this run. Repeated evaluation would be needed to assess how consistently it outperforms logistic regression.

## Key Findings

- **Defaults are the minority outcome.** There are 2,632 observed defaults among 12,000 customers, representing 21.9% of the dataset. The test-set default rate is 22.2%.
- **The customer population is concentrated among younger and middle-aged adults.** Median age is 34, and 72.5% of customers are aged 21–40. This describes the sample and does not establish that younger customers have greater default risk.
- **University education is the most common education category.** It accounts for 5,596 customers, or 46.6% of the dataset.
- **The final logistic model’s test AUC was reproduced at approximately 0.731.** The neural network achieved 0.770 in the seeded verification run, showing stronger discrimination on this particular test set.

## Limitations

- **Preprocessing uses the full dataset.** Scaling minima and maxima are calculated before the split, allowing test-set information into preprocessing.
- **Evaluation uses one fixed row split.** Performance may depend on the ordering of the data. The notebook does not use cross-validation.
- **Neural-network fitting is not seeded in the uploaded code.** Results may differ between runs.
- **Categorical predictors retain numeric codes.** Education and marital status are not explicitly converted to factors, so their numeric treatment imposes relationships between categories.
- **AUC does not establish operational usefulness.** The workflow does not evaluate calibration, decision costs, or recall and precision at a chosen classification threshold.
- **The data represents a historical customer population.** Performance on other institutions or current customers has not been established.

## Potential Application

Credit-default models can support research into account monitoring and customer outreach. Applying this workflow to lending decisions would require further validation, updated data, and evaluation of prediction thresholds and their consequences.

## Data Fields

### Customer Information

| Variable | Description |
|---|---|
| `LIMIT_BAL` | Credit limit in New Taiwan dollars |
| `SEX` | Recorded sex: 1 = male; 2 = female |
| `EDUCATION` | 1 = graduate school; 2 = university; 3 = high school; 4 = others |
| `MARRIAGE` | 1 = married; 2 = single; 3 = others |
| `AGE` | Age in years |

### Repayment Status

| Variable | Month in 2005 |
|---|---|
| `PAY_0` | September |
| `PAY_2` | August |
| `PAY_3` | July |
| `PAY_4` | June |
| `PAY_5` | May |
| `PAY_6` | April |

These variables contain repayment-status codes. Positive codes represent payment delays. The supplied data also includes −2, −1, and 0; the notebook’s variable description does not fully document all these codes.

### Bill Statements and Previous Payments

| Variables | Description |
|---|---|
| `BILL_AMT1`–`BILL_AMT6` | Bill statement amounts from September through April 2005 |
| `PAY_AMT1`–`PAY_AMT6` | Previous payment amounts from September through April 2005 |

All amounts are in New Taiwan dollars.

### Target

`default.payment.next.month`: next-month default indicator, renamed to `default` in the analysis.

## File Descriptions

- `FinalProject_Codes_credit_card.Rmd`: R Markdown workflow containing data preparation, exploratory plots, model specifications, and ROC/AUC evaluation.
- `credit_default.csv`: The 12,000-record project dataset. The current notebook imports an online CSV rather than reading this local file.
- `README.md`: Project overview, methodology, results, and limitations.

## Tools and Skills

- R and R Markdown
- `dplyr` for data manipulation
- Base R graphics for exploratory visualization
- `glm` for regression modeling
- `neuralnet` for neural-network modeling
- `ROCR` for ROC curves and AUC
- Feature engineering with interaction and quadratic terms
