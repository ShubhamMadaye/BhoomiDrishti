# Data Dictionary — BhoomiDrishti (SIH26017)
**Dataset Reference:** `ml/data/synthetic_land_acquisition_demo.csv`  
**Classification:** Synthetic Demonstration Dataset — Not SIH Operational Data

---

## 1. Project Overview & Schema Structure

The synthetic demonstration dataset consists of **2,500 project records** across **29 attributes**, systematically modeling the land acquisition lifecycle for infrastructure projects.

---

## 2. Field-Level Specification

| Field Name | Data Type | Role | Allowed Range / Categories | Generation Logic | Leakage Risk & Prevention | SIH PS Relevance |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `project_id` | String | Metadata | `PRJ-SYNTH-00001` to `PRJ-SYNTH-02500` | Deterministic sequential synthetic identifier. | None. Dropped from ML feature matrix. | Unique identification across national hierarchy. |
| `project_name` | String | Metadata | String format `Demo [Type] Sector-N` | Formatted demonstration label. | None. Dropped from ML feature matrix. | Administrative UI display. |
| `project_type` | Categorical | Predictor | `Highway Expansion`, `Freight Rail Corridor`, `Metro / Urban Transit`, `Renewable Solar Park`, `Industrial Corridor Hub` | Weighted sample reflecting major national infrastructure sectors. | None. Available at project inception. | Multi-sector infrastructure modeling. |
| `state` | Categorical | Predictor | `State-North (Demo)`, `State-West (Demo)`, `State-South (Demo)`, `State-East (Demo)`, `State-Central (Demo)` | Clustered synthetic state assignments. | None. Available at project inception. | State-level comparative governance analytics. |
| `district` | Categorical | Predictor | 15 synthetic district identifiers (`Dist-N1` through `Dist-C3`) | Hierarchically linked to demo states. | None. Available at project inception. | District-level bottleneck localization. |
| `land_area_ha` | Float | Predictor | `15.0` to `2000.0` hectares | Uniform distribution tailored to project type scale. | None. Determined during preliminary project report. | Primary physical land acquisition scope. |
| `affected_families` | Integer | Predictor | `10` to `1200` families | Uniform/Poisson distribution scaled with project type. | None. Measured during Social Impact Assessment (SIA). | R&R scale & stakeholder complexity. |
| `latitude` | Float | GIS Metadata | `12.20000` to `30.50000` | Random coordinate bounded inside regional demo box. | None. Marked as synthetic; excluded from ML tabular training. | Interactive Leaflet GIS visualization. |
| `longitude` | Float | GIS Metadata | `72.80000` to `88.10000` | Random coordinate bounded inside regional demo box. | None. Marked as synthetic; excluded from ML tabular training. | Interactive Leaflet GIS visualization. |
| `current_stage` | Categorical | Predictor | `Notification`, `Documentation`, `Approval`, `Compensation`, `Legal`, `R&R`, `Possession` | Random lifecycle phase assignment. | None. Snapshot state at prediction time. | Stage-wise risk awareness and tracking. |
| `notification_status`| Categorical | Predictor | `Preliminary Issued`, `Declaration Published`, `Inquiry Ongoing`, `Pending Review` | Discrete categorical sample. | None. Statutory procedural state. | Administrative milestone tracking. |
| `documentation_completion_pct` | Float | Predictor | `5.0` to `100.0` % | Beta distribution ($\alpha=3, \beta=2$) simulating field paperwork pace. | None. Real-time document telemetry. | Land title records verification. |
| `approval_status` | Categorical | Predictor | `Approved`, `Conditionally Approved`, `In Review`, `Pending Inter-Departmental NOC` | Categorical distribution modeling multi-agency approvals. | None. Current clearance milestone. | Inter-departmental coordination bottleneck. |
| `approval_delay_days` | Integer | Predictor | `0` to `400` days | Exponential distribution scaled by approval status. | None. Operational latency prior to completion. | Core early delay indicator. |
| `compensation_status` | Categorical | Predictor | `Disbursed`, `Partial Disbursement`, `Disbursement Initiated`, `Pending Hearing` | Categorical disbursement state. | None. Current financial disbursement milestone. | Direct land acquisition compensation tracking. |
| `compensation_progress_pct` | Float | Predictor | `0.0` to `100.0` % | Uniform bounds mapped to compensation status. | None. Financial disbursement telemetry. | Financial disbursement progress. |
| `legal_dispute_count` | Integer | Predictor | `0` to `8` cases | Discrete choice distribution with 55% zero-dispute baseline. | None. Active court litigation count. | Legal litigation risk factor. |
| `legal_dispute_status` | Categorical | Predictor | `No Active Dispute`, `Hearing Scheduled`, `Active Stay`, `Resolved / Quashed` | Dependent on dispute count. | None. Current legal posture. | Injunctions and legal stay orders. |
| `possession_status` | Categorical | Predictor | `Not Commenced`, `Partial Handover`, `Joint Inspection Complete`, `Full Possession` | Categorical handover status. | None. Physical possession milestone. | Site handover to project concessionaire. |
| `rehabilitation_progress_pct` | Float | Predictor | `0.0` to `100.0` % | Correlated with compensation progress plus Gaussian variation. | None. Current R&R execution progress. | Rehabilitation & Resettlement of PAPs. |
| `stakeholder_responsiveness_score` | Float | Predictor | `1.0` to `10.0` | Normal distribution ($\mu=6.5, \sigma=1.8$) clipped to bounds. | None. Survey/feedback composite score. | Community and landowner engagement. |
| `historical_authority_performance_score` | Float | Predictor | `1.0` to `10.0` | Normal distribution ($\mu=7.0, \sigma=1.5$) clipped to bounds. | None. Historical project completion rate of the executing authority. | Authority institutional capacity. |
| `project_start_date` | Date | Predictor / Temporal | `2021-01-01` to `2023-09-27` | Uniform temporal spread from base date. | None. Baseline starting epoch. | Temporal train/validation splitting. |
| `planned_completion_date` | Date | Predictor / Temporal | `2022-03-30` to `2026-12-25` | Start date + estimated baseline duration. | None. Contractual target date. | Planned horizon baseline. |
| `actual_completion_date` | Date | **TARGET / OUTCOME** | `2022-04-15` to `2027-08-30` | Planned completion date + simulated delay days. | **HIGH LEAKAGE RISK**: Must NEVER enter feature preprocessing or training pipeline. | Empirical outcome validation. |
| `delay_days` | Integer | **TARGET / OUTCOME** | `0` to `503` days | Simulated days past planned completion date. | **HIGH LEAKAGE RISK**: Strictly a regression target variable. Excluded from predictors. | Quantitative delay horizon modeling. |
| `delay_occurred` | Binary (0/1) | **PRIMARY TARGET** | `0` (On Time) or `1` (Delayed) | Thresholded binary outcome ($P_{\text{delay}} > 0.52$). | **HIGH LEAKAGE RISK**: Primary classification target. Excluded from predictors. | Primary binary classification objective. |
| `is_synthetic` | Boolean | Metadata | `True` | Constant boolean flag for governance compliance. | None. Dropped from training matrix. | Section 4 & 6 data honesty compliance. |
| `dataset_version` | String | Metadata | `v1.0-synthetic-demo` | Static version tag. | None. Dropped from training matrix. | Reproducibility & version tracking. |
