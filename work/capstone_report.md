# Capstone Report — Google Search Ranking & Discoverability

- **Author:** A D S ABHISHEK
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/addadugurudurga2024-lang/Flyrank
- **Date:** 2026-08-15

## 1. Problem framing

This work supports the decision of which content items should receive human editorial review first. The unit of analysis is a pseudonymized content item within a client grouping and observation window. The output is a ranked refresh-opportunity score and a short action queue. A human editor can use the ranking to prioritize pages for review rather than treating the score as an automatic instruction to rewrite content.

The cost of a wrong call is asymmetric: a false positive can waste editorial time, while a false negative can leave a potentially weak page unreviewed. Data and ML are useful because multiple search and engagement signals can be combined consistently into one prioritization score.

This is decision-support analysis. It does not attempt to predict Google's ranking algorithm or prove that refreshing content causes performance improvement.

## 2. Data safety

The analysis uses the FlyRank Internship Warehouse release, specifically the `fact_content_daily_performance` table. The table is daily at the report-date, pseudonymized-client, and pseudonymized-content level.

The analysis uses safe aggregate performance signals including `report_date`, `gsc_impressions`, `gsc_clicks`, `gsc_avg_position`, `ga4_pageviews`, `ga4_sessions`, `ga4_engaged_sessions`, `ga4_total_engagement_sec`, `sessions_organic`, `sessions_ai`, and `scroll_events`.

The pseudonymous fields `client_hash_id` and `content_hash_id` are retained only for grouping. They are not used as predictive model features.

Client names, domains, URLs, private queries, credentials, raw exports, and other client-identifying information are excluded from the public work.

Label-derived or future-derived fields such as `trend_direction` and `trend_pct` are not used as input features.

## 3. Baseline

The transparent baseline ranks observations using a simple search-performance rule based on average position. The baseline and machine-learning ranking are evaluated on the same held-out observations and with the same metrics.

**Baseline metrics from the final notebook run:**

- ROC-AUC: **0.858955**
- Precision@20: **0.700000**
- Recall@20: **0.016627**
- Lift@20: **2.371021**

**Held-out base rate:** **0.2952**

## 4. Model / analysis

The method is a Random Forest classifier used as a ranking model for refresh-opportunity prioritization.

Features:

- impressions
- clicks
- click-through rate
- average GSC position
- pageviews
- sessions
- engaged sessions
- total engagement seconds
- organic sessions
- AI sessions
- scroll events
- engagement rate

Pseudonymous identifiers and label-derived or future-derived fields are excluded.

### Target definition

For this compact capstone analysis, the target is a **proxy refresh-opportunity label** based on comparatively weaker search visibility and performance within the analyzed observation sample. It is used to prioritize human editorial review; it does not establish that a page objectively needs refreshing.

The model uses random seed `42` and a deterministic 80/20 stratified hold-out split.

## 5. Evaluation

The evaluation uses an 80/20 stratified hold-out split with `random_state=42`. The baseline and Random Forest model are evaluated on exactly the same test observations.

| Metric | Baseline | Random Forest |
|---|---:|---:|
| ROC-AUC | 0.858955 | 1.000000 |
| Precision@20 | 0.700000 | 1.000000 |
| Recall@20 | 0.016627 | 0.023753 |
| Lift@20 | 2.371021 | 3.387173 |

The base rate is reported next to ranking metrics because a high precision value can otherwise be misleading when the positive class is common.

False positives represent content that receives editorial attention but may not require a refresh. False negatives represent potentially useful review opportunities that remain lower in the queue. The ranking should therefore be treated as a prioritization aid rather than an automatic content decision.

## 6. Interpretation

The model combines search visibility, click activity, position, traffic, and engagement signals into a single prioritization score.

Feature importance is interpreted directionally: higher importance means that a feature contributed more to model decisions in this dataset. It does not mean that the feature causes ranking or traffic changes.

Related performance measures such as impressions, clicks, sessions, and engagement can overlap. A weak model-vs-baseline result would also be a valid finding: it would mean that additional model complexity did not provide enough evidence of better discrimination for this decision-support task.

## 7. Recommendation

The output should be used as a ranked editorial review queue.

1. **High-score content — review first.** Check search visibility, click capture, relevance, freshness, and content quality.
2. **Medium-score content — review selectively.** Compare performance with comparable content before changing the page.
3. **Low-score content — monitor.** Do not automatically rewrite or remove content solely because its score is low.
4. **Use supporting signals.** Impressions, clicks, position, traffic, and engagement provide context for human review.
5. **Re-measure after changes.** Any later movement should be described as observed movement, not proof that a refresh caused it.

Confidence is moderate for prioritization and limited for causal interpretation. The score is decision-support ranking, not a guarantee of future search performance.

## 8. Reproducibility

The analysis was developed in Google Colab and saved as `work/notebooks/capstone.ipynb`.

To reproduce the analysis:

1. Open the capstone notebook.
2. Provide a valid Hugging Face read token through the `HF_TOKEN` secret.
3. Install the required Python packages.
4. Run the notebook from top to bottom.
5. Use random seed `42`.

The notebook connects to the gated FlyRank Internship Warehouse through DuckDB and regenerates the reported evaluation metrics.

No raw dataset is committed to the repository.

## Claims checklist

- Observed / measured / directional / decision-support language is used.
- Base rate is reported next to ranking metrics.
- No causal claim is made.
- The work does not claim to predict Google's ranking algorithm.
- No client-identifying details are included.
- Pseudonymous IDs are not used as model features.
- Final numbers must match a fresh notebook run.

## Acknowledgments & data credit

Built on the FlyRank ML Internship dataset.

Source: https://flyrank.ai
