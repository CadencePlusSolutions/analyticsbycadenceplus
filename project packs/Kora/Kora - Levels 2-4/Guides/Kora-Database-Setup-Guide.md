
<!-- page break -->

# Kora Database Setup Guide

## What you are installing

This four-year synthetic extract covers 1 January 2022 through 31 December 2025: 40 tables and 630,743 rows shared across the financial, people and operations projects. Choose SQL Server, PostgreSQL or MySQL. Each engine has separate KoraOperations and KoraPeople databases.

Start from an empty database pair. Do not append to Kora Starter. The source CSVs in `Data/` are the canonical rows for every engine.

## SQL Server — `Database/SQL_Server/`

Open these files in SQL Server Management Studio (SSMS) in order. SSMS handles the `GO` lines in the scripts.

1. `00_Create_Databases.sql` — create the two databases.
2. `01_Load_Operations.sql` — load KoraOperations.
3. `02_Load_People.sql` — load KoraPeople.
4. `03_Check_Load.sql` — validate the load.
5. `04_Reporting_Indexes.sql` — optional, after validation passes.

## PostgreSQL — `Database/PostgreSQL/`

Use `psql` from the extracted `Kora-Levels-2-4` folder so the `Data/` paths resolve. The first comment in each file identifies the target database.

1. `00_Create_Databases.sql` — create the two databases.
2. `01_Load_Operations.sql` — load KoraOperations.
3. `02_Load_People.sql` — load KoraPeople.
4. `03_Check_Load.sql` — validate the load.

## MySQL — `Database/MySQL/`

Use the MySQL client. The first comment in each file identifies the target database.

1. `00_Create_Databases.sql` — create the two databases.
2. `01_Load_Operations.sql` — load KoraOperations.
3. `02_Load_People.sql` — load KoraPeople.
4. `03_Check_Load.sql` — validate the load.
