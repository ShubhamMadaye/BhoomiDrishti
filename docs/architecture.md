# System Architecture Document — BhoomiDrishti
**Problem Statement ID:** SIH26017  
**Sub-theme:** AI, ML & Predictive Analytics for Infrastructure Governance and Public Administration

---

## 1. High-Level Modular Monolith Architecture

BhoomiDrishti is designed as a **Modular Monolith** rather than fragmented microservices for the MVP phase. This architectural choice delivers:
- Atomic database transactions across projects, predictions, interventions, and audit records.
- Deterministic operational debugging and straightforward local execution for evaluators.
- Clear module boundaries facilitating future microservice decomposition if national transaction volume requires it.

```
                         +-----------------------------------+
                         |         CLIENT INTERFACES         |
                         +-----------------+-----------------+
                                           |
                   +-----------------------+-----------------------+
                   |                                               |
                   ↓                                               ↓
        WEB COMMAND CENTER (React 18)                   MOBILE APP (Flutter)
     - Executive KPI Dashboards                       - Field Intelligence & Action
     - State & District Comparative GIS               - Offline Update Queue
     - Model Performance Monitoring                   - Geo-Tagged Evidence Capture
     - Intervention Creation & Admin                  - Local Idempotent Sync
                   |                                               |
                   +-----------------------+-----------------------+
                                           |
                                           ↓ (HTTPS / REST / JWT)
                         +-----------------------------------+
                         |      FASTAPI APPLICATION CORE     |
                         +-----------------+-----------------+
                                           |
        +------------------+---------------+------------------+------------------+
        |                  |                                  |                  |
        ↓                  ↓                                  ↓                  ↓
  AUTH & RBAC       PROJECT SERVICE                     INTERVENTION        ALERT ENGINE
  - JWT Tokens      - Stage Lifecycles                  - Closed Loop       - Severity Tiers
  - Object Scopes   - Field Updates                     - Officer Assign    - Real-Time Push
  - Audit Events    - Recalculation Trigger             - Verification      - In-App Alerts
        |                  |                                  |                  |
        +------------------+---------------+------------------+------------------+
                                           |
                                           ↓
                         +-----------------------------------+
                         |             ML SUBSYSTEM          |
                         | - Model Registry (Artifact Hash)  |
                         | - Calibrated Risk Scorer (0-100)  |
                         | - SHAP Explainability Engine      |
                         | - Recommendation Rule Mapper      |
                         | - Drift & Calibration Health      |
                         +-----------------+-----------------+
                                           |
                                           ↓
                         +-----------------------------------+
                         |       PERSISTENCE & STORAGE       |
                         | - PostgreSQL 16 (Relational/GIS)  |
                         | - Alembic Schema Migrations       |
                         | - Immutable Audit Log Store       |
                         +-----------------------------------+
```

---

## 2. Closed-Loop Operational Workflow

BhoomiDrishti replaces reactive discovery with an end-to-end active feedback loop:

1. **Ingestion & Validation**: Project data is ingested (currently from the reproducible synthetic generator; modularly swappable for future government sources).
2. **Prediction**: The calibrated classifier predicts delay probability ($P_{\text{delay}}$) and projects risk onto a standardized 0–100 scale.
3. **Explanation**: SHAP feature attributions isolate top positive and negative risk contributors (e.g. approval stagnation, compensation lag).
4. **Prioritization**: Projects are categorized into HIGH, MEDIUM, and LOW tiers using configurable administrative thresholds.
5. **Recommendation**: Context-aware, traceable recommendations are generated for designated administrative roles.
6. **Intervention**: An authorized officer creates a trackable intervention with due date and assigned field personnel.
7. **Field Verification**: District officers use the mobile app (online or offline) to submit field updates and attached evidence.
8. **Risk Recalculation**: Supported field updates trigger automated score re-evaluation and delta tracking ($\Delta \text{Risk}$).
9. **Audit Trail**: Every transaction is immutably logged with timestamp, user ID, and diff.
10. **Continuous Learning**: Retraining pipelines ingest validated outcomes under strict data drift and calibration guards.
