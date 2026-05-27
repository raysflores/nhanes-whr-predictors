# Predictors of Waist-to-Hip Ratio Among U.S. Adults: A Multiple Linear Regression Analysis Using NHANES 2017–2018

**Raymond Santiago Flores · STAT 4120: Applied Linear Models · Spring 2026 · University of Virginia**

A multiple linear regression analysis identifying physiological, behavioral, and socioeconomic predictors of waist-to-height ratio (WHR) in a nationally representative cross-sectional sample of U.S. adults, using data from the National Health and Nutrition Examination Survey (NHANES) 2017–2018 cycle.

---

## Overview

Waist-to-height ratio is a composite anthropometric index with clinical and epidemiological significance as a marker of central adiposity and cardiometabolic risk. Unlike BMI, WHR is not conflated by lean mass and captures abdominal fat distribution more directly. This project applies OLS regression to a curated subsample of the NHANES 2017–2018 dataset (n ≈ 4,382 observations after listwise deletion) to identify which demographic, clinical, and lifestyle factors are independently associated with WHR after controlling for correlated predictors.

The analysis follows a full inferential workflow: exploratory data analysis, distribution diagnostics, variance-stabilizing transformations, candidate variable selection via bidirectional AIC-based stepwise regression, nested F-tests for variable exclusion, multicollinearity assessment (VIF), two-way ANOVA for interaction effects, influential observation diagnostics (Cook's distance), and a formal evaluation of Gauss-Markov assumptions.

---

## Research Questions

1. Which physiological, behavioral, and socioeconomic variables independently predict waist-to-height ratio in U.S. adults after adjusting for correlates?
2. Do sex and race/ethnicity interact in their association with WHR, and does this interaction remain significant after controlling for other predictors?
3. Does the estimated model satisfy classical linear regression assumptions, and what are the implications of identified violations?

---

## Data

**National Health and Nutrition Examination Survey (NHANES) — 2017–2018 Cycle**

- **Source:** U.S. Centers for Disease Control and Prevention (CDC) / National Center for Health Statistics (NCHS)
- **Design:** Stratified, multistage probability sample with oversampling of racial and ethnic minorities, low-income populations, and adults 80 years and older
- **Analytic sample:** n ≈ 4,382 (after exclusion of observations with missing values on any model variable)
- **Outcome variable:** `whr` — waist circumference (cm) divided by height (cm)

> **Sampling design note:** NHANES employs a complex sampling design with unequal selection probabilities and survey weights. This analysis treats the analytic subsample as a simple random sample, which introduces a limitation: standard errors may be underestimated relative to design-based estimates. Results should be interpreted as exploratory rather than nationally representative after reweighting.

---

## Methods

### Preprocessing and Transformations

Several continuous predictors exhibited right skew inconsistent with linearity assumptions. Two variables required log transformation prior to inclusion:

- **`log_BMI`** — log(BMI): Shapiro-Wilk statistic improved from W = 0.945 (untransformed) to W = 0.994 (log-transformed)
- **`log_triglcd`** — log(triglycerides): applied for similar distributional reasons

All other continuous variables were entered without transformation after inspection of Q-Q plots and residual plots.

### Variable Selection

Candidate predictors were selected on substantive grounds from NHANES domains covering anthropometrics, blood chemistry, demographics, and lifestyle. Bidirectional stepwise selection using AIC was applied to determine the final model, with manual follow-up nested F-tests to confirm exclusions:

- **Creatinine (CAP):** excluded; nested F-test F = 0.63, p = 0.428
- **ALT (liver enzyme):** excluded; nested F-test F = 2.73, p = 0.099

### Final Model

```
whr ~ waist + sex + log_BMI + age + race_eth + educ_lev +
      hvy.drk + bp.di.m + log_triglcd + bp.sy.m + income + phy.act
```

| Term | Description |
|---|---|
| `waist` | Waist circumference (cm) |
| `sex` | Biological sex (binary indicator) |
| `log_BMI` | Log-transformed body mass index |
| `age` | Age in years |
| `race_eth` | Race/ethnicity (6-level factor) |
| `educ_lev` | Educational attainment (ordinal) |
| `hvy.drk` | Heavy alcohol use (binary) |
| `bp.di.m` | Mean diastolic blood pressure (mmHg) |
| `log_triglcd` | Log-transformed triglycerides (mg/dL) |
| `bp.sy.m` | Mean systolic blood pressure (mmHg) |
| `income` | Household income-to-poverty ratio |
| `phy.act` | Physical activity level (categorical) |

### Model Fit

| Statistic | Value |
|---|---|
| Residual standard error | 0.02876 on 4,363 degrees of freedom |
| R² | > 0.90 |
| Adjusted R² | > 0.90 |

### Multicollinearity

Variance Inflation Factors (VIF) were computed for all predictors. The two largest were:

| Predictor | VIF |
|---|---|
| `waist` | 9.05 |
| `log_BMI` | 8.53 |

Both values fall below the conventional threshold of 10, suggesting moderate but not severe multicollinearity. This collinearity is expected given the mathematical and physiological relationship between waist circumference and BMI, and is acknowledged as a limitation of the model.

### Interaction Analysis

A two-way ANOVA was conducted for `sex × race_eth` to assess whether the association between sex and WHR varies across racial/ethnic groups. Post-hoc comparisons used Tukey's Honest Significant Difference (HSD) correction to control familywise error rate.

### Influential Observation Diagnostics

Cook's distance was used to identify high-leverage, high-influence observations. Three observations exceeded the conventional threshold and were flagged for sensitivity analysis:

- Observation **3459**
- Observation **4712**
- Observation **2399**

### Gauss-Markov Assumption Evaluation

The analysis includes a formal discussion of the five classical Gauss-Markov conditions (linearity, exogeneity, homoscedasticity, no autocorrelation, no perfect multicollinearity). Heteroscedasticity was assessed via residual-vs-fitted plots; the complex survey design implies residuals may not be fully i.i.d., representing a limitation for inference.

---

## Technical Stack

- **Language:** R
- **Key packages:** `tidyverse`, `car`, `leaps`, `reshape2`, `knitr`, `kableExtra`
- **Methods:** OLS multiple linear regression, bidirectional AIC stepwise selection, nested F-tests, VIF, Cook's distance, two-way ANOVA, Tukey HSD

---

## Repository Structure

```
├── 4120_Project.Rmd    # Full analysis — preprocessing, EDA, modeling, diagnostics
└── README.md
```

> **Data note:** `nhanes17.RData` is not included in this repository. The NHANES 2017–2018 public-use data files are available for download at [https://wwwn.cdc.gov/nchs/nhanes/continuousnhanes/default.aspx?BeginYear=2017](https://wwwn.cdc.gov/nchs/nhanes/continuousnhanes/default.aspx?BeginYear=2017). Load the `.RData` workspace into the project directory before knitting.

---

## Limitations

- **Complex sampling design:** Survey weights and design effects are not incorporated; standard errors and p-values should be interpreted as approximate.
- **Collinearity:** Moderate VIF inflation between `waist` and `log_BMI` limits interpretability of their individual coefficients.
- **Cross-sectional design:** Temporal precedence cannot be established; no causal inference is warranted.
- **Listwise deletion:** Exclusion of observations with any missing covariate (n removed not reported here) may introduce selection bias if missingness is not completely at random.

---

## About

Completed as the course project for STAT 4120: Applied Linear Models at the University of Virginia, Spring 2026.

**Author:** Raymond Santiago Flores | [raymondosf@gmail.com](mailto:raymondosf@gmail.com)
