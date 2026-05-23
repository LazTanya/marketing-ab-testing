# Marketing A/B Test: Ad Attribution & Campaign ROI Analysis

## Project Overview

A marketing team ran a controlled experiment to measure the true impact of paid advertising on product conversions. Users were randomly split into two groups: an **Ad group** (treatment) who saw the product advertisement, and a **PSA group** (control) who saw a Public Service Announcement in the same placement and size.

The PSA control design is methodologically significant — it isolates the ad content itself as the variable, controlling for the mere exposure effect (the possibility that simply seeing *something* in an ad slot influences behavior regardless of content).

### The Two Business Questions

1. **Was the campaign statistically successful?** Did the ad group convert at a meaningfully higher rate than the control?
2. **How much revenue can be attributed to the ads?** This is the budget justification question — the difference between a campaign that *worked* and one that was *worth the spend*.

---

## Dataset

**Source:** [Marketing A/B Testing — Kaggle](https://www.kaggle.com/datasets/faviovaz/marketing-ab-testing/data)

**Size:** 588,101 rows × 6 columns (one row per unique user)

| Column | Type | Description |
|--------|------|-------------|
| `user_id` | int | Unique user identifier |
| `test_group` | string | `ad` = saw advertisement, `psa` = saw public service announcement |
| `converted` | bool | `True` if user purchased, `False` otherwise |
| `total_ads` | int | Total number of ads seen by the user |
| `most_ads_day` | string | Day of the week the user saw the most ads |
| `most_ads_hour` | int | Hour of day (0–23) the user saw the most ads |

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python (pandas, numpy) | Data manipulation and feature engineering |
| scipy, statsmodels | Statistical hypothesis testing |
| matplotlib, seaborn | Data visualization |
| Jupyter Notebook | Analysis environment and narrative documentation |

---

## Analytical Approach

The notebook is structured as five sequential acts. Each act builds on the previous and moves from data validation through to a business decision.

### Act 1 — Experiment Validation

Before analyzing results, I validated the experiment design itself. 

Four checks were performed:

| Check | Result |
|-------|--------|
| Null values | 0 across all 6 columns |
| Duplicate user IDs | 0 — one row per user confirmed |
| Sample Ratio Mismatch (Chi-squared test) | Pass — 96/4 split matches intended design |
| Group contamination | 0 users appeared in both groups |

**Conclusion:** All validation checks passed. The experiment design is sound and the dataset is cleared for analysis.

### Act 2 — EDA & Behavioral Profiling

Three key behavioral patterns were identified:

**Ad Exposure Distribution**
The distribution of `total_ads` is heavily right-skewed (skewness = 7.50). The median user saw 13 ads; the mean was 24.8, inflated by a long tail of heavy-exposure users. The maximum was 2,065 ads. Only 3.9% of users saw more than 100 ads.

**Dose-Response Curve**
Conversion rate increases monotonically with ad frequency across all six exposure buckets:

| Frequency Bucket | Users | Conversion Rate |
|-----------------|-------|----------------|
| 1–5 ads | 169,962 | 0.25% |
| 6–15 ads | 146,101 | 0.61% |
| 16–30 ads | 122,920 | 1.40% |
| 31–60 ads | 74,470 | 5.00% |
| 61–100 ads | 29,070 | 13.39% |
| 100+ ads | 22,054 | 17.14% |

**Important:** This monotonic increase may reflect selection bias rather than causal persuasion — users with pre-existing purchase intent naturally accumulate more impressions before buying. A randomized frequency cap experiment would be required to establish causality.

**Temporal Analysis**
- Best performing day: **Monday** (3.32% CR vs 2.55% overall)
- Best performing hour: **16:00** (3.09% CR)

### Act 3 — Hypothesis Testing

**Null Hypothesis (H₀):** The ad has no effect. Any difference in conversion rates is due to random chance.

**Test used:** Two-proportion z-test (appropriate for comparing binary outcome rates between two independent groups).

| Metric | Value |
|--------|-------|
| Ad group conversion rate | 2.55% (14,423 / 564,577) |
| PSA group conversion rate | 1.79% (420 / 23,524) |
| Absolute lift | +0.77 percentage points |
| Relative lift | +43.1% |
| Z-statistic | 7.37 |
| P-value | 1.71e-13 |
| 95% CI on lift | [0.595pp, 0.943pp] |
| Cohen's h | 0.053 (small) |

**Result:** We reject H₀. The lift is statistically significant at p = 1.71e-13.

**On effect size:** Cohen's h of 0.053 classifies the effect as small on a standardized scale. This does not contradict the hypothesis test result — statistical significance and effect size answer different questions. With 588,101 users, the test detects even modest real effects with certainty. The commercial value of a small-but-real effect at this scale is evaluated in Act 4.

### Act 4 — Revenue Attribution Model

The PSA group serves as the counterfactual baseline — the conversion rate we would expect with no advertising. This allows us to isolate *incremental* conversions that are directly attributable to the ads.

**Model assumptions**:
- Average Order Value (AOV): $45.00
- Cost per 1,000 impressions (CPM): $5.00

| Metric | Value |
|--------|-------|
| Total ad group conversions | 14,423 |
| Baseline converters (organic intent) | 10,080 |
| **Incremental converters (ad-driven)** | **4,343** |
| Attribution rate | 30.1% of conversions credited to ads |
| Total revenue (ad group) | $649,035 |
| Attributable revenue | $195,434 |
| Estimated ad spend | $70,074 |
| **Net attributable profit** | **$125,361** |
| **ROAS** | **2.79x** |
| **ROI** | **178.9%** |
| Cost per incremental conversion | $16.13 |
| Breakeven AOV | $16.13 |
| Breakeven CPM | $13.94 |

**Sensitivity analysis** was performed across a 5×7 matrix of AOV ($20–$80) and CPM ($2–$12) combinations. Of 35 tested scenarios, only 4 produced negative ROI — all requiring a sub-$25 AOV combined with above-market CPM. The campaign has wide margins of safety on both dimensions.

### Act 5 — Executive Recommendations

---

## ✅ Verdict: SHIP

The campaign is statistically proven and financially positive. It should be continued.

---

### Recommendations

**1. Continue and Scale the Campaign**
The 178.9% ROI and $125,361 net profit provide a clear case for scaling. Recommend increasing spend while monitoring ROAS weekly. Set a ROAS floor of 1.5x as the threshold for campaign review.

**2. Implement an Impression Frequency Cap**
The dose-response curve shows conversion peaks in the 61–100 ad bucket. Cap impressions at 100 per user per campaign cycle and reallocate excess budget toward reaching new unique users. This expands reach without increasing total spend.

**3. Weight Delivery Toward Peak Windows**
Monday delivers a 3.32% CR vs the 2.55% overall average. Configure dayparting to increase bid multipliers on Monday afternoons (14:00–18:00). Target: shift 20–30% of weekly budget toward this window.

**4. Set CPM Guardrails for Media Buying**
The breakeven CPM is $13.94. Set a hard CPM ceiling of $12.00 in platform bidding rules — providing a $1.94 buffer and preventing overspend on premium placements.

---

### Limitations

**Assumed AOV and CPM:** Revenue and spend figures were not in the dataset. All financial conclusions depend on the $45 AOV and $5 CPM assumptions. 

**Frequency-conversion causality:** The dose-response curve may reflect selection bias. Users with pre-existing purchase intent naturally accumulate more impressions before converting. A randomized frequency cap test is required to establish true causal frequency effects.

**Short-term measurement window:** The analysis captures direct conversions within the experiment window only. Brand awareness effects, repeat purchase behavior, and customer lifetime value differences between groups are not captured. True campaign ROI is likely higher than reported.

**External validity:** Results reflect this specific campaign, product, audience, and time period. Performance may vary across seasons, geographies, or audience segments not represented in this data.

---


## Repository Structure

```
marketing-ab-test/
├── README.md
├── notebooks/
│   └── ab_analysis.ipynb        # Full analysis — Acts 1 through 5
├── data/
│   └── marketing_ab_data.csv         # Raw dataset (from Kaggle)
