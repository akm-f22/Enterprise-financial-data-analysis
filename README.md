# Enterprise Data Architecture & Reconciliation

## Financial Audit Dashboard | FinTech Data Analytics Project

A complete end-to-end data analytics pipeline designed to reconcile data from multiple messy enterprise systems and prepare reliable financial data for an executive/CFO dashboard.

The project combines **SQL, Python, Pandas, Regular Expressions, data reconciliation, financial calculations, data validation, and Power BI** to transform raw data into an analysis-ready financial dataset.

---

## 📌 Project Overview

The CFO of a FinTech company requires a Q1–Q4 Financial Audit Dashboard. However, the required information is distributed across three different systems:

1. **HR Database** — SQLite database containing historical user account status records.
2. **Server Logs** — 100,000 raw text records containing transaction information inside unstructured strings.
3. **Finance Server** — Daily EUR-to-USD exchange rates with missing values for weekends and market-closure dates.

The objective was to build a Python-based data pipeline that could:

* Identify users based on their latest account status.
* Extract transaction information from unstructured server logs.
* Remove invalid/error transactions.
* Handle missing exchange rates.
* Reconcile transactions with active users.
* Convert EUR transaction values into USD revenue.
* Validate the final dataset.
* Produce dashboard-ready data for Power BI.
* Analyze monthly revenue and the Top 5 customers.

---

# 🎯 Business Problem

The source systems contain inconsistent, historical, and partially unstructured data.

A simple aggregation of the raw data could produce inaccurate financial results because:

* Users have multiple historical status records.
* Transaction information is embedded in text logs.
* ERROR transactions must be excluded.
* Exchange-rate data contains missing dates.
* Transactions must be matched to the correct exchange rate.
* Only users whose latest status is Active should be included.

Therefore, the project focuses on building a **reliable and reproducible financial reconciliation pipeline** before the data reaches the executive dashboard.

---

# 💡 Solution

The project implements a five-phase data pipeline.

### Phase 1 — SQL User Extraction

SQLite is used to identify the most recent status for every user.

A SQL window function ranks records by:

```sql
ROW_NUMBER() OVER (
    PARTITION BY user_id
    ORDER BY updated_at DESC
)
```

Only records where:

```text
rn = 1
AND status = 'Active'
```

are retained.

---

### Phase 2 — Regex Transaction Extraction

The raw server logs contain transaction information inside text strings.

The pipeline:

1. Loads the log file into Pandas.
2. Removes records containing `ERROR`.
3. Uses Regex with Pandas `.str.extract()`.
4. Extracts:

```text
Date
User_ID
Product_ID
Euro_Value
```

This converts the unstructured log data into structured tabular data.

---

### Phase 3 — Exchange Rate Reconciliation

The daily EUR-to-USD exchange-rate dataset contains missing values because financial markets are closed on certain dates.

The pipeline:

1. Converts the Date column to datetime.
2. Sorts the exchange-rate data chronologically.
3. Forward-fills missing exchange rates.

```python
exchange_rates["EUR_to_USD"] = (
    exchange_rates["EUR_to_USD"].ffill()
)
```

The transactions are then merged with the exchange-rate data using the transaction date.

---

### Phase 4 — Active User Reconciliation

The cleaned transactions are merged with the latest Active users extracted from the HR database.

This ensures that the final financial dataset contains only transactions associated with users whose latest recorded status is Active.

---

### Phase 5 — USD Revenue Calculation

USD revenue is calculated using:

```python
df_final["USD_Revenue"] = (
    df_final["Euro_Value"] *
    df_final["EUR_to_USD"]
)
```

The final dataset is then validated and exported as:

```text
power_bi_ready_dataset.csv
```

---

# 🛠️ Technologies Used

| Technology          | Purpose                                    |
| ------------------- | ------------------------------------------ |
| Python              | End-to-end data pipeline                   |
| Pandas              | Data cleaning, transformation and analysis |
| SQLite              | Database querying                          |
| SQL                 | Historical user-status extraction          |
| Regular Expressions | Server-log parsing                         |
| Matplotlib          | Data visualization                         |
| Power BI            | Interactive dashboard                      |
| Jupyter Notebook    | Development and documentation              |
| CSV                 | Data exchange/output                       |

---

# 📊 Project Results

The completed pipeline produced:

| Metric                                  |                   Result |
| --------------------------------------- | -----------------------: |
| Raw server log records                  |                  100,001 |
| ERROR records removed                   |                    5,098 |
| Valid transactions extracted            |                   94,902 |
| Latest Active users                     |                   14,183 |
| Final reconciled transactions           |                   53,899 |
| Unique customers in final data          |                   13,888 |
| Total USD Revenue                       |          $148,274,622.91 |
| Average Transaction Value               |                $2,750.97 |
| Missing exchange rates after processing |                        0 |
| Missing USD Revenue values              |                        0 |
| Duplicate transactions identified       |                        0 |
| Reporting period                        | 2023-01-01 to 2023-12-31 |

---

# 📈 Dashboard

The resulting dataset is designed for a Power BI Financial Audit Dashboard.

### Dashboard Components

#### 1. Total USD Revenue

A DAX measure can be created using:

```DAX
Total USD Revenue =
SUM('power_bi_ready_dataset'[USD_Revenue])
```

#### 2. Revenue Trend

A line chart displays:

```text
Monthly Revenue
```

across the 2023 reporting period.

#### 3. Top 5 Customers

A bar chart identifies the five customers with the highest total USD revenue.

#### 4. Date Filtering

The dashboard can include a date slicer allowing users to analyze:

```text
Year
Quarter
Month
Date
```

---

# 📷 Dashboard Screenshot

<img width="1128" height="840" alt="Screenshot of powerbi dashboard" src="https://github.com/user-attachments/assets/028321ba-6f8f-4874-a636-c82aae7706d5" />

---

# 📦 Output

The primary output is:

```text
/power_bi_ready_dataset.csv
```

The dataset contains:

```text
Date
User_ID
name
Product_ID
Euro_Value
EUR_to_USD
USD_Revenue
```

This dataset can be imported directly into Power BI.

---

# 🔍 Data Validation

The pipeline performs several validation checks:

### Missing Values

```python
df_final.isna().sum()
```

### Duplicate Transactions

```python
df_final.duplicated(
    subset=[
        "Date",
        "User_ID",
        "Product_ID",
        "Euro_Value"
    ]
).sum()
```

### Revenue Validation

```python
df_final["USD_Revenue"].sum()
```

### Date Range

```python
df_final["Date"].min()
df_final["Date"].max()
```

These checks help ensure that the final dataset is suitable for financial reporting.

---

# ⚡ Performance Approach

The project avoids row-by-row Pandas loops.

Instead, it uses vectorized operations including:

* `.str.extract()`
* `.merge()`
* `.groupby()`
* `.ffill()`
* `.dropna()`
* SQL window functions

This approach allows the pipeline to process the large server-log dataset efficiently.

---

# 📋 Key Business Insights

The final analysis supports two primary business questions:

### 1. How does revenue change over time?

Monthly aggregation allows the CFO to identify revenue trends throughout the year.

### 2. Who are the most valuable customers?

Customer-level aggregation identifies the Top 5 customers based on total reconciled USD revenue.

These outputs can be used to support financial review, customer-value analysis and executive reporting.

---

# 📄 Project Documentation

The repository also contains a detailed project report covering:

* Problem Statement
* Project Solution
* Data Pipeline
* Data Validation
* Project Results
* Visualizations
* Dashboard Outputs

---

# 👨‍💻 Author

**Anand Kumar Mishra**

Data Analyst | SQL | Python | Power BI | Data Engineering

---

## ⭐ Project Highlights

* Advanced SQL window functions
* Unstructured data extraction using Regex
* Large-scale log processing with Pandas
* Financial data reconciliation
* Missing exchange-rate handling
* Data-quality validation
* Revenue analytics
* Power BI dashboard preparation
* End-to-end ETL/data pipeline development
