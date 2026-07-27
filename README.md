# 💳 Credit Card Fraud Detection & Transaction Analytics (SQL + Python)

A portfolio project that detects fraudulent credit card transactions and analyzes spending behavior using **MySQL** for all analytics/fraud logic and **Python** strictly for data cleaning and loading.

## 🛠️ Tech Stack
- **Database:** MySQL 8+ / MariaDB 10.2+
- **Language:** Python 3 (`pandas`, `sqlalchemy`, `pymysql`) — cleaning & loading only
- **Analysis & Fraud Detection:** 100% SQL (CTEs, window functions, rule-based scoring)

## 📂 Project Structure
```
credit-card-fraud-detection-analytics/
├── data_raw/                    # messy raw CSVs (simulates real bank export data)
├── scripts/
│   ├── generate_raw_data.py     # STEP 2: generate messy raw data
│   └── clean_and_load.py        # STEP 4: clean data + load into MySQL
├── sql/
│   ├── schema.sql                # STEP 3: MySQL table structure
│   └── analysis_and_fraud.sql    # STEP 5: analytics + fraud detection (pure SQL)
├── requirements.txt
└── README.md
```

---

## 🧭 Build It Step-By-Step

### STEP 1 — Set up MySQL
Install MySQL/MariaDB if you don't have it, then create the database and a dedicated user:
```sql
CREATE DATABASE IF NOT EXISTS fintech_db;
CREATE USER IF NOT EXISTS 'fintech_user'@'localhost' IDENTIFIED BY 'fintech_pass123';
GRANT ALL PRIVILEGES ON fintech_db.* TO 'fintech_user'@'localhost';
FLUSH PRIVILEGES;
```
Also install Python dependencies:
```bash
pip install -r requirements.txt
```

### STEP 2 — Generate realistic (and messy) raw transaction data
```bash
cd scripts
python generate_raw_data.py
```
This creates `data_raw/customers_raw.csv`, `merchants_raw.csv`, and `transactions_raw.csv` — deliberately messy, like a real bank export: duplicate rows, missing values, inconsistent text casing, mixed date formats, currency symbols in amounts.

**Why this step matters for your portfolio:** it proves you understand that real data is never clean — and shows you can simulate that realistically.

### STEP 3 — Design the database schema
`sql/schema.sql` defines 3 tables: `customers`, `merchants`, `transactions` — with primary keys, foreign keys, and indexes on `txn_datetime` and `category` for query performance. This runs automatically in Step 4, or run it manually:
```bash
mysql -u fintech_user -p fintech_db < ../sql/schema.sql
```

### STEP 4 — Clean the data and load it into MySQL (Python's only job)
```bash
python clean_and_load.py
```
This script:
1. Strips whitespace, standardizes text casing (`"  DELHI"` → `"Delhi"`)
2. Removes duplicate rows
3. Parses multiple inconsistent date formats into one standard format
4. Cleans the amount field (removes `₹`, commas; drops negative/invalid values)
5. Fills missing numeric fields (age, credit score) with the median
6. Loads the cleaned DataFrames into MySQL using SQLAlchemy

**Why this step matters:** this is the ETL / data-cleaning skill recruiters specifically test for — showing you didn't just work with a pre-cleaned Kaggle CSV.

### STEP 5 — Run SQL analytics
```bash
mysql -u fintech_user -p fintech_db < ../sql/analysis_and_fraud.sql
```
Or run queries individually in MySQL Workbench / DBeaver. This file has two sections:

**Section A — Business Analytics**
- Transaction volume & value by category
- Monthly trend + cumulative revenue (`SUM() OVER (ORDER BY month)`)
- Top customers by spend
- Customer ranking within each city (`RANK() OVER (PARTITION BY city ...)`)
- RFM segmentation (Recency via `DATEDIFF`, Frequency, Monetary)

**Section B — Fraud Detection Rule Engine**
A transaction earns 1 point per rule triggered, using a CTE (`WITH ... AS`):
1. **Statistical outlier** — amount > category mean + 2×stddev (`AVG()`/`STDDEV() OVER PARTITION BY category`)
2. **Odd-hour transaction** — between 12 AM–5 AM
3. **City mismatch** — transaction city ≠ customer's home city
4. **High-risk channel** — large P2P/ATM transaction via Wallet/Net Banking

Transactions scoring **3+** are flagged high-risk. No ML — this is exactly how a first-pass fraud rules engine works in production before/alongside a model.

### STEP 6 — Interpret the results
From a real test run:
- 8,670 raw transactions → 8,153 clean rows (170 duplicates + 347 invalid rows removed)
- 47 transactions flagged high-risk fraud, concentrated in P2P Transfer (6.04% fraud rate) — consistent with real fraud patterns, since peer-to-peer transfers are a common laundering/fraud vector
- Category-level and merchant-level fraud rate breakdowns show which merchants carry the most risk

### STEP 7 — Push to GitHub
```bash
git init
git add .
git commit -m "Credit card fraud detection & transaction analytics (SQL + Python)"
git remote add origin <your-repo-url>
git push -u origin main
```

---

## 🎤 How to Explain This in Your Interview
1. **"Why SQL for fraud detection instead of Python/ML?"** — Rule-based SQL scoring is how most real fraud systems start (fast, explainable, no training data needed); ML models are usually layered on top later.
2. **"Why clean in Python but analyze in SQL?"** — Separation of concerns: Python/pandas is better for messy, row-level ETL logic; SQL is better for set-based aggregation and is what the data actually lives in.
3. **Walk through the CTE** in `analysis_and_fraud.sql` — explain `category_stats` computes the outlier threshold per category, then `scored_transactions` applies the 4 rules, then the outer query sums them into a `fraud_score`.
4. Be ready to explain **window functions** (`RANK() OVER`, `SUM() OVER`) — these are what separate junior from mid-level SQL skill in interviews.

---
*Built as a portfolio project demonstrating MySQL analytical SQL (CTEs, window functions, rule-based fraud scoring) and Python-based ETL/data cleaning skills.*
