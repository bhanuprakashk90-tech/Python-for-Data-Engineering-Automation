# Web Scraper & Automated SQL Ingestion Pipeline

A lightweight, end-to-end Python pipeline that scrapes multi-page web data, cleans it up, and loads it into a SQL database. It uses batch upserts to handle duplicate records safely and runs on a schedule with built-in logging and alert notifications.

I built this project across three main steps:
1. Scraping data safely from paginated pages.
2. Building an idempotent database loader (preventing duplicate entries).
3. Wiring everything together into an automated scheduled job.

---

## What It Does

```text
[ Web Pages ] ──> [ BeautifulSoup Scraper ] ──> [ Data Cleaner ] ──> [ SQL Database (Upsert) ]
                                                                             │
                                                                             ▼
                                                                  [ Logging & Webhook Alert ]

1. Web Extraction (requests + BeautifulSoup4)
Crawls paginated listings while handling basic anti-scraping checks (rotating User-Agent headers and adding request delays).

Parses raw HTML tags to extract fields like titles, authors/prices, and metadata tags.

Exports a standalone copy of the dataset as quotes_100_records.csv and quotes_100_records.json (100+ items minimum).

2. Database Loading & Upsert Engine (SQLite / SQLAlchemy)
Auto-creates the database schema on launch if it doesn't already exist.

Enforces a UNIQUE constraint on natural keys (the quote text).

Uses INSERT ... ON CONFLICT DO UPDATE (upsert logic). Re-running the pipeline updates existing rows instead of crashing or inserting duplicate entries.

Employs atomic transactions (commit and rollback) to protect data integrity if a network error occurs mid-load.

Pipeline Scheduling & Alerts (logging + schedule)
Wraps Extract, Transform, and Load operations inside an object-oriented Python runner (ETLPipeline).

Writes operational logs (timestamps, success metrics, error stack traces) to both stdout and pipeline.log.

Dispatches execution summaries via Webhook (Discord/Slack) or console alerts.

Configured to run on a timer loop or as a background Linux cron job.


Database SchemaTable Name: target_recordsColumnTypeConstraintsNotesidINTEGERPRIMARY KEY AUTOINCREMENTAuto-generated row IDquoteTEXTNOT NULL UNIQUETargeted for conflict checkingauthorTEXTNOT NULLItem author or entitytagsTEXTNULLABLEComma-separated categoriesingested_atTIMESTAMPDEFAULT CURRENT_TIMESTAMPLast updated timestamp

Project Structure
├── etl_pipeline.py          # Main ETL script with scheduler and logging
├── quotes_100_records.csv   # Scraped CSV dataset output
├── quotes_100_records.json  # Scraped JSON dataset output
├── etl_production.db        # SQLite database file
├── pipeline.log             # Execution and error logs
└── README.md                # Project documentation

How to Run It
1. Install Dependencies
pip install requests beautifulsoup4 pandas sqlalchemy schedule

2. Run the Script Directly
python etl_pipeline.py

3. Set Up as a Daily Cron Job (Linux)
To run the job automatically every night at 2:00 AM:
crontab -e

0 2 * * * /usr/bin/python3 /path/to/etl_pipeline.py >> /path/to/cron_output.log 2>&1
