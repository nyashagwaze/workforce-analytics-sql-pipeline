# Assumptions And Rules

## Temporal match priority

1. Floor match first: latest position record where `effective_start_date <= entry_date`.
2. If none exists, use earliest future record where `effective_start_date > entry_date`.
3. Tie-break on most recent `updated_at` when candidates are otherwise equal.

## Data quality assumptions

- Position history may contain gaps and overlaps.
- Projects may contain duplicate `project_code` rows.
- Timesheets may contain expected duplicate business rows.

## Duplicate handling

- Expected duplicates are preserved for auditability.
- Unexpected duplicates are surfaced in validation checks.

## Recommendation rules in project activity output

- `Not applicable (Overhead/Off_Work)`: row is excluded from assignment quality checks.
- `Unmapped assignment`: no assignment candidate was found for the row.
- `Weak match`: assignment found but low mapping strength.
- `No resource FTE`: match exists but resource-side FTE is unavailable.
- `Aligned`: absolute FTE variance is below threshold.
- `Mismatch`: absolute FTE variance meets or exceeds threshold.
