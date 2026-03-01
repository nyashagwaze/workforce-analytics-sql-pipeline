# Validation Strategy

## Row fidelity

- Reconcile row counts from `timesheets` to `position_history_timesheets` and `project_activity_timesheets`.
- Reconcile distinct row keys from `source_row_hash` to `unique_row_id` in both outputs.
- Confirm no accidental row loss through joins, ranking, and candidate selection.

## Key and duplicate checks

- Validate expected duplicate patterns via `source_row_hash` and `unique_row_id`.
- Validate project-key resolution coverage (`project_id_resolved` null-rate).
- Distinguish expected duplicate groups from unexpected duplication behavior.

## Temporal logic tests

- Confirm `FLOOR` match behavior for prior-effective records.
- Confirm fallback `CEILING` behavior when no prior record exists.
- Check that temporal match distribution is stable and explainable.

## Coverage and anomalies

- Null coverage checks for `job_title`, `business_title`, `mapping_strength`, `fte_variance`.
- Mapping quality checks: `name_match`, `role_match`, `date_match`, `activity_match`.
- Recommendation distribution checks: `Aligned`, `Mismatch`, `No resource FTE`, `Not applicable`.
- Spot-check high-hour outliers and high absolute FTE variance rows.

## Where To Run

- SQL-only build + validation notebooks:
  - `notebooks/05_sql_only_position_history_timesheets.ipynb`
  - `notebooks/06_sql_only_project_activity_timesheets.ipynb`
- Validation SQL files:
  - `sql/30_validation/301_row_count_reconciliation.sql`
  - `sql/30_validation/302_duplicate_and_key_checks.sql`
  - `sql/30_validation/303_temporal_logic_tests.sql`
  - `sql/30_validation/304_null_coverage_and_quality.sql`
  - `sql/30_validation/305_anomaly_spot_checks.sql`
