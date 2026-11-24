
<!-- page break -->

# Kora Database Setup Guide

## What you are installing

This four-year synthetic extract covers 1 January 2022 through 31 December 2025: 40 tables and 630,743 rows shared across the financial, people and operations projects. Choose SQL Server, PostgreSQL, MySQL or SQLite. Each engine has separate KoraOperations and KoraPeople databases; SQLite represents them as two files.

Start from an empty database pair. Do not append to Kora Starter. The source CSVs in Data are the canonical rows for every engine. Reference/Load_Control_Totals.json lists row counts, hashes and reconciliation controls.

## Choose an engine

- **SQL Server:** Open the scripts in `Database/SQL_Server` with SQL Server Management Studio (SSMS).
- **PostgreSQL:** Use `psql` for the scripts in `Database/PostgreSQL`. Open a terminal in the extracted `Kora-Levels-2-4` folder (the folder containing `Data` and `Database`) so the CSV paths resolve.
- **MySQL 8.0.16 or later:** Use the `mysql` command line for `Database/MySQL`. The generated INSERT scripts load the CSV rows without server file access.
- **SQLite:** Use Python 3 to build two database files from the supplied CSVs. SQLite supports views and saved parameterized queries but not stored procedures.

## Validation status

Run the check script for your chosen engine after loading. It checks row counts and other data controls.


<!-- page break -->

# SQL Server and PostgreSQL

## SQL Server

Open each script in SSMS and select **Execute**, in this order:

1. `Database/SQL_Server/00_Create_Databases.sql`
2. `Database/SQL_Server/01_Load_Operations.sql`
3. `Database/SQL_Server/02_Load_People.sql`
4. `Database/SQL_Server/03_Check_Load.sql`
5. After a successful check, optionally run `Database/SQL_Server/04_Reporting_Indexes.sql`.

Keep the `GO` lines in the files. SSMS uses them to separate batches; `GO` is a client command, not SQL sent directly to the server. Each loader uses a transaction and refuses existing `kora` tables.

## PostgreSQL

Open a terminal in the extracted `Kora-Levels-2-4` folder. Run these commands in order:

1. `psql -X -v ON_ERROR_STOP=1 -d postgres -f Database/PostgreSQL/00_Create_Databases.sql`
2. `psql -X -v ON_ERROR_STOP=1 -d KoraOperations -f Database/PostgreSQL/01_Load_Operations.sql`
3. `psql -X -v ON_ERROR_STOP=1 -d KoraPeople -f Database/PostgreSQL/02_Load_People.sql`
4. `psql -X -v ON_ERROR_STOP=1 -d postgres -f Database/PostgreSQL/03_Check_Load.sql`

The client-side `\copy` commands read `Data/` from the folder where you ran `psql`. Stop at any error and inspect the transaction before retrying.


<!-- page break -->

# MySQL and SQLite

## MySQL

Use MySQL 8.0.16 or later. Open a terminal in the extracted `Kora-Levels-2-4` folder. Run these commands in order:

1. `mysql --default-character-set=utf8mb4 < Database/MySQL/00_Create_Databases.sql`
2. `mysql --default-character-set=utf8mb4 KoraOperations < Database/MySQL/01_Load_Operations.sql`
3. `mysql --default-character-set=utf8mb4 KoraPeople < Database/MySQL/02_Load_People.sql`
4. `mysql --default-character-set=utf8mb4 < Database/MySQL/03_Check_Load.sql`

Every table must show `PASS`; a SQL error or `FAIL` invalidates the load. The scripts do not replace existing tables. MySQL does not offer a single atomic transaction over DDL and inserts; after a failed load, create a fresh database pair before retrying.

## SQLite

Open a terminal in the extracted `Kora-Levels-2-4` folder. Run these commands in order:

1. `python3 Database/SQLite/00_Build_Databases.py`
2. `python3 Database/SQLite/03_Check_Load.py`

The builder creates `Database/SQLite/KoraOperations.sqlite` and `Database/SQLite/KoraPeople.sqlite` from the CSVs. It refuses to overwrite an existing database file and checks every table and relationship before renaming its temporary output.


<!-- page break -->

# Use the data in your reporting tool

## Connections and naming

Connect to KoraOperations for financial and operations cases and KoraPeople for people cases. SQL Server and PostgreSQL use the kora schema. MySQL database names provide the namespace; SQLite uses unqualified tables within each file. The two databases share branch IDs logically, without a cross-database foreign key.

CSV import is also possible: preserve text IDs, parse ISO dates, use decimal NGN amounts and retain blank nullable values. Do not append CSVs after using a supplied loader.

## Reusable SQL by engine

The project guides show SQL Server signatures to make parameters and expected output concrete. Implement equivalent behavior in your chosen engine: stored procedures in SQL Server and MySQL, parameterized functions in PostgreSQL, or a reusable view with a saved parameterized query in SQLite. Explain any limitations of the chosen engine in your methodology.

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
