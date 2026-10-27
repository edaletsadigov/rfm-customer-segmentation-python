# Dataset Overview — `rfm_sales.csv`

A transactional sales file: **one row = one order line** (one product on one order). It is the only input of the RFM segmentation in this repository.

## At a glance

| Item | Value |
|---|---|
| File | `rfm_sales.csv` (≈ 92 MB, UTF-8, comma-separated) |
| **Rows** | **500,000** order lines |
| **Columns** | **25** (6 integer, 3 float, 16 text) |
| Orders | 150,000 unique `ORDERNUMBER` (3.3 lines per order on average) |
| Customers | 25,097 unique `CUSTOMERNAME` |
| Products | 225 product codes in 7 product lines |
| Geography | 19 countries, 27 cities |
| Period | 2022-01-01 → 2025-12-31 (1,461 days, every day present) |
| Total sales | $155,293,902 (including cancelled orders) |
| Duplicate rows | 0 |
| Schema | Same columns as `sales_data_sample.csv` (Kaggle “Sample Sales Data”) but a much larger file |

> The file is **not stored in this repository** (92 MB is above GitHub's 50 MB warning size). Put it next to the notebook as `rfm_sales.csv` before running.

## Order status

| Status | Order lines | Share of lines |
|---|---|---|
| Shipped | 445,199 | 89.0% |
| Resolved | 19,519 | 3.9% |
| Cancelled | 10,397 | 2.1% |
| On Hold | 10,077 | 2.0% |
| In Process | 9,827 | 2.0% |
| Disputed | 4,981 | 1.0% |

## Volume over time

| Year | Orders | Sales |
|---|---|---|
| 2022 | 26,348 | $27.2M |
| 2023 | 26,182 | $27.0M |
| 2024 | 35,569 | $36.9M |
| 2025 | 61,901 | $64.2M |

Order volume is 2.3× higher in 2025 than in 2022, so a large part of the customer base is recent. Q4 is the busiest quarter (29.6% of lines).

## Columns

| # | Column | Type | Missing | Distinct | Meaning | Example values |
|---|---|---|---|---|---|---|
| 1 | `ORDERNUMBER` | int64 | 0 (0.0%) | 150,000 | Order identifier. One order has one or more lines; one order always belongs to one customer, one date and one status. | `104396`, `105216` |
| 2 | `QUANTITYORDERED` | int64 | 0 (0.0%) | 16 | Units ordered on the line (values 1–50; 1 and 2 units make up half of all lines). | `4`, `5` |
| 3 | `PRICEEACH` | float64 | 0 (0.0%) | 1,469 | Unit price actually charged. On average $5.84 below `MSRP`. | `76.3`, `169.68` |
| 4 | `ORDERLINENUMBER` | int64 | 0 (0.0%) | 6 | Line number inside the order (1–6). Not a unique key: 11,433 (order, line) pairs repeat. | `1`, `2` |
| 5 | `SALES` | float64 | 0 (0.0%) | 16,321 | Line revenue in $ = `QUANTITYORDERED` × `PRICEEACH` (verified on every row). Used as **Monetary** in the RFM analysis. | `305.2`, `848.4` |
| 6 | `ORDERDATE` | text | 0 (0.0%) | 1,461 | Order date, `YYYY-MM-DD`, 2022-01-01 to 2025-12-31 (every day present). Stored as text in the CSV. | `2022-01-01`, `2022-01-02` |
| 7 | `STATUS` | text | 0 (0.0%) | 6 | Order status: Shipped, Resolved, Cancelled, On Hold, In Process, Disputed. | `Shipped`, `Disputed` |
| 8 | `QTR_ID` | int64 | 0 (0.0%) | 4 | Calendar quarter (1–4) of `ORDERDATE`; consistent with the date on every row. | `1`, `2` |
| 9 | `MONTH_ID` | int64 | 0 (0.0%) | 12 | Calendar month (1–12) of `ORDERDATE`; consistent with the date on every row. | `1`, `2` |
| 10 | `YEAR_ID` | int64 | 0 (0.0%) | 4 | Calendar year of `ORDERDATE` (2022–2025); consistent with the date on every row. | `2022`, `2023` |
| 11 | `PRODUCTLINE` | text | 0 (0.0%) | 7 | Product category (7 lines: Motorcycles, Classic Cars, Trains, Planes, Ships, Vintage Cars, Trucks and Buses). | `Trains`, `Motorcycles` |
| 12 | `MSRP` | float64 | 0 (0.0%) | 222 | Manufacturer suggested retail price per unit; fixed per `PRODUCTCODE`. | `76.3`, `169.68` |
| 13 | `PRODUCTCODE` | text | 0 (0.0%) | 225 | Product identifier (225 products); each code belongs to exactly one product line. | `S319`, `S190` |
| 14 | `CUSTOMERNAME` | text | 0 (0.0%) | 25,097 | Customer name in the form `Cust_<id> <company>`; unique per customer (one phone, address and contact per name). Used as the customer key in the RFM analysis. | `Cust_16943 Omni Designs`, `Cust_05635 Apex Diecast` |
| 15 | `PHONE` | text | 0 (0.0%) | 25,097 | Customer phone number; one per customer. | `+59 692 713 1445`, `+14 826 178 1433` |
| 16 | `ADDRESSLINE1` | text | 0 (0.0%) | 22,126 | Street address. | `4879 King St.`, `8669 Washington St.` |
| 17 | `ADDRESSLINE2` | text | 268,099 (53.6%) | 4 | Second address line (Floor 2, Building C, Suite 100, Apt. 4B). Missing for 53.6% of rows. | `Building C`, `Apt. 4B` |
| 18 | `CITY` | text | 0 (0.0%) | 27 | Customer city (27 cities; Allentown, Tokyo, Oslo, Stockholm and New Bedford are the most frequent). | `New Bedford`, `NYC` |
| 19 | `STATE` | text | 153,570 (30.7%) | 21 | State, province or region. Filled for USA and Canada, mostly empty elsewhere (49–77% missing per country). Contains a few non-state values such as `Paris` and `Bavaria`. | `NY`, `CA` |
| 20 | `POSTALCODE` | text | 0 (0.0%) | 22,371 | Postal code, stored as text. | `39125`, `62462` |
| 21 | `COUNTRY` | text | 0 (0.0%) | 19 | Customer country (19 countries; USA is 47.8% of lines). | `USA`, `France` |
| 22 | `TERRITORY` | text | 0 (0.0%) | 3 | Sales territory: `EMEA`, `APAC` or `NA` (North America = USA and Canada). **`NA` is text, not a missing value** — see the note below. | `NA`, `EMEA` |
| 23 | `CONTACTLASTNAME` | text | 0 (0.0%) | 29 | Contact person last name (29 distinct values). | `Nelson`, `Pedro` |
| 24 | `CONTACTFIRSTNAME` | text | 0 (0.0%) | 30 | Contact person first name (30 distinct values). | `Armand`, `Julie` |
| 25 | `DEALSIZE` | text | 0 (0.0%) | 3 | Size class derived from `SALES`: Small < $3,000, Medium $3,000–7,000, Large > $7,000. 99.7% of lines are Small, so it separates customers poorly. | `Small`, `Medium` |

Numeric ranges: `QUANTITYORDERED` 1–50 (mean 3.54), `PRICEEACH` $22.60–$216.82 (mean $87.63), `SALES` $22.60–$10,458.50 (mean $310.59, median $201.21 — right-skewed), `MSRP` $30.14–$212.57.

## Data quality notes

1. **`TERRITORY = "NA"` is North America, not a null.** `pd.read_csv` converts the text `NA` to `NaN` by default, which makes 55.8% of the rows look “missing”. Read the file with `keep_default_na=False` to keep it.
2. **`ADDRESSLINE2` stores missing values as the text `nan`** (a leftover of an earlier export). The notebook reads it with `na_values=['', 'nan']`, which gives the true 53.6% missing.
3. **`STATE` is missing by design** outside the USA and Canada; it is not an error and is not used in the analysis.
4. **`ORDERLINENUMBER` repeats inside orders** (1,423 orders have more than 6 lines while the maximum line number is 6), so do not use `(ORDERNUMBER, ORDERLINENUMBER)` as a unique key.
5. **`DEALSIZE` is derived from `SALES`** and is 99.7% Small.
6. **Extreme concentration.** The largest customer has 11,804 orders and $13.2M of sales (8.7% of non-cancelled revenue). Together with the `Cust_<id>` naming pattern, this suggests the data may be synthetically generated; treat business conclusions as a demonstration of the method unless the source is confirmed.

## Columns used in the RFM analysis

| RFM element | Column(s) |
|---|---|
| Customer key | `CUSTOMERNAME` |
| Recency | `ORDERDATE` (days from the last order to 2026-01-01) |
| Frequency | `ORDERNUMBER` (count of unique orders) |
| Monetary | `SALES` (sum) |
| Filter | `STATUS` (`Cancelled` orders excluded) |

All other columns are descriptive and are not used in the segmentation.

