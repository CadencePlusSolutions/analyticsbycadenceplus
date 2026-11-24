
<!-- page break -->

# Kora Database Setup Guide

## What you are installing

This four-year synthetic extract covers 1 January 2022 through 31 December 2025: 40 tables and 630,743 rows shared across the financial, people and operations projects. Choose SQL Server, PostgreSQL or MySQL. Each engine has separate KoraOperations and KoraPeople databases.

Start from an empty database pair. Do not append to Kora Starter. The source CSVs in `Data/` are the canonical rows for every engine.

## Database setup

- **SQL Server:** `Database/SQL_Server/` — use SQL Server Management Studio.
- **PostgreSQL:** `Database/PostgreSQL/` — use `psql` from the extracted `Kora-Levels-2-4` folder so the `Data/` paths resolve.
- **MySQL 8.0.16 or later:** `Database/MySQL/` — use the MySQL client.

The SQL files in each folder are numbered in run order. Each file identifies its target database in the opening comment. Use the `03_Check_Load.sql` file in the same folder to validate the load.
