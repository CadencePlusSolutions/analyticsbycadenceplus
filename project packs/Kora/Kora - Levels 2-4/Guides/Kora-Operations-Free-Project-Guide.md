
<!-- page break -->

# Kora Operations - Free Project Guide

Use the SQL objects required for the chosen case. SQL Server and MySQL support stored procedures; PostgreSQL can use a parameterized function. SQL Server and PostgreSQL use the kora schema; MySQL uses tables in the selected database without that prefix.

## Business problem A

## The decision

Better branch service with the same budget

The Operations Manager wants a branch improvement plan using existing resources. Diagnose service pressure, then compare feasible reallocation or process-change scenarios without assuming that a better load ratio guarantees faster deliveries.

## Your data

KoraOperations: orders, order_lines, deliveries, delivery_lines, branch_monthly_resources, open_order_monthly_snapshot, branches and months. Individual HR records are not required for this operations case.

## Possible questions

1. Compare demand, indicative capacity, delivery reliability, rework, complaints and outstanding demand across branches and aligned periods.

2. Which patterns remain after comparing Standard and Complex work? Distinguish order counts, consignments and estimated hours.

3. Quantify two plausible bottlenecks. Test competing explanations such as growth, complexity, rework and stock shortages before assigning blame.

4. Design at least two scenarios using existing hours or reducing avoidable work. Show the donor impact, recipient impact, feasible work types and implementation assumptions.

5. Stress-test each proposal under less favourable demand or productivity assumptions. Explain what happens if hours cannot be transferred as assumed.

6. Recommend a preferred action and an alternative, with cost constraints, service safeguards and a pilot review trigger.

## Think It Through

Challenge reasoning: a branch can look more efficient because its work is easier. Act & explain: make the donor cost and service risk explicit, and distinguish a scenario from an observed improvement.


<!-- page break -->

## Main measures

On-time rate = consignments with delivery_date on or before promised_date divided by all consignments completed in the period. Report consignments still In transit separately; they are absent from this completed-only rate.

Incoming load ratio = estimated standard_work_hours for orders received in the month divided by available_service_hours for that branch-month. It includes orders later cancelled and is an estimate of pressure, not actual utilization.

Available hours = non-manager scheduled minus absence plus overtime hours, aggregated to branch-month. Rework is an additional burden indicator; do not blindly add it to a workload estimate already assumed to include rework.

Complaint share = completed consignments with complaint_flag = 1 divided by all completed consignments in the same period. Compare sample sizes and work complexity.

Open demand is a month-end balance. Total scenario hours must equal baseline hours across all branches. Budget neutrality also requires checking costs, feasibility and donor service effects.

## Required SQL objects

Create kora.vw_BranchService with one branch-month row: incoming orders/hours, available hours, completed and on-time consignments, rework, complaints and load ratio. Keep event-date populations explicit.

Create kora.usp_CapacityScenario(@MonthEnd, @DonorBranch, @ReceivingBranch, @Hours). Return baseline and scenario hours and load ratios for all six branches. Preserve total hours; reject invalid months, equal/unknown branches, non-positive hours and an overdrawn donor. Do not modify source data.


<!-- page break -->

## Business problem B

## The decision

Excess stock and shortages at the same time

The Supply Chain Manager wants to improve stock availability without increasing total inventory investment. Assess purchasing, supplier delivery and branch stock allocation, then compare a transfer with a replenishment-policy change.

## Your data

KoraOperations: inventory_movements, inventory_monthly_snapshot, products, purchase_orders, purchase_order_lines, suppliers, goods_receipts, stock_shortage_events, sales_lines, open_order_monthly_snapshot, orders and order_lines.

## Possible questions

1. Reconcile stock movements and month-end balances by branch-product. Locate persistent excess holdings relative to recent issues, and distinguish slow movement from zero demand.

2. Analyze recorded shortage events, emergency purchases and outstanding demand. Explain what these reveal and why month-end stock alone cannot prove availability throughout a month.

3. Compare Standard supplier orders by promised dates, partial receipts, completion dates and outstanding units. Keep not-yet-due and overdue open orders separate.

4. Test whether shortages align with replenishment timing, supplier delays or branch allocation. Do not assume a shortage event proves the cause of a delivery delay.

5. Compare an inventory transfer with revised replenishment timing or quantity. Quantify donor safety stock, recipient need, estimated cost and assumptions about transit time.

6. Recommend a preferred change and an alternative. Show how existing budget offsets cover estimated transfer costs, and define stock and service measures for a pilot.

## Think It Through

Gather & validate data: use daily events to explain monthly balances. Challenge reasoning: could the transfer merely move the shortage? Act & explain: balance availability, cost and donor risk.


<!-- page break -->

## Main measures

Closing stock = opening stock + receipts + restocked returns - sales issues. The Opening movements on 1 January 2022 establish the starting balance; use signed inventory movements to reconcile each month end.

Use sales issue units for the same product over a stated lookback period as a demand proxy. Days of cover = on-hand units divided by average daily issues in that period; report it as undefined when there were no issues.

Supplier outstanding units at a reporting date = ordered units minus units received by that date. The complete receipt date is the first date cumulative receipts reach the ordered quantity.

For Standard purchase-order lines due by the analysis date, on-time completion rate = lines fully received by their promised date divided by all lines due by then, including overdue open lines. Show a separate completed-only rate and explain the different denominators.

Stock-shortage events record failed pre-fulfilment checks, covered by same-day emergency procurement. They do not measure shortage duration or lost sales. Emergency premiums and transfer freight are not recorded; scenario costs are assumptions.

## Required SQL objects

Create kora.vw_PurchaseLinePerformance at purchase-order-line grain, with supplier, branch, order type, promise, ordered/received/outstanding units, completion date and on-time completion flag as of cutoff. Retain incomplete orders.

Create kora.usp_StockTransferScenario(@MonthEnd, @ProductID, @DonorBranch, @ReceivingBranch, @Quantity, @DonorSafetyUnits, @CostPerUnit, @AvailableBudgetOffset). Return before/after stock and estimated cost. Reject unknown combinations, invalid dates, non-positive transfers, donor safety-stock breaches and costs exceeding the stated offset.


<!-- page break -->

# Submission

## Expected deliverables

Pick one business problem. Submit one reproducible SQL file containing analysis queries, the reusable objects required for your chosen problem, readable aliases, explanatory comments and executable validation tests.

One report page in a BI tool of your choice, with clear labels. Supply the editable report and a PDF or image export. *EVERY VISUAL SHOULD HAVE A JOB.*

One recommendation page with at least two actions, supporting numbers, reasoning, limitations, responsible stakeholders and proposed follow-up measures.

One methodology page organized around the Think It Through Framework. Explain any definitions you developed for metrics beyond those provided in the project guide, your quality checks, assumptions and limitations.
