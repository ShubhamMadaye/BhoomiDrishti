# Machine Learning Subsystem & Model Evaluation Report — BhoomiDrishti
**Problem Statement Reference:** SIH26017  
**Sub-Theme:** AI, ML & Predictive Analytics for Infrastructure Governance and Public Administration  
**Operational Status:** Prototype Demonstration (Synthetic Data Pipeline)  
**Registered Model Version:** `v0.1.0-synthetic-demo`  
**Model Artifact SHA-256 Checksum:** `d92fb3db06dbf42c6731e8461b2b800ca33cbb493fb0ea6baae448c26bbd9241`

---

## 1. MANDATORY DATA HONESTY DISCLAIMER
> [!IMPORTANT]
> **Synthetic Demonstration Data Notice:**  
> Smart India Hackathon (SIH) has **NOT** provided an operational dataset for SIH26017. 
> 
> All model evaluation metrics reported herein (Accuracy, Precision, Recall, F1, ROC-AUC, PR-AUC, Brier Score, MAE, RMSE, R²) were computed directly by code from a temporal split of the **reproducible synthetic demonstration dataset** (`synthetic_land_acquisition_demo.csv`).
>
> - **Never** claim that synthetic demonstration accuracy represents real-world government predictive accuracy.
> - **Never** assume field readiness of a model validated solely on synthetic distributions.
> - Real-world deployment requires representative historical land acquisition records from authorized departments.

---

## 2. ML Problem Formulation

The BhoomiDrishti ML subsystem addresses land acquisition delay risk via two complementary supervised objectives:

```
                            INPUT PREDICTOR MATRIX
                     (20 Raw Features + 4 Engineered = 50 Encoded)
                                      |
                 +--------------------+--------------------+
                 |                                         |
                 ↓                                         ↓
     PRIMARY TASK: CLASSIFICATION              SECONDARY TASK: REGRESSION
          Target: delay_occurred                     Target: delay_days
      (Binary: 0 = On Time, 1 = Delayed)          (Continuous: Estimated Horizon)
                 |                                         |
                 ↓                                         ↓
         Calibrated Platt Scaling                       Ridge Regressor
      Outputs: Delay Probability (0-1)              Outputs: Predicted Delay Days
               Standardized Risk Score (0-100)               (e.g., 185 days)
               Risk Tier (LOW / MED / HIGH)
```

---

## 3. Temporal Data Partitioning Strategy

To eliminate time-travel data leakage and mirror real-world government deployments, records are partitioned strictly by `project_start_date`:

| Partition | Date Horizon | Sample Count | Proportion | Simulated Delay Rate |
| :--- | :--- | :--- | :--- | :--- |
| **Train Set** | `2021-01-01` to `2022-11-28` | 1,750 | 70.0% | 62.7% |
| **Validation Set** | `2022-11-28` to `2023-04-22` | 375 | 15.0% | 64.8% |
| **Test Set (Held-Out)** | `2023-04-22` to `2023-09-27` | 375 | 15.0% | 64.5% |

Transformers (StandardScaler, OneHotEncoder) were fitted **strictly on the Train partition**; the Validation and Test partitions were transformed out-of-sample.

---

## 4. Primary Task: Classification Model Comparison & Calibration

### 4.1 Validation Performance Across Candidates
Three candidate models were evaluated on the validation partition:

| Model Architecture | Val Accuracy | Val Precision | Val Recall | Val F1-Score | Val ROC-AUC | Val PR-AUC | Val Brier Score |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Baseline: Logistic Regression** *(Selected)* | **0.8427** | **0.8708** | **0.8848** | **0.8778** | **0.9341** | **0.9632** | **0.1004** |
| **Candidate: Random Forest** | 0.8000 | 0.8242 | 0.8807 | 0.8527 | 0.8760 | 0.9168 | 0.1413 |
| **Candidate: Gradient Boosting** | 0.8320 | 0.8538 | 0.8930 | 0.8722 | 0.9087 | 0.9416 | 0.1174 |

> [!NOTE]
> Per Permanent Rule 21 ("Always implement a baseline ... Do not assume the most complex model is best"): Logistic Regression outperformed tree ensembles on this synthetic distribution in both ROC-AUC (0.9341 vs 0.9087) and probability calibration (Brier score 0.1004 vs 0.1174).

### 4.2 Probability Calibration
The model was calibrated using **Platt Scaling** (`CalibratedClassifierCV(method='sigmoid', cv=5)`). This guarantees that predicted probabilities reflect true empirical delay frequencies rather than uncalibrated margin scores.

### 4.3 Final Held-Out Test Evaluation (Test Set, N = 375)
- **Accuracy:** `0.8427` (84.27%)
- **Precision:** `0.8735` (87.35%)
- **Recall:** `0.8843` (88.43%)
- **F1-Score:** `0.8789`
- **ROC-AUC:** `0.9243`
- **PR-AUC:** `0.9600`
- **Brier Score:** `0.1075`
- **Confusion Matrix:**
  $$\begin{pmatrix} \text{True Negatives (TN): } 102 & \text{False Positives (FP): } 31 \\ \text{False Negatives (FN): } 28 & \text{True Positives (TP): } 214 \end{pmatrix}$$

---

## 5. Secondary Task: Regression Model Comparison (Delay Horizon)

For projects flagged at risk, estimating the delay horizon in days enables capital budgeting and resource planning.

| Model Architecture | Val MAE (Days) | Val RMSE (Days) | Val $R^2$ Score |
| :--- | :--- | :--- | :--- |
| **Baseline: Ridge Regression** *(Selected)* | 59.27 days | **71.04 days** | **0.5843** |
| **Candidate: Random Forest Regressor** | 60.77 days | 75.85 days | 0.5261 |
| **Candidate: Gradient Boosting Regressor**| **56.86 days** | 71.69 days | 0.5767 |

### Final Test Partition Regression Metrics (Held-Out Test Set)
- **Mean Absolute Error (MAE):** `57.37` days
- **Root Mean Squared Error (RMSE):** `70.57` days
- **Coefficient of Determination ($R^2$):** `0.6155`

---

## 6. Stage-Level Prediction Feasibility Determination

Per Permanent Rule 33 ("Document unsupported requirements. If a problem statement item lacks sufficient data, explicitly note 'Stage-wise prediction requires stage-specific historical data'"):

### Findings
1. **Current Capability (Stage-Conditioned Unified Model)**:
   - The unified model encodes `current_stage` as a multi-class categorical feature across all 7 lifecycle stages (`Notification`, `Documentation`, `Approval`, `Compensation`, `Legal`, `R&R`, `Possession`).
   - This captures stage-conditioned risk variations across the full 2,500 sample volume.
2. **Limitation for Isolated Stage Sub-Models**:
   - Splitting the dataset into 7 distinct stage sub-models yields only ~350 records per stage, leading to high variance and poor generalization.
   - True stage progression forecasting requires granular stage-to-stage transition telemetry (e.g. stage entry timestamp, stage SLA target, intra-stage milestone completions).
3. **Official Governance Notice**:
   > *"Stage-wise prediction requires stage-specific historical telemetry."*

---

## 7. Explainable AI & Feature Attribution

Global and instance-level explainability is computed via Shapley-consistent feature attributions:

### Top 10 Global Risk Drivers by Mean Attribution
1. **Judicial Injunction / High Court Stay Order** (`2.0263`)
2. **Number of Active Court Disputes** (`1.2224`)
3. **Pending Inter-Departmental Clearance (NOC)** (`1.1452`)
4. **Dispute Status: No Active Dispute** (`1.0136`, risk mitigating)
5. **Land Possession Not Commenced** (`0.9629`)
6. **Approval Stagnation / Processing Delay** (`0.8705`)
7. **Historical Authority Execution Capacity** (`0.8655`, risk mitigating when high)
8. **Pace of Land Compensation Disbursement** (`0.8247`, risk mitigating when high)
9. **Stakeholder Responsiveness Score** (`0.7493`, risk mitigating when high)
10. **Possession Status: Joint Inspection Complete** (`0.7049`)

### Instance-Level Explainability Protocol
Every prediction returned by the API provides:
- Top risk contributors (increasing delay risk)
- Top mitigating factors (reducing delay risk)
- Strictly non-causal phrasing: *"Approval delay is contributing positively to the predicted delay risk"* (adhering to Permanent Rule 14).

---

## 8. Persisted ML Artifacts Registry

All model artifacts are versioned in `ml/artifacts/`:

```
ml/artifacts/
├── models/
│   ├── classifier_calibrated.joblib    # Production CalibratedClassifierCV
│   ├── classifier_best_tree.joblib      # Base model for explainability
│   ├── classifier_baseline.joblib       # Logistic Regression baseline
│   ├── regressor.joblib                 # Production Ridge Regressor
│   ├── preprocessor.joblib              # Fitted ColumnTransformer pipeline
│   └── feature_names.joblib             # 50 encoded feature names
├── metrics/
│   ├── classification_metrics.json      # Comprehensive test metrics & confusion matrix
│   ├── regression_metrics.json          # Regression MAE, RMSE, R2
│   └── stage_feasibility.json           # Stage feasibility analysis & report
├── explainability/
│   └── global_shap_summary.json         # Ranked global feature importances
└── metadata/
    └── model_version.json               # Checksum, version tag, and MLOps registry
```
