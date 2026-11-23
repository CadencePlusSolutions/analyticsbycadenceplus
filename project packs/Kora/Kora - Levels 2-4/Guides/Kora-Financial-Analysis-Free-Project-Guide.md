
<!-- page break -->

# Kora Financial Analysis - Free Project Guide

Use the SQL objects required for the chosen case. SQL Server and MySQL support stored procedures; PostgreSQL can use a parameterized function. SQLite has no stored procedures: submit a reusable view and a saved parameterized query with equivalent output. SQL Server and PostgreSQL use the kora schema; MySQL and SQLite use tables in the selected database without that prefix.

## Business problem A

## The decision

Sales are growing, but is profit keeping up?

The Finance Manager must decide where to protect branch profitability in the next quarter.

You are required to assess whether sales growth is translating into gross profit and operating contribution, then recommend two financially justified actions.

## Your data

KoraOperations: sales_lines, returns, order_lines, orders, products, customers, operating_expenses, monthly_budgets, months and branches. Invoices provide a reconciliation to recognized sales.

## Possible questions

1. Build monthly and annual revenue, gross-profit and operating-contribution trends for all 48 months. Compare the same months across years, then calculate actual-versus-budget variances for net sales, goods costs, operating expenses and operating contribution.

2. What drives the changes? Distinguish total contribution from margin rates. Keep branch overhead separate unless you justify an allocation.

3. Quantify discounts and returns. Does a branch-level result persist within comparable categories and customer segments?

4. Which expense components explain budget variances? Check that shipping and resource-table costs are not counted twice.

5. Test an alternative explanation for the strongest finding. Separate what the data quantifies from what it cannot establish.

6. Recommend at least two actions, with the value at stake, a responsible stakeholder, assumptions and review measures.

## Think It Through

Challenge reasoning: does the profitability story survive a change in category mix? Act & explain: quantify a defensible action rather than simply asking finance to investigate the same data.


<!-- page break -->

## Main measures

Net sales for a period = sales_lines.sales_value_ngn on sale_date minus return credits on return_date. returns.refund_ngn and credit_notes.credit_amount_ngn record the same credit; subtract it only once.

Net goods cost for a period = sales_lines.cost_of_goods_ngn on sale_date minus returns.recovered_cost_ngn on return_date. Gross profit = net sales minus net goods cost.

Gross margin = total gross profit divided by total net sales for the same population and period. Return NULL when net sales are zero; do not average individual margin percentages.

Operating contribution = gross profit minus recorded operating_expenses. This is not audited company net profit; tax, interest, depreciation and complete corporate accounts are absent.

Budget variance = actual amount minus budget amount for the same branch and month. A positive sales variance differs from a positive cost variance. Budget contribution = budget net sales minus budget goods cost minus budget operating expenses.

## Required SQL objects

Create kora.vw_MonthlyFinancial: one row per branch-month, including zero activity. Return invoiced sales, credits, net sales, net goods cost, gross profit, operating expenses, contribution, budgets and variances.

Create kora.usp_FinancialSummary(@StartDate, @EndDate, @BranchID = NULL). Return monthly financial measures and gross margin, ordered by month and branch. Require complete months within 2022-2025; reject missing/reversed dates and unknown branches.


<!-- page break -->

## Business problem B

## The decision

We made the sales. Where is the cash?

The Credit Control Manager needs a collection priority list and a review of payment terms. Identify the customers and invoices driving overdue balances and recommend targeted actions without treating every unpaid invoice as overdue.

## Your data

KoraOperations: invoices, payment_terms, credit_notes, cash_receipts, payment_allocations, cash_refunds, receivables_monthly_snapshot, customers, branches and months.

## Possible questions

1. Reconstruct balances at selected historical month ends and compare with supplied receivables snapshots. Keep not-yet-due balances and customer credits visible.

2. How do overdue amount, overdue share and aging change over four years? Separate growth in sales from deterioration in payment behaviour.

3. Which customers account for the greatest overdue exposure? Compare amounts, age, payment history and concentration; do not choose an arbitrary top five.

4. How do partial payments, contractual terms and later return credits change the interpretation? Use only events known through the reporting date.

5. Compare customers with similar terms. Distinguish invoices still awaiting payment from fully observed payment histories.

6. Recommend at least two collection or terms-review priorities. Quantify exposure and propose measures to review progress.

## Think It Through

Gather & validate data: prove your date filters prevent future payments from rewriting earlier balances. Challenge reasoning: does a large unpaid balance reflect delinquency or simply longer agreed terms?


<!-- page break -->

## Main measures

For each invoice, calculate its balance at the reporting date: original amount minus credits issued and payments allocated by that date, plus any related cash refunds paid by that date. A negative balance is a customer credit.

Add positive invoice balances to report receivables. Show negative invoice balances separately as amounts owed to customers; do not offset them against overdue positive balances.

For a positive invoice balance, days past due is zero through its due date; after that, count the days from due_date to reporting date. Use not overdue, 1-30, 31-60, 61-90 and over 90 days as buckets.

Overdue share = positive overdue balances / all positive balances at the same date. Receivables are balances and must not be summed across months.

Customer net cash for a period = receipts on receipt_date minus refunds on refund_date. This is not company cash flow: supplier and staff payments are absent. The extract has no opening receivables from before 2022.

## Required SQL objects

Create kora.usp_ReceivablesAging(@AsOfDate, @BranchID = NULL). Return invoice, customer, branch, invoice and due dates, balance, days past due and aging bucket. Support any date within 2022-2025, including non-month-end dates.

Exclude settled invoices, preserve customer credit rows and reject NULL/out-of-window dates or unknown branches. Aggregate allocations, credits and cash refunds separately before joining invoices.


<!-- page break -->

# Submission

## Expected deliverables

Pick one business problem. Submit one reproducible SQL file containing analysis queries, the reusable objects required for your chosen problem, readable aliases, explanatory comments and executable validation tests.

One report page in a BI tool of your choice, with clear labels. Supply the editable report and a PDF or image export. *EVERY VISUAL SHOULD HAVE A JOB.*

One recommendation page with at least two actions, supporting numbers, reasoning, limitations, responsible stakeholders and proposed follow-up measures.

One methodology page organized around the Think It Through Framework. Explain any definitions you developed for metrics beyond those provided in the project guide, your quality checks, assumptions and limitations.
