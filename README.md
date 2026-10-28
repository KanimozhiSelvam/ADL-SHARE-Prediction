# Activities of Daily Living (ADL) Decline Prediction

## Project Overview

This project investigates whether baseline characteristics can predict the development of a **new Activities of Daily Living (ADL) limitation** among adults aged 65+ using longitudinal SHARE/easySHARE survey data.

### Research Question

> **Among adults aged 65+ with no ADL limitation at baseline (ADL = 0), can baseline demographic, health, physical, mental-health, socioeconomic, cognitive, and functional characteristics predict a new ADL limitation at the next available SHARE wave?**

The prediction outcome is:

- `0` = no new ADL limitation at follow-up
- `1` = new ADL limitation at follow-up (ADL ≥ 1)

---

## Dataset

The analysis uses the **easySHARE Release 9.0.0** longitudinal dataset.

The study uses the following baseline → follow-up wave transitions:

- Wave 1 → Wave 2
- Wave 4 → Wave 5
- Wave 5 → Wave 6
- Wave 6 → Wave 7
- Wave 7 → Wave 8
- Wave 8 → Wave 9

Wave 3 is not used for the regular ADL transition because it is a SHARELIFE life-history wave with a different questionnaire structure.

The raw SHARE/easySHARE dataset is **not included in this repository**. Users must obtain the data through the appropriate SHARE data-access process.

---

## Study Population and Outcome

Participants were eligible for prediction if they:

1. were aged 65 or older at baseline;
2. had no ADL limitation at baseline (`ADL = 0`);
3. had a valid ADL measurement at the next available wave.

The primary outcome is defined as:

```text
Baseline ADL = 0
        ↓
Next available SHARE wave
        ↓
Follow-up ADL ≥ 1
        ↓
New ADL decline = 1
```

The final test-period prevalence of new ADL decline is approximately **9.05%**.

Because the outcome is imbalanced, accuracy alone is not considered an adequate evaluation metric.

---

## Analysis Pipeline

```text
Raw easySHARE data
        ↓
Data-quality audit
        ↓
Special missing-code cleaning
        ↓
Plausibility checks and feature engineering
        ↓
Exploratory data analysis
        ↓
Variable classification
        ↓
Same-wave association analysis
        ↓
Baseline → future ADL association
        ↓
Longitudinal prediction dataset
        ↓
Feature-set construction
        ↓
PCA
        ↓
Development / validation / test design
        ↓
Predictive model comparison
        ↓
Threshold selection
        ↓
Final test evaluation
        ↓
Sensitivity and robustness analyses
```

---

## Data Cleaning

easySHARE contains special negative codes representing different types of missing or unavailable information.

These special codes (`-3` to `-16`) are converted to missing values before analysis. Plausibility checks are also applied to variables with known ranges or extreme distributions.

For example, after cleaning:

- EURO-D has a plausible distribution;
- grip strength has a substantially more plausible distribution;
- BMI is restricted to a plausible range;
- extreme household-income values are controlled;
- missingness becomes explicitly visible.

Missing predictor values are retained and handled inside the modeling pipelines rather than deleting observations solely because of missing predictors.

### Effect of the missing-code problem

A direct comparison using the original model design showed:

| Design | Test AUC |
|---|---:|
| Special codes treated as real numbers | 0.7343 |
| Special codes correctly treated as missing | 0.7485 |

Thus, correct handling of the easySHARE special missing codes improved the original model's test AUC by approximately **0.014**.

---

## Exploratory Analysis

Exploratory analyses examined:

- ADL distributions;
- age-group differences;
- demographic characteristics;
- self-rated health;
- chronic disease burden;
- physical function;
- cognitive measures;
- mental health;
- socioeconomic characteristics;
- activity and social-help variables;
- missingness;
- predictor correlations.

Future ADL decline increased strongly with age:

| Age group | New ADL decline |
|---|---:|
| 65–69 | 5.05% |
| 70–74 | 6.90% |
| 75–79 | 10.41% |
| 80–84 | 15.29% |
| 85+ | 25.57% |

Baseline self-rated health was the strongest individual candidate predictor in the univariate future-outcome analysis, followed by age, CASP, EURO-D, grip strength, recall, and activity.

---

## Feature Selection

Feature selection combined statistical evidence with substantive considerations.

The following were considered:

- missingness;
- variable type;
- same-wave associations;
- baseline-to-future associations;
- univariate ROC-AUC;
- redundancy;
- conceptual relevance;
- multivariable predictive performance.

A data-driven ranking identified the strongest individual predictors as:

1. Self-rated health (`sphus`)
2. CASP
3. Age
4. EURO-D
5. Grip strength
6. Recall
7. Activity
8. Healthcare utilization
9. Chronic disease burden
10. Income-related measures

However, univariate AUC was not used as the sole criterion for the final model.

---

## Functional Indicators

Additional functional indicators were evaluated because ADL decline is closely related to physical and functional capacity.

The analysis considered:

- mobility difficulties;
- large-muscle difficulties;
- IADL difficulties;
- grip strength.

These variables showed meaningful associations with future ADL decline.

For example, the univariate AUCs were approximately:

- `mobilityind`: **0.668**
- `lgmuscle`: **0.682**

The functional indicators substantially improved the multivariable model.

---

## Principal Component Analysis

PCA was performed after imputation and standardization to assess whether the predictor space could be represented using fewer dimensions.

For the primary eight-variable PCA analysis, the first four components explained:

| Component | Explained variance |
|---|---:|
| PC1 | 26.34% |
| PC2 | 16.47% |
| PC3 | 12.19% |
| PC4 | 11.48% |

The first four components explained approximately **66.48%** of the variance.

PCA was also evaluated on the larger feature sets.

The main finding was that PCA did **not** provide useful compression for prediction:

- many components were required to retain 80–90% of the variance;
- the best validation-selected number of components was close to the original number of variables;
- PCA + Logistic Regression consistently performed slightly worse than Logistic Regression using the original variables.

Therefore:

> **PCA was not used in the final predictive model.**

Keeping the original variables also preserves interpretability through variable-level odds ratios.

---

## Longitudinal Prediction Design

The analysis uses a temporal prediction design.

Earlier wave transitions are used for model development and validation, while later transitions are reserved for final evaluation.

The main design separates:

```text
Development
Wave 1→2
Wave 4→5
Wave 5→6

Validation
Wave 6→7

Final test
Wave 7→8
Wave 8→9
```

This reflects the real prediction task:

> use information available at baseline to predict a future ADL outcome.

---

## Feature Sets

Nested feature sets were developed to assess the value of additional information.

### Feature Set A
Core demographic, health, chronic-disease, and physical-function variables.

### Feature Set B
Feature Set A plus mental health, BMI, income, and cognitive recall.

### Feature Set C
Feature Set B plus activity, social-help, household, education, healthcare-use, and country variables.

### Feature Set D
Feature Set C plus functional indicators such as mobility, large-muscle function, and IADL difficulties.

### Feature Set D+T
Feature Set D plus changes in selected characteristics between the previous wave and baseline.

---

## Predictive Models

The following approaches were evaluated:

- Logistic Regression;
- PCA + Logistic Regression;
- Random Forest;
- Histogram-based Gradient Boosting;
- Elastic-Net Logistic Regression;
- LightGBM;
- XGBoost;
- probability blending;
- models with functional indicators;
- models with longitudinal trend features.

The main evaluation metrics were:

- ROC-AUC;
- PR-AUC;
- Brier score;
- precision;
- recall;
- F1-score;
- calibration;
- top-20% risk-group recall;
- lift;
- decision-curve net benefit.

---

## Main Model Comparison

The basic Logistic Regression model achieved a test ROC-AUC of approximately **0.750**.

Adding increasingly informative variables improved performance:

| Model | Test ROC-AUC |
|---|---:|
| Logistic Regression (B) | 0.750 |
| PCA + Logistic Regression (B) | 0.749 |
| Random Forest (B) | 0.751 |
| Logistic Regression (C) | 0.763 |
| Logistic Regression + Functional Indicators (D) | **0.784** |
| Logistic Regression + Trend (D+T) | 0.790 |
| HGB + Trend (D+T) | ~0.793 |
| XGBoost + Trend (D+T) | ~0.792 |

The largest improvement came from adding **functional indicators**, rather than simply changing the machine-learning algorithm.

Trend features provided only a modest additional improvement over the functional model.

---

# Primary Final Model

## Logistic Regression with Functional Indicators — Feature Set D

The primary model selected for detailed interpretation is **Logistic Regression with functional indicators (Feature Set D)**.

### Test performance

| Metric | Result |
|---|---:|
| ROC-AUC | **0.784** |
| 95% CI | **0.777–0.790** |
| PR-AUC | **0.295** |
| Brier score | **0.073** |
| Sensitivity / Recall | **43.1%** |
| Specificity | **89.9%** |
| Precision / PPV | **29.9%** |
| NPV | **94.1%** |
| F1-score | **0.353** |
| Top-20% recall | **55.6%** |
| Top-20% lift | **2.78** |

The validation-selected probability threshold was **0.183**.

---

## Why Logistic Regression Was Selected

The trend-based models achieved slightly higher ROC-AUC values, approximately 0.79, but the improvement over the functional Logistic Regression model was small.

The functional Logistic Regression model was selected as the primary model because it provides:

- strong discrimination;
- good calibration;
- interpretable coefficients and odds ratios;
- stable performance across different time periods;
- robustness to repeated observations;
- robustness to the changing missing-data pattern in SHARELIFE;
- a transparent relationship between predictors and predicted risk.

The trend model is retained as an important comparison showing that additional longitudinal information provides only a modest gain.

---

## Interpretation of Important Predictors

The final logistic regression showed that poorer current health and greater functional difficulty are associated with higher predicted risk of future ADL decline.

Important predictors included:

- self-rated health;
- age;
- large-muscle difficulties;
- mobility difficulties;
- IADL difficulties;
- grip strength;
- sex;
- activity;
- received help.

In the primary nine-variable model, higher self-rated health score, age, EURO-D, BMI, and chronic disease burden were associated with higher odds, while stronger grip strength, better recall, and female sex were associated with lower odds.

After adding functional indicators, some of the apparent effects of age, self-rated health, and depression were reduced, indicating that part of their predictive information overlaps with physical and functional status.

These results represent **predictive associations, not causal effects**.

---

## Risk Stratification

The final model produces a clear gradient of observed risk across predicted-risk deciles.

The observed new-ADL rate increased from approximately:

- **1.0%** in the lowest-risk decile
- to **32.3%** in the highest-risk decile.

Among the 20% of participants with the highest predicted risk:

- approximately **55.6% of all new ADL cases** were captured;
- the lift was approximately **2.78** compared with random selection.

This indicates that the model is particularly useful for **risk ranking and population-level targeting**.

---

## Threshold Analysis

The threshold of **0.183** maximized F1-score on the validation period.

At this threshold on the test period:

- approximately 13% of participants were flagged;
- precision was approximately 30%;
- recall was approximately 43%;
- F1-score was approximately 0.352.

Lower thresholds capture more future cases but create more false positives. Higher thresholds increase precision but miss more cases.

Therefore, the final threshold should be chosen according to the practical costs and capacity of the intended intervention rather than treated as a universal clinical threshold.

---

## Decision Curve Analysis

Decision-curve analysis showed that the final Logistic Regression model had higher net benefit than both "flag everyone" and "flag nobody" across the examined threshold range.

This supports the potential use of the model for **risk-based screening or prevention targeting**, particularly when identifying high-risk individuals is useful and follow-up resources are available.

---

## Robustness and Sensitivity Analyses

Several additional analyses were performed.

### Alternative outcome definition

Using ADL ≥ 2 instead of ADL ≥ 1 produced the same overall model ranking.

The functional model achieved approximately **0.829 AUC** for the more severe outcome.

This supports the robustness of the functional-feature finding.

### Person/household grouping

Household-grouped cross-validation continued to show an advantage for the functional Logistic Regression model.

### One baseline per person

Restricting training data to one randomly selected baseline pair per person produced very similar Logistic Regression performance.

For the functional model:

```text
Main AUC:                  0.7837
One pair per person:       0.7826
```

This indicates that repeated observations of the same person do not drive the Logistic Regression result.

### Rolling-origin validation

The functional Logistic Regression model remained competitive across successive wave transitions, with AUC values around 0.78–0.80.

### Missing-data patterns

Wave 7 contains a substantial SHARELIFE-related missing-data pattern for several variables.

The analysis showed that Logistic Regression handled this changing missingness pattern more robustly than some tree-based alternatives.

### Smoking

Adding smoking did not meaningfully improve prediction.

### Follow-up interval

Adding the follow-up interval as a predictor did not improve the extended Logistic Regression model.

### Doctor visits

Doctor visits are strongly right-skewed, but capping and log-transforming the variable changed the test AUC by only approximately 0.0001. The raw variable was therefore retained.

---

## Important Limitations

- Only participants with a valid follow-up ADL measurement contribute to the prediction analysis, resulting in attrition.
- Follow-up corresponds to the next available SHARE interview rather than a fixed time horizon.
- The outcome is relatively uncommon, with approximately 9% prevalence in the final test period.
- ADL is self-reported.
- Country and survey-period differences may influence the results.
- Some later waves have substantial changes in missing-data patterns.
- The model predicts risk and should not be interpreted as establishing causal effects.
- The model is intended for risk ranking and population-level screening rather than individual diagnosis.
- The selected threshold depends on the relative costs of false positives and missed cases.

---

## Key Conclusions

1. **Data quality matters.** Correcting easySHARE special missing codes improved the original model's AUC from 0.734 to 0.749.

2. **Baseline health characteristics contain predictive information.** Age, self-rated health, mental health, cognition, and physical capacity are associated with future ADL decline.

3. **Functional indicators provide the largest improvement.** Adding mobility, large-muscle, and IADL measures increased test AUC from approximately 0.750 to **0.784**.

4. **PCA did not improve prediction.** Most of the original information was required to maintain predictive performance, so the original variables were retained.

5. **Algorithm choice matters less than information content.** Logistic Regression, Elastic-Net, and tree-based models perform similarly when given the same information. Adding relevant functional information produces a much larger improvement.

6. **Longitudinal trends provide only a modest additional gain.** Adding previous-wave changes increases AUC from approximately 0.784 to 0.790 for Logistic Regression.

7. **The primary model is interpretable and useful for risk ranking.** Logistic Regression with functional indicators achieves a test AUC of **0.784** and captures approximately **56% of future cases within the highest-risk 20% of participants**.

8. **The model should be used as a risk-ranking and screening tool, not as a standalone individual-level diagnostic system.**

---

## Repository Structure

```text
adl-prediction/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── notebooks/
│   └── ADL_prediction_analysis.ipynb
│
├── outputs/
│   ├── figures/
│   └── tables/
│
├── docs/
│   └── ADL_Project_Team_Summary.docx
│
└── presentation/
    └── ADL_Prediction_Presentation.pptx
```

---

## Reproducibility

Install the required Python packages:

```bash
pip install -r requirements.txt
```

The SHARE/easySHARE dataset is not included in this repository.

Set the local path to the easySHARE `.dta` file using the `EASYSHARE_PATH` environment variable.

Example:

```text
EASYSHARE_PATH=/path/to/easySHARE_rel9-0-0.dta
```

The analysis uses a fixed random seed (`42`) where applicable.

---

## Project Status

**Analysis completed.**

The project includes:

- data-quality auditing;
- data cleaning;
- feature engineering;
- exploratory analysis;
- longitudinal prediction design;
- feature selection;
- PCA;
- Logistic Regression;
- Random Forest;
- gradient-boosting models;
- model comparison;
- threshold analysis;
- calibration;
- risk stratification;
- decision-curve analysis;
- and sensitivity/robustness analyses.

The next stage is the final project presentation.
