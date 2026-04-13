# Credit Risk Decision System
### Stage 2 — Data Understanding

---

## Overview

This stage moves from problem framing to data reality. Every claim made in Stage 1 is now tested against 307,511 actual loan applications. No assumptions carry forward unless the data confirms them.

The dataset is `application_train.csv` from the Home Credit Default Risk dataset (Kaggle). Each row represents a single loan application captured at the point of decision — containing only information available to the lender at approval time. No post-loan behaviour. No future leakage. This makes it directly analogous to a real lending system input.

---

## Dataset at a Glance

| Property | Value |
|---|---|
| Source | Home Credit Default Risk (Kaggle) |
| File | `application_train.csv` |
| Rows | 307,511 |
| Columns | 122 |
| Target | Binary — 0 (repaid), 1 (defaulted) |
| Default Rate | **8.07%** (24,825 defaults) |
| Class Balance | 91.93% repaid / 8.07% defaulted |

The 8.07% default rate is not a limitation of the dataset — it is a realistic property of lending portfolios, where defaults are rare events but individually costly. Any model and policy built on this data operates under the same conditions a real NBFC faces.

---

## Feature Set

11 features were selected across three tiers of the business argument established in Stage 1:

**Tier 1 — Financial Capacity** *(traditional indicators — included to test their sufficiency)*
- `AMT_INCOME_TOTAL` — annual income
- `AMT_CREDIT` — total loan amount requested *(also serves as revenue proxy in Stage 5)*
- `AMT_ANNUITY` — monthly loan repayment amount

**Tier 2 — Demographic & Stability** *(socioeconomic and lifecycle signals)*
- `DAYS_BIRTH` → converted to age in years
- `DAYS_EMPLOYED` → converted to employment duration; contains the 365243 anomaly (see below)
- `NAME_EDUCATION_TYPE` — highest education level
- `NAME_INCOME_TYPE` — employment category
- `CODE_GENDER`

**Tier 3 — Behavioural & External** *(compressed credit bureau intelligence)*
- `EXT_SOURCE_1` — external risk score
- `EXT_SOURCE_2` — external risk score
- `EXT_SOURCE_3` — external risk score

**Missing value summary for selected features:**

| Feature | Missing Count | Missing % | Action |
|---|---|---|---|
| AMT_ANNUITY | 12 | 0.0% | Median imputation |
| EXT_SOURCE_1 | 173,378 | 56.4% | Median imputation |
| EXT_SOURCE_2 | 660 | 0.2% | Median imputation |
| EXT_SOURCE_3 | 60,965 | 19.8% | Median imputation |
| All others | 0 | 0.0% | No action required |

EXT_SOURCE_1's 56% missingness is a known characteristic of this dataset. The score is unavailable for a large segment of applicants. Rows are not dropped — median imputation preserves the full 307k population for modelling.

---

## Finding 1 — Income Cannot Separate Defaulters from Non-Defaulters

| Metric | Defaulters | Non-Defaulters | Difference |
|---|---|---|---|
| Mean income | ₹1,65,612 | ₹1,69,078 | **2.0%** |
| Median income | ₹1,35,000 | ₹1,48,500 | 9.9% |

The income distributions of the two groups overlap almost completely. A 2.0% difference in means provides no meaningful basis for a lending decision. No income threshold can cleanly partition borrowers by risk.

**This directly confirms Stage 1's central claim:** income reflects earning capacity, not repayment behaviour. Two borrowers with identical incomes can carry dramatically different default probabilities. A system calibrated on income alone is structurally blind to this.

---

## Finding 2 — EXT_SOURCE Variables Provide Strong, Consistent Separation

| Feature | Default Mean | Non-Default Mean | Gap | Relative Gap |
|---|---|---|---|---|
| EXT_SOURCE_1 | 0.3870 | 0.5115 | 0.1245 | 24.3% |
| EXT_SOURCE_2 | 0.4109 | 0.5235 | 0.1125 | 21.5% |
| EXT_SOURCE_3 | 0.3907 | 0.5210 | 0.1303 | 25.0% |

All three variables show a consistent distributional shift — defaulters score materially lower across the board. EXT_SOURCE_3 carries the strongest separation (gap: 0.1303), followed by EXT_SOURCE_1 (0.1245) and EXT_SOURCE_2 (0.1125).

Measured as relative gap between group means, this represents approximately **12× stronger separation than income** (24% vs 2%). These are the signals that income cannot see — compressed behavioural intelligence encoding past repayment history, credit utilisation, and delinquency patterns from external bureaus.

This is the empirical foundation for the model in Stage 3.

---

## Finding 3 — Age Shows Moderate Signal

| Group | Mean Age | Median Age |
|---|---|---|
| Defaulters | 40.8 years | 40.1 years |
| Non-Defaulters | 44.2 years | 44.4 years |

A 3.4-year gap in mean age. The default distribution peaks in the early-to-mid 30s — consistent with early-career financial instability, lower savings buffers, and shorter credit histories. Age is a supporting signal, not a primary driver.

---

## Finding 4 — Categorical Features Show Structured Risk Gradients

**Education Type** (baseline: 8.07%)

| Education Level | Default Rate | Share of Portfolio |
|---|---|---|
| Lower secondary | 10.93% | 1.2% |
| Secondary / secondary special | 8.94% | 71.0% |
| Incomplete higher | 8.48% | 3.3% |
| Higher education | 5.36% | 24.3% |
| Academic degree | 1.83% | 0.05% |

A 9.1 percentage point spread from lowest to highest education level. The gradient is monotonic — higher education correlates with lower default rates throughout.

**Income Type** (main categories, baseline: 8.07%)

| Income Type | Default Rate | Share of Portfolio |
|---|---|---|
| Working | 9.59% | 51.6% |
| Commercial associate | 7.48% | 23.3% |
| State servant | 5.75% | 7.1% |
| Pensioner | 5.39% | 18.0% |

**Gender** (baseline: 8.07%)

| Gender | Default Rate | Count |
|---|---|---|
| Male | 10.14% | 105,059 |
| Female | 7.00% | 202,448 |

These are not noise. They reflect real socioeconomic risk gradients — employment stability, income predictability, and financial lifecycle stage all manifest in default behaviour.

---

## Finding 5 — DAYS_EMPLOYED = 365243 Is a Deliberate Encoding, Not Missing Data

18.01% of applicants (55,374 rows) carry the value `365243` in `DAYS_EMPLOYED`. This is not a measurement error — it is a business-specific encoding for a specific borrower category.

| Group | Count | Default Rate |
|---|---|---|
| Recorded employment | 252,137 | 8.66% |
| Flag = 365243 | 55,374 | **5.40%** |

Flagged borrowers default at a **lower** rate than the overall population. This immediately rules out the "unemployed" interpretation. The value most likely encodes pensioners, retirees, or self-employed borrowers without a standard employment record — groups with stable income and lower default risk.

**Handling decision:**
- `DAYS_EMPLOYED_CLEAN`: replace 365243 with `NaN`, then median-impute for continuous employment signal
- `EMPLOYED_FLAG_SPECIAL`: binary indicator (0/1) retained as a separate feature

Treating this value as standard missing data without preserving the flag would silently remove a meaningful risk signal and introduce systematic bias into the model. Both the continuous and binary representations are carried into Stage 3.

---

## Stage 2 Conclusion

Across financial and demographic features, separation between defaulters and non-defaulters remains limited — confirming that traditional indicators alone are insufficient for risk assessment. In contrast, the EXT_SOURCE behavioural variables show clear, consistent, and quantified differentiation between the two groups, validating the probability-based decision framework proposed in Stage 1.

The feature set is confirmed. The data is understood. Modelling proceeds in Stage 3.

---

*Next: [Stage 3 — Risk Driver Analysis & Model](README_Stage3.md)*
