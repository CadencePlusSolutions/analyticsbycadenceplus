# Kora reporting databases data dictionary

Version 0.2-review | Four reporting years: 1 January 2022 to 31 December 2025 | Currency NGN | Synthetic training data

This page documents both databases, every table and every field. SQL Server and PostgreSQL use schema `kora`; MySQL uses its database as the namespace; SQLite uses one file per database. Primary and foreign keys are identified below. The same canonical CSV rows are supplied for all four database engines. The SQL type column below uses SQL Server names; PostgreSQL maps `nvarchar` to `varchar`, `decimal` to `numeric`, and `bit` to `boolean`; MySQL uses `varchar`, `decimal`, and `tinyint(1)`; SQLite uses `TEXT`, `NUMERIC`, and `INTEGER` affinities with ISO date strings.

## Database definitions

| Database | Purpose | Main project use |
| --- | --- | --- |
| KoraOperations (Operations and Financials) | Customer demand, delivery, recognized sales, returns, inventory, procurement, recorded costs, budgets and customer receivables | Financial analysis and operations |
| KoraPeople | Employee identity, continuous employment spells, effective-dated assignments, workforce snapshots, hours and promotion assessments | People analysis |

## Interpretation rules

- Event tables record dated transactions; effective-dated tables record changing states; snapshot tables record month-end balances. A snapshot date is not imposed on transaction records because that would conflate event timing with observation timing.
- Reporting covers 48 month ends. Known staff histories before 2022 support opening populations. Promises and due dates may extend beyond cutoff; actual events after cutoff are absent.
- Start dates are inclusive; employment and assignment end dates are exclusive. NULL end dates mean current through cutoff. IDs identify records, not chronological order.
- NULL means no recorded value, with field-specific meaning below. It does not mean zero, false or an inferred category. Empty CSV cells represent SQL NULL.
- Amounts exclude tax and use NGN. Decimal ratios are fractions, not percentages. Inventory uses fixed standard unit costs. Snapshot balances must not be summed across months.
- Repeated customer, product, employee or branch IDs in event tables are expected. One-to-many joins can duplicate measures: aggregate independent child records before joining.
- `returns.refund_ngn` is a legacy name for the recognized sales credit. It matches `credit_notes.credit_amount_ngn`; actual cash refunds are separate. Count the revenue credit once.
- `operating_expenses` contains the costs also summarized in `branch_monthly_resources`. Do not add both. Core staffing allowance is not a payroll reconciliation to annual employee salaries.
- Receivables cover invoices issued in this extract only. No pre-2022 opening receivables, bad-debt provisions or write-offs are supplied.
- Candidate records cover eligible considered employees, not everyone who might have sought progression. There are no ethnicity or self-identification fields in this edition.
- Category examples below list observed values, not a promise that unobserved values could never occur in a real source system.

## Important connection examples

**Employee and manager:** `assignments.spell_id -> employment_spells.spell_id -> employees.employee_id` identifies the employee. `assignments.manager_employee_id -> employees.employee_id` identifies the manager. EMP0020 holds spell 20 and assignment 60; EMP0002 is their manager.

**Promotion assessment:** `promotion_candidates.assignment_id_at_assessment -> assignments.assignment_id` identifies the role held at assessment. `promotion_candidates.round_id -> promotion_rounds.round_id` supplies decision and effective dates. EMP0135 was assessed on assignment 538 in round 10; promoted assignment 1342 starts 1 April 2024; later assignment 539 starts 1 July 2024.

**Receivables:** invoices connect to delivery headers, while sales lines connect through delivery_id. Cash receipts connect to invoices through payment_allocations. Credit notes connect returns to invoices; cash_refunds references credit notes. Aggregate each event stream independently before reconstructing invoice balances.

**Procurement:** purchase_orders -> purchase_order_lines -> goods_receipts. Each goods receipt references one inventory movement. A stock shortage references the sales line being fulfilled and its emergency purchase order. Use movement_date then movement_id for within-day replay.

## Table index


### KoraOperations

| Table | Grain and purpose | Rows |
| --- | --- | ---: |
| [dataset_info](#koraoperations-dataset-info) | One row defining the reporting window and version | 1 |
| [months](#koraoperations-months) | One row per month end; 48 reporting periods | 48 |
| [branches](#koraoperations-branches) | One row per distribution centre; same IDs in both databases | 6 |
| [customers](#koraoperations-customers) | One fictional organization; includes customers with no orders | 800 |
| [products](#koraoperations-products) | One product; standard unit cost is fixed in this teaching model | 36 |
| [orders](#koraoperations-orders) | One order; standard work hours reflect its handling complexity, not actual labour | 27,999 |
| [order_lines](#koraoperations-order-lines) | One product line per order; net value excludes tax and includes cancelled demand | 56,009 |
| [deliveries](#koraoperations-deliveries) | One consignment; split orders have several. Unresolved shipments have no delivery date | 30,357 |
| [delivery_lines](#koraoperations-delivery-lines) | One delivered order line in a consignment; only completed deliveries have rows | 60,366 |
| [sales_lines](#koraoperations-sales-lines) | One invoiced delivered line; this teaching model invoices at completed delivery, excluding tax | 60,366 |
| [returns](#koraoperations-returns) | One full or partial return of an invoiced line; recovery only for restocked units | 2,310 |
| [inventory_movements](#koraoperations-inventory-movements) | One stock movement; opening balances on 1 January 2022 plus receipts sales returns and adjustments | 71,208 |
| [inventory_monthly_snapshot](#koraoperations-inventory-monthly-snapshot) | One product at one branch at month end; balances are not additive across months | 10,368 |
| [open_order_monthly_snapshot](#koraoperations-open-order-monthly-snapshot) | One unfulfilled noncancelled order line at month end; completed lines are absent | 12,520 |
| [branch_monthly_resources](#koraoperations-branch-monthly-resources) | One branch month; period flows despite month-end date. Hours reconcile to non-manager HR hours; costs exclude goods shipping and refunds | 288 |
| [payment_terms](#koraoperations-payment-terms) | One payment term; copied to each invoice at issue | 3 |
| [invoices](#koraoperations-invoices) | One invoice per completed delivery; amount equals its sales lines, excluding tax | 30,194 |
| [credit_notes](#koraoperations-credit-notes) | One recognized sales credit per return; a credit is not itself a cash refund | 2,310 |
| [cash_receipts](#koraoperations-cash-receipts) | One customer cash receipt; all receipts fully allocated in this teaching extract | 29,019 |
| [payment_allocations](#koraoperations-payment-allocations) | One receipt allocation to an invoice; a receipt may settle several invoices | 29,622 |
| [cash_refunds](#koraoperations-cash-refunds) | One cash repayment of a customer credit balance; decreases cash and increases net receivables | 437 |
| [operating_expenses](#koraoperations-operating-expenses) | One recognized branch expense component; fixed costs, overtime and shipping, excluding goods, tax, interest and capital expenditure | 31,509 |
| [monthly_budgets](#koraoperations-monthly-budgets) | One approved branch budget for a month; frozen planning assumptions, not fitted actuals | 288 |
| [receivables_monthly_snapshot](#koraoperations-receivables-monthly-snapshot) | One nonzero invoice balance at month end; negative balances are customer credits, not overdue debt | 120,995 |
| [suppliers](#koraoperations-suppliers) | One fictional supplier with standard planning lead time | 6 |
| [purchase_orders](#koraoperations-purchase-orders) | One supplier purchase order; Standard or Emergency. Some remain outstanding at cutoff | 7,947 |
| [purchase_order_lines](#koraoperations-purchase-order-lines) | One product line on a supplier order; current extract uses one line per purchase order | 7,947 |
| [goods_receipts](#koraoperations-goods-receipts) | One receipt against a purchase line; partial receipts are possible | 8,938 |
| [stock_shortage_events](#koraoperations-stock-shortage-events) | One failed stock check immediately before fulfilment; same-day emergency procurement supplies the shortage. Not a duration measure or proof of the cause of delivery lateness | 3,816 |

### KoraPeople

| Table | Grain and purpose | Rows |
| --- | --- | ---: |
| [dataset_info](#korapeople-dataset-info) | One row defining the reporting window and version | 1 |
| [months](#korapeople-months) | One row per month end; 48 reporting periods | 48 |
| [branches](#korapeople-branches) | One row per distribution centre; same IDs in both databases | 6 |
| [employees](#korapeople-employees) | One row per person; names are synthetic labels and persist across rehires | 372 |
| [employment_spells](#korapeople-employment-spells) | One continuous employment period; rehire creates a new spell | 383 |
| [assignments](#korapeople-assignments) | One effective-dated branch role manager and pay assignment within an employment spell | 1,357 |
| [staff_monthly_snapshot](#korapeople-staff-monthly-snapshot) | One active employee at each month end; terminated employees disappear after their exclusive termination date | 11,112 |
| [staff_monthly_hours](#korapeople-staff-monthly-hours) | One assignment active for any part of a month; hours prorated by weekdays including midmonth hires and leavers | 11,227 |
| [promotion_rounds](#korapeople-promotion-rounds) | One branch promotion round; frozen eligibility and scores precede the decision | 18 |
| [promotion_candidates](#korapeople-promotion-candidates) | One eligible considered employee in a round; includes non-selected employees and missing scores | 500 |
| [grades](#korapeople-grades) | One internal responsibility band; promotions may retain the same job family title | 6 |

<a id="koraoperations-dataset-info"></a>

## KoraOperations dataset_info

One row defining the reporting window and version.

**Primary key:** `dataset_version`. **Rows:** 1.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `dataset_version` | varchar(30) | No | PK | Synthetic extract release identifier. |
| `start_date` | date | No | - | First day of the reporting window. |
| `cutoff_date` | date | No | - | Last observed reporting date; no later outcomes are provided. |
| `currency` | varchar(3) | No | - | Currency code NGN; monetary values exclude tax. |

<a id="koraoperations-months"></a>

## KoraOperations months

One row per month end; 48 reporting periods.

**Primary key:** `snapshot_date`. **Rows:** 48.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `month_start` | date | No | - | First day of the reporting month. |
| `calendar_year` | int | No | - | Four-digit calendar year. |
| `month_number` | int | No | - | Month number from 1 to 12. |

<a id="koraoperations-branches"></a>

## KoraOperations branches

One row per distribution centre; same IDs in both databases.

**Primary key:** `branch_id`. **Rows:** 6.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `branch_id` | varchar(8) | No | PK | Distribution centre key; shared consistently across both databases. |
| `branch_name` | nvarchar(80) | No | - | Fictional distribution centre name. |
| `region_name` | nvarchar(40) | No | - | Geographical region; customer and branch geography have different roles. |
| `city` | nvarchar(40) | No | - | Branch city. |

<a id="korapeople-dataset-info"></a>

## KoraPeople dataset_info

One row defining the reporting window and version.

**Primary key:** `dataset_version`. **Rows:** 1.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `dataset_version` | varchar(30) | No | PK | Synthetic extract release identifier. |
| `start_date` | date | No | - | First day of the reporting window. |
| `cutoff_date` | date | No | - | Last observed reporting date; no later outcomes are provided. |
| `currency` | varchar(3) | No | - | Currency code NGN; monetary values exclude tax. |

<a id="korapeople-months"></a>

## KoraPeople months

One row per month end; 48 reporting periods.

**Primary key:** `snapshot_date`. **Rows:** 48.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `month_start` | date | No | - | First day of the reporting month. |
| `calendar_year` | int | No | - | Four-digit calendar year. |
| `month_number` | int | No | - | Month number from 1 to 12. |

<a id="korapeople-branches"></a>

## KoraPeople branches

One row per distribution centre; same IDs in both databases.

**Primary key:** `branch_id`. **Rows:** 6.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `branch_id` | varchar(8) | No | PK | Distribution centre key; shared consistently across both databases. |
| `branch_name` | nvarchar(80) | No | - | Fictional distribution centre name. |
| `region_name` | nvarchar(40) | No | - | Geographical region; customer and branch geography have different roles. |
| `city` | nvarchar(40) | No | - | Branch city. |

<a id="korapeople-employees"></a>

## KoraPeople employees

One row per person; names are synthetic labels and persist across rehires.

**Primary key:** `employee_id`. **Rows:** 372.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `employee_id` | varchar(12) | No | PK | Stable fictional person identifier retained across rehires. |
| `employee_name` | nvarchar(80) | No | - | Synthetic person label; encodes no demographic identity. |
| `first_hire_date` | date | No | - | Earliest known employment start; may precede 2022. |

<a id="korapeople-employment-spells"></a>

## KoraPeople employment_spells

One continuous employment period; rehire creates a new spell.

**Primary key:** `spell_id`. **Rows:** 383.

**SQL checks:** `termination_date IS NULL OR termination_date > hire_date`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `spell_id` | int | No | PK | Key of an uninterrupted employment period; rehire creates a new spell. |
| `employee_id` | varchar(12) | No | FK to KoraPeople.employees.employee_id | Stable fictional person identifier retained across rehires. |
| `hire_date` | date | No | - | First employed day of this spell, inclusive. |
| `termination_date` | date | Yes | - | First day no longer employed; exclusive interval end |
| `exit_type` | varchar(30) | Yes | - | Voluntary resignation, Contract completion, Dismissal or Redundancy; NULL for active spells. Observed values: Contract completion, Dismissal, Redundancy, Voluntary resignation. |
| `exit_reason` | nvarchar(100) | Yes | - | Recorded primary explanation: employee-reported for voluntary resignations, employer-recorded for other exits. NULL on active spells or where no reason was recorded. Not an independently verified cause. Observed values: Attendance requirements not met, Career opportunity, Conduct concerns, Department restructuring, Fixed term ended, Limited career progression, Pay and benefits, Performance expectations not met, Personal reasons, Probation requirements not met, Project completed, Reduced business demand, Relocation, Role consolidation, Temporary cover ended, Unpleasant Work Environment, Workload and work-life balance. |

<a id="korapeople-assignments"></a>

## KoraPeople assignments

One effective-dated branch role manager and pay assignment within an employment spell.

**Primary key:** `assignment_id`. **Rows:** 1,357.

**SQL checks:** `effective_end IS NULL OR effective_end > effective_start`; `fte > 0 AND fte <= 1`; `annual_salary_ngn > 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `assignment_id` | int | No | PK | Effective-dated assignment key. Sort by effective_start, not by this ID. |
| `spell_id` | int | No | FK to KoraPeople.employment_spells.spell_id | Key of an uninterrupted employment period; rehire creates a new spell. |
| `branch_id` | varchar(8) | No | FK to KoraPeople.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `manager_employee_id` | varchar(12) | Yes | FK to KoraPeople.employees.employee_id | Reporting manager employee ID, not the employee holding the assignment. NULL for branch managers. |
| `role_name` | nvarchar(50) | No | - | Job family title; seniority is recorded separately in grade. |
| `grade` | int | No | - | Internal responsibility band, defined in KoraPeople.grades. Promotions can retain job family titles. |
| `annual_salary_ngn` | decimal(18,2) | No | - | Annualized contractual base salary; not actual payroll or a monthly expense. |
| `fte` | decimal(5,2) | No | - | Full-time equivalent fraction: 1.00 full-time; 0.80 four-fifths. |
| `effective_start` | date | No | - | First date the assignment applies, inclusive. |
| `effective_end` | date | Yes | - | Exclusive end; NULL means still current at cutoff |
| `change_type` | varchar(25) | No | - | Hire, Rehire, Transfer, Pay review or Promotion. A pay review alone is not promotion. Observed values: Hire, Pay review, Promotion, Rehire, Transfer. |

<a id="korapeople-staff-monthly-snapshot"></a>

## KoraPeople staff_monthly_snapshot

One active employee at each month end; terminated employees disappear after their exclusive termination date.

**Primary key:** `snapshot_date, employee_id`. **Rows:** 11,112.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK; FK to KoraPeople.months.snapshot_date | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `employee_id` | varchar(12) | No | PK; FK to KoraPeople.employees.employee_id | Stable fictional person identifier retained across rehires. |
| `spell_id` | int | No | FK to KoraPeople.employment_spells.spell_id | Key of an uninterrupted employment period; rehire creates a new spell. |
| `assignment_id` | int | No | FK to KoraPeople.assignments.assignment_id | Effective-dated assignment key. Sort by effective_start, not by this ID. |
| `branch_id` | varchar(8) | No | FK to KoraPeople.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `manager_employee_id` | varchar(12) | Yes | FK to KoraPeople.employees.employee_id | Reporting manager employee ID, not the employee holding the assignment. NULL for branch managers. |
| `role_name` | nvarchar(50) | No | - | Job family title; seniority is recorded separately in grade. |
| `grade` | int | No | - | Internal responsibility band, defined in KoraPeople.grades. Promotions can retain job family titles. |
| `annual_salary_ngn` | decimal(18,2) | No | - | Annualized contractual base salary; not actual payroll or a monthly expense. |
| `fte` | decimal(5,2) | No | - | Full-time equivalent fraction: 1.00 full-time; 0.80 four-fifths. |

<a id="korapeople-staff-monthly-hours"></a>

## KoraPeople staff_monthly_hours

One assignment active for any part of a month; hours prorated by weekdays including midmonth hires and leavers.

**Primary key:** `snapshot_date, assignment_id`. **Rows:** 11,227.

**SQL checks:** `scheduled_hours >= absence_hours AND absence_hours >= 0`; `overtime_hours >= 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK; FK to KoraPeople.months.snapshot_date | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `assignment_id` | int | No | PK; FK to KoraPeople.assignments.assignment_id | Effective-dated assignment key. Sort by effective_start, not by this ID. |
| `scheduled_hours` | decimal(12,2) | No | - | Assignment hours scheduled in the month, prorated by weekdays in its effective interval. |
| `absence_hours` | decimal(12,2) | No | - | Scheduled hours not worked; included in scheduled_hours. |
| `overtime_hours` | decimal(12,2) | No | - | Additional hours beyond scheduled hours. |

<a id="korapeople-promotion-rounds"></a>

## KoraPeople promotion_rounds

One branch promotion round; frozen eligibility and scores precede the decision.

**Primary key:** `round_id`. **Rows:** 18.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `round_id` | int | No | PK | Branch promotion assessment round key. |
| `branch_id` | varchar(8) | No | FK to KoraPeople.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `decision_date` | date | No | - | Date assessment and selection were recorded. |
| `effective_date` | date | No | - | Date selected promotions take effect, distinct from decision_date. |
| `target_grade` | int | No | - | Responsibility band awarded to selected candidates. |
| `positions` | int | No | - | Number of promotion places. |
| `eligibility_policy` | nvarchar(200) | No | - | Recorded rules for consideration and selection. |

<a id="korapeople-promotion-candidates"></a>

## KoraPeople promotion_candidates

One eligible considered employee in a round; includes non-selected employees and missing scores.

**Primary key:** `round_id, employee_id`. **Rows:** 500.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `round_id` | int | No | PK; FK to KoraPeople.promotion_rounds.round_id | Branch promotion assessment round key. |
| `employee_id` | varchar(12) | No | PK; FK to KoraPeople.employees.employee_id | Stable fictional person identifier retained across rehires. |
| `assignment_id_at_assessment` | int | No | FK to KoraPeople.assignments.assignment_id | Assignment held on the promotion round decision date; not the resulting promoted assignment |
| `eligible` | bit | No | - | 1 means eligible under the recorded policy. Only eligible considered candidates appear; absence is not proof of ineligibility. |
| `assessment_score` | decimal(5,2) | Yes | - | Recorded score on a 0 to 100 scale; NULL is missing, not zero. |
| `selected` | bit | No | - | 1 selected, 0 not selected in this round. People can appear in multiple rounds. |
| `decision_reason` | nvarchar(100) | Yes | - | Recorded selection explanation; NULL means unrecorded. Not independent proof of decision quality. |

<a id="koraoperations-customers"></a>

## KoraOperations customers

One fictional organization; includes customers with no orders.

**Primary key:** `customer_id`. **Rows:** 800.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `customer_id` | varchar(12) | No | PK | Stable customer key; some registered customers have no orders. |
| `customer_name` | nvarchar(80) | No | - | Fictional business customer name. |
| `region_name` | nvarchar(40) | No | - | Geographical region; customer and branch geography have different roles. |
| `segment` | varchar(30) | No | - | Customer business segment. Observed values: Corporate, Public sector, Small business. |

<a id="koraoperations-products"></a>

## KoraOperations products

One product; standard unit cost is fixed in this teaching model.

**Primary key:** `product_id`. **Rows:** 36.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `product_id` | varchar(12) | No | PK | Product key; quantities across categories may use unlike units. |
| `product_name` | nvarchar(80) | No | - | Fictional product description. |
| `category` | varchar(30) | No | - | Product category. Observed values: Furniture, Supplies, Technology. |
| `standard_unit_cost_ngn` | decimal(18,2) | No | - | Fixed standard unit cost used for inventory and goods cost; no cost inflation, FIFO or weighted-average costing. |

<a id="koraoperations-orders"></a>

## KoraOperations orders

One order; standard work hours reflect its handling complexity, not actual labour.

**Primary key:** `order_id`. **Rows:** 27,999.

**SQL checks:** `promised_date >= order_date`; `cancelled_date IS NULL OR cancelled_date >= order_date`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `order_id` | int | No | PK | Customer order key; can have several lines and consignments. |
| `customer_id` | varchar(12) | No | FK to KoraOperations.customers.customer_id | Stable customer key; some registered customers have no orders. |
| `branch_id` | varchar(8) | No | FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `order_date` | date | No | - | Date demand was recorded. |
| `promised_date` | date | No | - | Promised delivery completion date; may follow the cutoff. |
| `cancelled_date` | date | Yes | - | Date cancellation takes effect; NULL means no recorded cancellation. |
| `channel` | varchar(20) | No | - | Online, Account team or Branch order entry channel. Observed values: Account team, Branch, Online. |
| `complexity` | varchar(20) | No | - | Standard or Complex work classification. Observed values: Complex, Standard. |
| `standard_work_hours` | decimal(10,2) | No | - | Estimated whole-order effort; count once per order. Not observed productive time. |

<a id="koraoperations-order-lines"></a>

## KoraOperations order_lines

One product line per order; net value excludes tax and includes cancelled demand.

**Primary key:** `order_line_id`. **Rows:** 56,009.

**SQL checks:** `quantity > 0`; `discount_rate >= 0 AND discount_rate <= 1`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `order_line_id` | int | No | PK | One product line within a customer order. |
| `order_id` | int | No | FK to KoraOperations.orders.order_id | Customer order key; can have several lines and consignments. |
| `product_id` | varchar(12) | No | FK to KoraOperations.products.product_id | Product key; quantities across categories may use unlike units. |
| `quantity` | int | No | - | Units ordered on the customer line. |
| `unit_price_ngn` | decimal(18,2) | No | - | Selling price per unit before discount, excluding tax. |
| `discount_rate` | decimal(5,4) | No | - | Discount fraction; 0.05 means 5%. Weight by pre-discount value for aggregation. |
| `net_order_value_ngn` | decimal(18,2) | No | - | Ordered quantity times discounted selling price; demand, not recognized revenue. |

<a id="koraoperations-deliveries"></a>

## KoraOperations deliveries

One consignment; split orders have several. Unresolved shipments have no delivery date.

**Primary key:** `delivery_id`. **Rows:** 30,357.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `delivery_id` | int | No | PK | Consignment key; several can belong to one order. |
| `order_id` | int | No | FK to KoraOperations.orders.order_id | Customer order key; can have several lines and consignments. |
| `promised_date` | date | No | - | Promised delivery completion date; may follow the cutoff. |
| `delivery_date` | date | Yes | - | Actual completion date; NULL for unresolved consignments. |
| `status` | varchar(20) | No | - | Delivered or In transit at cutoff in this edition. Reconstruct historical state from dates. Observed values: Delivered, In transit. |
| `shipping_cost_ngn` | decimal(18,2) | No | - | Recorded consignment cost recognized at order_date in this simplified model. |
| `rework_hours` | decimal(10,2) | No | - | Recorded corrective work on a completed consignment; zero means none recorded. |
| `complaint_flag` | bit | No | - | 1 if a complaint was recorded, otherwise 0. |

<a id="koraoperations-delivery-lines"></a>

## KoraOperations delivery_lines

One delivered order line in a consignment; only completed deliveries have rows.

**Primary key:** `delivery_id, order_line_id`. **Rows:** 60,366.

**SQL checks:** `delivered_quantity > 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `delivery_id` | int | No | PK; FK to KoraOperations.deliveries.delivery_id | Consignment key; several can belong to one order. |
| `order_line_id` | int | No | PK; FK to KoraOperations.order_lines.order_line_id | One product line within a customer order. |
| `delivered_quantity` | int | No | - | Units of this order line completed in this consignment. |

<a id="koraoperations-sales-lines"></a>

## KoraOperations sales_lines

One invoiced delivered line; this teaching model invoices at completed delivery, excluding tax.

**Primary key:** `sales_line_id`. **Rows:** 60,366.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `sales_line_id` | int | No | PK | Invoiced delivered line key. |
| `delivery_id` | int | No | FK to KoraOperations.deliveries.delivery_id | Consignment key; several can belong to one order. |
| `order_line_id` | int | No | FK to KoraOperations.order_lines.order_line_id | One product line within a customer order. |
| `sale_date` | date | No | - | Revenue and invoice date, equal to delivery completion in this model. |
| `quantity` | int | No | - | Units invoiced on this delivered line. |
| `sales_value_ngn` | decimal(18,2) | No | - | Invoiced line value after discount, excluding tax and subsequent credits. |
| `cost_of_goods_ngn` | decimal(18,2) | No | - | Standard cost of invoiced units, before later restocking recoveries. |

<a id="koraoperations-returns"></a>

## KoraOperations returns

One full or partial return of an invoiced line; recovery only for restocked units.

**Primary key:** `return_id`. **Rows:** 2,310.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `return_id` | int | No | PK | Return event key. |
| `sales_line_id` | int | No | FK to KoraOperations.sales_lines.sales_line_id | Invoiced delivered line key. |
| `return_date` | date | No | - | Date return credit and any restocking recovery are recognized. |
| `quantity` | int | No | - | Units returned from the referenced sales line. |
| `refund_ngn` | decimal(18,2) | No | - | Recognized return credit excluding tax, on return_date. Legacy field name; actual cash repayments are in cash_refunds. Do not deduct both this and credit_notes from revenue. |
| `recovered_cost_ngn` | decimal(18,2) | No | - | Standard cost recovered for restocked returned units; zero otherwise. |
| `restocked` | bit | No | - | 1 units re-enter stock; 0 no cost recovery. |
| `return_reason` | varchar(40) | Yes | - | Recorded return explanation; NULL unrecorded. Does not independently establish fault. Observed values: Customer changed mind, Damaged, Wrong item. |

<a id="koraoperations-inventory-movements"></a>

## KoraOperations inventory_movements

One stock movement; opening balances on 1 January 2022 plus receipts sales returns and adjustments.

**Primary key:** `movement_id`. **Rows:** 71,208.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `movement_id` | int | No | PK | Stock event key; replay by movement_date then movement_id for within-day order. |
| `branch_id` | varchar(8) | No | FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `product_id` | varchar(12) | No | FK to KoraOperations.products.product_id | Product key; quantities across categories may use unlike units. |
| `movement_date` | date | No | - | Stock booking date. |
| `movement_type` | varchar(20) | No | - | Opening, Receipt, Sale or Return. Opening is initial stock, not purchases. Observed values: Opening, Receipt, Return, Sale. |
| `quantity_change` | int | No | - | Signed units; sales negative, opening, receipts and returns positive. |
| `sales_line_id` | int | Yes | FK to KoraOperations.sales_lines.sales_line_id | Invoiced delivered line key. |
| `return_id` | int | Yes | FK to KoraOperations.returns.return_id | Return event key. |

<a id="koraoperations-inventory-monthly-snapshot"></a>

## KoraOperations inventory_monthly_snapshot

One product at one branch at month end; balances are not additive across months.

**Primary key:** `snapshot_date, branch_id, product_id`. **Rows:** 10,368.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK; FK to KoraOperations.months.snapshot_date | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `branch_id` | varchar(8) | No | PK; FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `product_id` | varchar(12) | No | PK; FK to KoraOperations.products.product_id | Product key; quantities across categories may use unlike units. |
| `on_hand_quantity` | int | No | - | Units held at month end; do not add balances across months. |
| `stock_value_ngn` | decimal(18,2) | No | - | Month-end units times standard cost; not period cash investment. |

<a id="koraoperations-open-order-monthly-snapshot"></a>

## KoraOperations open_order_monthly_snapshot

One unfulfilled noncancelled order line at month end; completed lines are absent.

**Primary key:** `snapshot_date, order_line_id`. **Rows:** 12,520.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK; FK to KoraOperations.months.snapshot_date | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `order_line_id` | int | No | PK; FK to KoraOperations.order_lines.order_line_id | One product line within a customer order. |
| `open_quantity` | int | No | - | Units unfulfilled on a non-cancelled order at month end; no stock reservation implied. |
| `open_value_ngn` | decimal(18,2) | No | - | Outstanding units at discounted order price; not revenue or proven lost sales. |

<a id="koraoperations-branch-monthly-resources"></a>

## KoraOperations branch_monthly_resources

One branch month; period flows despite month-end date. Hours reconcile to non-manager HR hours; costs exclude goods shipping and refunds.

**Primary key:** `snapshot_date, branch_id`. **Rows:** 288.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK; FK to KoraOperations.months.snapshot_date | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `branch_id` | varchar(8) | No | PK; FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `available_service_hours` | decimal(12,2) | No | - | Non-manager scheduled minus absence plus overtime hours for the branch-month; indicative capacity, not actual productive utilization. |
| `fixed_operating_cost_ngn` | decimal(18,2) | No | - | Simplified core branch costs including staffing allowance; do not add employee annual salaries. |
| `overtime_cost_ngn` | decimal(18,2) | No | - | Recognized branch overtime expense; not individual payroll. |
| `service_budget_ngn` | decimal(18,2) | No | - | Legacy operational budget excluding goods and shipping; use monthly_budgets for financial comparisons. |

<a id="korapeople-grades"></a>

## KoraPeople grades

One internal responsibility band; promotions may retain the same job family title.

**Primary key:** `grade`. **Rows:** 6.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `grade` | int | No | PK | Internal responsibility band, defined in KoraPeople.grades. Promotions can retain job family titles. |
| `grade_label` | nvarchar(60) | No | - | Readable responsibility-band name. |
| `definition` | nvarchar(200) | No | - | Scope of the internal responsibility band. |

<a id="koraoperations-payment-terms"></a>

## KoraOperations payment_terms

One payment term; copied to each invoice at issue.

**Primary key:** `payment_term_id`. **Rows:** 3.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `payment_term_id` | varchar(10) | No | PK | NET0, NET30 or NET60 term code. |
| `term_days` | int | No | - | Calendar days from invoice issue to due date. |
| `description` | nvarchar(100) | No | - | Human-readable payment-term definition. |

<a id="koraoperations-invoices"></a>

## KoraOperations invoices

One invoice per completed delivery; amount equals its sales lines, excluding tax.

**Primary key:** `invoice_id`. **Rows:** 30,194.

**SQL checks:** `due_date >= invoice_date`; `invoice_amount_ngn > 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `invoice_id` | int | No | PK | Invoice key, one per completed delivery. Do not repeat header values over joined lines. |
| `delivery_id` | int | No | FK to KoraOperations.deliveries.delivery_id | Consignment key; several can belong to one order. |
| `customer_id` | varchar(12) | No | FK to KoraOperations.customers.customer_id | Stable customer key; some registered customers have no orders. |
| `branch_id` | varchar(8) | No | FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `invoice_date` | date | No | - | Issue and revenue recognition date. |
| `due_date` | date | No | - | Payment deadline; overdue begins the next day. |
| `payment_term_id` | varchar(10) | No | FK to KoraOperations.payment_terms.payment_term_id | NET0, NET30 or NET60 term code. |
| `invoice_amount_ngn` | decimal(18,2) | No | - | Original invoice amount excluding tax; equals its sales lines. |

<a id="koraoperations-credit-notes"></a>

## KoraOperations credit_notes

One recognized sales credit per return; a credit is not itself a cash refund.

**Primary key:** `credit_note_id`. **Rows:** 2,310.

**SQL checks:** `credit_amount_ngn > 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `credit_note_id` | int | No | PK | Sales credit key linked to a return and invoice. |
| `invoice_id` | int | No | FK to KoraOperations.invoices.invoice_id | Invoice key, one per completed delivery. Do not repeat header values over joined lines. |
| `return_id` | int | No | FK to KoraOperations.returns.return_id | Return event key. |
| `credit_date` | date | No | - | Date credit reduces net sales and invoice balance. |
| `credit_amount_ngn` | decimal(18,2) | No | - | Recognized sales credit; not automatically a cash repayment. |

<a id="koraoperations-cash-receipts"></a>

## KoraOperations cash_receipts

One customer cash receipt; all receipts fully allocated in this teaching extract.

**Primary key:** `receipt_id`. **Rows:** 29,019.

**SQL checks:** `amount_ngn > 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `receipt_id` | int | No | PK | Customer cash receipt key; may cover several invoices. |
| `customer_id` | varchar(12) | No | FK to KoraOperations.customers.customer_id | Stable customer key; some registered customers have no orders. |
| `receipt_date` | date | No | - | Date cash was received and allocated. |
| `amount_ngn` | decimal(18,2) | No | - | Customer cash received; not revenue. |

<a id="koraoperations-payment-allocations"></a>

## KoraOperations payment_allocations

One receipt allocation to an invoice; a receipt may settle several invoices.

**Primary key:** `allocation_id`. **Rows:** 29,622.

**SQL checks:** `allocated_amount_ngn > 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `allocation_id` | int | No | PK | Key of one receipt application to an invoice. |
| `receipt_id` | int | No | FK to KoraOperations.cash_receipts.receipt_id | Customer cash receipt key; may cover several invoices. |
| `invoice_id` | int | No | FK to KoraOperations.invoices.invoice_id | Invoice key, one per completed delivery. Do not repeat header values over joined lines. |
| `allocated_amount_ngn` | decimal(18,2) | No | - | Receipt amount applied to this invoice. |

<a id="koraoperations-cash-refunds"></a>

## KoraOperations cash_refunds

One cash repayment of a customer credit balance; decreases cash and increases net receivables.

**Primary key:** `cash_refund_id`. **Rows:** 437.

**SQL checks:** `amount_ngn > 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `cash_refund_id` | int | No | PK | Customer cash repayment key. |
| `credit_note_id` | int | No | FK to KoraOperations.credit_notes.credit_note_id | Sales credit key linked to a return and invoice. |
| `refund_date` | date | No | - | Date cash was repaid for a credit balance. |
| `amount_ngn` | decimal(18,2) | No | - | Cash repaid; not a second revenue credit. |

<a id="koraoperations-operating-expenses"></a>

## KoraOperations operating_expenses

One recognized branch expense component; fixed costs, overtime and shipping, excluding goods, tax, interest and capital expenditure.

**Primary key:** `expense_id`. **Rows:** 31,509.

**SQL checks:** `amount_ngn >= 0`.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `expense_id` | int | No | PK | Recognized expense component key. |
| `branch_id` | varchar(8) | No | FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `recognition_date` | date | No | - | Expense recognition date, not necessarily cash payment date. |
| `expense_category` | varchar(40) | No | - | Premises and facilities, Core staffing and administration, Utilities and services, Overtime or Shipping. Observed values: Core staffing and administration, Overtime, Premises and facilities, Shipping, Utilities and services. |
| `amount_ngn` | decimal(18,2) | No | - | Recognized expense component; do not add resource-table costs again. |
| `delivery_id` | int | Yes | FK to KoraOperations.deliveries.delivery_id | Consignment key; several can belong to one order. |

<a id="koraoperations-monthly-budgets"></a>

## KoraOperations monthly_budgets

One approved branch budget for a month; frozen planning assumptions, not fitted actuals.

**Primary key:** `snapshot_date, branch_id`. **Rows:** 288.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK; FK to KoraOperations.months.snapshot_date | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `branch_id` | varchar(8) | No | PK; FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `approved_date` | date | No | - | Budget approval date before the budget year; values remain frozen. |
| `budget_net_sales_ngn` | decimal(18,2) | No | - | Approved monthly revenue net of credits. |
| `budget_net_cogs_ngn` | decimal(18,2) | No | - | Approved monthly goods cost net of cost recoveries. |
| `budget_operating_expenses_ngn` | decimal(18,2) | No | - | Approved monthly branch costs including shipping; excludes goods, tax, interest and capital expenditure. |

<a id="koraoperations-receivables-monthly-snapshot"></a>

## KoraOperations receivables_monthly_snapshot

One nonzero invoice balance at month end; negative balances are customer credits, not overdue debt.

**Primary key:** `snapshot_date, invoice_id`. **Rows:** 120,995.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `snapshot_date` | date | No | PK; FK to KoraOperations.months.snapshot_date | Calendar month end. Snapshot tables are balances as of this date; hours, resources and budgets describe the whole month. |
| `invoice_id` | int | No | PK; FK to KoraOperations.invoices.invoice_id | Invoice key, one per completed delivery. Do not repeat header values over joined lines. |
| `balance_ngn` | decimal(18,2) | No | - | Invoice minus dated credits and allocations plus dated cash refunds. Negative means customer credit. |
| `days_past_due` | int | No | - | Maximum of zero and snapshot_date minus due_date. Only positive debit balances enter overdue debt buckets. |

<a id="koraoperations-suppliers"></a>

## KoraOperations suppliers

One fictional supplier with standard planning lead time.

**Primary key:** `supplier_id`. **Rows:** 6.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `supplier_id` | varchar(8) | No | PK | Fictional supplier key. |
| `supplier_name` | nvarchar(80) | No | - | Fictional supplier name. |
| `standard_lead_days` | int | No | - | Expected procurement lead time in calendar days; not observed performance. |

<a id="koraoperations-purchase-orders"></a>

## KoraOperations purchase_orders

One supplier purchase order; Standard or Emergency. Some remain outstanding at cutoff.

**Primary key:** `purchase_order_id`. **Rows:** 7,947.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `purchase_order_id` | int | No | PK | Supplier order key, distinct from customer order_id. |
| `supplier_id` | varchar(8) | No | FK to KoraOperations.suppliers.supplier_id | Fictional supplier key. |
| `branch_id` | varchar(8) | No | FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `ordered_date` | date | No | - | Date supplier order was placed. |
| `promised_receipt_date` | date | No | - | Supplier commitment date; may follow cutoff. |
| `order_type` | varchar(20) | No | - | Standard replenishment or Emergency same-day procurement; compare reliability separately. Observed values: Emergency, Standard. |

<a id="koraoperations-purchase-order-lines"></a>

## KoraOperations purchase_order_lines

One product line on a supplier order; current extract uses one line per purchase order.

**Primary key:** `purchase_order_line_id`. **Rows:** 7,947.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `purchase_order_line_id` | int | No | PK | Supplier order product line key. |
| `purchase_order_id` | int | No | FK to KoraOperations.purchase_orders.purchase_order_id | Supplier order key, distinct from customer order_id. |
| `product_id` | varchar(12) | No | FK to KoraOperations.products.product_id | Product key; quantities across categories may use unlike units. |
| `ordered_quantity` | int | No | - | Units requested from supplier. |
| `unit_cost_ngn` | decimal(18,2) | No | - | Standard purchase unit cost; emergency premiums and procurement freight are not modelled. |

<a id="koraoperations-goods-receipts"></a>

## KoraOperations goods_receipts

One receipt against a purchase line; partial receipts are possible.

**Primary key:** `goods_receipt_id`. **Rows:** 8,938.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `goods_receipt_id` | int | No | PK | Supplier partial or full receipt key. |
| `purchase_order_line_id` | int | No | FK to KoraOperations.purchase_order_lines.purchase_order_line_id | Supplier order product line key. |
| `movement_id` | int | No | FK to KoraOperations.inventory_movements.movement_id | Stock event key; replay by movement_date then movement_id for within-day order. |
| `received_date` | date | No | - | Date received units enter stock. |
| `received_quantity` | int | No | - | Units accepted in this receipt; sum partial receipts against ordered units. |

<a id="koraoperations-stock-shortage-events"></a>

## KoraOperations stock_shortage_events

One failed stock check immediately before fulfilment; same-day emergency procurement supplies the shortage. Not a duration measure or proof of the cause of delivery lateness.

**Primary key:** `shortage_id`. **Rows:** 3,816.

| Field | SQL type | Nullable | Key or connection | Definition |
| --- | --- | --- | --- | --- |
| `shortage_id` | int | No | PK | Failed pre-fulfilment stock check key. |
| `sales_line_id` | int | No | FK to KoraOperations.sales_lines.sales_line_id | Invoiced delivered line key. |
| `branch_id` | varchar(8) | No | FK to KoraOperations.branches.branch_id | Distribution centre key; shared consistently across both databases. |
| `product_id` | varchar(12) | No | FK to KoraOperations.products.product_id | Product key; quantities across categories may use unlike units. |
| `event_date` | date | No | - | Shortage date immediately before same-day emergency receipt and stock issue. |
| `required_quantity` | int | No | - | Units required for this sales-line fulfilment attempt. |
| `available_quantity` | int | No | - | On-hand units immediately before shortage check. |
| `shortage_quantity` | int | No | - | Required minus available units, strictly positive. |
| `emergency_purchase_order_id` | int | No | FK to KoraOperations.purchase_orders.purchase_order_id | Same-day emergency order raised to cover shortage. |
