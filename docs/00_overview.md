# Overview

This repository builds an end-to-end temporal matching pipeline for workforce analytics.

## Core outputs

- Deduplicated projects
- Timesheet records enriched and matched to historical position context
- Project activity fact table with assignment scoring and recommendation flags
- Validation outputs that prove row fidelity and match quality

## Main tables

- `employees`
- `employee_position_history`
- `projects`
- `timesheets`
- `projects_dedup`
- `timesheets_enriched`
- `position_history_timesheets`
- `project_activity_timesheets`
