# PIPELINE OVERVIEW

## Project Overview

This project implements a **retail e-commerce data pipeline** for the Brazilian e-commerce dataset (Kaggle: Olist Brazil E-commerce). The pipeline ingests raw CSV files containing customer, order, product, payment, and order item data, performs cleaning, validation, and normalization, and produces **analytics-ready Gold layer tables** for BI and reporting.

The pipeline handles data spanning **2016 – 2018**, covering approximately 99K customers and their associated orders, products, payments, and order items.

---

## Pipeline Layers

The pipeline is organized into multiple layers, each progressively refining the data:

### 1. Exploration Layer

* **Purpose:** Understand the dataset’s composition, quality, and business meaning.
* **Workflow:**

  * Load tables into pandas or Spark DataFrames.
  * Inspect structure, row counts, column types, summary statistics, and null counts.
  * Visualize geographic and categorical distributions (e.g., number of customers per state).
* **Notebook Example:** `01_explore_customers.ipynb`
* **Outcome:** Foundational understanding of table composition, patterns, and potential data quality issues.
* **Note:** Similar exploration occurs for orders, order items, products, payments, and geolocation data in other notebooks under `01_exploration/`.


### 2. Raw Layer

* **Purpose:** Capture original CSV data with minimal transformations.
* **Workflow:**

  * Scans source directories for raw CSV files.
  * Reads each file into Spark DataFrames with header and schema inference.
  * Writes data into Delta tables in the `bronze` schema.
* **Notebook Example:** `02_raw_to_bronze_ingestion.ipynb`
* **Outcome:** Reproducible ingestion of raw data into managed Delta tables for downstream processing.


### 3. Silver Layer

* **Purpose:** Transform raw data into a structured, normalized, and high-quality format.
* **Workflow:**

  * Trim string columns, standardize types, and parse timestamps.
  * Handle missing values and remove rows with critical nulls (e.g., `order_id`, `customer_id`).
  * Write cleaned DataFrames into Delta tables under the silver schema.
* **Notebooks:**

  * Customers → `03_silver_customers.ipynb`
  * Orders → `03_silver_orders.ipynb`
  * Order Items → `03_silver_order_items.ipynb`
  * Products → `03_silver_products.ipynb`
  * Payments → `03_silver_payments.ipynb`
* **Outcome:** Reliable silver tables ready for quality validation and business transformations.

### 4. Quality / Validation Layer

* **Purpose:** Ensure data integrity, correctness, and consistency.
* **Workflow:**

  * Use configuration-driven validation rules for null checks, formats, enums, and data types.
  * Run primary key uniqueness checks and referential integrity checks across related tables.
  * Aggregate results into a **metrics DataFrame** for monitoring and orchestration.

* **Outcome:** Automated, reusable checks ensure only high-quality data progresses downstream.

### 5. Gold Layer

* **Purpose:** Transform silver tables into **analytics-ready fact and dimension tables**.
* **Workflow:**

  * Build dimension tables (e.g., `dim_customers`, `dim_products`) by integrating semantic features and removing duplicates.
  * Build fact tables (e.g., `fact_orders`, `fact_order_items`) by joining and aggregating order, customer, and product data.
  * Normalize timestamps, calculate business metrics, and enforce relational integrity.
* **Notebooks:**

  * `05_1_orders_integrated.ipynb`
  * `05_2_customer_semantics.ipynb`
  * `05_3_product_category_reference_table.ipynb`
  * `05_4_gold_layer_design.ipynb`
* **Outcome:** Star-schema tables optimized for reporting, dashboards, and BI analytics.

---

## Table Flow Diagram (Conceptual)

```text
Customers --------┐
                 ├─> Orders → Fact_Orders
Orders Items -----┘
Products --------> Fact_OrderItems
Payments --------> Fact_Orders
Products --------> Dim_Products
Customers --------> Dim_Customers
```

* **Fact Tables:**

  * `fact_orders` → integrated order and payment data
  * `fact_order_items` → detailed line items per order
* **Dimension Tables:**

  * `dim_customers` → enriched customer details
  * `dim_products` → product and category attributes
  * `dim_dates` → for efficient querying, simplified reporting

---

## ETL Steps Summary

| Step                        | Description                                                                                |
| --------------------------- | ------------------------------------------------------------------------------------------ |
| **1. Exploration**          | Inspect tables, understand column types, nulls, duplicates, and basic distributions.       |
| **2. Raw Ingestion/Bronze**        | Load CSVs into Delta Lake bronze tables with minimal transformation.                       |
| **3. Silver**               | Normalize data types, handle nulls, trim strings, and standardize timestamps.              |
| **4. Quality Checks**       | Validate data via null, format, type, enum, primary key, and referential integrity checks. |
| **5. Business Rules**       | Standardize statuses, calculate order values, and apply domain-specific transformations.   |
| **6. Dimensional Modeling** | Build fact and dimension tables for gold layer analytics.                                  |
| **7. Analysis / Metrics**   | Generate metrics, KPIs, and summary statistics for reporting or monitoring.                |
| **8. Consolidation**        | Aggregate quality metrics from all tables into a central repository.                       |
| **9. Gold Layer Delivery**  | Provide analytics-ready tables for BI dashboards, reporting, or ML pipelines.              |

---

## Dependencies / Relationships

* **Primary Key → Foreign Key links in Gold layer:**

  * `fact_orders.customer_id → dim_customers.customer_id`
  * `fact_order_items.order_id → fact_orders.order_id`
  * `fact_order_items.product_id → dim_products.product_id`


