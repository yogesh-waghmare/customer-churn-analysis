# Customer Churn Analysis

End-to-end customer churn analysis built in Python, using a SQLite database (`customer_churn.db`) as the data source. The project cleans three raw tables, engineers churn features, and computes the KPIs needed to understand *who* churns, *where*, and *how much revenue is at stake*.

## Tech Stack

- Python (pandas, numpy)
- SQLite (`sqlite3`)
- matplotlib, seaborn
- Jupyter Notebook

## Data

Loaded from `customer_churn.db` — each table is read into its own dataframe.

| Table | Description | Key fields |
|---|---|---|
| `db_customer` | Customer master data | `customer_id`, `customer_name`, `gender`, `dob`, `country`, `state` |
| `db_subscription` | Subscription lifecycle & billing | `customer_id`, `plan_type`, `subscription_type`, `monthly_charges`, `subscription_start_date`, `renewal_date`, `cancellation_date`, `churn_score` |
| `db_support` | Support complaints | `customer_id`, `complaint_date`, `escalations` |

## Data Cleaning

**Customer table**
- Renamed `customerid` → `customer_id`, `name` → `customer_name`
- Standardised gender values (`Men` → `Male`, `Women` → `Female`)
- Converted `dob` to datetime
- Dropped unused columns (`interests`, `pincode`)
- Imputed missing `country` values using a state → country mapping

**Subscription table**
- Renamed `customerid` → `customer_id`
- Converted `subscription_start_date`, `renewal_date`, `cancellation_date` to datetime

**Support table**
- Renamed `customerid` → `customer_id`
- Dropped unused columns (`col_1`, `comment`)
- Converted `complaint_date` to datetime
- Deduplicated to one row per customer (kept latest complaint) after storing `complaint_count`

## Feature Engineering

- `churn_flag` — 1 if `cancellation_date` is present, else 0
- `complaint_count` — number of complaints raised per customer
- `tenure_days` — days between subscription start and cancellation date (or today, if still active)
- `escalations` — encoded Y/N to 1/0
- `churn_risk` — bucketed from `churn_score`: `low` (<50), `med` (50–69), `high` (>=70)

All three tables are merged on `customer_id` (subscription as the base, left joins) and exported to `exported churn data.csv` for downstream BI use.

## Analysis / KPIs

1. Overall churn rate and retention rate
2. Churn rate by plan type
3. Churn rate by state, with revenue and user counts per state
4. Churn rate by subscription type
5. ARPU (average revenue per user)
6. Average customer tenure (days)
7. Revenue at risk (monthly charges from churned customers)
8. Escalation rate
9. Average complaints per user
10. Correlation between escalations and churn
11. Churn risk segmentation (low / medium / high)

## How to Run

```bash
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook churn_analysis.ipynb
```

Place `customer_churn.db` in the same folder as the notebook, then run all cells top to bottom.

## Project Structure

```
.
├── churn_analysis.ipynb      # Full cleaning + analysis workflow
├── customer_churn.db         # Source SQLite database
├── exported churn data.csv   # Cleaned, merged dataset
└── README.md
```

## Next Steps

- Visual dashboard (Power BI / Tableau) on the exported dataset
- Predictive churn model (logistic regression / gradient boosting) using tenure, complaints, escalations and plan type
- Cohort and retention curve analysis by signup month
