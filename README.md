# Baseball Statistical Modelling in R

Statistical modelling project using **R** and historical baseball data from the `Lahman` package.

The project applies linear regression, logistic regression, Poisson regression and mixed-effects modelling, alongside model diagnostics, prediction and classification evaluation.

---

## Project Overview

This project was completed as part of an **MSc Data Science Statistics module**.

The coursework explores three statistical modelling problems using historical Major League Baseball data:

**Linear Regression → Logistic Regression → Poisson and Mixed-Effects Modelling**

The analysis demonstrates a broad statistical workflow including:

- data preparation and joining;
- exploratory data analysis;
- regression modelling;
- statistical inference;
- confidence intervals;
- model diagnostics;
- prediction;
- train-test evaluation;
- ROC analysis;
- classification thresholds;
- confusion matrices;
- model comparison; and
- mixed-effects modelling.

---

## Data

The analysis uses datasets supplied through the **Lahman R package**, which contains historical baseball statistics.

The main datasets used are:

- `Managers`
- `Teams`
- `AwardsShareManagers`
- `Pitching`
- `People`

Because these datasets are available directly through the `Lahman` package, no separate raw data files are required in this repository.

---

# 1. Linear Regression

The first part of the project investigates factors associated with the proportion of award voting points received by baseball managers.

## Data Preparation

A manager-level dataset is constructed from `Managers`.

The analysis calculates managerial winning percentage as:

```r
win_pct = W / (W + L)
```

and retains:

- `playerID`
- `teamID`
- `yearID`
- `lgID`
- `plyrMgr`
- `win_pct`

Additional information is then joined from the `Teams` and `AwardsShareManagers` datasets.

A transformed response variable is created as:

```r
sqr_point_pct = sqrt(pointsWon / pointsMax)
```

Incomplete observations are removed before modelling.

---

## Linear Regression Model

A Gaussian linear regression model is fitted using:

```r
spp_mod <- lm(
  sqr_point_pct ~ win_pct + DivWin + CS,
  data = awards_man
)
```

The fitted coursework model was approximately:

```text
sqr_point_pct =
-0.73
+ 1.8 × win_pct
+ 0.14 × DivWin
+ 0.0027 × CS
```

The model was statistically significant overall and produced an R-squared of approximately:

```text
0.27
```

meaning that the included predictors explained around 27% of the observed variation in the transformed award-point response.

The coursework found statistically significant positive associations for:

- managerial winning percentage;
- division-winning status; and
- the `CS` variable included from the `Teams` dataset.

---

## Model Diagnostics

Several diagnostic plots are used to evaluate the assumptions of the fitted linear model.

These include:

- residual distributions;
- residuals against predictors;
- residuals versus fitted values;
- Normal Q-Q plots; and
- scale-location plots.

The diagnostics are used to investigate:

- normality of residuals;
- linearity;
- homoscedasticity;
- possible outliers; and
- potential model misspecification.

The coursework found that the residuals were approximately normally distributed, although the scale-location plot suggested that the assumption of constant residual variance may not be completely satisfied.

---

## Prediction

The fitted model is used to estimate the transformed award-point response for a manager with:

```text
win_pct = 0.8
DivWin = Yes
CS = 8
```

The predicted value was approximately:

```text
0.88
```

This demonstrates the use of a fitted regression model to generate predictions for new observations.

---

## Confidence Intervals

95% confidence intervals are calculated for the model coefficients using:

```r
confint(spp_mod)
```

The coursework reported approximate intervals of:

| Parameter | 95% Confidence Interval |
|---|---:|
| Intercept | -1.00 to -0.45 |
| `win_pct` | 1.31 to 2.30 |
| `DivWinY` | 0.08 to 0.19 |
| `CS` | 0.0015 to 0.0039 |

The intervals for the three predictors do not include zero, supporting the significance indicated by the fitted regression model.

---

# 2. Logistic Regression

The second part investigates how the prevalence of **player-managers** changed over time.

A player-manager is represented by the `plyrMgr` variable in the `Managers` dataset.

---

## Exploratory Analysis

Player-manager status is plotted against year using vertically jittered observations.

The visualisation shows a clear historical decline in the frequency of player-managers, with the role becoming increasingly uncommon over time.

---

## Logistic Regression Model

A logistic regression model is fitted using:

```r
logit_model <- glm(
  as.factor(plyrMgr) ~ yearID,
  family = "binomial",
  data = df_managers
)
```

The fitted coursework model was approximately:

```text
logit(P(player-manager)) =
89 - 0.047 × year
```

The negative year coefficient indicates that the probability of a manager also being a player decreases over time.

The coursework reports a substantial reduction from null deviance to residual deviance, indicating that year provides useful information for predicting player-manager status.

---

## Train-Test Evaluation

The data is split into:

```text
80% training data
20% testing data
```

using:

```r
set.seed(123)
```

The logistic model is fitted to the training data and evaluated on both the training and testing sets.

ROC curves are generated using predicted probabilities.

The coursework reported approximately:

| Dataset | ROC AUC |
|---|---:|
| Training | 0.905 |
| Testing | 0.900 |

The similar AUC values indicate that the model performed consistently on unseen data and showed little evidence of substantial overfitting.

---

## Classification Threshold

An optimal probability threshold is selected using **Youden's Index**.

The coursework obtained a threshold of approximately:

```text
0.112
```

with:

```text
Sensitivity ≈ 0.956
Specificity ≈ 0.753
```

The model therefore demonstrated particularly strong sensitivity for identifying player-managers, although specificity was lower.

Confusion matrices were produced for both the training and testing datasets to examine classification performance in more detail.

---

## League-Level Evaluation

The coursework also attempts to compare the sum of sensitivity and specificity across different league IDs.

The submitted analysis notes that the resulting values showed no variation and explicitly identifies that this calculation may not have been implemented correctly.

This is retained in the original coursework as an example of recognising when analytical output requires further investigation rather than assuming that an unexpected result is valid.

---

## Model Comparison

The final logistic-regression section explores whether adding managerial winning percentage improves prediction of player-manager status.

Conceptually, the extended model compares:

```r
plyrMgr ~ year + win_pct
```

against the simpler year-only model.

The coursework reports that `win_pct` was not statistically significant and provided only a negligible change in model fit.

The simpler model was therefore preferred because it achieved similar performance with fewer predictors.

### Reproducibility Note

The submitted R Markdown contains a variable-naming inconsistency in this section, using `year` rather than the `yearID` variable used elsewhere in the analysis.

The original coursework file has been retained unchanged in this repository. This portfolio README documents the intended statistical comparison while preserving the submitted source code.

---

# 3. Poisson Regression and Mixed-Effects Modelling

The third part applies count-data modelling to pitcher statistics from the `Pitching` dataset.

## Pitcher Dataset

A pitcher-level dataset is created using:

```r
df_pitchers <- Pitching %>%
  filter(BFP > 0) %>%
  mutate(innings = IPouts / 3) %>%
  left_join(
    People %>% select(playerID, weight, height, throws),
    by = "playerID"
  ) %>%
  drop_na()
```

The preprocessing therefore:

- retains pitchers who faced at least one batter;
- converts recorded outs into innings pitched;
- adds player characteristics from `People`; and
- removes incomplete observations.

The additional characteristics include:

- weight;
- height; and
- throwing hand.

---

## Important Variable Note

The original submitted R Markdown contains an inconsistency in the response variable used in Question 3.

Within the `Lahman::Pitching` data:

```text
SH   = sacrifices by opposing batters
SHO  = shutouts
```

The coursework text describes `SH` as shutouts in several sections, and the initial Poisson regression is therefore fitted to `SH` while being discussed as a shutout model.

The later mixed-effects model uses `SHO`.

The original `.Rmd` has been retained unchanged to preserve the submitted coursework. This README distinguishes between the variables so that the portfolio accurately describes what the code actually models.

---

## Initial Poisson Regression

The first count model is:

```r
poisson_mod1 <- glm(
  SH ~ innings + weight + height + throws,
  family = poisson(link = "log"),
  data = df_pitchers
)
```

The model investigates the relationship between the `SH` count response and:

- innings pitched;
- weight;
- height; and
- throwing hand.

Because the response consists of non-negative counts, the analysis demonstrates the use of a **Poisson generalised linear model with a logarithmic link function**.

---

## Analysis of Deviance

Predictor contribution is assessed using:

```r
anova(poisson_mod1, test = "Chisq")
```

This demonstrates the use of likelihood-based statistical testing within a generalised linear modelling framework.

The coursework interprets small p-values as evidence against the null hypothesis that a predictor makes no contribution to the model.

Innings pitched was identified as a particularly important predictor in the submitted analysis.

---

## Mixed-Effects Poisson Model

The analysis is extended using a **generalised linear mixed-effects model**.

Unlike the initial model, this model uses the actual shutout variable `SHO`:

```r
poisson_mod2 <- glmer(
  SHO ~ innings + weight + height + throws + (1 | teamID),
  data = df_pitchers,
  family = poisson,
  nAGQ = 0
)
```

The fixed effects are:

- innings pitched;
- weight;
- height; and
- throwing hand.

`teamID` is incorporated as a random intercept.

This allows the expected shutout count to vary between teams while simultaneously estimating the effects of pitcher-level characteristics.

---

## Team-Level Variation

The coursework reports an estimated random-effect variance for `teamID` of approximately:

```text
0.04887
```

with a standard deviation of approximately:

```text
0.22113
```

The relatively small variance suggests that some between-team variability exists, although the coursework concludes that individual pitcher characteristics and pitching exposure appear more influential than team membership.

---

## Model Diagnostics

A scale-location diagnostic is produced for the initial Poisson model:

```r
plot(poisson_mod1, which = 3)
```

The coursework examines:

- residual spread;
- possible systematic patterns;
- potential outliers; and
- evidence of model misspecification.

The diagnostic relates to `poisson_mod1` and therefore to the `SH` response rather than the later `SHO` shutout model.

---

## Predictor Interpretation

The coursework uses predictions from `poisson_mod1` to compare left- and right-handed pitchers while holding other variables constant.

It also examines coefficient direction for:

- height; and
- weight.

The submitted analysis reports a ratio of approximately:

```text
1.1
```

between the predicted counts for left- and right-handed pitchers.

Because these calculations use `poisson_mod1`, they apply to the `SH` response and should **not** be interpreted as an estimate of differences in shutouts.

The `poisson_mod2` mixed-effects model is the section that directly models pitcher shutouts through `SHO`.

---

## Key Statistical Techniques

This project demonstrates the use of:

- linear regression;
- logistic regression;
- Poisson regression;
- generalised linear models;
- generalised linear mixed-effects models;
- random effects;
- data transformation;
- data joining and preprocessing;
- confidence intervals;
- hypothesis testing;
- analysis of deviance;
- residual diagnostics;
- train-test splitting;
- ROC curves;
- ROC AUC;
- Youden's Index;
- confusion matrices;
- sensitivity and specificity;
- prediction;
- AIC-based model comparison;
- model parsimony; and
- statistical visualisation.

---

## Data Analysis Workflow

The project demonstrates a general modelling workflow of:

```text
Data Preparation
      ↓
Exploratory Analysis
      ↓
Model Selection
      ↓
Model Fitting
      ↓
Statistical Inference
      ↓
Diagnostic Assessment
      ↓
Prediction / Evaluation
      ↓
Interpretation
```

---

## R Packages

The submitted R Markdown loads packages including:

```text
sandwich
broom
ggplot2
dplyr
tidyverse
magrittr
janitor
gridExtra
readxl
Lahman
viridis
lindia
lme4
caret
pROC
car
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

The complete analysis is contained in:

```text
rmd file statistics .Rmd
```

Open the file in **RStudio**.

Install the required packages if they are not already available:

```r
install.packages(c(
  "sandwich",
  "broom",
  "ggplot2",
  "dplyr",
  "tidyverse",
  "magrittr",
  "janitor",
  "gridExtra",
  "readxl",
  "Lahman",
  "viridis",
  "lindia",
  "lme4",
  "caret",
  "pROC",
  "car"
))
```

The R Markdown document can then be run interactively or rendered as an HTML document.

The baseball datasets are supplied through the `Lahman` package, so no external raw-data download is required.

---

## Coursework Notes

The original R Markdown has been retained unchanged as the submitted academic work.

While preparing the project for this portfolio, two reproducibility issues were identified:

1. The Poisson-regression section uses `SH` while referring to it as shutouts, whereas the later mixed-effects model uses `SHO`.
2. The final logistic model-comparison code switches from `yearID` to `year`.

These issues are documented rather than silently altering the submitted coursework.

This reflects an important part of practical data science: reviewing variable definitions, checking code consistency and being transparent about limitations when presenting previous analytical work.

---

## Skills Demonstrated

This project demonstrates experience in:

- R programming;
- statistical modelling;
- data manipulation;
- exploratory data analysis;
- linear regression;
- classification modelling;
- count-data modelling;
- mixed-effects modelling;
- statistical inference;
- predictive model evaluation;
- model diagnostics;
- data visualisation;
- model interpretation;
- critical review of analytical outputs; and
- communicating quantitative results.

---

## Academic Context

This repository contains work completed as part of an **MSc Data Science Statistics module**.

The original coursework has been retained alongside this portfolio documentation to demonstrate statistical modelling, R programming, model evaluation and quantitative-analysis skills.

---

## Author

**Gabriella Davis**

MSc Data Science

GitHub: `gabriella-davis0505`
