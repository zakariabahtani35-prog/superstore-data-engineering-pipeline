# 🛒 Superstore Data Engineering Pipeline

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791?logo=postgresql)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?logo=pandas)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.x-red)
![Status](https://img.shields.io/badge/Status-Production--Ready-brightgreen)

> A professional data engineering pipeline that ingests a cleaned Superstore retail dataset, normalizes it into a relational schema, and loads it into a PostgreSQL database — ready for analytical SQL queries and dashboard integrations.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Project Objectives](#project-objectives)
- [Dataset Description](#dataset-description)
- [Data Engineering Pipeline](#data-engineering-pipeline)
- [Technologies Used](#technologies-used)
- [Database Architecture](#database-architecture)
- [Relational Schema & ERD](#relational-schema--erd)
- [Data Normalization Strategy](#data-normalization-strategy)
- [Table Descriptions](#table-descriptions)
- [Data Loading Process](#data-loading-process)
- [SQL Query Examples](#sql-query-examples)
- [Project Folder Structure](#project-folder-structure)
- [Installation & Setup Guide](#installation--setup-guide)
- [How to Run the Project](#how-to-run-the-project)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

## 📌 Project Overview

This project implements a complete **data engineering pipeline** for a retail Superstore dataset. Starting from a flat, denormalized CSV file (`superstore_clean.csv`), the pipeline transforms the raw data into a structured, normalized **PostgreSQL relational database** consisting of six related tables.

The pipeline follows standard data engineering best practices: ingestion, transformation, normalization, loading, and validation. The resulting database is optimized for analytical queries, reporting, and integration with BI tools such as Tableau, Power BI, or Metabase.

---

## 🎯 Project Objectives

- **Ingest** a cleaned CSV dataset using Python and Pandas
- **Standardize** column names for consistency
- **Convert** date fields to proper `datetime` types
- **Normalize** the flat dataset into six relational tables following 1NF, 2NF, and 3NF principles
- **Design** a relational schema with primary keys, foreign keys, and referential integrity
- **Load** all normalized tables into a PostgreSQL database via SQLAlchemy
- **Validate** data integrity after loading
- **Enable** complex analytical SQL queries across the normalized schema

---

## 📊 Dataset Description

**File:** `superstore_clean.csv`

The Superstore dataset is a widely-used retail dataset containing transactional sales data for a fictional US-based retail company. It includes information about orders, customers, products, and geographic regions.

| Column | Type | Description |
|---|---|---|
| `order_id` | STRING | Unique identifier for each order |
| `order_date` | DATE | Date the order was placed |
| `ship_date` | DATE | Date the order was shipped |
| `ship_mode` | STRING | Shipping method used |
| `customer_id` | STRING | Unique customer identifier |
| `customer_name` | STRING | Full name of the customer |
| `segment` | STRING | Customer market segment |
| `postal_code` | STRING | Delivery postal code |
| `city` | STRING | Delivery city |
| `state` | STRING | Delivery state |
| `region` | STRING | Geographic region |
| `country` | STRING | Country |
| `product_id` | STRING | Unique product identifier |
| `product_name` | STRING | Full product name |
| `category` | STRING | Product category |
| `sub_category` | STRING | Product sub-category |
| `sales` | FLOAT | Revenue from the order line |
| `cost` | FLOAT | Cost of goods sold |
| `profit` | FLOAT | Profit margin on the line |

---

## ⚙️ Data Engineering Pipeline

The pipeline executes the following sequential steps:

```
┌─────────────────────────────────────────────────────────────────────┐
│                     DATA ENGINEERING PIPELINE                       │
├──────────────┬──────────────┬──────────────┬──────────────┬─────────┤
│  STEP 1      │  STEP 2      │  STEP 3      │  STEP 4      │ STEP 5  │
│              │              │              │              │         │
│  Ingest CSV  │  Clean &     │  Normalize   │  Build FK    │  Load   │
│  via Pandas  │  Standardize │  into 6      │  References  │  into   │
│              │  Columns     │  Tables      │              │ Postgres│
└──────────────┴──────────────┴──────────────┴──────────────┴─────────┘
```

**Step-by-step breakdown:**

1. **Load CSV** — Read `superstore_clean.csv` into a Pandas DataFrame
2. **Clean & Standardize** — Normalize column names, parse date fields
3. **Build Dimension Tables** — Extract `regions`, `customers`, `categories`, `products`
4. **Build Fact Table** — Extract `orders` and `order_details`
5. **Resolve Foreign Keys** — Merge `category_id` into `products` table
6. **Connect to PostgreSQL** — Establish connection using SQLAlchemy + psycopg2
7. **Load Data** — Insert all tables in dependency order within a single transaction
8. **Validate** — Confirm row counts and referential integrity

---

## 🛠️ Technologies Used

| Technology | Version | Role |
|---|---|---|
| **Python** | 3.10+ | Core programming language |
| **Pandas** | 2.x | Data manipulation and transformation |
| **PostgreSQL** | 15+ | Target relational database |
| **SQLAlchemy** | 2.x | Database ORM and connection management |
| **psycopg2** | 2.9+ | PostgreSQL adapter for Python |
| **Jupyter Notebook** | 7.x | Interactive development environment |

---

## 🗄️ Database Architecture

The database follows a **star schema** design pattern, with `order_details` as the central fact table and surrounding dimension tables.

```
superstore_db
├── regions          (dimension)
├── customers        (dimension, FK → regions)
├── categories       (dimension)
├── products         (dimension, FK → categories)
├── orders           (fact-adjacent, FK → customers)
└── order_details    (fact table, FK → orders, products)
```

**Database name:** `superstore_db`  
**Schema:** `public`  
**Engine:** PostgreSQL 15+  
**Charset:** UTF-8

---

## 🔗 Relational Schema & ERD

### Entity Relationship Diagram (Text Format)

```
┌──────────────────┐         ┌──────────────────────────┐
│     REGIONS      │         │        CUSTOMERS         │
├──────────────────┤         ├──────────────────────────┤
│ PK postal_code   │◄────────│ PK customer_id           │
│    city          │  1 : N  │    customer_name         │
│    state         │         │    segment               │
│    region        │         │ FK postal_code           │
│    country       │         └──────────────────────────┘
└──────────────────┘                      │
                                          │ 1 : N
                               ┌──────────▼───────────┐
                               │        ORDERS        │
                               ├──────────────────────┤
                               │ PK order_id          │
                               │    order_date        │
                               │    ship_date         │
                               │    ship_mode         │
                               │ FK customer_id       │
                               └──────────┬───────────┘
                                          │ 1 : N
┌──────────────────┐         ┌────────────▼─────────────┐
│    CATEGORIES    │         │       ORDER_DETAILS      │
├──────────────────┤         ├──────────────────────────┤
│ PK category_id   │         │ FK order_id              │
│    category_name │         │ FK product_id            │
└──────────────────┘         │    sales                 │
          │                  │    cost                  │
          │ 1 : N            │    profit                │
┌─────────▼────────┐         └──────────────────────────┘
│     PRODUCTS     │                    ▲
├──────────────────┤                    │ N : 1
│ PK product_id    │────────────────────┘
│    product_name  │
│    sub_category  │
│ FK category_id   │
└──────────────────┘
```

### Foreign Key Relationships

| Child Table | Foreign Key | Parent Table | Parent Key |
|---|---|---|---|
| `customers` | `postal_code` | `regions` | `postal_code` |
| `products` | `category_id` | `categories` | `category_id` |
| `orders` | `customer_id` | `customers` | `customer_id` |
| `order_details` | `order_id` | `orders` | `order_id` |
| `order_details` | `product_id` | `products` | `product_id` |

---

## 📐 Data Normalization Strategy

The original CSV is a **flat, denormalized** file where every attribute — customer details, product info, geographic data, and transaction records — co-exist in a single row. This violates multiple normal forms and leads to significant data redundancy.

### First Normal Form (1NF)
All columns contain atomic values. No repeating groups or arrays. Each row is uniquely identifiable.

### Second Normal Form (2NF)
All non-key attributes are fully functionally dependent on the entire primary key. By splitting the data into dimension tables (`customers`, `products`, `regions`, `categories`), we eliminate partial dependencies.

### Third Normal Form (3NF)
All non-key attributes depend **only** on the primary key — not on other non-key attributes (transitive dependencies are eliminated). For example:
- `city`, `state`, `region` depend on `postal_code`, not on `customer_id` → extracted to `regions`
- `product_name`, `sub_category` depend on `product_id`, not `order_id` → extracted to `products`
- `category_name` depends on `category_id`, not `product_id` → extracted to `categories`

---

## 📁 Table Descriptions

### `regions`
Stores geographic location data keyed by postal code.

| Column | Type | Constraint |
|---|---|---|
| `postal_code` | VARCHAR | PRIMARY KEY |
| `city` | VARCHAR | NOT NULL |
| `state` | VARCHAR | NOT NULL |
| `region` | VARCHAR | NOT NULL |
| `country` | VARCHAR | NOT NULL |

### `customers`
Stores unique customer profiles with a reference to their location.

| Column | Type | Constraint |
|---|---|---|
| `customer_id` | VARCHAR | PRIMARY KEY |
| `customer_name` | VARCHAR | NOT NULL |
| `segment` | VARCHAR | |
| `postal_code` | VARCHAR | FOREIGN KEY → regions |

### `categories`
Lookup table for product categories.

| Column | Type | Constraint |
|---|---|---|
| `category_id` | INTEGER | PRIMARY KEY |
| `category_name` | VARCHAR | NOT NULL |

### `products`
Stores product catalog with reference to its category.

| Column | Type | Constraint |
|---|---|---|
| `product_id` | VARCHAR | PRIMARY KEY |
| `product_name` | VARCHAR | NOT NULL |
| `sub_category` | VARCHAR | |
| `category_id` | INTEGER | FOREIGN KEY → categories |

### `orders`
Stores order header information linked to a customer.

| Column | Type | Constraint |
|---|---|---|
| `order_id` | VARCHAR | PRIMARY KEY |
| `order_date` | DATE | NOT NULL |
| `ship_date` | DATE | |
| `ship_mode` | VARCHAR | |
| `customer_id` | VARCHAR | FOREIGN KEY → customers |

### `order_details`
Central fact table — one row per order-product combination.

| Column | Type | Constraint |
|---|---|---|
| `order_id` | VARCHAR | FOREIGN KEY → orders |
| `product_id` | VARCHAR | FOREIGN KEY → products |
| `sales` | FLOAT | |
| `cost` | FLOAT | |
| `profit` | FLOAT | |

---

## 🚀 Data Loading Process

Tables are loaded in **strict dependency order** to respect referential integrity:

```python
# Insertion order respects FK constraints
1. regions        → no dependencies
2. customers      → depends on regions
3. categories     → no dependencies
4. products       → depends on categories
5. orders         → depends on customers
6. order_details  → depends on orders + products
```

All inserts are executed within a **single SQLAlchemy transaction** (`engine.begin()`), ensuring atomicity — if any insert fails, the entire operation rolls back.

```python
with engine.begin() as conn:
    regions.to_sql("regions", conn, if_exists="append", index=False)
    customers.to_sql("customers", conn, if_exists="append", index=False)
    categories.to_sql("categories", conn, if_exists="append", index=False)
    products.to_sql("products", conn, if_exists="append", index=False)
    orders.to_sql("orders", conn, if_exists="append", index=False)
    order_details.to_sql("order_details", conn, if_exists="append", index=False)
```

---

## 🔍 SQL Query Examples

### Total Revenue and Profit by Region
```sql
SELECT
    r.region,
    SUM(od.sales)  AS total_revenue,
    SUM(od.profit) AS total_profit,
    ROUND(SUM(od.profit) / NULLIF(SUM(od.sales), 0) * 100, 2) AS profit_margin_pct
FROM order_details od
JOIN orders       o  ON od.order_id   = o.order_id
JOIN customers    c  ON o.customer_id = c.customer_id
JOIN regions      r  ON c.postal_code = r.postal_code
GROUP BY r.region
ORDER BY total_revenue DESC;
```

### Top 10 Most Profitable Products
```sql
SELECT
    p.product_name,
    cat.category_name,
    SUM(od.profit) AS total_profit
FROM order_details od
JOIN products p   ON od.product_id   = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
GROUP BY p.product_name, cat.category_name
ORDER BY total_profit DESC
LIMIT 10;
```

### Monthly Sales Trend
```sql
SELECT
    DATE_TRUNC('month', o.order_date) AS month,
    SUM(od.sales)                     AS monthly_revenue
FROM order_details od
JOIN orders o ON od.order_id = o.order_id
GROUP BY month
ORDER BY month;
```

### Customer Lifetime Value (CLV)
```sql
SELECT
    c.customer_id,
    c.customer_name,
    c.segment,
    COUNT(DISTINCT o.order_id) AS total_orders,
    SUM(od.sales)              AS lifetime_revenue,
    SUM(od.profit)             AS lifetime_profit
FROM customers c
JOIN orders       o  ON c.customer_id = o.customer_id
JOIN order_details od ON o.order_id  = od.order_id
GROUP BY c.customer_id, c.customer_name, c.segment
ORDER BY lifetime_revenue DESC
LIMIT 20;
```

### Sales by Category and Sub-Category
```sql
SELECT
    cat.category_name,
    p.sub_category,
    SUM(od.sales)  AS total_sales,
    SUM(od.profit) AS total_profit
FROM order_details od
JOIN products   p   ON od.product_id  = p.product_id
JOIN categories cat ON p.category_id  = cat.category_id
GROUP BY cat.category_name, p.sub_category
ORDER BY cat.category_name, total_sales DESC;
```

---

## 📂 Project Folder Structure

```
superstore-pipeline/
│
├── data/
│   └── superstore_clean.csv        # Source dataset (not tracked in git)
│
├── notebooks/
│   └── superstore_pipeline_v2.ipynb  # Main pipeline notebook
│
├── sql/
│   ├── create_schema.sql           # DDL for table creation with constraints
│   └── analytical_queries.sql      # Sample analytical SQL queries
│
├── docs/
│   ├── README.md                   # This file
│   └── technical_documentation.pdf # Full technical report
│
├── requirements.txt                # Python dependencies
├── .env.example                    # Environment variable template
└── .gitignore
```

---

## 🛠️ Installation & Setup Guide

### Prerequisites

- Python 3.10+
- PostgreSQL 15+ (running locally or remote)
- pip

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/superstore-pipeline.git
cd superstore-pipeline
```

### 2. Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate           # Windows
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

**`requirements.txt`:**
```
pandas>=2.0.0
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0
jupyter>=7.0.0
python-dotenv>=1.0.0
```

### 4. Create the PostgreSQL Database

```sql
-- Connect to PostgreSQL as superuser
CREATE DATABASE superstore_db;
CREATE USER superstore_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE superstore_db TO superstore_user;
```

### 5. Configure Environment Variables

Create a `.env` file:

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=superstore_db
DB_USER=postgres
DB_PASSWORD=your_password
```

### 6. Add Dataset

Place `superstore_clean.csv` in the `data/` directory.

---

## ▶️ How to Run the Project

### Option A — Jupyter Notebook (Recommended)

```bash
jupyter notebook notebooks/superstore_pipeline_v2.ipynb
```

Run each cell sequentially. The notebook is divided into logical sections:

| Cell | Description |
|---|---|
| 1 | Import libraries |
| 2 | Load CSV dataset |
| 3 | Clean date columns |
| 4–9 | Build normalized tables |
| 10 | Connect to PostgreSQL |
| 11 | Load all tables |

### Option B — Command Line (Python Script)

Convert the notebook to a script and run:

```bash
jupyter nbconvert --to script notebooks/superstore_pipeline_v2.ipynb
python notebooks/superstore_pipeline_v2.py
```

### Expected Output

```
✅ regions       — 631 rows inserted
✅ customers     — 793 rows inserted
✅ categories    —   3 rows inserted
✅ products      — 1850 rows inserted
✅ orders        — 5009 rows inserted
✅ order_details — 9994 rows inserted

🎉 Pipeline complete — all tables loaded into PostgreSQL!
```

---

## 🔮 Future Improvements

- [ ] **Schema DDL Script** — Add explicit `CREATE TABLE` statements with all constraints, data types, and indexes
- [ ] **Data Validation Layer** — Add pre-load validation (null checks, type validation, range checks)
- [ ] **Incremental Loading** — Support upsert logic (`INSERT ... ON CONFLICT`) for production pipelines
- [ ] **Airflow Integration** — Orchestrate the pipeline with Apache Airflow DAGs
- [ ] **dbt Models** — Add analytical transformations using dbt on top of raw tables
- [ ] **Unit Tests** — Add pytest-based tests for each transformation step
- [ ] **Docker Compose** — Containerize Python + PostgreSQL for portable deployment
- [ ] **Logging** — Add structured logging with log levels for production monitoring
- [ ] **CI/CD** — Add GitHub Actions workflow for automated testing

---

## 👤 Author

**Data Engineering Project**  
Built with Python, Pandas, SQLAlchemy, and PostgreSQL

> *This project demonstrates end-to-end data engineering skills: ingestion, transformation, normalization, relational modeling, and database loading — core competencies for a production data engineering role.*

---

*Licensed under MIT License*
