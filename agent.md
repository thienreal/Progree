# IMPLEMENTATION GUIDELINES & TECHNICAL BLUEPRINT

## I. General Principles & Coding Standards

* **Language Standard:** All variable names, function names, docstrings, code comments, and technical notes (Markdown cells) within the notebook files MUST be written entirely in **professional/academic English**.
* **Code Commenting & Formatting:**
  * Write code in complete, logical code blocks, minimizing fragmentation.
  * Every complex processing logic must include inline comments clearly explaining: the purpose, input assumptions, and output transformations.
* **Deterministic Execution:** Fix `random_state=42` for all algorithms that involve randomness (train/test split, clustering, K-Means) to guarantee 100% reproducibility.
* **Decision-Making & Proactive Proposals:** For steps involving technical branching (e.g., selecting an imputation method, outlier treatment threshold, choosing the optimal number of clusters $K$, or forecasting model), the code must output specific comparative metrics and state the reasoning for the selection instead of imposing a fixed method.
* **Dataset Specification:** This project utilizes the **Online Retail Dataset** (from UCI Machine Learning Repository / Kaggle) simulating the transactions of a UK-based online retail company. The core required fields are: `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country`.

---

## II. Detailed Technical Workflows

### PART 1: Exploratory Data Analysis & Integrity Cleaning Pipeline

* **Execution File:** `01_eda_and_cleaning.ipynb`
* **Objective:** Comprehensively process raw data quality, eliminate anomalies, standardize the schema, and extract overall distributional characteristics.

#### Detailed Execution Steps

1. **Environment Setup & Data Ingestion:**
   * Import libraries: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`.
   * Load raw data and inspect metadata: `.shape`, `.info()`, `.dtypes`, and memory footprint.

2. **Schema & Structural Validation:**
   * Convert data to correct formats: cast `InvoiceDate` to datetime, and `CustomerID` to string/object or an integer type that supports missing values.
   * Scan for junk values: trailing whitespaces in text strings (`Description`), encoding errors.

3. **Data Integrity & Anomaly Filtering:**
   * Filter out invalid transaction records: return transactions (encoded with a 'C' prefix in `InvoiceNo` or `Quantity < 0`), and records with illogical unit prices (`UnitPrice <= 0`).
   * Handle duplicate rows.

4. **Systematic Missing Value Handling (Imputation):**
   * Assess the missing value ratio across columns.
   * `CustomerID` field: Analyze the risk of dropping rows versus assigning an anonymous identifier (`Guest_XXXX`). Proposed action: drop rows missing `CustomerID` for the dataset intended for customer analysis.
   * Other numerical or categorical fields: Compare Median Imputation and Mode Imputation, then execute the optimal approach.

5. **Outlier Detection & Treatment:**
   * Apply IQR (Interquartile Range) or Z-score methods on the two sensitive continuous variables: `Quantity` and `UnitPrice`.
   * Weigh the pros and cons between Capping (Winsorization at the 1st and 99th percentiles) and Trimming (complete removal). Proposed action: Capping to preserve the large sample size of transactions.

6. **Univariate & Bivariate Exploratory Data Analysis (EDA):**
   * Visualize the distribution of quantity and revenue variables (apply Log-transform to handle heavy right-skewed distributions).
   * Analyze sales distribution over time (by month/quarter) and by geographical market (`Country`).

7. **Export Pipeline:**
   * Export the cleaned dataset to the processed data directory in an optimized format: `cleaned_transactions.parquet` (or `.csv`).

#### Expected Deliverables

* A complete notebook file that runs from start to finish without generating any errors or critical warnings (clean run).
* A clean dataset with 100% of illogical values removed and missing values/outliers thoroughly handled.
* A set of distribution charts (Histograms, Boxplots, Correlation Heatmap) clearly demonstrating the data variations before and after cleaning.

#### Definition of Done (DoD)

* The clean data contains zero `null` values in primary key columns.
* No illogical negative values remain in quantitative fields (`Quantity`, `UnitPrice`).
* An Integrity Check confirms correct data types and that the valid row count is maintained at a minimum of 70–80% of the raw dataset.

---

### PART 2: Customer Segmentation & Behavioral Trend Preparation

* **Execution File:** `02_customer_segmentation.ipynb`
* **Objective:** Compute the customer behavioral index system RFM (Recency, Frequency, Monetary), segment customers, and export a standardized data structure for Business Intelligence tools.

#### Detailed Execution Steps

1. **Data Ingestion & Feature Engineering:**
   * Read the clean data from the `cleaned_transactions.parquet` file generated in Part 1.
   * Create a new variable: `TotalSpend = Quantity * UnitPrice`.
   * Define the Analysis Snapshot Date = the maximum transaction date in the dataset + 1 day.

2. **RFM Metrics Computation:**
   * Aggregate data by `CustomerID`:
     * **Recency ($R$):** Number of days since the last purchase to the Snapshot Date.
     * **Frequency ($F$):** Total number of unique invoices/transactions generated (`InvoiceNo.nunique()`).
     * **Monetary ($M$):** Total amount of money spent by the customer (`TotalSpend.sum()`).

3. **Statistical Scaling & Distribution Normalization:**
   * Analyze the skewness of the 3 variables $R, F, M$.
   * Apply logarithmic transformation (`np.log1p`) or Box-Cox to normalize the distributions prior to clustering.
   * Standardize the scales using `StandardScaler`.

4. **Segmentation Modeling (Proposal & Execution):**
   * *Propose 2 approaches:*
     * Approach 1: Traditional Quantiles (Quantile Scoring 1–5 for each index $R, F, M$).
     * Approach 2: Unsupervised machine learning via $K$-Means Clustering.
   * *Execute $K$-Means:* Utilize the Elbow Method (Inertia) combined with the Silhouette Score to find the optimal number of clusters $K$ ($K \in [3, 6]$).
   * Assign clear business labels to each segment (e.g., *Champions/VIP*, *Loyal Customers*, *Potential Loyalists*, *At Risk*, *Lost*).

5. **Cohort Analysis Matrix Preparation:**
   * Identify each customer's first purchase month (Cohort Month) and current transaction month (Transaction Month).
   * Calculate the Cohort Index (month distance $0, 1, 2, ...$).
   * Construct the Retention Rate Matrix as a pivot table.

6. **Data Structuring for BI Suite (Power BI / Tableau):**
   * Standardize the customer table: `CustomerID`, $R$, $F$, $M$, `Cluster_Label`, `Segment_Name`, `Country`.
   * Export 2 files for Data Modeling in Power BI:
     * Fact Table: `fact_transactions.csv` (the linked clean transaction data).
     * Dimension Table: `dim_customers_rfm.csv` (customer segment information).

#### Expected Deliverables

* A detailed segmentation analysis notebook, clearly displaying the Silhouette score chart and Elbow curve to justify the chosen number of clusters.
* A customer data table with explicit segment labels, including a mean characteristics analysis ($R, F, M$ mean/median) for each group.
* Two standardized CSV files (`fact_transactions.csv`, `dim_customers_rfm.csv`) ready to be loaded directly into Power BI without requiring further processing via Power Query.

#### Definition of Done (DoD)

* 100% of valid customers are assigned a specific segment label.
* There is clear statistical separation between clusters (Silhouette Score $\ge 0.4$, or logical business reasoning if using quantiles).
* The schema between the Fact and Dimension tables ensures relational integrity (the `CustomerID` foreign key in the Fact table fully references the primary key in the Dimension table).

---

### PART 3: Inferential Hypothesis Testing & Predictive Time-Series Forecasting

* **Execution File:** `03_hypothesis_and_forecasting.ipynb`
* **Objective:** Perform inferential statistical tests to verify business hypotheses and construct a time-series forecasting model with confidence intervals.

#### Detailed Execution Steps

1. **Module A - Statistical Inferential Hypothesis Testing:**
   * **Construct Hypothesis 1 (Testing revenue differences among customer groups):**
     * Formulate hypotheses: $H_0$ (There is no difference in average spending among the customer segments created in Part 2); $H_1$ (At least one segment has a significantly different average spending).
     * Check application conditions: Test for normal distribution (Shapiro-Wilk) and homogeneity of variances (Levene's Test).
     * Execute test: Use **One-Way ANOVA** (if assumptions are met) or the non-parametric **Kruskal-Wallis** test (if assumptions are violated).
     * Run Post-hoc tests (Tukey HSD or Dunn's test) to identify pairs of segments with significant differences at the $\alpha = 0.05$ level.
   * **Construct Hypothesis 2 (Testing the independence between geographical location and customer behavior):**
     * Establish a Contingency Table between Customer Segments and Geographical Regions. *(Note for the Online Retail Dataset: Since the vast majority of customers are from the UK, it is necessary to group the `Country` variable into "UK" and "Non-UK" (International) to satisfy the expected frequency condition > 5 for the Chi-Square test).*
     * Execute the **Chi-Square Test of Independence**, outputting the $p$-value and Cramér's V correlation coefficient.

2. **Module B - Time-Series Aggregation & Stationarity Diagnostics:**
   * Aggregate clean transaction data along the time axis: Resample total revenue by Day (`'D'`) or by Week (`'W'`). Proposed action: use the Weekly (`'W'`) level to reduce weekend noise and short-term volatility.
   * Decompose the time series (Seasonal Decomposition: Trend, Seasonal, Residual) using an Additive or Multiplicative model.
   * Check for stationarity using the **Augmented Dickey-Fuller (ADF Test)** and **KPSS Test**.
   * Apply Differencing (order $d$) if the time series is non-stationary.

3. **Module C - Predictive Modeling (ARIMA/SARIMA vs. Baselines):**
   * Data splitting: Train/Test split in chronological order (Chronological Split: e.g., 80% train, 20% test - absolutely no data shuffling).
   * Hyperparameter selection $(p, d, q) \times (P, D, Q)_s$:
     * Plot the Autocorrelation Function (ACF) and Partial Autocorrelation Function (PACF).
     * Utilize `pmdarima.auto_arima` (or configure a Grid Search loop) based on minimizing the AIC/BIC criteria.
   * Train the SARIMA model on the Train set and forecast on the Test set.

4. **Model Evaluation & Confidence Boundary Projection:**
   * Calculate error measurement metrics: MAE (Mean Absolute Error), RMSE (Root Mean Squared Error), MAPE (Mean Absolute Percentage Error).
   * Retrain the model on the entire dataset (Full Dataset).
   * Project future values (e.g., the next 8–12 weeks) and extract the 95% Confidence Intervals (Lower Bound and Upper Bound).
   * Visualize the results: Actual historical line, forecast line, and a shaded confidence area.

#### Expected Deliverables

* Detailed quantitative statistical conclusions for each hypothesis: complete with $F$-statistic / $\chi^2$-statistic values, $p$-values, and a conclusion to reject or accept $H_0$ at the $\alpha = 0.05$ significance level.
* A performance comparison table of the time-series model on the Test set (along with remarks on its stability).
* A standard forecasting visualization chart clearly showing the 95% confidence boundaries for future periods.
* A `forecast_results.csv` file containing the columns: `Date`, `Forecasted_Sales`, `Lower_Bound_95`, `Upper_Bound_95`.

#### Definition of Done (DoD)

* All statistical tests undergo a distribution assumption check before selecting the test type (parametric vs. non-parametric).
* The time-series data splitting process must contain absolutely no data leakage from the future into the past.
* Future forecasts must strictly include clear upper and lower confidence boundaries (satisfying the *confidence performance boundaries* requirement).
