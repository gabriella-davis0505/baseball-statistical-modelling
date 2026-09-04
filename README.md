# Baseball Statistical Modelling in R

Statistical modelling project using **R** and historical baseball data from the `Lahman` package.

The project applies linear regression, logistic regression and Poisson regression to investigate managerial and pitching performance, with additional model diagnostics, prediction and classification evaluation.

---

## Project Overview

This project was completed as part of an MSc Data Science Statistics module.

The analysis explores several statistical modelling problems using historical Major League Baseball data.

The coursework covers three main areas:

**Linear Regression → Logistic Regression → Poisson and Mixed-Effects Modelling**

The project demonstrates the complete statistical modelling workflow, including:

- data preparation and joining;
- exploratory visualisation;
- model fitting;
- coefficient interpretation;
- hypothesis testing;
- confidence intervals;
- regression diagnostics;
- prediction;
- train-test evaluation;
- ROC analysis;
- classification thresholds;
- confusion matrices; and
- mixed-effects modelling.

---

## Dataset

The analysis uses datasets provided by the **Lahman R package**, which contains historical baseball statistics.

The main datasets used are:

- `Managers`
- `Teams`
- `AwardsShareManagers`
- `Pitching`
- `People`

Because the data is provided directly through the `Lahman` package, no separate raw dataset files are required in this repository.

---

# 1. Linear Regression

The first section investigates factors associated with the proportion of award voting points received by baseball managers.

## Data Preparation

A manager-level dataset is constructed from `Managers`, including:

- player ID;
- team ID;
- year;
- league;
- player-manager status; and
- managerial win percentage.

Manager records are then combined with information from:

- `Teams`; and
- `AwardsShareManagers`.

A transformed response variable is created as:

```text
sqr_point_pct = sqrt(pointsWon / pointsMax)
```

Incomplete observations are removed before modelling.

---

## Linear Regression Model

A Gaussian linear regression model is fitted using:

```r
sqr_point_pct ~ win_pct + DivWin + CS
```

where:

- `win_pct` represents managerial winning percentage;
- `DivWin` indicates whether the team won its division; and
- `CS` represents championship-series information used in the coursework data.

The fitted coursework model was approximately:

```text
sqr_point_pct =
-0.73
+ 1.8 × win_pct
+ 0.14 × DivWin
+ 0.0027 × CS
```

The model was statistically significant overall and explained approximately **27% of the variation** in the transformed award-point percentage.

The analysis found positive associations between the response and:

- winning percentage;
- division-winning status; and
- the `CS` variable.

---

## Linear Model Diagnostics

Regression assumptions are investigated using several diagnostic plots, including:

- residual distributions;
- residuals versus predictors;
- residuals versus fitted values;
- Normal Q-Q plots; and
- scale-location plots.

The diagnostics are used to assess:

- normality of residuals;
- linearity;
- homoscedasticity;
- possible outliers; and
- potential model misspecification.

The coursework identified generally reasonable residual behaviour while also noting some evidence that residual variance may not be completely constant.

---

## Prediction

The model is also used for prediction.

For a manager with:

```text
win_pct = 0.8
DivWin = Yes
CS = 8
```

the fitted model produced an expected transformed award-point value of approximately:

```text
0.88
```

Confidence intervals are additionally calculated for each regression coefficient to assess uncertainty in the parameter estimates.

---

# 2. Logistic Regression

The second section investigates how the prevalence of **player-managers** has changed over time.

A player-manager is an individual who simultaneously held both playing and managerial responsibilities.

---

## Exploratory Analysis

Player-manager status is plotted against year using jittered observations.

The visual analysis shows a strong historical decline in player-managers, with the role becoming increasingly uncommon over time.

---

## Logistic Regression Model

A logistic regression model is fitted using:

```r
plyrMgr ~ yearID
```

The fitted coursework model was approximately:

```text
logit(P(player-manager)) =
89 - 0.047 × year
```

The negative coefficient for year indicates that the probability of a manager also being a player decreases as time progresses.

---

## Train-Test Evaluation

To evaluate model generalisation, the data is divided into:

```text
80% training data
20% testing data
```

using:

```r
set.seed(123)
```

ROC curves are generated for both datasets.

The coursework produced approximately:

| Dataset | ROC AUC |
|---|---:|
| Training | 0.905 |
| Testing | 0.900 |

The very similar training and testing AUC values suggest that the model generalises well and shows little evidence of substantial overfitting.

---

## Classification Threshold

An optimal classification threshold is estimated using **Youden's Index**.

The threshold is then used to construct confusion matrices for both the training and testing data.

The analysis examines:

- sensitivity;
- specificity;
- true positives;
- true negatives;
- false positives;
- false negatives; and
- overall classification performance.

The coursework reported high sensitivity, indicating that the model was effective at detecting historical player-managers, although specificity was lower.

---

## Model Comparison

A second logistic model adds managerial winning percentage:

```r
plyrMgr ~ year + win_pct
```

The additional `win_pct` variable did not provide meaningful improvement to the model and was not statistically significant.

The simpler year-only model was therefore preferred based on:

- similar model fit;
- lower complexity; and
- slightly better AIC.

This demonstrates the use of **model parsimony** when comparing alternative statistical models.

---

# 3. Poisson Regression

The third section investigates the number of **shutouts achieved by baseball pitchers**.

Because shutouts are non-negative count data, Poisson regression provides a natural modelling framework.

---

## Pitcher Dataset

A new dataset is constructed from `Pitching`.

The preprocessing includes:

- retaining pitchers who faced at least one batter;
- calculating innings pitched from recorded outs;
- joining player characteristics from `People`; and
- removing incomplete observations.

Additional player characteristics include:

- weight;
- height; and
- throwing hand.

---

## Exploratory Analysis

The distribution of shutouts is first explored using a histogram.

The data is strongly right-skewed and consists of non-negative integer counts, supporting the use of a count-based modelling approach.

A second visualisation examines shutouts against innings pitched while distinguishing between left- and right-handed pitchers.

The plot indicates a strong relationship between pitching exposure and the number of shutouts recorded.

---

## Multiple Poisson Regression

The principal Poisson model is:

```r
SH ~ innings + weight + height + throws
```

using a logarithmic link function.

The model investigates whether shutout counts are associated with:

- innings pitched;
- player weight;
- player height; and
- throwing hand.

Analysis of deviance is used to assess the contribution of the predictors.

The coursework identifies innings pitched as a particularly important predictor of shutout counts.

---

## Mixed-Effects Poisson Model

The analysis is extended using a **generalised linear mixed-effects model**.

`teamID` is incorporated as a random effect to investigate whether pitchers belonging to different teams exhibit additional variation beyond the individual-level predictors.

Conceptually, the model takes the form:

```r
SH ~ innings + weight + height + throws + (1 | teamID)
```

This allows the intercept to vary between teams.

The coursework found that the estimated team-level variance was relatively small compared with the effects associated with individual pitcher characteristics.

---

## Poisson Model Diagnostics

A scale-location diagnostic is produced for the Poisson model to examine residual behaviour.

The analysis considers:

- residual spread;
- possible heteroscedasticity;
- clustering;
- potential outliers; and
- evidence of model misspecification.

---

## Interpreting Poisson Effects

The fitted model is also used to compare pitchers while holding other characteristics constant.

The coursework analysis estimated that left-handed pitchers produced approximately:

```text
1.1 ×
```

as many shutouts as right-handed pitchers under the model.

Coefficient signs were also used to investigate the relationship between shutouts and:

- height; and
- weight.

This demonstrates how coefficients from a log-link Poisson regression can be translated into interpretable comparisons.

---

## Statistical Techniques Demonstrated

This project demonstrates experience with:

- linear regression;
- logistic regression;
- Poisson regression;
- generalised linear models;
- generalised linear mixed models;
- random effects;
- data transformation;
- model diagnostics;
- confidence intervals;
- hypothesis testing;
- analysis of variance and deviance;
- train-test splitting;
- ROC curves;
- ROC AUC;
- Youden's Index;
- confusion matrices;
- sensitivity and specificity;
- prediction;
- model comparison;
- AIC;
- residual analysis; and
- statistical visualisation.

---

## R Packages

The analysis uses packages including:

```text
broom
caret
car
dplyr
ggplot2
gridExtra
janitor
Lahman
lindia
lme4
magrittr
pROC
readxl
sandwich
tidyverse
viridis
```

---

## Repository Structure

```text
baseball-statistical-modelling/
│
├── README.md
├── .gitignore
│
└── rmd file statistics .Rmd
```

---

## How to Run the Project

The analysis is contained in:

```text
rmd file statistics .Rmd
```

Open the file in **RStudio** and ensure the required packages are installed.

For example:

```r
install.packages(c(
  "broom",
  "caret",
  "car",
  "dplyr",
  "ggplot2",
  "gridExtra",
  "janitor",
  "Lahman",
  "lindia",
  "lme4",
  "magrittr",
  "pROC",
  "readxl",
  "sandwich",
  "tidyverse",
  "viridis"
))
```

The R Markdown document can then be run interactively or rendered to produce the complete analysis and figures.

No external data download is required because the baseball datasets are supplied by the `Lahman` package.

---

## Skills Demonstrated

This project demonstrates experience in:

- R programming;
- statistical modelling;
- data manipulation;
- exploratory data analysis;
- regression analysis;
- classification modelling;
- count-data modelling;
- mixed-effects modelling;
- statistical inference;
- predictive model evaluation;
- model diagnostics;
- data visualisation;
- interpreting statistical results; and
- communicating quantitative findings.

---

## Academic Context

This repository contains work completed as part of an **MSc Data Science Statistics module**.

The project has been included in my data science portfolio to demonstrate statistical modelling, R programming, model evaluation and quantitative analysis skills.

---

## Author

**Gabriella Davis**

MSc Data Science

GitHub: `gabriella-davis0505`
