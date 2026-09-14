# Cookie Cats A/B Testing: Gate Placement & Player Retention

An end-to-end A/B test analysis examining whether moving a progression gate from level 30 to level 40 in the mobile game **Cookie Cats** affects player retention and engagement.

![Python](https://img.shields.io/badge/Python-3.12-blue)
![pandas](https://img.shields.io/badge/pandas-data--analysis-150458)
![scipy](https://img.shields.io/badge/scipy-statistics-8CAAE6)
![scikit--learn](https://img.shields.io/badge/scikit--learn-outlier--detection-F7931E)

## Business Problem

Cookie Cats gates progression at level 30, requiring players to either wait or make an in-app purchase to continue. The product team wants to know whether moving this gate to level 40 changes:

- **Retention** — do players come back after 1 day? After 7 days?
- **Engagement** — how many game rounds do players complete?

The goal is a data-backed launch decision: keep the gate at level 30, or move it to level 40.

## Dataset

- **Source:** [Mobile Games A/B Testing (Cookie Cats)](https://www.kaggle.com/datasets/yufengsui/mobile-games-ab-testing) on Kaggle
- **Size:** 90,189 users
- **Columns:**

| Column | Description |
|---|---|
| `userid` | Unique player identifier |
| `version` | Experiment group — `gate_30` (control) or `gate_40` (treatment) |
| `sum_gamerounds` | Total game rounds played in the first 14 days |
| `retention_1` | Whether the player returned 1 day after install |
| `retention_7` | Whether the player returned 7 days after install |

## Methodology

The analysis follows a structured **7-step A/B testing framework**:

1. Clarify the business problem
2. State the experiment hypothesis
3. Design the experiment
4. Run the experiment
5. Assess validity threats
6. Conduct statistical inference
7. Decide whether to launch

### 1. Exploratory Data Analysis
Distribution checks and summary statistics for group sizes, retention rates, and game rounds played.

### 2. Data Quality Checks
Verified zero missing values and zero duplicate user IDs before proceeding.

### 3. Validity Check — Sample Ratio Mismatch (SRM)
A chi-square goodness-of-fit test on the observed group split (44,700 vs. 45,489) returned **χ² = 6.90, p = 0.0086**. This flags at conventional thresholds (α = 0.05, 0.01) but not at the stricter α = 0.001 threshold some experimentation platforms (e.g., Microsoft ExP) use specifically to control false positives at scale. Treated as a caveat rather than a disqualifier.

### 4. Outlier Detection & Treatment
Cross-validated outliers in `sum_gamerounds` using four methods:
- Boxplot visualization
- Z-score thresholding (|z| > 3)
- IQR filtering
- Isolation Forest and Local Outlier Factor (scikit-learn)

One extreme outlier (49,854 rounds played) was removed prior to hypothesis testing.

### 5. Hypothesis Testing

| Metric | Type | Tests Used |
|---|---|---|
| `retention_1`, `retention_7` | Binary | Chi-square test of independence, two-proportion z-test |
| `sum_gamerounds` | Continuous, skewed | Shapiro-Wilk → Levene's → Mann-Whitney U / Welch's t-test |

## Key Results

| Metric | Test | p-value | Significant? |
|---|---|---|---|
| Day-1 Retention | Chi-square / Z-test | 0.075 / 0.074 | No |
| Day-7 Retention | Chi-square / Z-test | 0.0016 / 0.0016 | **Yes** |
| Game Rounds Played | Mann-Whitney / Welch's t-test | 0.051 / 0.804 | No |

**Lift (gate_40 vs. gate_30):**
- Day-1 retention: **-1.32%**
- Day-7 retention: **-4.30%**

## Recommendation

**Keep the gate at level 30.** It shows significantly better 7-day retention — the more meaningful long-term metric — with no offsetting gain in engagement or short-term retention from delaying the gate to level 40.

## Tech Stack

- **Language:** Python
- **Data handling:** pandas, numpy
- **Visualization:** matplotlib
- **Statistics:** scipy.stats (chi-square, Shapiro-Wilk, Levene's, Mann-Whitney U, Welch's t-test), statsmodels (two-proportion z-test)
- **Outlier detection:** scikit-learn (Isolation Forest, Local Outlier Factor)
- **Data source:** kagglehub

## Repository Structure

```
.
├── ab_testing_cookieCat.ipynb   # Full analysis notebook
└── README.md
```

## Getting Started

```bash
# Install dependencies
pip install pandas numpy matplotlib scipy statsmodels scikit-learn kagglehub

# Download the dataset (or place cookie_cats.csv in the working directory)
python -c "import kagglehub; kagglehub.dataset_download('yufengsui/mobile-games-ab-testing')"

# Run the notebook
jupyter notebook ab_testing_cookieCat.ipynb
```

## Notes on Interpretation

- The SRM flag at conventional significance levels means the group split wasn't perfectly random; results are directionally trustworthy but not bulletproof, and would warrant a re-run with corrected randomization in a production setting.
- Multiple statistical approaches (parametric and non-parametric) were run in parallel for the engagement metric to guard against conclusions that depend on a single test's assumptions.
