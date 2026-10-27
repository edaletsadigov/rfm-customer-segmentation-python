# Chart Guide

Six charts from `RFM_python_analysis.ipynb`, in the order they appear. Each entry has a link to the PNG, how to read it and what it tells us. Segment colours are the same in every chart.

| # | Chart | File |
|---|---|---|
| 1 | [RFM metric distributions](#1-rfm-metric-distributions) | [`01_rfm_metric_distributions.png`](01_rfm_metric_distributions.png) |
| 2 | [Segment sizes](#2-segment-sizes) | [`02_segment_sizes.png`](02_segment_sizes.png) |
| 3 | [Customer share vs revenue share](#3-customer-share-vs-revenue-share) | [`03_customer_vs_revenue_share.png`](03_customer_vs_revenue_share.png) |
| 4 | [Recency vs Monetary by segment](#4-recency-vs-monetary-by-segment) | [`04_recency_vs_monetary_scatter.png`](04_recency_vs_monetary_scatter.png) |
| 5 | [Average RFM score heatmap](#5-average-rfm-score-heatmap) | [`05_rfm_score_heatmap.png`](05_rfm_score_heatmap.png) |
| 6 | [3D view of R, F and M](#6-3d-view-of-recency-frequency-and-monetary) | [`06_rfm_3d_scatter.png`](06_rfm_3d_scatter.png) |

Segment colours: Champions (blue), Loyal (green), Promising (cyan), Needs Attention (orange), At Risk (red), Lost (grey).

---

## 1. RFM metric distributions

**File:** [`01_rfm_metric_distributions.png`](01_rfm_metric_distributions.png)

![RFM metric distributions](01_rfm_metric_distributions.png)

**What it shows.** Histograms of Recency (days since last order), Frequency (orders) and Monetary (total sales) for the 24,916 scored customers. Frequency and Monetary use a log x-axis.

**Analysis**
- **Recency is spread widely**: median 201 days, 25% of customers ordered within the last 72 days and 25% about 435 days ago or more (maximum 1,461).
- **Frequency is extremely skewed**: 35.0% of customers placed exactly one order and the median is 2, but the maximum is 11,804. The mean (5.9) is three times the median.
- **Monetary is skewed the same way**: median $2,230 against a mean of $6,104; the top value is $13.2M.
- **Why it matters:** fixed thresholds (e.g. “more than $10,000 is high”) would be set by a handful of very large accounts. This is the reason the analysis scores customers by quartile rank (`pd.qcut`).

**Caveat.** The log axes hide how few customers sit at the far right; the long tail is real but thin.

---

## 2. Segment sizes

**File:** [`02_segment_sizes.png`](02_segment_sizes.png)

![Segment sizes](02_segment_sizes.png)

**What it shows.** Number of customers in each of the six RFM segments, sorted from largest to smallest, with the share of all customers.

**Analysis**
- **Needs Attention is the largest segment** (6,078 customers, 24.4%) — a mixed middle group with average scores on most dimensions.
- **Loyal (4,916, 19.7%) and Lost (4,712, 18.9%)** come next; almost one customer in five is already inactive and low-value.
- **Champions are 3,105 customers (12.5%)** and **At Risk is the smallest group (2,579, 10.4%)**.
- Roughly **one in three customers (32.2%) is Champions or Loyal**, and **34.7% are Needs Attention or At Risk**, i.e. drifting or already quiet.

**Caveat.** Size says nothing about value — see chart 3. The size of At Risk and Needs Attention is sensitive to how Frequency ties are broken (At Risk 2,579 → 3,056 in the sensitivity test).

---

## 3. Customer share vs revenue share

**File:** [`03_customer_vs_revenue_share.png`](03_customer_vs_revenue_share.png)

![Customer share vs revenue share](03_customer_vs_revenue_share.png)

**What it shows.** For each segment, the share of customers (light blue) next to the share of revenue (dark blue).

**Analysis**
- **Champions: 12.5% of customers produce 60.4% of revenue** — 4.8 times their customer share.
- **Loyal is proportional** (19.7% of customers, 19.0% of revenue). Champions + Loyal together are 32.2% of customers and **79.4% of revenue**.
- **Promising, Needs Attention and Lost carry far less revenue than their size**: 14.2% / 24.4% / 18.9% of customers but only 3.4% / 6.5% / 2.9% of revenue (0.2–0.3 times their customer share).
- **At Risk is the notable exception among inactive groups**: 10.4% of customers and 7.9% of revenue (0.8 times), because these customers used to buy often and in volume.

**Caveat.** One account alone is 8.7% of all revenue and sits in Champions, so the 60.4% is concentrated in a small number of very large customers.

---

## 4. Recency vs Monetary by segment

**File:** [`04_recency_vs_monetary_scatter.png`](04_recency_vs_monetary_scatter.png)

![Recency vs Monetary by segment](04_recency_vs_monetary_scatter.png)

**What it shows.** Every customer as a point: days since last order (x) against total sales on a log scale (y), coloured by segment.

**Analysis**
- **Champions** occupy the top-left: ordered within the last 72 days and spent more than $4,922 (median recency 25 days, median spend $12,426). A few points reach $1M–$13M.
- **Lost customers** fill the bottom-right: last order about 435 days ago or more and low spend (median 889 days, $844).
- **At Risk (red)** forms a band above $2,230 but beyond 200 days: valuable customers who have gone quiet. **Promising (cyan)** is the mirror image — recent, but with few orders and a lower typical spend (median $1,222).
- The spend level falls as recency grows: the further back the last order, the lower the typical total spend, which matches the negative Spearman correlation (ρ = −0.53).

**Caveat.** The segments are defined by rules on score cut-offs, so the straight vertical and horizontal borders are a result of the rules, not a natural cluster structure. The chart checks the rules; it does not discover groups. The negative correlation is an association, not a cause.

---

## 5. Average RFM score heatmap

**File:** [`05_rfm_score_heatmap.png`](05_rfm_score_heatmap.png)

![Average RFM score heatmap](05_rfm_score_heatmap.png)

**What it shows.** Average R, F and M score (1 = worst, 4 = best) per segment.

**Analysis**
- **Champions** score 4.00 on all three (the rule).
- **Loyal** is strong across the board (R 3.30, F 3.48, M 3.44).
- **At Risk has high F and M (3.29, 3.33) but a low R (1.78)**, while **Promising has the opposite profile (R 3.37, F 1.59, M 1.78)**. These two groups need opposite actions: win back vs nurture.
- **Lost has R = 1.00** and low F and M (1.38, 1.39).
- **F and M move together in every segment** (e.g. Loyal 3.48 / 3.44, At Risk 3.29 / 3.33), consistent with their rank correlation of 0.87. The M score largely repeats the F score; Recency is the independent dimension.

**Caveat.** Averages of scores are a profile summary, not a measure of distance between customers.

---

## 6. 3D view of Recency, Frequency and Monetary

**File:** [`06_rfm_3d_scatter.png`](06_rfm_3d_scatter.png)

![3D view of Recency, Frequency and Monetary](06_rfm_3d_scatter.png)

**What it shows.** A random sample of 5,000 customers (`random_state=42`) in R/F/M space. Frequency and Monetary are on log scales.

**Analysis**
- The customers form **one long diagonal band**, from “recent, frequent, high spend” (Champions, top left) to “old, rare, low spend” (Lost, bottom right): R, F and M are not independent.
- **Champions form the narrow upper tail** and **Lost forms the long, flat grey tail** at low frequency and low spend.
- **Needs Attention (orange) overlaps the other groups** — it sits in the middle of the cloud and has the least clear identity.
- **At Risk (red)** sits above the Lost tail: higher frequency and spend, but already beyond the recent zone.

**Caveat.** This PNG is a static copy. The notebook contains the **interactive Plotly version** (rotate, zoom, hover on customer name), which is easier to read. In the notebook the Plotly colours are identical.

