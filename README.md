# Workforce Analytics SQL Pipeline

I built this project as a SQL case study around one practical workforce analytics issue:
when employees move between roles over time, timesheet reporting is often joined to the wrong role context.

This is a recreation of real work I delivered during my apprenticeship, rebuilt end-to-end with synthetic data so I can safely show the business use case and my SQL decision-making.

My objective was to show that I can design a robust SQL pipeline, model it for analytics, and prove data quality with explicit validation evidence.

## Problem I Solved

In many reporting stacks, timesheets are joined to the latest employee role record, not the role that was valid on the work date.
That creates incorrect historical labor attribution by role, business unit, and cost centre.

This was the same core reporting problem I addressed in production during my apprenticeship, and here I have reproduced the logic with synthetic data to make the approach portfolio-safe.

I solved this by:
- Matching each timesheet row to position history using temporal logic.
- Building a project activity fact output at row-level grain.
- Adding validation layers to prove row fidelity, key integrity, and explainable variance outcomes.

## What I Built

Main SQL outputs:
- `position_history_timesheets`
  This output enriches each timesheet row with matched position history and capacity context.
- `project_activity_timesheets`
  This output adds project mapping, assignment scoring, FTE variance signals, and recommendation flags.

## STAR Summary (Scenario, Task, Action, Reflection)

### Scenario
During my apprenticeship, I worked on workforce reporting where timesheets needed to be attributed to the correct role and capacity context on the exact work date.
The recurring issue was that many joins used "latest role" logic, which distorted historical reporting.

### Task
Recreate that business problem as a portfolio project and demonstrate, with SQL, how to:
- perform correct temporal matching,
- preserve row-level traceability,
- and validate output quality in a way that is reviewable by employers.

### Action
I built a two-stage SQL pipeline:
- `position_history_timesheets` to perform deterministic temporal matching (`FLOOR`/`CEILING` logic with tie-break ranking),
- `project_activity_timesheets` to add project mapping, capacity context, and FTE variance signals.

I then added validation queries directly under the build SQL to prove:
- row-count and key parity,
- matching behavior distributions,
- and explainable recommendation outcomes.

### Reflection
This recreation let me show the same decision quality I used in production, while using synthetic data.
The strongest lesson was that temporal joins and validation design are as important as transformation logic if the output will drive workforce decisions.

## Data Model Choice

I kept the model intentionally practical for analytics and auditability:
- Source tables: `employees`, `employee_position_history`, `projects`, `timesheets`
- Enrichment layer: `position_history_timesheets` (time-correct role/FTE context)
- Final fact-style output: `project_activity_timesheets` at timesheet-row grain
- Lineage key: `unique_row_id` for traceability back to source records

## Core SQL Logic

### Temporal Position Matching

For each timesheet row:
1. Match on employee.
2. Prefer the most recent prior effective position (`FLOOR`).
3. If no prior record exists, use earliest future effective position (`CEILING`).
4. Use deterministic tie-breaking with `ROW_NUMBER()`.

### FTE Capacity and Variance Method

After temporal matching, I pull the matched `fte` from position history and calculate expected capacity:
- `calculated_daily_fte = daily_bookable_hours * fte`
- `calculated_weekly_fte = weekly_bookable_hours * fte`

Then I calculate consumed and variance metrics:
- `timesheet_fte_consumed = (hours / calculated_daily_fte) / dup_count` for valid work rows
- `fte_variance = timesheet_fte_consumed - resource_consumed_fte`

Final recommendation flags:
- `Aligned`
- `Mismatch`
- `No resource FTE`
- `Not applicable (Overhead/Off_Work)`

## Validation Framework

I built validation as a first-class output, not an afterthought:
- Row-count reconciliation from source to both outputs.
- Distinct key parity (`source_row_hash` to `unique_row_id`).
- Temporal match distribution checks (`FLOOR` vs `CEILING`).
- Mapping strength and assignment-source coverage checks.
- FTE variance distribution and mismatch drilldown.

## SQL Showcase Notebooks

These are the main artifacts I would ask an employer to review.

Run in order:
1. [05_sql_only_position_history_timesheets.ipynb](notebooks/05_sql_only_position_history_timesheets.ipynb)
2. [06_sql_only_project_activity_timesheets.ipynb](notebooks/06_sql_only_project_activity_timesheets.ipynb)

Both notebooks are SQL-only and include build logic plus validations directly underneath.

## Visual Evidence

### Pipeline

`Base Timesheets -> Dedup -> Enrich -> Temporal Match -> Activity Mapping -> Validate`

![Pipeline flow](docs/screenshots/pipeline_flow.png)

### Data Model (ERD)

![ERD](docs/screenshots/erd.png)

### Position History Build

![Position history build](docs/screenshots/position_history_build.png)

### Project Activity Build

![Project activity build](docs/screenshots/project_activity_build.png)

### Validation Snapshot

![Validation checks](docs/screenshots/validation_checks.png)

### Mismatch Drilldown

![Mismatch drilldown](docs/screenshots/mismatch_drilldown.png)

### Dashboard Mock

![Dashboard mock](docs/screenshots/dashboard_mock.png)

## Example Results (Local Run, March 1, 2026)

- `timesheets`: `40,000` rows
- `position_history_timesheets`: `40,000` rows
- `project_activity_timesheets`: `40,000` rows
- Distinct business row keys preserved across source and outputs: `36,800`

Recommendation split:
- `Not applicable (Overhead/Off_Work)`: `14,821`
- `Mismatch`: `12,603`
- `No resource FTE`: `7,192`
- `Aligned`: `5,384`

## How To Review

Minimum prerequisite:
- PostgreSQL with source tables loaded:
  `employees`, `employee_position_history`, `projects`, `timesheets`

Review flow:
1. Run notebook `05`
2. Run notebook `06`
3. Compare SQL outputs with validation screenshots in `docs/screenshots`

## Repository Structure

- `notebooks`: SQL-only showcase notebooks
- `diagrams`: ERD and pipeline visuals
- `docs`: business context, assumptions, data dictionary, validation strategy
