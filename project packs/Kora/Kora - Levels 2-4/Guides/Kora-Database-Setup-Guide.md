
<!-- page break -->

# Kora Database Setup Guide

## What you are installing

This four-year synthetic extract covers 1 January 2022 through 31 December 2025: 40 tables and 630,743 rows shared across the financial, people and operations projects. Choose SQL Server, PostgreSQL or MySQL. Each engine has separate KoraOperations and KoraPeople databases.

Start from an empty database pair. Do not append to Kora Starter. The source CSVs in `Data/` are the canonical rows for every engine.

## Choose an engine

- **SQL Server:** `Database/SQL_Server/` (use SQL Server Management Studio)
- **PostgreSQL:** `Database/PostgreSQL/` (use `psql`)
- **MySQL 8.0.16 or later:** `Database/MySQL/` (use the `mysql` client)

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

# MySQL

Use MySQL 8.0.16 or later. Open a terminal in the extracted `Kora-Levels-2-4` folder. Run these commands in order:

1. `mysql --default-character-set=utf8mb4 < Database/MySQL/00_Create_Databases.sql`
2. `mysql --default-character-set=utf8mb4 KoraOperations < Database/MySQL/01_Load_Operations.sql`
3. `mysql --default-character-set=utf8mb4 KoraPeople < Database/MySQL/02_Load_People.sql`
4. `mysql --default-character-set=utf8mb4 < Database/MySQL/03_Check_Load.sql`

Every table must show `PASS`; a SQL error or `FAIL` invalidates the load. The scripts do not replace existing tables. MySQL does not offer a single atomic transaction over DDL and inserts; after a failed load, create a fresh database pair before retrying.
