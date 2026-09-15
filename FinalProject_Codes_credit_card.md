# Credit Card Default Prediction in R

Rendered analysis · 15 September 2026

## Setup and data

Place `credit_default.csv` beside this notebook. Required packages are `dplyr`,
`ROCR`, `neuralnet`, `knitr`, and `rmarkdown`.


``` r
knitr::opts_chunk$set(error = FALSE, fig.width = 8, fig.height = 5)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

``` r
library(ROCR)
library(neuralnet)
```

```
## 
## Attaching package: 'neuralnet'
```

```
## The following object is masked from 'package:ROCR':
## 
##     prediction
```

```
## The following object is masked from 'package:dplyr':
## 
##     compute
```

``` r
credit_default <- read.csv("credit_default.csv")
credit_default <- rename(credit_default, default = default.payment.next.month)
stopifnot(!anyNA(credit_default), all(credit_default$default %in% 0:1))
# Explicit categories preserve nominal rather than numeric relationships.
credit_default$SEX <- factor(credit_default$SEX, levels = c(1, 2),
                             labels = c("Male", "Female"))
credit_default$EDUCATION <- factor(credit_default$EDUCATION, levels = 1:4,
                                  labels = c("Graduate", "University", "HighSchool", "Other"))
credit_default$MARRIAGE <- factor(credit_default$MARRIAGE, levels = 1:3,
                                 labels = c("Married", "Single", "Other"))
stopifnot(!anyNA(credit_default))
table(credit_default$default)
```

```
## 
##    0    1 
## 9368 2632
```

Repayment-status variables retain their original numeric codes. The supplied
notebook documents −1 as payment made duly and positive values as months of
payment delay, but does not establish the definitions of −2 and 0. These codes
are retained without assigning new meanings; numeric modeling assumes a common
ordered scale and should be compared with categorical encoding in future work.

## Constructing new variables


``` r
# Create a new variable named 'BILL_AMT1_to_3' by summing up the bill amounts 
# for the first three months (BILL_AMT1, BILL_AMT2, and BILL_AMT3) for each observation
credit_default2 <- 
  mutate(credit_default, 
         BILL_AMT1_to_3 = BILL_AMT1 + BILL_AMT2 + BILL_AMT3)

summary(credit_default2$BILL_AMT1_to_3)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## -141129   14102   67642  147718  191246 2483462
```

## Creating sub-samples for specific needs


``` r
# Create a subset 'credit_single' containing observations for clients who are single
credit_single <- 
  filter(credit_default, MARRIAGE == "Single")

# Create a subset 'credit_married' containing observations for clients who are married
credit_married <- 
  filter(credit_default, MARRIAGE == "Married")


# Generate summary statistics for the 'PAY_AMT1' variable for single clients
summary(credit_single$PAY_AMT1)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    1012    2184    5900    5043  493358
```

``` r
# Generate summary statistics for the 'PAY_AMT1' variable for married clients
summary(credit_married$PAY_AMT1)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0     794    2108    5635    5000  302000
```

# Exploratory Analyses & Visualization

### 1. Exploring the distribution of the dependent/independent variables


``` r
summary(credit_default)
```

```
##    LIMIT_BAL           SEX            EDUCATION       MARRIAGE   
##  Min.   :  10000   Male  :4765   Graduate  :4228   Married:5525  
##  1st Qu.:  50000   Female:7235   University:5596   Single :6332  
##  Median : 140000                 HighSchool:1992   Other  : 143  
##  Mean   : 167501                 Other     : 184                 
##  3rd Qu.: 240000                                                 
##  Max.   :1000000                                                 
##       AGE           PAY_0              PAY_2            PAY_3        
##  Min.   :21.0   Min.   :-2.00000   Min.   :-2.000   Min.   :-2.0000  
##  1st Qu.:28.0   1st Qu.:-1.00000   1st Qu.:-1.000   1st Qu.:-1.0000  
##  Median :34.0   Median : 0.00000   Median : 0.000   Median : 0.0000  
##  Mean   :35.5   Mean   :-0.01575   Mean   :-0.128   Mean   :-0.1667  
##  3rd Qu.:41.0   3rd Qu.: 0.00000   3rd Qu.: 0.000   3rd Qu.: 0.0000  
##  Max.   :79.0   Max.   : 8.00000   Max.   : 7.000   Max.   : 7.0000  
##      PAY_4             PAY_5             PAY_6           BILL_AMT1     
##  Min.   :-2.0000   Min.   :-2.0000   Min.   :-2.0000   Min.   :-15308  
##  1st Qu.:-1.0000   1st Qu.:-1.0000   1st Qu.:-1.0000   1st Qu.:  3690  
##  Median : 0.0000   Median : 0.0000   Median : 0.0000   Median : 22658  
##  Mean   :-0.2256   Mean   :-0.2678   Mean   :-0.2931   Mean   : 51392  
##  3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 67207  
##  Max.   : 7.0000   Max.   : 7.0000   Max.   : 8.0000   Max.   :964511  
##    BILL_AMT2        BILL_AMT3         BILL_AMT4        BILL_AMT5     
##  Min.   :-33350   Min.   :-157264   Min.   :-81334   Min.   :-81334  
##  1st Qu.:  3156   1st Qu.:   2980   1st Qu.:  2411   1st Qu.:  1863  
##  Median : 21652   Median :  20330   Median : 19078   Median : 18244  
##  Mean   : 49246   Mean   :  47080   Mean   : 43102   Mean   : 40314  
##  3rd Qu.: 63786   3rd Qu.:  59662   3rd Qu.: 53117   3rd Qu.: 49927  
##  Max.   :983931   Max.   : 855086   Max.   :891586   Max.   :927171  
##    BILL_AMT6          PAY_AMT1         PAY_AMT2            PAY_AMT3     
##  Min.   :-339603   Min.   :     0   Min.   :      0.0   Min.   :     0  
##  1st Qu.:   1309   1st Qu.:  1000   1st Qu.:    944.8   1st Qu.:   400  
##  Median :  17130   Median :  2128   Median :   2013.0   Median :  1827  
##  Mean   :  38821   Mean   :  5766   Mean   :   6250.1   Mean   :  5195  
##  3rd Qu.:  48938   3rd Qu.:  5006   3rd Qu.:   5000.0   3rd Qu.:  4505  
##  Max.   : 961664   Max.   :493358   Max.   :1227082.0   Max.   :896040  
##     PAY_AMT4         PAY_AMT5         PAY_AMT6           default      
##  Min.   :     0   Min.   :     0   Min.   :     0.0   Min.   :0.0000  
##  1st Qu.:   300   1st Qu.:   300   1st Qu.:   142.8   1st Qu.:0.0000  
##  Median :  1500   Median :  1518   Median :  1500.0   Median :0.0000  
##  Mean   :  4878   Mean   :  4868   Mean   :  5432.7   Mean   :0.2193  
##  3rd Qu.:  4078   3rd Qu.:  4121   3rd Qu.:  4061.8   3rd Qu.:0.0000  
##  Max.   :432130   Max.   :426529   Max.   :528666.0   Max.   :1.0000
```

``` r
# Draw a pie chart to show the proportion of default clients
pie(table(credit_default$default))
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)


``` r
# Create a pie chart with customized colors, labels, and a title
default_table <- table(credit_default$default)
colors <- c("#FF9999", "#66B2FF")  # Define custom colors for the slices
labels <- c("Not Default", "Default")  # Define custom labels for the slices
percentages <- round(default_table / sum(default_table) * 100, 1)  # Calculate percentages
pie(default_table, 
    col = colors, 
    labels = paste(labels, "\n (", percentages, "%)"),  # Include percentages in labels
    main = "Credit Default Status")
legend("topright", legend = labels, fill = colors, cex = 0.8)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)


``` r
# Create a histogram to visualize the distribution of ages in the dataset
hist(credit_default$AGE, main = "Distribution of Age")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)

``` r
# Create a histogram to visualize the distribution of balance limits in the dataset
hist(credit_default$LIMIT_BAL, main = "Distribution of Balance Limit")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-2.png)


``` r
# Generate a table to display the frequency of each education level in the dataset
table(credit_default$EDUCATION)
```

```
## 
##   Graduate University HighSchool      Other 
##       4228       5596       1992        184
```

``` r
# Create a bar plot to visualize the frequency of each education level
barplot(table(credit_default$EDUCATION))
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

``` r
# Generate a table to display the frequency of each gender in the dataset
table(credit_default$SEX)
```

```
## 
##   Male Female 
##   4765   7235
```

### 2. Exploring the association between default indicator and other numerical variables


``` r
# Exploring the association between default indicator and other numerical variables
boxplot(credit_default$AGE ~ credit_default$default, horizontal = T)
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png)

``` r
boxplot(credit_default$BILL_AMT1 ~ credit_default$default, horizontal = T)
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-2.png)

### 3. Exploring the association between default indicator and categorical variables

Counts describe sample composition; within-group proportions describe observed default rates. 


``` r
# Compare observed default status across categories
table(credit_default$default, credit_default$SEX, 
      dnn = c("Default or not","Sex: Gender (1 = male; 2 = female)")) 
```

```
##               Sex: Gender (1 = male; 2 = female)
## Default or not Male Female
##              0 3603   5765
##              1 1162   1470
```

``` r
# Create a contingency table of default status and marital status
myNewTable <- table(credit_default$default, credit_default$MARRIAGE) 

# Convert the contingency table into proportions
prop.table(myNewTable, margin = 2) 
```

```
##    
##       Married    Single     Other
##   0 0.7688688 0.7916930 0.7482517
##   1 0.2311312 0.2083070 0.2517483
```

``` r
# Create a stacked bar chart
barplot(myNewTable, 
        col=c('blue','red'),  # Define colors for the bars
        legend= paste("Default = ", rownames(myNewTable)),  
        # Add legend with default status labels
        xlab="Marital status",  # Label for x-axis
        ylab = "Count")  # Label for y-axis
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png)

# Building Models

## Preparing the training and testing datasets

The full customer sample is split randomly within each outcome class using a
fixed seed. Scaling is fitted on training rows only. Test values may lie outside
[0, 1]; they are not clipped. The exploratory bill-total variable is not modeled.


``` r
set.seed(20260915)
by_class <- split(seq_len(nrow(credit_default)), credit_default$default)
sample_index <- sort(unlist(lapply(by_class, function(i) sample(i, round(length(i) * 0.70)))))
train_raw <- credit_default[sample_index, ]
test_raw <- credit_default[-sample_index, ]
stopifnot(length(intersect(sample_index, setdiff(seq_len(nrow(credit_default)), sample_index))) == 0)
var_used <- c("LIMIT_BAL", "AGE", "PAY_0", "PAY_2", "PAY_3", "PAY_4", "PAY_5", "PAY_6",
              paste0("BILL_AMT", 1:6), paste0("PAY_AMT", 1:6))
fit_scaler <- function(data) {
  mins <- vapply(data[var_used], min, numeric(1))
  ranges <- vapply(data[var_used], max, numeric(1)) - mins
  ranges[ranges == 0] <- 1
  list(mins = mins, ranges = ranges)
}
apply_scaler <- function(data, scaler) {
  data[var_used] <- as.data.frame(scale(data[var_used], center = scaler$mins, scale = scaler$ranges))
  data
}
scaler <- fit_scaler(train_raw)
credit_train <- apply_scaler(train_raw, scaler)
credit_test <- apply_scaler(test_raw, scaler)
rbind(Training = table(credit_train$default), Testing = table(credit_test$default))
```

```
##             0    1
## Training 6558 1842
## Testing  2810  790
```

## First Method: Logistic regression models

### A simple model based on your intuition: only select a few relevant predictors 


``` r
# Build a model based on your intuition: only select a few relevant predictors 
DefaultModel1 <- 
  glm(default ~ LIMIT_BAL + SEX + MARRIAGE + AGE + PAY_0 + BILL_AMT1 + PAY_AMT1,
     data = credit_train, family = "binomial")

summary(DefaultModel1, digits = 3)
```

```
## 
## Call:
## glm(formula = default ~ LIMIT_BAL + SEX + MARRIAGE + AGE + PAY_0 + 
##     BILL_AMT1 + PAY_AMT1, family = "binomial", data = credit_train)
## 
## Coefficients:
##                Estimate Std. Error z value Pr(>|z|)    
## (Intercept)    -2.32338    0.11804 -19.683  < 2e-16 ***
## LIMIT_BAL      -1.68255    0.26745  -6.291 3.15e-10 ***
## SEXFemale      -0.19071    0.05774  -3.303 0.000956 ***
## MARRIAGESingle -0.19539    0.06470  -3.020 0.002529 ** 
## MARRIAGEOther  -0.01248    0.25104  -0.050 0.960341    
## AGE             0.12309    0.17895   0.688 0.491552    
## PAY_0           7.11886    0.28389  25.076  < 2e-16 ***
## BILL_AMT1      -0.70997    0.45019  -1.577 0.114786    
## PAY_AMT1       -5.00185    1.62675  -3.075 0.002107 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 8836.8  on 8399  degrees of freedom
## Residual deviance: 7774.3  on 8391  degrees of freedom
## AIC: 7792.3
## 
## Number of Fisher Scoring iterations: 5
```

### A simple model without imposing any constraints on predictor selection


``` r
# Build a simple model without imposing any constraints on predictor selection
DefaultModelFull <- 
  glm(default ~ .,
     data = credit_train, 
     family = "binomial")

summary(DefaultModelFull, digits = 3)
```

```
## 
## Call:
## glm(formula = default ~ ., family = "binomial", data = credit_train)
## 
## Coefficients:
##                     Estimate Std. Error z value Pr(>|z|)    
## (Intercept)         -2.57221    0.15797 -16.283  < 2e-16 ***
## LIMIT_BAL           -1.13930    0.30070  -3.789 0.000151 ***
## SEXFemale           -0.17236    0.05831  -2.956 0.003116 ** 
## EDUCATIONUniversity -0.09073    0.06811  -1.332 0.182872    
## EDUCATIONHighSchool -0.17943    0.09166  -1.958 0.050273 .  
## EDUCATIONOther      -1.19854    0.35116  -3.413 0.000642 ***
## MARRIAGESingle      -0.19226    0.06605  -2.911 0.003603 ** 
## MARRIAGEOther        0.01297    0.25305   0.051 0.959128    
## AGE                  0.23735    0.18619   1.275 0.202385    
## PAY_0                5.80459    0.34195  16.975  < 2e-16 ***
## PAY_2                0.89381    0.34582   2.585 0.009749 ** 
## PAY_3                0.35510    0.38608   0.920 0.357703    
## PAY_4                0.35860    0.42752   0.839 0.401578    
## PAY_5                0.47930    0.45319   1.058 0.290236    
## PAY_6                0.20633    0.37917   0.544 0.586332    
## BILL_AMT1           -4.67971    1.87977  -2.490 0.012792 *  
## BILL_AMT2            4.45808    2.44971   1.820 0.068783 .  
## BILL_AMT3            0.93742    1.95370   0.480 0.631356    
## BILL_AMT4           -7.26418    2.60710  -2.786 0.005331 ** 
## BILL_AMT5            2.73559    3.27988   0.834 0.404251    
## BILL_AMT6            3.76190    2.62019   1.436 0.151078    
## PAY_AMT1            -5.88861    1.88224  -3.129 0.001757 ** 
## PAY_AMT2            -7.14260    4.20996  -1.697 0.089773 .  
## PAY_AMT3             0.98606    2.55869   0.385 0.699958    
## PAY_AMT4            -2.13120    1.33360  -1.598 0.110026    
## PAY_AMT5            -0.84134    1.28553  -0.654 0.512811    
## PAY_AMT6            -0.79647    1.16773  -0.682 0.495198    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 8836.8  on 8399  degrees of freedom
## Residual deviance: 7675.9  on 8373  degrees of freedom
## AIC: 7729.9
## 
## Number of Fisher Scoring iterations: 6
```

### Advanced model development: Interaction variables


``` r
# A potential model with interaction term
DefaultModel_inter <- 
  glm(default ~ . + AGE * MARRIAGE,
     data = credit_train, 
     family = "binomial")

summary(DefaultModel_inter, digits = 3)
```

```
## 
## Call:
## glm(formula = default ~ . + AGE * MARRIAGE, family = "binomial", 
##     data = credit_train)
## 
## Coefficients:
##                     Estimate Std. Error z value Pr(>|z|)    
## (Intercept)         -2.70370    0.17113 -15.799  < 2e-16 ***
## LIMIT_BAL           -1.12763    0.30089  -3.748 0.000178 ***
## SEXFemale           -0.16920    0.05835  -2.900 0.003736 ** 
## EDUCATIONUniversity -0.08671    0.06829  -1.270 0.204153    
## EDUCATIONHighSchool -0.17295    0.09175  -1.885 0.059435 .  
## EDUCATIONOther      -1.19885    0.35137  -3.412 0.000645 ***
## MARRIAGESingle      -0.02206    0.11976  -0.184 0.853845    
## MARRIAGEOther        1.58780    0.61434   2.585 0.009751 ** 
## AGE                  0.56088    0.24785   2.263 0.023637 *  
## PAY_0                5.82624    0.34242  17.015  < 2e-16 ***
## PAY_2                0.89993    0.34628   2.599 0.009354 ** 
## PAY_3                0.33946    0.38626   0.879 0.379485    
## PAY_4                0.36926    0.42743   0.864 0.387631    
## PAY_5                0.47370    0.45355   1.044 0.296278    
## PAY_6                0.21319    0.37961   0.562 0.574397    
## BILL_AMT1           -4.68070    1.88316  -2.486 0.012935 *  
## BILL_AMT2            4.44884    2.45572   1.812 0.070045 .  
## BILL_AMT3            0.92846    1.95646   0.475 0.635100    
## BILL_AMT4           -7.36736    2.61480  -2.818 0.004839 ** 
## BILL_AMT5            2.81084    3.28488   0.856 0.392169    
## BILL_AMT6            3.81830    2.62563   1.454 0.145879    
## PAY_AMT1            -5.92182    1.88826  -3.136 0.001712 ** 
## PAY_AMT2            -7.10733    4.21228  -1.687 0.091548 .  
## PAY_AMT3             1.01600    2.54391   0.399 0.689608    
## PAY_AMT4            -2.19403    1.33755  -1.640 0.100936    
## PAY_AMT5            -0.85958    1.28901  -0.667 0.504864    
## PAY_AMT6            -0.81417    1.16271  -0.700 0.483780    
## MARRIAGESingle:AGE  -0.58389    0.36450  -1.602 0.109179    
## MARRIAGEOther:AGE   -3.67418    1.35492  -2.712 0.006693 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 8836.8  on 8399  degrees of freedom
## Residual deviance: 7666.8  on 8371  degrees of freedom
## AIC: 7724.8
## 
## Number of Fisher Scoring iterations: 6
```

### Advanced model development: nonlinear term


``` r
# Fit a logistic regression model with quadratic age term
DefaultModel_quad <- 
  glm(default ~ . + I(AGE^2),  # Formula including quadratic term for age
     data = credit_train,  # Dataset
     family = "binomial")  # Binomial family for binary outcome
summary(DefaultModel_quad, digits = 3)  # Summary of model with 3 significant digits
```

```
## 
## Call:
## glm(formula = default ~ . + I(AGE^2), family = "binomial", data = credit_train)
## 
## Coefficients:
##                     Estimate Std. Error z value Pr(>|z|)    
## (Intercept)         -2.60928    0.17400 -14.996  < 2e-16 ***
## LIMIT_BAL           -1.16038    0.30364  -3.822 0.000133 ***
## SEXFemale           -0.17038    0.05844  -2.916 0.003549 ** 
## EDUCATIONUniversity -0.09092    0.06812  -1.335 0.182000    
## EDUCATIONHighSchool -0.17785    0.09171  -1.939 0.052462 .  
## EDUCATIONOther      -1.20017    0.35123  -3.417 0.000633 ***
## MARRIAGESingle      -0.18542    0.06735  -2.753 0.005907 ** 
## MARRIAGEOther        0.01760    0.25304   0.070 0.944539    
## AGE                  0.51830    0.58212   0.890 0.373271    
## PAY_0                5.80517    0.34194  16.977  < 2e-16 ***
## PAY_2                0.89596    0.34585   2.591 0.009580 ** 
## PAY_3                0.35562    0.38611   0.921 0.357030    
## PAY_4                0.35568    0.42762   0.832 0.405543    
## PAY_5                0.48621    0.45335   1.072 0.283504    
## PAY_6                0.20545    0.37922   0.542 0.587980    
## BILL_AMT1           -4.69225    1.88038  -2.495 0.012582 *  
## BILL_AMT2            4.47195    2.44969   1.826 0.067923 .  
## BILL_AMT3            0.94483    1.95422   0.483 0.628752    
## BILL_AMT4           -7.27745    2.60759  -2.791 0.005257 ** 
## BILL_AMT5            2.73950    3.28044   0.835 0.403661    
## BILL_AMT6            3.75763    2.62038   1.434 0.151572    
## PAY_AMT1            -5.89004    1.88328  -3.128 0.001763 ** 
## PAY_AMT2            -7.14441    4.21028  -1.697 0.089716 .  
## PAY_AMT3             0.98414    2.56603   0.384 0.701328    
## PAY_AMT4            -2.13486    1.33389  -1.600 0.109492    
## PAY_AMT5            -0.84392    1.28601  -0.656 0.511676    
## PAY_AMT6            -0.79695    1.16868  -0.682 0.495287    
## I(AGE^2)            -0.39796    0.78187  -0.509 0.610760    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 8836.8  on 8399  degrees of freedom
## Residual deviance: 7675.7  on 8372  degrees of freedom
## AIC: 7731.7
## 
## Number of Fisher Scoring iterations: 6
```

``` r
# Fit a logistic regression model with quadratic PAY_AMT1 term
DefaultModel_quad2 <- 
  glm(default ~ . + I(PAY_AMT1^2),  # Formula including quadratic term for PAY_AMT1
     data = credit_train,  # Dataset
     family = "binomial")  # Binomial family for binary outcome
summary(DefaultModel_quad2, digits = 3)  # Summary of model with 3 significant digits
```

```
## 
## Call:
## glm(formula = default ~ . + I(PAY_AMT1^2), family = "binomial", 
##     data = credit_train)
## 
## Coefficients:
##                      Estimate Std. Error z value Pr(>|z|)    
## (Intercept)          -2.54520    0.15813 -16.096  < 2e-16 ***
## LIMIT_BAL            -1.10322    0.30119  -3.663 0.000249 ***
## SEXFemale            -0.17357    0.05832  -2.976 0.002920 ** 
## EDUCATIONUniversity  -0.09292    0.06813  -1.364 0.172586    
## EDUCATIONHighSchool  -0.17910    0.09166  -1.954 0.050709 .  
## EDUCATIONOther       -1.19895    0.35119  -3.414 0.000640 ***
## MARRIAGESingle       -0.19073    0.06607  -2.887 0.003894 ** 
## MARRIAGEOther         0.02038    0.25335   0.080 0.935893    
## AGE                   0.23211    0.18620   1.247 0.212546    
## PAY_0                 5.76308    0.34228  16.837  < 2e-16 ***
## PAY_2                 0.80425    0.34716   2.317 0.020522 *  
## PAY_3                 0.43740    0.38492   1.136 0.255814    
## PAY_4                 0.36855    0.42693   0.863 0.387994    
## PAY_5                 0.47799    0.45316   1.055 0.291518    
## PAY_6                 0.20498    0.37918   0.541 0.588800    
## BILL_AMT1            -4.53350    1.86867  -2.426 0.015264 *  
## BILL_AMT2             4.37020    2.41012   1.813 0.069790 .  
## BILL_AMT3             1.15432    1.93481   0.597 0.550770    
## BILL_AMT4            -7.21079    2.59427  -2.780 0.005444 ** 
## BILL_AMT5             2.77307    3.26625   0.849 0.395877    
## BILL_AMT6             3.54479    2.60805   1.359 0.174092    
## PAY_AMT1            -10.38168    2.59652  -3.998 6.38e-05 ***
## PAY_AMT2             -7.67663    3.67775  -2.087 0.036860 *  
## PAY_AMT3              1.31555    2.50645   0.525 0.599676    
## PAY_AMT4             -1.99313    1.32203  -1.508 0.131649    
## PAY_AMT5             -0.72828    1.27544  -0.571 0.568001    
## PAY_AMT6             -0.77311    1.16757  -0.662 0.507871    
## I(PAY_AMT1^2)        15.48585    5.55837   2.786 0.005336 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 8836.8  on 8399  degrees of freedom
## Residual deviance: 7671.2  on 8372  degrees of freedom
## AIC: 7727.2
## 
## Number of Fisher Scoring iterations: 6
```

### Evaluating the logistic regression model 

1. In-sample AUC-ROC (or training set)


``` r
# Predicted probability for training set
pred_train_logit <- predict(DefaultModel_quad2, newdata = credit_train, type="response")

library(ROCR)
# This line of code creates a prediction object 'pred' using the 'ROCR::prediction()' 
pred_obj_train_logit <- ROCR::prediction(pred_train_logit, credit_train$default)

# This line of code calculates True Positive Rate (TPR) and False Positive Rate
# (FPR) based on the prediction object 'pred_obj_train' using the 'performance()'
# function from the 'ROCR' package. 
# It stores the resulting performance object in the variable 'perf_train'.
perf_train_logit <- performance(pred_obj_train_logit, "tpr", "fpr")

# Draw the ROC curve
plot(perf_train_logit, colorize=TRUE)
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png)

``` r
# This line of code calculates the Area Under the ROC Curve (AUC) using the 'performance()' function with "auc" as the argument. 
# It extracts the AUC value from the resulting performance object and returns it as
# a numeric value.
# The 'unlist()' function is used to convert the AUC value from a list to a numeric vector.
unlist(slot(performance(pred_obj_train_logit, "auc"), "y.values"))
```

```
## [1] 0.7366379
```

2. Out-of-sample AUC-ROC performance (testing set is more important)


``` r
# Predicted probability for testing set
pred_test_logit <- predict(DefaultModel_quad2, newdata = credit_test, type="response")

library(ROCR)
# This line of code creates a prediction object 'pred' using the 'ROCR::prediction()' 
pred_obj_test_logit <- ROCR::prediction(pred_test_logit, credit_test$default)

perf_test_logit <- performance(pred_obj_test_logit, "tpr", "fpr")

# Draw the ROC curve
plot(perf_test_logit, colorize=TRUE)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png)

``` r
# This line of code calculates the Area Under the ROC Curve (AUC) using the 'performance()' function with "auc" as the argument. 
unlist(slot(performance(pred_obj_test_logit, "auc"), "y.values"))
```

```
## [1] 0.7119618
```


## Second Method: Neural Network

Categorical predictors are represented by indicator columns using the same
training-derived design specification for both partitions. The network retains
one hidden layer with three neurons.


``` r
nn_terms <- terms(~ ., data = credit_train[, setdiff(names(credit_train), "default")])
x_train <- model.matrix(nn_terms, credit_train)[, -1, drop = FALSE]
x_test <- model.matrix(nn_terms, credit_test)[, -1, drop = FALSE]
colnames(x_train) <- make.names(colnames(x_train), unique = TRUE)
colnames(x_test) <- make.names(colnames(x_test), unique = TRUE)
stopifnot(identical(colnames(x_train), colnames(x_test)))
nn_train <- data.frame(default = credit_train$default, x_train)
set.seed(20260916)
default_nn1 <- neuralnet(default ~ ., data = nn_train,
                         hidden = 3, linear.output = FALSE)
if (is.null(default_nn1$result.matrix)) stop("Neural network did not converge; no results reported.")
plot(default_nn1)
```

### Evaluating the neural network model

We calculate the AUC-ROC for the training set. 


``` r
# Predicted probability for training set
pred_train_nn1 <- as.vector(predict(default_nn1, newdata = as.data.frame(x_train)))

library(ROCR)
# This line of code creates a prediction object 'pred' using the 'ROCR::prediction()' 
pred_obj_train_nn1 <- ROCR::prediction(pred_train_nn1, credit_train$default)

# This line of code calculates True Positive Rate (TPR) and False Positive Rate
# (FPR) based on the prediction object 'pred_obj_train' using the 'performance()'
# function from the 'ROCR' package. 
# It stores the resulting performance object in the variable 'perf_train'.
perf_train_nn1 <- performance(pred_obj_train_nn1, "tpr", "fpr")

plot(perf_train_nn1, colorize=TRUE)
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15-1.png)

``` r
# This line of code calculates the Area Under the ROC Curve (AUC) using the 'performance()' function with "auc" as the argument. 
# It extracts the AUC value from the resulting performance object and returns it as
# a numeric value.
# The 'unlist()' function is used to convert the AUC value from a list to a numeric vector.
unlist(slot(performance(pred_obj_train_nn1, "auc"), "y.values"))
```

```
## [1] 0.7858157
```


2. Out-of-sample AUC-ROC performance (testing set is more important)


``` r
# Predicted probability for testing set
pred_test_nn1 <- as.vector(predict(default_nn1, newdata = as.data.frame(x_test)))

library(ROCR)
# This line of code creates a prediction object 'pred' using the 'ROCR::prediction()' 
pred_obj_test_nn1 <- ROCR::prediction(pred_test_nn1, credit_test$default)

perf_test_nn1 <- performance(pred_obj_test_nn1, "tpr", "fpr")

# Draw the ROC curve
plot(perf_test_nn1, colorize=TRUE)
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-16-1.png)

``` r
# This line of code calculates the Area Under the ROC Curve (AUC) using the 'performance()' function with "auc" as the argument. 
unlist(slot(performance(pred_obj_test_nn1, "auc"), "y.values"))
```

```
## [1] 0.7571715
```


# Model evaluation summary

The logistic specification was retained from the original project rather than
selected using test performance. A threshold of 0.5 is illustrative and was not
tuned on the test set. Brier score measures squared probability error (lower is
better); the calibration table compares average predictions with observed rates.


``` r
auc_value <- function(p, y) as.numeric(performance(ROCR::prediction(p, y), "auc")@y.values[[1]])
metrics <- function(p, y) {
  stopifnot(length(p) == length(y), all(is.finite(p)), all(p >= 0 & p <= 1))
  labels <- as.integer(p >= 0.5)
  tp <- sum(labels == 1 & y == 1); fp <- sum(labels == 1 & y == 0)
  fn <- sum(labels == 0 & y == 1)
  c(AUC = auc_value(p, y), Accuracy = mean(labels == y),
    Recall = tp / (tp + fn), Precision = if (tp + fp == 0) NA_real_ else tp / (tp + fp),
    Brier = mean((p - y)^2))
}
results <- rbind(
  Logistic_training = metrics(pred_train_logit, credit_train$default),
  Logistic_testing = metrics(pred_test_logit, credit_test$default),
  Neural_network_training = metrics(pred_train_nn1, credit_train$default),
  Neural_network_testing = metrics(pred_test_nn1, credit_test$default))
knitr::kable(results, digits = 4)
```



|                        |    AUC| Accuracy| Recall| Precision|  Brier|
|:-----------------------|------:|--------:|------:|---------:|------:|
|Logistic_training       | 0.7366|   0.8167| 0.2600|    0.7302| 0.1412|
|Logistic_testing        | 0.7120|   0.8108| 0.2481|    0.6926| 0.1459|
|Neural_network_training | 0.7858|   0.8331| 0.4099|    0.7056| 0.1291|
|Neural_network_testing  | 0.7572|   0.8114| 0.3582|    0.6220| 0.1402|

``` r
calibration <- function(p, y) {
  bins <- cut(p, breaks = seq(0, 1, 0.1), include.lowest = TRUE)
  aggregate(data.frame(Predicted = p, Observed = y), list(Probability_bin = bins), mean)
}
calibration(pred_test_logit, credit_test$default)
```

```
##    Probability_bin  Predicted  Observed
## 1          [0,0.1] 0.06026492 0.1153324
## 2        (0.1,0.2] 0.15045725 0.1386986
## 3        (0.2,0.3] 0.23674339 0.1756885
## 4        (0.3,0.4] 0.33953454 0.3566879
## 5        (0.4,0.5] 0.44921918 0.5247525
## 6        (0.5,0.6] 0.54514662 0.6524390
## 7        (0.6,0.7] 0.63947174 0.7468354
## 8        (0.7,0.8] 0.74840669 0.8636364
## 9        (0.8,0.9] 0.85352847 0.6153846
## 10         (0.9,1] 0.95589655 0.6000000
```

``` r
calibration(pred_test_nn1, credit_test$default)
```

```
##    Probability_bin  Predicted   Observed
## 1          [0,0.1] 0.07119408 0.09495102
## 2        (0.1,0.2] 0.14390830 0.13560976
## 3        (0.2,0.3] 0.25533766 0.27663230
## 4        (0.3,0.4] 0.34215835 0.34042553
## 5        (0.4,0.5] 0.44493199 0.47142857
## 6        (0.5,0.6] 0.54580778 0.40384615
## 7        (0.6,0.7] 0.66317410 0.59585492
## 8        (0.7,0.8] 0.74456630 0.66666667
## 9        (0.8,0.9] 0.84269947 0.78571429
## 10         (0.9,1] 0.91536737 0.75000000
```

## Logistic stability within the training data

Five stratified folds assess the retained logistic specification without using
the held-out test set. Scaling is fitted separately within each fold. This is
not a cross-validation comparison of the two model families; the neural network
is evaluated using a single seeded holdout run.


``` r
set.seed(20260917)
fold <- integer(nrow(train_raw))
for (value in 0:1) {
  idx <- which(train_raw$default == value)
  fold[idx] <- sample(rep(1:5, length.out = length(idx)))
}
cv_auc <- vapply(1:5, function(k) {
  fold_train <- train_raw[fold != k, ]; fold_valid <- train_raw[fold == k, ]
  fold_scaler <- fit_scaler(fold_train)
  fold_train <- apply_scaler(fold_train, fold_scaler)
  fold_valid <- apply_scaler(fold_valid, fold_scaler)
  fit <- glm(default ~ . + I(PAY_AMT1^2), data = fold_train, family = binomial())
  auc_value(predict(fit, fold_valid, type = "response"), fold_valid$default)
}, numeric(1))
data.frame(Fold = 1:5, AUC = cv_auc)
```

```
##   Fold       AUC
## 1    1 0.7543191
## 2    2 0.7014612
## 3    3 0.7391304
## 4    4 0.6980110
## 5    5 0.7566556
```

# Conclusion

On the held-out test set, logistic regression achieved AUC
0.7120, and the neural network
achieved AUC 0.7572.
These values are generated from the current run. Higher AUC indicates better
ranking of defaulting versus non-defaulting customers on this split; it does
not demonstrate why one model performs better or establish consistent superiority.

The mean five-fold training-validation AUC for the logistic specification was
0.7299 (fold standard deviation
0.0284).

## Business interpretation and limitations

The models provide a starting point for studying default-risk scoring. This
analysis does not establish which predictors are strongest or demonstrate that
behavioral variables outperform demographics. Operational use would require
validation on representative current customers, threshold selection based on
error costs, and further calibration assessment.

The network uses one initialization and a small architecture without systematic
tuning. Its holdout result is not a repeated cross-validation estimate. The
historical sample may not generalize to other institutions or periods, and
income, employment, and broader economic conditions are not included.
Repayment codes are modeled numerically, an assumption worth testing separately.

## Reproducibility


``` r
dir.create("results", showWarnings = FALSE)
write.csv(data.frame(Model = rownames(results), results, row.names = NULL),
          "results/model_metrics.csv", row.names = FALSE)
write.csv(data.frame(Fold = 1:5, AUC = cv_auc), "results/logistic_cv.csv", row.names = FALSE)
write.csv(data.frame(Row = seq_len(nrow(credit_default)),
                     Partition = ifelse(seq_len(nrow(credit_default)) %in% sample_index, "Training", "Testing")),
          "results/data_split.csv", row.names = FALSE)
writeLines(capture.output(sessionInfo()), "results/session_info.txt")
sessionInfo()
```

```
## R version 4.4.2 (2024-10-31)
## Platform: aarch64-apple-darwin20
## Running under: macOS 26.2
## 
## Matrix products: default
## BLAS:   /Library/Frameworks/R.framework/Versions/4.4-arm64/Resources/lib/libRblas.0.dylib 
## LAPACK: /Library/Frameworks/R.framework/Versions/4.4-arm64/Resources/lib/libRlapack.dylib;  LAPACK version 3.12.0
## 
## locale:
## [1] C.UTF-8/C.UTF-8/C.UTF-8/C/C.UTF-8/C.UTF-8
## 
## time zone: America/New_York
## tzcode source: internal
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] neuralnet_1.44.2 ROCR_1.0-11      dplyr_1.1.4     
## 
## loaded via a namespace (and not attached):
##  [1] R6_2.5.1         xfun_0.50        tidyselect_1.2.1 magrittr_2.0.3  
##  [5] glue_1.8.0       tibble_3.2.1     knitr_1.49       pkgconfig_2.0.3 
##  [9] generics_0.1.3   lifecycle_1.0.4  cli_3.6.5        grid_4.4.2      
## [13] vctrs_0.6.5      withr_3.0.2      compiler_4.4.2   tools_4.4.2     
## [17] evaluate_1.0.3   pillar_1.10.1    rlang_1.1.5
```
