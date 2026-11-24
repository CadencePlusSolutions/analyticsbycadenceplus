
<!-- page break -->

# Kora Database Setup Guide

## What you are installing

This four-year synthetic extract covers 1 January 2022 through 31 December 2025: 40 tables and 630,743 rows shared across the financial, people and operations projects. Choose SQL Server, PostgreSQL or MySQL. Each engine has separate KoraOperations and KoraPeople databases.

Start from an empty database pair. Do not append to Kora Starter. The source CSVs in `Data/` are the canonical rows for every engine.

## Database folders

- **SQL Server:** `Database/SQL_Server/` (use SQL Server Management Studio)
- **PostgreSQL:** `Database/PostgreSQL/` (use `psql`)
- **MySQL 8.0.16 or later:** `Database/MySQL/` (use the `mysql` client)

Each folder has numbered SQL files for creating, loading and checking the databases. The opening comments in each file give the exact command and target database. For PostgreSQL, open the terminal in the extracted `Kora-Levels-2-4` folder so the `Data/` paths resolve. SQL Server's optional `04_Reporting_Indexes.sql` can be run after the load passes its checks.

<!-- page break -->

# Use the data in your reporting tool

## Connections and naming

Connect to KoraOperations for financial and operations cases and KoraPeople for people cases. SQL Server and PostgreSQL use the kora schema. MySQL database names provide the namespace. The two databases share branch IDs logically, without a cross-database foreign key.

CSV import is also possible: preserve text IDs, parse ISO dates, use decimal NGN amounts and retain blank nullable values. Do not append CSVs after using a supplied loader.

## Reusable SQL by engine

The project guides show SQL Server signatures to make parameters and expected output concrete. Implement equivalent behavior in your chosen engine: stored procedures in SQL Server and MySQL or parameterized functions in PostgreSQL. Explain any limitations of the chosen engine in your methodology.

## Dates and joins

Employment and assignment starts are inclusive; ends are exclusive. NULL end dates mean current through 31 December 2025. Use that cutoff rather than today's date. Assignment IDs are not chronological. promotion_candidates.assignment_id_at_assessment is the role at assessment; the round supplies the decision and effective dates.

Use events for activity within a period and snapshots for month-end states. Avoid summing month-end stock, staff or receivables across months. Aggregate independent child tables before joining to avoid multiplying amounts.


<!-- page break -->

# Reporting conventions and limits

## Financial conventions

Revenue is invoiced at completed delivery. Return credits reduce revenue on their own dates. Cash receipts and refunds have their own dates. Shipping is recognized at order date in this simplified model.

Recorded operating costs contain a simplified staffing allowance. Do not add salaries, resource summaries and expense ledger costs together. Credit notes duplicate return credits; count the credit once. These are not complete company accounts.

## Operational conventions

Inventory uses fixed standard costs. Standard procurement has dated promises and partial receipts. Shortages are covered by same-day emergency procurement; no shortage duration, emergency premium or lost-sales amount is supplied. Capacity and stock transfer results are scenarios based on stated assumptions, not causal forecasts.

## People conventions

Grade is a responsibility band within a job family. Promotion selection data covers eligible considered candidates, not everyone who might have sought progression. No ethnicity or self-identification fields are included in this edition. Report missing exit explanations and assessments as missing.

## Reuse and credit

You may analyze, modify and publish SQL, dashboards and findings with credit to CadencePlus Solutions. Keep the fictional and synthetic labels. Do not present values as real employee, customer or market records. Third-party software and fonts retain their own licenses.
