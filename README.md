# Customer Segmentation — RFM Analysis & K-Means Clustering

Segmenting 5,771 customers of a UK online retailer into four behavioural groups using RFM (Recency, Frequency, Monetary) features and K-Means clustering, on the [Online Retail II dataset](https://archive.ics.uci.edu/dataset/502/online+retail+ii) (about two years of transactions). Built end to end: cleaning → RFM feature engineering → outlier handling → scaling → clustering → segment profiling.

**Headline finding:** the most valuable segment is 22% of customers but about 64% of total spend, while a lapsing segment (25% of customers, about 23% of spend) has not bought in roughly eight months despite averaging £1,710 across 4.6 orders. That makes it the most obvious win-back target.

---

## Results at a Glance

| Segment (cluster) | Customers | Avg recency | Avg orders | Avg spend | Share of spend* |
|---|---|---|---|---|---|
| **Most Valuable** (2) | 1,273 (22.1%) | 31 days | 13.8 | £5,527 | **63.8%** |
| **Likely to Churn** (1) | 1,455 (25.2%) | 247 days | 4.6 | £1,710 | 22.6% |
| **Recently Acquired** (0) | 1,187 (20.6%) | 30 days | 2.8 | £790 | 8.5% |
| **Least Engaged** (3) | 1,856 (32.2%) | 401 days | 1.3 | £302 | 5.1% |

\*Approximate: each segment's average spend multiplied by its customer count, as a share of the total across all 5,771 clustered customers.

**Silhouette score: 0.37** — moderate separation. The segments overlap at the edges, which is typical for behavioural data.

---

## Dataset

Online Retail II (UCI Machine Learning Repository): 1,067,371 transaction lines from a UK-based online retailer selling gift-ware, 1 December 2009 – 9 December 2011. Many of its customers are wholesalers, which is why average spend per customer is high. Prices are in pounds sterling.

The notebook reads the CSV version (`online_retail_II.csv`), which is not included in this repo because of its size.

**Attribution (CC BY 4.0):** Chen, D. (2012). *Online Retail II* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5CG6D

---

## Method

**1. Cleaning.** 243,007 rows (22.8%) had no Customer ID and were dropped, since they can't be attributed to a customer. Cancellations (invoices containing "C") were removed, and only rows with positive quantity and price were kept. A `TotalSum` column (`Quantity × Price`) was added. Result: 1,067,371 → 805,549 rows (24.5% removed).

**2. RFM features.** Per customer, relative to a snapshot date one day after the last transaction in the data:
- **Recency:** days since the customer's last purchase
- **Frequency:** number of unique invoices
- **Monetary:** total spend

**3. Outlier removal.** Percentile-based fences on each feature (5th and 95th percentile, extended by 1.5× their range), leaving 5,771 customers.

**4. Transformation.** `log1p` to reduce right skew, then `StandardScaler`, because K-Means is distance-based and sensitive to scale.

**5. Choosing k.** Elbow method over k = 1–10. The curve drops steeply to k = 2 and then flattens gradually, with no sharp elbow, so k = 4 was a judgement call: it was chosen because it gave four distinct, interpretable segments.

![Elbow method for choosing k](images/elbow_method.png)

**6. Profiling.** Mean RFM values per cluster, then business names for each segment (below).

![Segments: recency vs. monetary](images/segments_recency_vs_monetary.png)

![Segments: frequency vs. monetary](images/segments_frequency_vs_monetary.png)

The segments overlap on these raw axes because the clustering was done on log-scaled, standardised values.

---

## Segment Profiles & Suggested Actions

| Segment | Profile | Suggested action |
|---|---|---|
| **Most Valuable** | Bought within the last month on average, 13.8 orders, £5,527 spend | Protect and reward: loyalty perks, early access, priority service |
| **Likely to Churn** | Last purchase about 8 months ago, but 4.6 orders and £1,710 spend | Win-back campaign. Highest priority among lapsed customers because of their past value |
| **Recently Acquired** | Recent buyers with few orders (2.8) and modest spend | Nurture towards a second and third purchase |
| **Least Engaged** | Over a year since last purchase, about 1.3 orders, £302 spend | Low-cost reactivation, or deprioritise |

The actions are suggestions based on the segment profiles, not tested results.

---

## Limitations & Future Work

- **Segment names are interpretations.** The data has no sign-up date and no churn label, so "Recently Acquired" and "Likely to Churn" are inferred from recency and frequency. To check the churn label properly, segment customers using the first year of data and see who actually buys in the second.
- **k = 4 is a judgement call.** There's no sharp elbow, and the silhouette score was only computed for k = 4. A fuller comparison would score k = 2–8 and try other algorithms (hierarchical clustering, Gaussian mixture).
- **Outlier removal drops the biggest spenders.** No customer above roughly £21,000 appears in the plots, so the segments describe typical customers, and very large wholesale accounts would need separate treatment. The notebook doesn't log how many customers were removed.
- **Soft boundaries.** With a silhouette of 0.37, customers near segment edges could reasonably belong to a neighbouring group.
- **A few high-value customers have gone quiet.** Recency in the Most Valuable segment ranges from 1 to 497 days, so some members have not bought for over a year. Flagging those separately would be a useful refinement.
- **Only three features.** Adding average order value or product mix could separate customers that RFM treats as identical.
- **Recency is relative to the end of the data** (December 2011), not to today.

---

## Tools

Python, pandas, NumPy, scikit-learn (KMeans, StandardScaler, silhouette score), matplotlib, seaborn.

## How to Run

1. Download the Online Retail II CSV and save it as `online_retail_II.csv` in the same folder as the notebook.
2. Install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Open `customer_segmentation_rfm_kmeans.ipynb` and run all cells. The notebook also saves `sample_customers_per_cluster.csv` (7 random customers per segment) for spot-checking.
