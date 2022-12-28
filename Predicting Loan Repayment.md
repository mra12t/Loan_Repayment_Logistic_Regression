Predicting Loan Repayment
================
2022-12-19

### About the Study and the Data Set

In the lending industry, investors loan money to borrowers with the
expectation of receiving repayment with interest. If the borrower repays
the loan as agreed, the lender earns a profit from the interest.
However, if the borrower is unable to repay the loan, the lender incurs
a loss. To mitigate this risk, lenders need to be able to predict the
likelihood of a borrower being unable to repay a loan. To help address
this issue, we will use a dataset from LendingClub.com, a platform that
connects borrowers and investors online. The dataset consists of 9,578
3-year loans made through LendingClub.com from May 2007 to February
2010. The dependent variable “not.fully.paid” indicates whether the loan
was not fully repaid, either because the borrower defaulted or the loan
was charged off due to the borrower being unlikely to repay it.

The data set contains the following variables:

- credit.policy: A binary variable indicating whether the customer meets
  the credit underwriting criteria of LendingClub.com (1) or not (0).

- purpose: The reason for taking out the loan, with categories including
  “credit_card”, “debt_consolidation”, “educational”, “major_purchase”,
  “small_business”, and “all_other”.

- int.rate: The interest rate of the loan as a decimal (e.g. a rate of
  11% would be stored as 0.11). LendingClub.com tends to assign higher
  interest rates to borrowers considered to be more risky.  

- installment: The monthly payment amount owed by the borrower if the
  loan is funded.  

- log.annual.inc: The natural log of the borrower’s self-reported annual
  income.  

- dti: The debt-to-income ratio of the borrower, calculated as the
  amount of debt divided by annual income.  

- fico: The FICO credit score of the borrower.  

- days.with.cr.line: The number of days the borrower has had a credit
  line.  

- revol.bal: The borrower’s revolving balance, or the amount unpaid at
  the end of the credit card billing cycle.  

- revol.util: The borrower’s utilization rate of their credit line, or
  the amount of the credit line used relative to the total credit
  available.  

- inq.last.6mths: The number of inquiries by creditors made to the
  borrower in the last 6 months.  

- delinq.2yrs: The number of times the borrower has been 30+ days past
  due on a payment in the past 2 years.  

- pub.rec: The number of derogatory public records the borrower has,
  such as bankruptcy filings, tax liens, or judgments.

### EDA

``` r
loans = read.csv("loans.csv")
str(loans)
```

    ## 'data.frame':    9578 obs. of  14 variables:
    ##  $ credit.policy    : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ purpose          : chr  "debt_consolidation" "credit_card" "debt_consolidation" "debt_consolidation" ...
    ##  $ int.rate         : num  0.119 0.107 0.136 0.101 0.143 ...
    ##  $ installment      : num  829 228 367 162 103 ...
    ##  $ log.annual.inc   : num  11.4 11.1 10.4 11.4 11.3 ...
    ##  $ dti              : num  19.5 14.3 11.6 8.1 15 ...
    ##  $ fico             : int  737 707 682 712 667 727 667 722 682 707 ...
    ##  $ days.with.cr.line: num  5640 2760 4710 2700 4066 ...
    ##  $ revol.bal        : int  28854 33623 3511 33667 4740 50807 3839 24220 69909 5630 ...
    ##  $ revol.util       : num  52.1 76.7 25.6 73.2 39.5 51 76.8 68.6 51.1 23 ...
    ##  $ inq.last.6mths   : int  0 0 1 1 0 0 0 0 1 1 ...
    ##  $ delinq.2yrs      : int  0 0 0 0 1 0 0 0 0 0 ...
    ##  $ pub.rec          : int  0 0 0 0 0 0 1 0 0 0 ...
    ##  $ not.fully.paid   : int  0 0 0 0 0 0 1 1 0 0 ...

``` r
library(knitr)
kable(table(loans$not.fully.paid))
```

| Var1 | Freq |
|:-----|-----:|
| 0    | 8045 |
| 1    | 1533 |

``` r
barplot(table(loans$not.fully.paid))
```

![](Predicting%20Loan%20Repayment_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

Now that we have an idea about the distribution of the loan repayment
variable, let’s examine the missing data.

``` r
kable(summary(loans))
```

|     | credit.policy | purpose          | int.rate       | installment    | log.annual.inc | dti            | fico          | days.with.cr.line | revol.bal      | revol.util     | inq.last.6mths | delinq.2yrs     | pub.rec        | not.fully.paid |
|:----|:--------------|:-----------------|:---------------|:---------------|:---------------|:---------------|:--------------|:------------------|:---------------|:---------------|:---------------|:----------------|:---------------|:---------------|
|     | Min. :0.000   | Length:9578      | Min. :0.0600   | Min. : 15.67   | Min. : 7.548   | Min. : 0.000   | Min. :612.0   | Min. : 179        | Min. : 0       | Min. : 0.00    | Min. : 0.000   | Min. : 0.0000   | Min. :0.0000   | Min. :0.0000   |
|     | 1st Qu.:1.000 | Class :character | 1st Qu.:0.1039 | 1st Qu.:163.77 | 1st Qu.:10.558 | 1st Qu.: 7.213 | 1st Qu.:682.0 | 1st Qu.: 2820     | 1st Qu.: 3187  | 1st Qu.: 22.70 | 1st Qu.: 0.000 | 1st Qu.: 0.0000 | 1st Qu.:0.0000 | 1st Qu.:0.0000 |
|     | Median :1.000 | Mode :character  | Median :0.1221 | Median :268.95 | Median :10.928 | Median :12.665 | Median :707.0 | Median : 4140     | Median : 8596  | Median : 46.40 | Median : 1.000 | Median : 0.0000 | Median :0.0000 | Median :0.0000 |
|     | Mean :0.805   | NA               | Mean :0.1226   | Mean :319.09   | Mean :10.932   | Mean :12.607   | Mean :710.8   | Mean : 4562       | Mean : 16914   | Mean : 46.87   | Mean : 1.572   | Mean : 0.1638   | Mean :0.0621   | Mean :0.1601   |
|     | 3rd Qu.:1.000 | NA               | 3rd Qu.:0.1407 | 3rd Qu.:432.76 | 3rd Qu.:11.290 | 3rd Qu.:17.950 | 3rd Qu.:737.0 | 3rd Qu.: 5730     | 3rd Qu.: 18250 | 3rd Qu.: 71.00 | 3rd Qu.: 2.000 | 3rd Qu.: 0.0000 | 3rd Qu.:0.0000 | 3rd Qu.:0.0000 |
|     | Max. :1.000   | NA               | Max. :0.2164   | Max. :940.14   | Max. :14.528   | Max. :29.960   | Max. :827.0   | Max. :17640       | Max. :1207359  | Max. :119.00   | Max. :33.000   | Max. :13.0000   | Max. :5.0000   | Max. :1.0000   |
|     | NA            | NA               | NA             | NA             | NA’s :4        | NA             | NA            | NA’s :29          | NA             | NA’s :62       | NA’s :29       | NA’s :29        | NA’s :29       | NA             |

We can see that, the variables with missing data are log.annual.inc,
days.with.cr.line, revol.util, inq.last.6mths, delinq.2yrs, and pub.rec.

Lets further investigate the missing records to decide what to do,

``` r
missing = subset(loans, is.na(log.annual.inc) | is.na(days.with.cr.line) | is.na(revol.util) | is.na(inq.last.6mths) | is.na(delinq.2yrs) | is.na(pub.rec))
nrow(missing)
```

    ## [1] 62

``` r
kable(table(missing$not.fully.paid))
```

| Var1 | Freq |
|:-----|-----:|
| 0    |   50 |
| 1    |   12 |

Although the number of the missing data is only 62 records, we would
want to fill the missing value’s instead of ignoring these record so
that the model could perhaps cover the cases where some value’s does not
exist.

Let’s impute the missing data.

``` r
library(mice)
```

    ## 
    ## Attaching package: 'mice'

    ## The following object is masked from 'package:stats':
    ## 
    ##     filter

    ## The following objects are masked from 'package:base':
    ## 
    ##     cbind, rbind

``` r
#Setting Seeds so that the result is reproducible
set.seed(123)
vars_for_imputation = setdiff(names(loans), "not.fully.paid")
imputed = complete(mice(loans[vars_for_imputation]))
```

    ## 
    ##  iter imp variable
    ##   1   1  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   1   2  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   1   3  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   1   4  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   1   5  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   2   1  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   2   2  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   2   3  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   2   4  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   2   5  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   3   1  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   3   2  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   3   3  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   3   4  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   3   5  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   4   1  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   4   2  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   4   3  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   4   4  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   4   5  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   5   1  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   5   2  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   5   3  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   5   4  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec
    ##   5   5  log.annual.inc  days.with.cr.line  revol.util  inq.last.6mths  delinq.2yrs  pub.rec

    ## Warning: Number of logged events: 1

``` r
loans[vars_for_imputation] = imputed
```

``` r
kable(summary(loans))
```

|     | credit.policy | purpose          | int.rate       | installment    | log.annual.inc | dti            | fico          | days.with.cr.line | revol.bal      | revol.util    | inq.last.6mths | delinq.2yrs     | pub.rec         | not.fully.paid |
|:----|:--------------|:-----------------|:---------------|:---------------|:---------------|:---------------|:--------------|:------------------|:---------------|:--------------|:---------------|:----------------|:----------------|:---------------|
|     | Min. :0.000   | Length:9578      | Min. :0.0600   | Min. : 15.67   | Min. : 7.548   | Min. : 0.000   | Min. :612.0   | Min. : 179        | Min. : 0       | Min. : 0.0    | Min. : 0.000   | Min. : 0.0000   | Min. :0.00000   | Min. :0.0000   |
|     | 1st Qu.:1.000 | Class :character | 1st Qu.:0.1039 | 1st Qu.:163.77 | 1st Qu.:10.558 | 1st Qu.: 7.213 | 1st Qu.:682.0 | 1st Qu.: 2820     | 1st Qu.: 3187  | 1st Qu.: 22.6 | 1st Qu.: 0.000 | 1st Qu.: 0.0000 | 1st Qu.:0.00000 | 1st Qu.:0.0000 |
|     | Median :1.000 | Mode :character  | Median :0.1221 | Median :268.95 | Median :10.929 | Median :12.665 | Median :707.0 | Median : 4140     | Median : 8596  | Median : 46.2 | Median : 1.000 | Median : 0.0000 | Median :0.00000 | Median :0.0000 |
|     | Mean :0.805   | NA               | Mean :0.1226   | Mean :319.09   | Mean :10.932   | Mean :12.607   | Mean :710.8   | Mean : 4562       | Mean : 16914   | Mean : 46.8   | Mean : 1.581   | Mean : 0.1639   | Mean :0.06223   | Mean :0.1601   |
|     | 3rd Qu.:1.000 | NA               | 3rd Qu.:0.1407 | 3rd Qu.:432.76 | 3rd Qu.:11.290 | 3rd Qu.:17.950 | 3rd Qu.:737.0 | 3rd Qu.: 5730     | 3rd Qu.: 18250 | 3rd Qu.: 70.9 | 3rd Qu.: 2.000 | 3rd Qu.: 0.0000 | 3rd Qu.:0.00000 | 3rd Qu.:0.0000 |
|     | Max. :1.000   | NA               | Max. :0.2164   | Max. :940.14   | Max. :14.528   | Max. :29.960   | Max. :827.0   | Max. :17640       | Max. :1207359  | Max. :119.0   | Max. :33.000   | Max. :13.0000   | Max. :5.00000   | Max. :1.0000   |

### Building Logistic Regression Model

``` r
library(caTools)
set.seed(123)
spl = sample.split(loans$not.fully.paid, 0.7)
train = subset(loans, spl == TRUE)
test = subset(loans, spl == FALSE)
model = glm(not.fully.paid ~., data = train, family = binomial)
summary(model)
```

    ## 
    ## Call:
    ## glm(formula = not.fully.paid ~ ., family = binomial, data = train)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -1.9700  -0.6203  -0.4936  -0.3554   2.6537  
    ## 
    ## Coefficients:
    ##                             Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                9.282e+00  1.546e+00   6.004 1.93e-09 ***
    ## credit.policy             -3.168e-01  1.010e-01  -3.137 0.001707 ** 
    ## purposecredit_card        -6.643e-01  1.328e-01  -5.004 5.63e-07 ***
    ## purposedebt_consolidation -4.250e-01  9.232e-02  -4.603 4.15e-06 ***
    ## purposeeducational         1.202e-02  1.785e-01   0.067 0.946293    
    ## purposehome_improvement   -3.739e-02  1.517e-01  -0.246 0.805357    
    ## purposemajor_purchase     -3.182e-01  1.908e-01  -1.668 0.095399 .  
    ## purposesmall_business      5.374e-01  1.387e-01   3.875 0.000107 ***
    ## int.rate                   1.533e+00  2.062e+00   0.743 0.457189    
    ## installment                1.189e-03  2.108e-04   5.642 1.68e-08 ***
    ## log.annual.inc            -4.277e-01  7.152e-02  -5.980 2.23e-09 ***
    ## dti                        7.222e-03  5.455e-03   1.324 0.185519    
    ## fico                      -9.832e-03  1.715e-03  -5.734 9.83e-09 ***
    ## days.with.cr.line          2.716e-05  1.577e-05   1.723 0.084970 .  
    ## revol.bal                  2.141e-06  1.134e-06   1.889 0.058931 .  
    ## revol.util                 2.674e-03  1.538e-03   1.738 0.082127 .  
    ## inq.last.6mths             7.794e-02  1.619e-02   4.815 1.47e-06 ***
    ## delinq.2yrs               -6.192e-02  6.517e-02  -0.950 0.342053    
    ## pub.rec                    2.255e-01  1.133e-01   1.991 0.046495 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 5896.6  on 6704  degrees of freedom
    ## Residual deviance: 5482.6  on 6686  degrees of freedom
    ## AIC: 5520.6
    ## 
    ## Number of Fisher Scoring iterations: 5

We can see that there are 10 significant variables in our model.

### Applying the Model

``` r
test$predicted = predict(model, newdata = test, type = "response")
x = table(test$not.fully.paid, test$predicted >= 0.5)
kable(x)
```

|     | FALSE | TRUE |
|:----|------:|-----:|
| 0   |  2401 |   12 |
| 1   |   447 |   13 |

Now, let’s calculate the accuracy of the model.

``` r
sum(diag(x))/sum(x)
```

    ## [1] 0.8402367

Let’s see what a base line model accuracy will look like.

``` r
x = table(test$not.fully.paid)
kable(x)
```

| Var1 | Freq |
|:-----|-----:|
| 0    | 2413 |
| 1    |  460 |

A simple base line model that assumes a loan will be fully paid all the
time (not.fully.paid = 0) will have an accuracy of:

``` r
x[1]/sum(x)
```

    ##         0 
    ## 0.8398886

Lets calculate the AUC of the Model

``` r
library(ROCR)
pred = prediction(test$predicted, test$not.fully.paid)
as.numeric(performance(pred, "auc")@y.values)
```

    ## [1] 0.6699814

### A Possibly Better Base Line Model

LendingClub.com uses the interest rate of a loan as an indicator of the
risk associated with that loan. The interest rate is an independent
variable in our dataset, and we will use it to compare the risk of
different loans and determine whether it can be used as a baseline for
ranking loans according to their risk.

``` r
Model1 = glm(not.fully.paid ~ int.rate, family = binomial, data = train)
summary(Model1)
```

    ## 
    ## Call:
    ## glm(formula = not.fully.paid ~ int.rate, family = binomial, data = train)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -1.0834  -0.6284  -0.5386  -0.4213   2.3118  
    ## 
    ## Coefficients:
    ##             Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)  -3.7833     0.1689  -22.39   <2e-16 ***
    ## int.rate     16.7754     1.2679   13.23   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 5896.6  on 6704  degrees of freedom
    ## Residual deviance: 5715.6  on 6703  degrees of freedom
    ## AIC: 5719.6
    ## 
    ## Number of Fisher Scoring iterations: 4

As we can see, the variable int.rate is really significant in the new
model, yet it is not significant in a model that contains many
variables. This is common when the independent variables are highly
correlated, so let’s investigate this hypothesis and see.

``` r
cor(train$fico, train$int.rate)
```

    ## [1] -0.7103988

We can see that there’s a strong linear relation between the credit
score and the interest rate, the reason it is negative is because
,logically, lower credit score leads to higher interest rate.

``` r
predModel1 = predict(Model1, newdata = test, type = "response")
summary(predModel1)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ## 0.05859 0.11504 0.14995 0.15989 0.19029 0.44394

We can see that the maximum prediction is 0.44 meaning a 0.5 threshold
will lead to all the not.fully.paid being 0, meaning all loans will be
re-payed.

Let’s see the AUC of this Model

``` r
pred1 = prediction(predModel1, test$not.fully.paid)
as.numeric(performance(pred1, "auc")@y.values)
```

    ## [1] 0.6063582

So given that this model has a not so different AUC from the previous
one, while being way simpler given that it have only one independent
variable, it can be considered a better base line model.

### Different Investment Stratgies

The goal of an investor is to find loans that are likely to be
profitable, meaning that they will be paid back in full and the investor
will earn interest on the loan. However, there is also a risk that the
loan will not be paid back, in which case the investor will lose the
money invested. Therefore, the investor should seek loans that offer a
good balance between risk and reward, meaning they have a reasonable
chance of being paid back while still offering a potentially high return
on investment.

First, let’s establish some ideas:

- $c*exp(rt)$ is the interest revenue given that $c$ is the initial
  investment, $r$ is the interest rate, and $t$ is the time.

- $c*exp(rt)-c$ is the revenue of the said investment after deducting
  the initial investment capital given that the loan has been fully
  repaid.

- $-c$ is the maximum loss an investor could take in case the loan
  wasn’t repaid at all.

***To evaluate the effectiveness of an investment strategy, we need to
calculate the potential profit for each loan in the test set. We will
assume that the investor has invested \$1 (c=1) in each loan. To
calculate the profit for a fully paid loan, we will use the formula
exp(rt)-1, where r is the interest rate and t is the length of the loan
in years. In this case, all of the loans in the dataset are 3-year
loans, so t=3 in our calculations. We will assign this value to every
observation and then replace it with -1 for any loans that were not paid
in full. This will allow us to compare the potential profits for
different loans and evaluate the quality of the investment strategy.***

``` r
test$profit = exp(test$int.rate*3)-1
test$profit[test$not.fully.paid == 1] = -1 
```

``` r
summary(test$profit)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ## -1.0000  0.2940  0.4154  0.2110  0.4980  0.8895

An investment strategy of equally investing in all loans would yield a
profit of \$20.94 for a \$100 investment, but this approach does not
take advantage of the prediction model that was developed earlier.
Investors generally want to find loans that offer a good balance between
reward (high interest rates) and risk (low likelihood of not being paid
back).

To meet this goal, we will examine an investment strategy that involves
only investing in loans with high interest rates (at least 15%) and
selecting the ones with the lowest predicted risk of not being fully
paid back. We will model an investor who invests \$1 in each of the 100
most promising loans. This strategy aims to maximize the potential
return on investment while minimizing the risk of loss.

``` r
highinterest = subset(test, int.rate >= 0.15)
mean(highinterest$profit)
```

    ## [1] 0.2228367

``` r
x = table(highinterest$not.fully.paid)
kable(x)
```

| Var1 | Freq |
|:-----|-----:|
| 0    |  306 |
| 1    |  104 |

``` r
x[2]/sum(x)
```

    ##         1 
    ## 0.2536585

We can see that almost 25% of the loans with high interest rate didn’t
get paid in full, and the mean of profit for the whole is 22%.

To identify the 100 loans with the lowest predicted risk of not being
paid back in full, we will first sort the predicted risks in increasing
order. Then, we will select the 100th element of this sorted list, which
will be the 100th smallest predicted probability of not paying in full.
This will allow us to identify the 100 loans with the lowest predicted
risk and consider them for investment.

``` r
cutmark = sort(highinterest$predicted, decreasing = FALSE)[100]
selected_investments = subset(highinterest, predicted <= cutmark)
```

Now let’s see the result of the strategy

``` r
nrow(selected_investments)
```

    ## [1] 100

``` r
mean(selected_investments$profit)
```

    ## [1] 0.3890197

``` r
x = table(selected_investments$not.fully.paid)
kable(x)
```

| Var1 | Freq |
|:-----|-----:|
| 0    |   86 |
| 1    |   14 |

``` r
x[2]/sum(x)
```

    ##    1 
    ## 0.14

***As we can see, even though the model we have built is simple, coupled
with an investment strategy, it became quite effective given us a profit
rate of almost 39% and a loan repayment rate of 86%. This is a great
improvement from 22% and 75%, respectively.***
