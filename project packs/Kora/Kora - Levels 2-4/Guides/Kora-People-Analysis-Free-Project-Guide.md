
<!-- page break -->

# Kora People Analysis - Free Project Guide

Use the SQL objects required for the chosen case. SQL Server and MySQL support stored procedures; PostgreSQL can use a parameterized function. SQLite has no stored procedures: submit a reusable view and a saved parameterized query with equivalent output. SQL Server and PostgreSQL use the kora schema; MySQL and SQLite use tables in the selected database without that prefix.

## Business problem A

## The decision

Why are people leaving?

The People Manager wants a focused retention plan. Determine where departures represent a sustained concern, how hiring offsets them and which recorded explanations warrant action. These fictional records describe no real workplace.

## Your data

KoraPeople: employees, employment_spells, assignments, staff_monthly_snapshot, staff_monthly_hours, grades, branches and months. Exit events remain in spells even after employees disappear from active snapshots.

## Possible questions

1. Reconcile branch and organization workforce changes through hires, rehires, exits and transfers. Explain why promotions and pay reviews do not change headcount.

2. Compare voluntary and other exits by tenure, role, branch and hiring cohort. Show counts and appropriate rates together.

3. Which exit reasons recur, and where are reasons missing? Include Unpleasant Work Environment exactly as recorded; do not impute the seven missing voluntary reasons.

4. Does the pattern persist after considering role mix, tenure, branch moves and time at risk? Avoid attributing an entire employment history to the final manager.

5. Identify employees who joined and left within a month and rehires. Explain what month-end snapshots alone would miss.

6. Recommend at least two proportionate actions with supporting evidence, limitations and a follow-up plan.

## Think It Through

Frame the task: define whose retention matters and the observation window. Challenge reasoning: test whether role mix or short tenure explains the apparent branch difference before attributing it to management.


<!-- page break -->

## Main measures

An employment spell is active on a reporting date when hire_date is on or before that date and termination_date is later or NULL. The termination date is the first day the person is no longer employed.

For each branch, closing headcount = opening headcount + hires - exits + transfers in - transfers out. Transfers cancel when branches are combined. Count each employment-spell start, including a rehire, as a hire.

Approximate monthly turnover = departures during the month divided by (opening headcount + closing headcount) / 2. Show voluntary departures separately and explain volatile rates in small groups.

For retention, define a starting hire cohort and a fixed follow-up period. Do not compare first-year retention for hires without a full year of observation.

Exit reason is a recorded primary explanation, employee-reported for voluntary resignation and employer-recorded otherwise. Missing reasons are unknown; recorded reasons are not independently verified causes.

## Required SQL objects

Create kora.usp_StaffMovement(@StartDate, @EndDate, @BranchID = NULL). Return month, branch, opening and closing headcount, hires, exits, transfers in/out, promotions and reconciliation difference.

Require complete months in 2022-2025 and validate inputs. Derive events from dated histories, including movements invisible to month-end snapshots. Provide all 288 branch-month rows for the full period.


<!-- page break -->

## Business problem B

## The decision

Are progression opportunities distributed fairly?

Leadership has received concerns that opportunities vary across branches and managers. Assess whether recorded promotion and progression patterns support a targeted review, accounting for eligibility, assessment results and available places. The data cannot establish discriminatory intent.

## Your data

KoraPeople: promotion_rounds, promotion_candidates, assignments, employment_spells, employees, grades, branches and months. This edition contains no ethnicity or other self-identification fields; the investigation concerns recorded access and outcomes.

## Possible questions

1. Reconstruct employee journeys across roles, grades, branches, managers and employment spells. Distinguish a pay review from promotion.

2. Compare selection rates using eligible considered candidates within each round. Show available positions and denominator sizes alongside rates.

3. Compare assessment results, tenure and grade at assessment. Does an apparent branch or manager disparity persist within comparable groups?

4. How much assessment-score and decision-reason information is missing? Test how that affects your confidence; do not treat missing scores as zero.

5. Check whether selected promotions became effective as recorded. Separate decision dates from implementation dates.

6. Conclude which concerns are supported, weakened or unresolved. Recommend at least two proportionate actions and name what additional evidence would change your conclusion.

## Think It Through

Explore and Analyze: compare like with like. Challenge reasoning: could fewer available positions explain the selection-rate difference? State where access data is incomplete instead of claiming fairness is proved.


<!-- page break -->

## Main measures

Candidate selection rate = selected eligible candidate rows divided by all eligible considered candidate rows. One employee considered in two rounds contributes two observations.

The candidate table does not contain all staff or all potentially eligible people. You cannot establish complete fairness of access from selection outcomes alone.

assignment_id_at_assessment references the role held on decision_date. Join round_id to promotion_rounds for decision_date and effective_date; find the resulting promotion by spell and effective date.

An assignment starts on effective_start and stops before effective_end. A NULL effective_end means the assignment remains open through the dataset cutoff of 31 December 2025; do not use the computer current date.

Grade represents responsibility within a job family. A promotion to senior practitioner can retain the same role_name; grades 1-3 may be assessed for grade 4.

## Required SQL objects

Create kora.vw_EmployeeJourney and kora.usp_EmployeeJourney(@EmployeeID, @AsOfDate). Return spell, role, branch, manager, grade, salary, effective start/end, last observed date and change type in chronological order. Hide outcomes after @AsOfDate; reject unknown employees or dates outside the extract.

Create kora.vw_PromotionAssessment joining candidates, rounds and assessed assignments. Return assessment context and the resulting promotion assignment when selected. Preserve non-selected candidates and missing scores.


<!-- page break -->

# Submission

## Expected deliverables

Pick one business problem. Submit one reproducible SQL file containing analysis queries, the reusable objects required for your chosen problem, readable aliases, explanatory comments and executable validation tests.

One report page in a BI tool of your choice, with clear labels. Supply the editable report and a PDF or image export. *EVERY VISUAL SHOULD HAVE A JOB.*

One recommendation page with at least two actions, supporting numbers, reasoning, limitations, responsible stakeholders and proposed follow-up measures.

One methodology page organized around the Think It Through Framework. Explain any definitions you developed for metrics beyond those provided in the project guide, your quality checks, assumptions and limitations.
