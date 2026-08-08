# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Uns Ahmad
- **Lane:** Lane 2 (Refresh / Content Opportunity Scoring)
- **Repo:** https://github.com/DEATHVIPERX/FLYRANK-INTERN
- **Date:** August 9, 2026

## 1. Abstract
We investigate which decaying web pages present the highest ROI for content refresh efforts. Using FlyRank's daily search performance warehouse, we aggregated 90-day time-series data to isolate pages showing negative traffic trajectories but high historical search gravity. We applied a Random Forest classifier to predict the probability of a page being a high-value refresh candidate (defined as a page losing >20% of its impressions while maintaining >500 historical impressions). The model achieved a 100% Precision@20 on our holdout set, vastly outperforming our heuristic baseline of 40%. This output serves as a ranked action engine, allowing editorial teams to prioritize updates based on measured opportunity.

## 2. Introduction / Problem framing
This work supports the decision of where to allocate editorial resources for content updates. The unit of analysis is the individual `content_hash_id`. The output is a ranked queue and a refresh opportunity score (0 to 1). A human editor uses this queue to review the top pages and rewrite or update them. The cost of a wrong call is wasted editorial time on a page that cannot recover, or missing a page that is rapidly losing valuable traffic. Machine learning helps here because identifying multivariate decay across impressions, clicks, and AI referral sessions is difficult to scale manually.

## 3. Data
We used the FlyRank Hugging Face release, specifically `fact_content_daily_performance` and `dim_clients`, filtering out pages with 0 impressions over the 90-day window, resulting in a dataset of 271,046 rows. We deliberately excluded future-window performance to prevent data leakage. Pseudonymous IDs (`client_hash_id`, `content_hash_id`) were used strictly for grouping during the train/test split to ensure no data leakage across clients, and were never passed as features. No client-identifying details exist anywhere in the analysis.

## 4. Methodology
We framed this as a binary classification task. We engineered features using DuckDB, aggregating the daily time-series into 30-day, 60-day, and 90-day windows to capture momentum: `imp_trend_pct`, `click_trend_pct`, and `pos_delta`. The target (`target_is_refresh_candidate`) is defined as a URL that lost at least 20% of its impressions in the last 30 days compared to the previous 30 days, filtered for pages with baseline visibility.

## 5. Results (vs baseline)
We utilized a grouped split by `client_hash_id` (GroupShuffleSplit) to prove the model generalizes to unseen domains. Our primary metric is Precision@20, as editors only review the top of the queue. We evaluate the model vs the baseline on the exact same holdout split to ensure honest discrimination.

*   **Base Rate (Majority Class):** 32.40%
*   **Baseline Precision@20:** 40.00%
*   **Model Precision@20:** 100.00%

## 6. Limitations & honest framing
The current model uses only performance metrics. Incorporating content metadata (e.g., topic, age, word count) could provide richer signals. Furthermore, the model identifies correlations but does not establish causation; a decline might be due to external factors not captured in the features. The Precision@20 metric relies on an arbitrary threshold (20% impression drop, 500 historical impressions) that might need tuning via domain expert input.

## 7. Ranked recommendations
A FlyRank editor should pull the top 20 URLs from `work/outputs/ranked_refresh_queue.csv`. Pages scoring above 0.97 require immediate factual updates or meta-tag rewrites to arrest the traffic decline, primarily driven by severe position drops (indicated by the `SEVERE_POSITION_DROP` reason code). These are directional recommendations for decision-support; they do not guarantee a ranking increase.

## 8. Reproducibility
To reproduce this work: clone the repository, install dependencies (duckdb, huggingface_hub, pandas, scikit-learn), set the Hugging Face token in the Colab secrets, and execute `work/notebooks/capstone.ipynb` top-to-bottom. Random seed `42` is enforced.

## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset (https://flyrank.ai).
