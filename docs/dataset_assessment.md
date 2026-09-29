# Dataset Assessment & Quality Audit — BhoomiDrishti
**Problem Statement:** SIH26017  
**Sub-Theme:** AI, ML & Predictive Analytics for Infrastructure Governance and Public Administration  
**Evaluation Scope:** Synthetic Demonstration Dataset (`synthetic_land_acquisition_demo.csv`)

---

## 1. MANDATORY DATA DISCLAIMER
> [!IMPORTANT]
> **Synthetic Demonstration Dataset — Not SIH Operational Data.**  
> Smart India Hackathon (SIH) has not provided an operational dataset for SIH26017. All findings, metrics, distributions, and relationships documented herein reflect an engineered, reproducible synthetic demonstration dataset created for prototype validation.
>
> - **Never** interpret these simulated distributions as empirical performance benchmarks of Indian state administrations or land records authorities.
> - **Never** assume field readiness of machine learning models validated exclusively against this synthetic baseline.

---

## 2. Structural & Statistical Assessment

### 2.1 Volume & Dimensionality
- **Sample Size:** 2,500 independent project records.
- **Total Features:** 29 columns (20 candidate predictors, 3 outcome targets, 6 metadata/GIS identifiers).
- **Missing Value Count:** 0 (100% data completeness across all fields).
- **Synthetic Governance Integrity:** 100% rows have `is_synthetic = True`.

### 2.2 Class Balance Assessment (`delay_occurred`)
- **Delayed Projects (`delay_occurred = 1`):** 1,582 records (63.28%)
- **On-Time Projects (`delay_occurred = 0`):** 918 records (36.72%)
- **Assessment:** The target distribution exhibits a realistic positive delay rate (~63%), mirroring typical public infrastructure reporting where land acquisition delays represent a substantial challenge. The class distribution does not suffer from severe class imbalance, allowing balanced F1 and PR-AUC optimization without extreme synthetic oversampling.

### 2.3 Delay Horizon Distribution (`delay_days`)
- **Minimum Delay:** 0 days (projects delivered on or before planned date)
- **25th Percentile:** 21 days
- **Median Delay:** 186 days
- **Mean Delay:** 152.5 days ($\sigma = 111.4$ days)
- **75th Percentile:** 241 days
- **Maximum Delay:** 503 days
- **Assessment:** The distribution captures both minor administrative schedule adjustments (< 30 days) and severe multi-year acquisition stalemates (> 365 days) often triggered by court stay orders or uncompleted R&R packages.

---

## 3. Data Leakage Audit & Verification

A strict automated check is enforced:
1. `actual_completion_date`: Generated as `planned_completion_date + timedelta(days=delay_days)`. **Quarantined**: Excluded from any feature engineering or predictive preprocessing.
2. `delay_days`: The direct continuous outcome. **Quarantined**: Used exclusively as the target variable for the secondary regression model.
3. `delay_occurred`: The direct binary outcome. **Quarantined**: Used exclusively as the training target for classification.
4. **Conclusion:** Zero target leakage is guaranteed by automated column isolation in `ml/src/preprocess.py`.

---

## 4. Modeling Assumptions vs. Real-World Limitations

| Dimension | Demonstration Dataset Assumption | Real-World Operational Reality |
| :--- | :--- | :--- |
| **Approval Delays** | Simulated as single exponential delay metric with NOC categorical state. | Real data involves intricate multi-departmental clearance workflows (Forestry, Wildlife, Defense, PWD, Revenue). |
| **Dispute Records** | Integer counts and discrete status (`Active Stay`, etc.). | Real data involves multi-tier litigation (District Collector, High Court, Supreme Court, Land Tribunal). |
| **GIS Coordinates** | Random simulated lat/lon points inside state boxes. | Real data requires parcel-level survey boundary GeoJSON / Cadastral maps from Bhunaksha / state portals. |
| **Authority Performance** | Synthetic continuous rating (1.0 to 10.0). | Real data requires aggregated historical completion rates, audit compliance scores, and SLA metrics. |

---

## 5. Real-World Ingestion Protocol

When real data is provided:
1. **Schema Mapping**: Ingested via standardized intermediate adapters.
2. **Data Profiling**: Verification of null rates, outlier boundaries, and categorical cardinalities.
3. **Temporal Partitioning**: Splitting by fiscal years (e.g., Projects started $\le 2022$ for train, $> 2022$ for test).
4. **Domain Calibration**: Re-evaluating Brier scores and Platt scaling on operational distributions.
