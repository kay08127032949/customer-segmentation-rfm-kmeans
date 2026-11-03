# Customer Segmentation using RFM Analysis and K-Means Clustering

Segmenting 5,771 customers of a UK-based online retailer into four behavioural groups using Recency, Frequency and Monetary (RFM) features and K-Means clustering, so that marketing and retention effort can be targeted at the right customers.

**Dataset:** Online Retail II (transactions from a UK online retailer, December 2009 to December 2011), 1,067,371 rows and 8 columns.
**Tools:** Python, Pandas, NumPy, Scikit-learn, Matplotlib, Seaborn, Jupyter Notebook.

---

## Project Goal

Not every customer is worth the same. A retailer that treats a customer who bought 14 times last month the same as one who bought once last year wastes money and loses its best customers. This project answers:

- Who are the most valuable customers?
- Which customers are slipping away and worth a win-back campaign?
- Which customers are barely engaged?

## Workflow

### 1. Data Cleaning
Starting from 1,067,371 transaction rows, the cleaned dataset has 805,549 valid rows after:

- Dropping rows with no Customer ID (243,007 rows), since sales cannot be linked to a customer.
- Removing cancelled invoices (Invoice numbers starting with "C").
- Removing rows with zero or negative Quantity or Price.
- Creating a `TotalSum` (Quantity x Price) column for revenue.

### 2. RFM Feature Engineering
Using a snapshot date one day after the last transaction, one row per customer was built:

| Feature | Meaning |
|---|---|
| Recency | Days since the customer's last purchase |
| Frequency | Number of unique invoices |
| Monetary | Total amount spent |

### 3. Outlier Removal and Scaling
- Removed extreme outliers in Recency, Frequency and Monetary using an IQR rule based on the 5th and 95th percentiles, leaving 5,771 customers.
- Applied a log transformation (`log1p`) to reduce skew, then standardised the features with `StandardScaler` so no single feature dominates the clustering.

### 4. Clustering
- Used the Elbow Method (WCSS for k = 1 to 10) to select **k = 4**.
- Fitted K-Means (`k-means++`, `random_state=42`).
- Validated the result with a **Silhouette Score of 0.37**, which indicates moderate cluster separation (real customer behaviour is continuous, so some overlap between groups is expected).

## Results

| Segment | Customers | Avg Recency (days) | Avg Orders | Avg Spend |
|---|---|---|---|---|
| Most Valuable Customers | 1,273 (22%) | 31 | 13.8 | 5,527 |
| Customers Likely to Churn | 1,455 (25%) | 247 | 4.6 | 1,710 |
| Recently Acquired Customers | 1,187 (21%) | 30 | 2.8 | 790 |
| Least Engaged Customers | 1,856 (32%) | 401 | 1.3 | 302 |

(Spend is in the dataset's currency, pounds sterling.)

### Key Insights
- **The Most Valuable group is only 22% of customers but accounts for roughly 64% of the revenue** among the segmented customers (calculated from each segment's customer count and average spend).
- **Customers Likely to Churn** are a major opportunity: they have spent an average of 1,710 each but have been inactive for about 247 days. They are the best target for win-back campaigns.
- **Recently Acquired Customers** have purchased recently but only 2.8 times on average. Encouraging a second and third purchase could move them towards the Most Valuable group.
- **Least Engaged Customers** are the largest group but spend the least (302 on average, 1.3 orders), so heavy marketing spend on them is unlikely to pay off.

### Suggested Actions
| Segment | Suggested action |
|---|---|
| Most Valuable | Loyalty rewards, early access to new products, personal service |
| Likely to Churn | Win-back offers and reminder emails |
| Recently Acquired | Welcome series, incentives for a second purchase |
| Least Engaged | Low-cost automated campaigns only |

## Repository Contents

| File | Description |
|---|---|
| `Customer_segmentation.ipynb` | Full analysis notebook (cleaning, RFM, clustering, profiling) |
| `sample_customers_per_cluster.csv` | 7 sample customers from each cluster with their segment names |

## How to Run

1. Download the Online Retail II dataset and save it as `online_retail_II.csv` in the project folder.
2. Install the requirements: `pip install pandas numpy matplotlib seaborn scikit-learn jupyter`
3. Open the notebook and run all cells.

## Limitations and Next Steps
- Silhouette Score of 0.37 shows the segments overlap somewhat. Trying other algorithms (for example Gaussian Mixture Models) or other values of k could be tested.
- Outlier removal excluded the very highest spenders, who would likely need separate (for example key account) treatment.
- Segments are based only on purchase behaviour. Adding product categories or country could give richer segments.
- The segments were not validated against real campaign results.
