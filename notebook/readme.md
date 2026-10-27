# Notebook Guide — `RFM_python_analysis.ipynb`

The notebook builds a rule-based **RFM (Recency, Frequency, Monetary) segmentation** of 24,916 customers from raw order lines, names six business segments and attaches a marketing action to each. It runs top to bottom without errors and all outputs are saved in the file.

## Libraries — what is used and why

All imports are in `common.py`, loaded with `from common import *`.

| Library | Import | Where it is used | Why it was chosen |
|---|---|---|---|
| **pandas** | `import pandas as pd` | Every section: reading the CSV, `to_datetime`, `groupby().agg()`, `pd.qcut`, `rank`, `melt`, `corr`, `to_csv` | The task is customer-level aggregation of 500,000 order lines; `groupby` + named aggregation is the shortest and most readable way to build the RFM table. `pd.qcut` is required by the task for quantile scoring. |
| **NumPy** | `import numpy as np` | `np.select` in `assign_segment` | Applies the ordered segment rules to all 24,916 customers at once (vectorised), instead of a slow row-by-row `apply`. |
| **Matplotlib** | `import matplotlib.pyplot as plt` | Figure and axis layout, bar labels, log-scaled scatter | Full control over titles, labels, log axes and annotations for the required charts. |
| **Seaborn** | `import seaborn as sns` | `histplot`, `barplot`, `scatterplot`, `heatmap`, `set_theme` | Statistical charts with a consistent style in one line each; it handles `hue` for segment colours and annotated heatmaps. |
| **Plotly Express** | `import plotly.express as px` | `px.scatter_3d` (bonus 3D view) | Matplotlib 3D cannot rotate or hover in a notebook; Plotly gives an interactive 3D plot with hover on `customer_name`. |
| **SciPy** | `from scipy import stats as st` | Imported in `common.py` but **not used** in this notebook | Kept as the project's standard import for statistical tests; the Spearman correlation is computed with `DataFrame.corr(method='spearman')`. |

## Notebook structure

| Section | What happens | Key decisions |
|---|---|---|
| **Libraries / Loading** | Imports, `pd.read_csv(..., keep_default_na=False, na_values=['', 'nan'])`, `head`, shape | `keep_default_na=False` keeps `NA` (North America) as text instead of `NaN` (see the [dataset guide](README_dataset.md)). |
| **1. Data Overview and Cleaning** | `info`, `describe`, nulls, duplicates, order-level integrity checks, date conversion, status table, filter | Verified that one order = one customer / date / status, and `SALES = quantity × price`. **`Cancelled` orders are excluded** (3,080 orders, 2.1%); 181 customers with only cancelled orders drop out. |
| **2. RFM Calculation** | `reference_date = max(order_date) + 1 day` (2026-01-01); `groupby('customer_name').agg(...)` | Frequency counts **unique orders** (`nunique`), not rows. Distribution charts show strong skew. |
| **3. Quantile Scoring** | `pd.qcut(metric.rank(method='first'), 4, labels=...)`; `RFM_Segment` string | Recency labels are inverted (`[4, 3, 2, 1]`). `rank(method='first')` is needed because raw Frequency has duplicate quartile edges and `qcut` fails. The tie-breaking limitation is measured. |
| **4. Segment Definition** | `assign_segment()` with `np.select`; summary table | Six segments, rules checked in order, first match wins; coverage verified (no customer without a segment). |
| **5. Visualizations** | Segment sizes, customer vs revenue share, Recency vs Monetary, score heatmap, 3D scatter | Every chart has a title, axis labels and a printed `Takeaway` with a computed number. |
| **6. Statistical and Robustness Checks** | Spearman correlation, revenue concentration, tie-break sensitivity | Spearman because all metrics are skewed; sensitivity test shows which segments are stable. |
| **7. Marketing Actions** | Action table per segment with KPIs | Actions follow the R/F/M profile of each segment. |
| **8. Export** | `rfm_segments_report.csv` | `utf-8-sig` so the file opens correctly in Excel. |
| **9. Final Summary** | Findings and recommendations | Numbers in the text are the computed values from the sections above. |

## Functions and methods worth knowing

| Function | Purpose | Why this and not an alternative |
|---|---|---|
| `groupby().agg(name=(col, func))` | Build Recency/Frequency/Monetary per customer | Named aggregation gives readable column names without a `MultiIndex`. |
| `Series.nunique()` on `order_number` | Frequency | One order has several lines; `count()` would multiply Frequency by the number of lines. |
| `pd.qcut(..., 4, labels=...)` | Equal-size quartile scores | Fixed cut-offs would be dominated by a few very large customers; ranks are not. |
| `Series.rank(method='first')` | Make quartile edges unique | Breaks ties by row order so `qcut` always returns four groups. |
| `np.select(conditions, names, default=...)` | Segment assignment | Ordered rules in one vectorised call; the `default` is `Needs Attention`. |
| `pd.Categorical(..., ordered=True)` | Fixed segment order | Tables and charts always list segments Champions → Lost. |
| `DataFrame.corr(method='spearman')` | Relationship between R, F and M | Rank correlation is robust to the heavy skew. |
| `ax.set_yscale('log')` | Scatter of Monetary | Monetary ranges from $26 to $13M; a linear axis would squash 99% of the points. |

## Reproduce

```bash
pip install pandas numpy matplotlib seaborn plotly scipy jupyter
# put rfm_sales.csv in the same folder as the notebook, then
jupyter notebook RFM_python_analysis.ipynb
```

Run all cells in order. The notebook writes `rfm_segments_report.csv` next to itself. Python 3.10+ and pandas 2.x/3.x are expected.

## Outputs

| File | Content |
|---|---|
| `rfm_segments_report.csv` | 24,916 rows × 9 columns: `customer_name, recency, frequency, monetary, r_score, f_score, m_score, rfm_segment, segment` |
| `charts/*.png` | Static copies of the six charts ([chart guide](../charts/README.md)) |

## Known limitations of the code

- `rank(method='first')` splits customers with identical Frequency into different F scores (about 60% of customers have an F score partly decided by tie-breaking). Section 6.3 measures the effect: 6.4% of customers change segment.
- The 3D chart uses a random sample of 5,000 customers (`random_state=42`) to keep the notebook light.

