# Insurance Claims Risk Modeling Dashboard
##  Project Overview
This project presents an end-to-end risk analytics pipeline for insurance claims, combining data engineering, feature development, and executive-level reporting. The objective is to provide leadership with a clear view of where risk is concentrated, how losses are distributed, and which behavioral patterns are associated with fraud and elevated claim severity.
The dashboard integrates exposure segmentation, loss concentration analysis, and behavioral risk indicators into a unified framework, enabling more informed underwriting, pricing, and fraud mitigation decisions.
---
##  Key Business Questions
This solution is designed to answer the following:
- Where are losses most concentrated within the portfolio?
- How does claim severity vary across exposure tiers?
- Which behavioral patterns are most associated with fraud?
- Where (geographically) are risk concentrations emerging?
- How much of total loss is driven by a small subset of claims?
---
##  Data Pipeline Architecture
The project follows a structured transformation pipeline:
Raw Data → Pandas Cleansing → Feature Engineering → SQL Aggregation → Tableau Dashboard
---
##  Data Preparation (Python / Pandas)
Key data preparation steps included:
- Data ingestion from Excel into Pandas
- Data type normalization using `.convert_dtypes()`
- Column standardization and string casting
- Removal / handling of null values
- Feature creation using vectorized operations

### Example Transformations
```python
df["umbrella_limit"] = df["umbrella_limit"].fillna(0)
df["tenure_days"] = (df["incident_date"] - df["policy_bind_date"]).dt.days
df["is_fraud"] = df["fraud_reported"].map({"Y": 1, "N": 0}).astype("Int64")
df["vehicle_age"] = df["incident_date"].dt.year - df["auto_year"]
df["injury_ratio"] = df["injury_claim"] / df["total_claim_amount"]
### Data Engineering/structuring
Exposure Segmentation
df["exposure_tiers"] = pd.cut(
    df["umbrella_limit"],
    bins=[-1, 0, 500000, 1000000, 3000000, 5000000, float("inf")],
    labels=["None", "Low", "Moderate", "Elevated", "High", "Extreme"]
)
Claim Severity
df["claim_severity"] = pd.qcut(
    df["total_claim_amount"],
    3,
    labels=["Low", "Medium", "High"]
)
Behavioral Risk Flags
df["no_witness_flag"] = (df["witnesses"] == 0).astype(int)
df["late_night_flag"] = (df["incident_hour_of_the_day"].between(0, 5)).astype(int)
SQL Aggregation Layer (DuckDB)
1. Risk Flag Layer
2. Aggregation Layer
SELECT
    incident_state AS ag_state,
    exposure_tiers AS ag_tiers,
    COUNT(*) AS claims_count,
    AVG(total_claim_amount) AS avg_claim,
    AVG(is_fraud) AS fraud_rate
FROM risk_flags
GROUP BY incident_state, exposure_tiers
Key Metrics Derived:
Claim Volume
Average Claim Severity
Fraud Rate (proportion of fraudulent claims)
Output Generation

pantab.frame_to_hyper(
    claims_data_final,
    "claims_data_final.hyper",
    table="risk_model"
)
 Dashboard Overview (Tableau)
Executive KPIs
Total Claims
Total Loss ($)
Average Claim Severity
Fraud Rate (%)

Key Visual Components
1. Exposure Distribution
Portfolio segmentation by exposure tier
2. Loss Concentration Analysis
Top 10%, 25%, 50% of claims vs total losses
Highlights tail risk and severity skew
3. Behavioral Risk Indicators
Early claim behavior
Fraud pattern flags
No-witness high-loss scenarios
4. Geographic Risk (Recommended Extension)
State-level loss and fraud concentration
Identifies regional risk clusters
Key Insights Delivered
A minority of claims drive a disproportionate share of losses
Higher exposure tiers are associated with increased severity
Behavioral flags strongly correlate with elevated fraud probability
Risk distribution varies meaningfully by geographic region
Business Impact
This solution enables:
Improved underwriting and pricing decisions
Targeted fraud detection strategies
Identification of high-risk exposure segments
Data-driven portfolio optimization
Tools & Technologies
Python (Pandas) – Data cleansing & feature engineering
DuckDB (SQL) – Analytical transformations & aggregations
Pantab – Hyper file generation
Tableau – Dashboard visualization and reporting

