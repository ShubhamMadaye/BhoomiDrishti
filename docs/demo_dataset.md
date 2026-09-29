# Synthetic Demonstration Dataset Documentation — BhoomiDrishti
**Problem Statement Reference:** SIH26017  
**Sub-Theme:** AI, ML & Predictive Analytics for Infrastructure Governance and Public Administration  
**Classification:** Prototype Demonstration Data (Non-Government / Synthetic)

---

## CRITICAL NOTICE & MANDATORY DISCLAIMER
> [!IMPORTANT]
> **Synthetic Demonstration Dataset — Not SIH Operational Data.**
> Smart India Hackathon (SIH) has **NOT** provided an operational dataset for SIH26017. 
> 
> - This dataset was generated programmatically for development, architecture validation, ML experimentation, and user interface demonstration.
> - It must **never** be cited or represented as official government records, actual land acquisition data, or empirical ministry statistics.
> - Simulated correlations represent demonstration modeling assumptions to test ML pipelines, **not discovered real-world causal dynamics**.
> - All geographic coordinates are simulated within approximate regional bounding boxes solely to verify GIS mapping components.

---

## 1. Generation Methodology

The dataset is generated via [`ml/data/generate_demo_dataset.py`](file:///c:/Projects/bhmooidristi/ml/data/generate_demo_dataset.py) using a fixed pseudorandom seed:

```bash
python ml/data/generate_demo_dataset.py --num_samples 2500 --seed 2026 --output_dir ml/data
```

### Deterministic Reproducibility
- **Random Seed:** `2026`
- **Output Record Count:** 2,500 projects
- **Total Columns:** 29
- **Missing Values:** 0 (Null count: 0)
- **Synthetic Governance Flag:** `is_synthetic = True` on every row

---

## 2. Simulated Governance Variables

Variables are formulated directly from the factors highlighted in the SIH26017 problem description:

| Domain | Simulated Variables | Behavioral Logic / Generation Assumptions |
| :--- | :--- | :--- |
| **Physical Scope** | `land_area_ha`, `affected_families` | Scaled by project type (Rail/Highway require higher land area; Metro involves high families per hectare). |
| **Administrative** | `notification_status`, `approval_status`, `approval_delay_days`, `documentation_completion_pct` | Exponential approval delay distributions; NOC delays simulate inter-departmental latency. |
| **Compensation** | `compensation_status`, `compensation_progress_pct` | Progress percentages mapped across disbursement stages (Initiated, Partial, Disbursed). |
| **Legal** | `legal_dispute_count`, `legal_dispute_status` | Discrete Poisson/choice dispute counts with statuses: `Active Stay`, `Hearing Scheduled`, `Resolved / Quashed`. |
| **Possession & R&R**| `possession_status`, `rehabilitation_progress_pct` | Possession handover stages; R&R progress correlates with compensation pace. |
| **Performance** | `stakeholder_responsiveness_score`, `historical_authority_performance_score` | Standardized continuous scores (1.0 to 10.0) reflecting district operational efficiency. |
| **Temporal** | `project_start_date`, `planned_completion_date` | Multi-year timeline spreading from January 2021 to September 2023. |
| **Outcomes** | `actual_completion_date`, `delay_days`, `delay_occurred` | Strictly post-outcome target variables generated via latent logistic scoring with controlled Gaussian noise. |

---

## 3. Separation of Predictors vs. Outcomes (Zero Target Leakage)

To maintain rigorous machine learning integrity, features are strictly separated into two disjoint sets:

1. **Eligible Predictors (Available at Inference Time)**:
   - `project_type`, `state`, `district`, `land_area_ha`, `affected_families`
   - `current_stage`, `notification_status`, `documentation_completion_pct`
   - `approval_status`, `approval_delay_days`
   - `compensation_status`, `compensation_progress_pct`
   - `legal_dispute_count`, `legal_dispute_status`
   - `possession_status`, `rehabilitation_progress_pct`
   - `stakeholder_responsiveness_score`, `historical_authority_performance_score`
   - `project_start_date`, `planned_completion_date`
2. **Outcome / Target Variables (Excluded from Feature Matrix)**:
   - `delay_occurred` (Primary binary classification target)
   - `delay_days` (Secondary continuous regression target)
   - `actual_completion_date` (Post-event timestamp)

---

## 4. Dataset Summary Metrics

- **Total Projects:** 2,500
- **Delay Occurrence Rate:** 63.28% (Delayed: 1,582, On Time: 918)
- **Regression Target (`delay_days`):**
  - Minimum: 0 days
  - 25th Percentile: 21.0 days
  - Median: 186.0 days
  - Mean: 152.5 days
  - Maximum: 503.0 days
- **Temporal Horizon:**
  - Start Dates: 2021-01-01 to 2023-09-27
  - Planned Completion Dates: 2022-03-30 to 2026-12-25

---

## 5. How to Replace with Real Government Data

When operational land acquisition records become available, ingestion is performed modularly via `ml/src/preprocess.py` and `ml/src/validate_data.py` without modifying the core backend schema:
1. Ingest real records into standard intermediate schema.
2. Run automated schema validation and leakage detection.
3. Perform temporal train/test split.
4. Execute candidate model retraining, calibration, and drift checks.
5. Promote newly evaluated model to the production registry.
