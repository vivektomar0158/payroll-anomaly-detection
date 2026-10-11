# Payroll Anomaly Detection

**Flagged $3.25M in gross pay at risk across 303 records (57 employees); recommended payroll review of the 10 highest-severity salary-jump and duplicate-payment cases before the next pay cycle.**

Dashboard (`AnomalyDashboard.pbip`) and findings below. Flags salary jumps, duplicate payments, and overtime outliers using SQL rules (`sql/rules.sql`, window functions) plus an Isolation Forest model (`ml/isolation_forest.py`), merged into one `data/anomalies.csv`.

## Dashboard

`AnomalyDashboard.pbip` — same PBIP pattern as the Analytics Dashboard project (see its README for the two Power BI Desktop preview flags to enable). Anomalies page: table of flagged records (`employee_id`, `pay_period`, `rule`, `detail`) and a bar chart of count by `rule`.

Note: `AnomalyDashboard.SemanticModel/definition/tables/Anomalies.tmdl` hardcodes an absolute path to `data/anomalies.csv` (Power Query's `File.Contents()` requires an absolute path). Update it if you move this project to a different machine or folder.

## Findings

Evaluated against the seeded ground truth (`data/ground_truth.json` — the employees deliberately given an unexplained salary jump, a duplicate payment, or an OT outlier when the synthetic dataset was built):

| Detector | Precision | Recall | F1 | TP | FP | FN |
|---|---|---|---|---|---|---|
| SQL rules | 1.00 | 0.886 | 0.939 | 31 | 0 | 4 |
| Isolation Forest | 0.537 | 0.829 | 0.652 | 29 | 25 | 6 |
| Combined | 0.561 | 0.914 | 0.696 | 32 | 25 | 3 |

The rule-based layer alone is the stronger detector here (zero false positives) — the ML layer catches a few cases rules miss but at a real precision cost. Recommendation: keep SQL rules as the primary flagging mechanism, use the ML score as a secondary "worth a second look" signal rather than an independent alert.

Run `python eval.py` after `run_detection.py` to regenerate this table (`eval_report.json`).

## Run

```
pip install -r requirements.txt
python run_detection.py
python eval.py
```

Output: `data/anomalies.csv` with columns `employee_id, pay_period, rule, detail` — `rule` is one of `salary_jump`, `duplicate_payment`, `ot_outlier`, or `ml_isolation_forest`.

## Tests

`pytest tests/ -q` — covers feature engineering (salary delta %) and that an injected extreme outlier is correctly flagged by the Isolation Forest.

## Tools

SQL (window functions, CTEs), Power BI + DAX, pandas, scikit-learn (Isolation Forest).

**Resume framing:** business impact and recommendations — dollars at risk, which detector to trust, what to review first.
