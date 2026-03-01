# Data Dictionary

## employees

- `employee_id`: synthetic surrogate key
- `employee_number`: human-readable employee code
- `full_name`: employee display name
- `email`: employee email
- `hire_date`: first hire date
- `termination_date`: nullable end date
- `home_cost_centre`: default employee cost centre

## employee_position_history

- `position_history_id`: synthetic surrogate key
- `employee_id`: employee reference
- `effective_start_date`: row validity start
- `effective_end_date`: nullable row validity end
- `job_title`: role label
- `cost_centre`: reporting cost centre
- `fte`: full-time equivalent value
- `updated_at`: last update timestamp

## projects

- `project_id`: synthetic surrogate key
- `project_code`: business project key (dedup target)
- `project_name`: project display name
- `client_name`: client display name
- `project_status`: status category
- `start_date`: project start
- `end_date`: nullable project end
- `updated_at`: last update timestamp

## timesheets

- `timesheet_id`: synthetic surrogate key
- `employee_id`: employee reference
- `entry_date`: work date
- `project_code`: project business key
- `hours`: reported hours
- `activity_code`: work activity code
- `receiver`: receiver-style encoded identifier
- `source_row_hash`: stable hash for duplicate checks

## position_history_timesheets

- `unique_row_id`: preserved business row hash from source timesheets
- `match_type`: temporal match mode (`FLOOR` or `CEILING`)
- `job_title`: matched role title at work date
- `business_title`: matched business title at work date
- `cost_center`: matched cost centre at work date
- `finance_week`: finance week bucket for utilization calculations
- `calculated_daily_fte`: expected daily capacity from bookable hours and reported FTE
- `calculated_weekly_fte`: expected weekly capacity from bookable hours and reported FTE
- `fte_finance_week`: consumed FTE ratio based on weekly hours/bookable hours
- `overtime_day_flag`: overtime signal against daily expected capacity

## project_activity_timesheets

- `unique_row_id`: preserved business row hash from source timesheets
- `project_id_resolved`: resolved project identifier after key fallback logic
- `assignment_source`: mapped assignment channel used for scoring
- `mapping_strength`: weighted score from name/role/date/activity matches
- `timesheet_fte_consumed`: consumed FTE on the timesheet side
- `resource_consumed_fte`: consumed FTE on the assignment/resource side
- `fte_variance`: difference between timesheet and resource FTE consumption
- `amp_8_hours_match`: indicates whether an assignment match was found
- `recommendation`: final action label (`Aligned`, `Mismatch`, etc.)
