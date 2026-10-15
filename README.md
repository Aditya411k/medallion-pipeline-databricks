# End-to-End Batch Data Pipeline on Databricks

A Medallion (Bronze → Silver → Gold) batch pipeline built with PySpark and Delta Lake on Databricks, using the Berka banking dataset.

## Dataset
Berka (Czech bank) dataset: 8 related tables (account, client, disp, district, card, loan, order, trans), about 1.05 million transaction rows. Source: https://www.kaggle.com/datasets/marceloventura/the-berka-dataset (the data is not included in this repo).

## Architecture
Raw CSV (Unity Catalog Volume) → Bronze (Delta) → Silver (Delta) → Gold (Delta tables)

| Layer | What it does |
|---|---|
| Bronze | Ingests raw CSVs as-is (all columns as strings) and adds audit columns (ingestion timestamp, source file) |
| Silver | Cleans blanks to nulls, casts data types, parses YYMMDD dates, decodes birth_number into birth date, translates Czech codes to English, renames cryptic district columns, key-based deduplication |
| Gold | Business-ready aggregates for analytics |

## Gold tables
- monthly_cashflow: monthly credits, debits, net flow and transaction counts
- account_summary: per-account totals and average balance, joined with district and region
- loan_summary_by_region: loan count and amounts by region and loan status

Gold tables are registered in Unity Catalog (workspace.berka_gold) and queried with SQL.

## Tech stack
Databricks (Free Edition), PySpark, Delta Lake, SQL, Unity Catalog

## Notes
- Source data had no fully duplicate rows; key-based deduplication is kept as a safeguard for future batches.
- Missing partner bank/account values in transactions are genuine (cash transactions) and were kept as nulls.

## How to run
Upload the 8 CSV files to a Unity Catalog Volume, then run the notebook cells in order.
