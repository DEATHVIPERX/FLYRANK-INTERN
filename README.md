# FlyRank Content Refresh Prioritization

A machine-learning system for identifying and prioritizing content pages that show significant deterioration in organic search performance.

> **Internship Capstone Project — FlyRank**

## Overview

Content teams often manage large portfolios of pages whose organic search performance changes over time. The challenge is not simply identifying pages that are declining, but determining **which pages should be reviewed first**.

This project develops a machine-learning-based prioritization system that ranks content pages according to their likelihood of meeting a predefined **content-refresh criterion**.

The system combines historical Google Search Console performance signals with trend-based features and produces a ranked queue of high-priority refresh candidates.

### Key Result

| Approach | Precision@20 |
|---|---:|
| Test-set base rate | **32.40%** |
| Rule-based baseline | **40.00%** |
| **Random Forest** | **100.00%** |

The Random Forest placed **20/20 qualifying refresh candidates in the top 20 predictions** on the held-out test set.

> **Important:** 100% Precision@20 does not mean 100% overall model accuracy. It means that all 20 of the model's highest-ranked test-set recommendations satisfied the project's predefined refresh-candidate criterion.

---

## Project Objective

The central research question is:

> **Can historical organic-search performance signals be used to rank content pages according to their likelihood of meeting a predefined refresh-candidate criterion?**

The goal is to create an actionable queue for content strategists rather than an automated system that decides whether content should be changed.

---

## Dataset

The project uses the **FlyRank internship warehouse**, including content-level search-performance data.

The analysis uses a 90-day performance window and compares the most recent 30 days against the previous 30 days.

### Analytical Dataset

- **271,046** content-level observations
- **14** analytical columns
- 90-day performance window
- Recent vs. previous 30-day comparisons
- Client-grouped train/test split

Content with zero impressions across the 90-day window was excluded from the analytical dataset.

### Refresh Candidate Definition

A page is considered a refresh candidate when:

1. Its most recent 30-day impressions decreased by more than **20%** compared with the previous 30 days.
2. Its total 90-day impressions exceeded **500**.

This is an operational definition created for the project. It should not be interpreted as proof that a page is objectively low quality or that refreshing it will necessarily recover traffic.

---

## Methodology

### Feature Engineering

The model uses historical search-performance signals including:

- 90-day impressions
- Recent 30-day impressions
- Recent 30-day clicks
- Recent average search position
- Previous 30-day impressions
- Previous 30-day clicks
- Previous average search position

Derived trend features include:

- Impression trend percentage
- Click trend percentage
- Position delta

### Baseline

A rule-based baseline was created using a simple deterioration condition:

- At least 500 impressions over 90 days
- Current 30-day impressions below previous 30-day impressions

This provides a straightforward benchmark against which the machine-learning ranking can be evaluated.

### Model

The project uses a **Random Forest classifier** configured with:

- 100 trees
- Maximum depth: 5
- Random seed: 42

The model produces a refresh probability for each page. Pages are then sorted by this probability to create the final prioritization queue.

### Validation

The train/test split is grouped by `client_hash_id`.

This prevents pages belonging to the same client from being randomly distributed between training and testing, reducing the risk of client-level information leakage.

| Dataset | Observations |
|---|---:|
| Training | 221,203 |
| Test | 49,843 |
| **Total** | **271,046** |

---

## Results

The model was evaluated using **Precision@20**, which measures the proportion of genuine refresh candidates among the first 20 recommendations.

| Evaluation | Precision@20 |
|---|---:|
| Base rate | 32.40% |
| Rule-based baseline | 40.00% |
| **Random Forest** | **100.00%** |

The machine-learning model therefore improved Precision@20 by **60 percentage points** compared with the rule-based baseline.

### Feature Importance

![Feature Importance](../work/figures/feature.png)

*Relative feature importance from the trained Random Forest classifier.*

Feature importance provides an indication of which performance signals the model relies on most heavily. It should not be interpreted as causal evidence.

### Precision@20 Comparison

![Precision@20 Comparison](../work/figures/precision.png)

*Precision@20 comparison between the rule-based baseline and the Random Forest model.*

---

## Ranked Output

The model generates a ranked refresh queue containing:

- Anonymized content identifier
- Anonymized client identifier
- Model refresh probability
- Reason code
- Impression trend
- Position delta

Example high-ranking candidates:

| Rank | ML Probability | Reason | Impression Trend | Position Delta |
|---:|---:|---|---:|---:|
| 1 | 0.9763 | Severe position drop | -82.86% | 24.31 |
| 2 | 0.9763 | Severe position drop | -68.79% | 3.25 |
| 3 | 0.9763 | Severe position drop | -67.87% | 15.39 |
| 4 | 0.9761 | Severe position drop | -93.85% | 3.58 |
| 5 | 0.9759 | Severe position drop | -44.51% | 7.02 |

The complete ranked queue is generated when the notebook is executed rather than committed to the repository as a generated CSV artifact.

---

## Repository Structure

```text
FLYRANK-INTERN/
│
├── docs/
│   └── index.md
│
├── work/
│   ├── notebooks/
│   │   └── capstone.ipynb
│   │
│   └── figures/
│       ├── feature.png
│       └── precision.png
│
└── README.md
```

### Contents

**`work/notebooks/capstone.ipynb`**  
Complete analysis, feature engineering, model training, evaluation, and recommendation generation.

**`work/figures/`**  
Visualizations used in the project report.

**`docs/index.md`**  
The full project report published through GitHub Pages.

Generated recommendation outputs are intentionally not committed to the repository.

---

## Project Report

For the complete methodology, analysis, results, limitations, and recommendations:

**[Read the Full Project Report](https://deathviperx.github.io/FLYRANK-INTERN/)**

---

## Limitations

The model should be viewed as a **prioritization system**, not an automated content-quality evaluator.

Key limitations include:

- The feature set is primarily based on search-performance data.
- The refresh target is a binary operational definition.
- The model identifies patterns associated with deterioration but does not establish causation.
- Search performance can be affected by seasonality, competitors, search behaviour, and search-engine changes.
- The project does not experimentally measure whether refreshing a recommended page actually improves its performance.
- The chosen 20% decline and 500-impression thresholds may affect the resulting target distribution.

A future version could incorporate content age, publication history, topic, content length, historical refresh activity, and additional search-performance signals.

---

## Future Work

Potential improvements include:

1. **Expand the feature set** with content-level and historical metadata.
2. **Develop a continuous priority score** instead of a binary refresh label.
3. **Track recommendations longitudinally** to determine whether refreshed pages recover.
4. **Run controlled experiments** to measure the causal impact of content refreshes.
5. **Improve explainability** by providing richer reasons for individual recommendations.
6. **Evaluate across additional time periods** to test robustness against seasonal changes.

---

## Reproducibility

The complete analysis is contained in the capstone notebook.

The notebook can be used to:

1. Load the internship warehouse.
2. Construct the analytical dataset.
3. Perform feature engineering.
4. Generate the refresh-candidate labels.
5. Create the client-grouped train/test split.
6. Train the Random Forest classifier.
7. Evaluate Precision@20.
8. Generate the ranked refresh queue.
9. Produce the project visualizations.

### Technologies

- Python
- pandas
- NumPy
- DuckDB
- scikit-learn
- Matplotlib
- Seaborn
- Jupyter / Google Colab

---

## Acknowledgments

This project was completed as part of the **FlyRank internship**.

The analysis uses the `FlyRank/internship-warehouse` dataset provided for the internship project. Client and content identifiers used in the analysis are anonymized.

---

## Author

**DEATHVIPERX**

[GitHub](https://github.com/DEATHVIPERX)

---

## License

This repository is an internship/capstone project. Dataset access and reuse are subject to the terms and restrictions associated with the original FlyRank internship data.
