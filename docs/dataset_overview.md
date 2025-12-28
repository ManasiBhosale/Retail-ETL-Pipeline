# DATASET OVERVIEW

## Overview

This dataset represents a synthetic e-commerce platform containing customer, order, product, and payment information.
The data is split across **five core tables**, each with a clearly defined grain and role in the overall data model.

The dataset spans approximately **127K records per table**, covering customer activity and transactions from **September 2016 to October 2018**.

---

## Table: `customers`

### Structural Summary

* **Rows:** 127,595
* **Columns:** 4
* **Primary Key:** `customer_id`
* **Data Types:**

  * `customer_id` – string
  * `customer_zip_code_prefix` – integer
  * `customer_city` – string
  * `customer_state` – string

### Business Meaning

* One row represents **one unique customer**.
* This is **reference (dimension) data** used to enrich orders with geographic attributes.

### Data Quality Observations

* No missing values across any column.
* `customer_id` is fully unique.
* Geographic fields (`city`, `state`) show reasonable cardinality:

  * 27 states
  * ~4,100 cities

### Grain

* **One row per customer**

---

## Table: `orders`

### Structural Summary

* **Rows:** 127,595
* **Columns:** 7
* **Primary Key:** `order_id`
* **Foreign Key:** `customer_id`
* **Data Types (raw):**

  * All date fields stored as strings in `dd-MM-yyyy HH:mm` or `dd-MM-yyyy` formats.

### Business Meaning

* One row represents **one customer order**.
* This is **transactional / event data** capturing the lifecycle of an order.

### Data Quality Observations

* Significant null presence in lifecycle-related fields:

  * `order_status` missing for ~30% of rows
  * `order_delivered_timestamp` missing for undelivered orders
* Multiple valid order states observed:

  * delivered, shipped, canceled, processing, invoiced, approved, unavailable
* Dates span:

  * **Purchases:** Sep 2016 – Sep 2018
  * **Deliveries:** Oct 2016 – Oct 2018

### Grain

* **One row per order**

---

## Table: `order_items`

### Structural Summary

* **Rows:** 127,595
* **Columns:** 5
* **Primary Key:** Composite (`order_id`, `product_id`)
* **Foreign Keys:** `order_id`, `product_id`, `seller_id`
* **Data Types:**

  * Prices and shipping charges stored as floats.

### Business Meaning

* One row represents **one product purchased within an order**.
* This is **transaction-level line item data**.

### Data Quality Observations

* No missing values.
* Price distribution shows:

  * Right skew
  * High-value outliers (max ≈ 6,700)
* Shipping charges include zero values, which may represent promotions or bundled shipping.

### Grain

* **One row per product per order**

---

## Table: `payments`

### Structural Summary

* **Rows:** 127,595
* **Columns:** 5
* **Primary Key:** (`order_id`, `payment_sequential`)
* **Foreign Key:** `order_id`
* **Data Types:**

  * Monetary values stored as floats
  * Installments stored as integers

### Business Meaning

* One row represents **a payment attempt or installment for an order**.
* This is **financial transactional data**.

### Data Quality Observations

* No missing values.
* Valid payment types observed:

  * credit_card, debit_card, wallet, voucher
* Some orders have multiple payment records (installments or split payments).
* Payment values range from 0 to ~13,600.

### Grain

* **One row per payment transaction per order**

---

## Table: `products`

### Structural Summary

* **Rows:** 127,595
* **Columns:** 6
* **Primary Key:** `product_id`
* **Data Types:**

  * Dimensions and weight stored as floats
  * Category stored as string

### Business Meaning

* One row represents **a unique product SKU**.
* This is **reference (dimension) data** describing physical attributes of products.

### Data Quality Observations

* Missing values present:

  * ~476 missing product categories
  * ~25 missing dimensional values
* Product categories:

  * 70 distinct values
  * Includes some naming inconsistencies and typos (e.g., `costruction_tools_*`)
* Physical attributes contain large outliers (e.g., very heavy or large items).

### Grain

* **One row per product**

---

## Cross-Table Relationships (High-Level)

* `customers.customer_id` → `orders.customer_id`
* `orders.order_id` → `order_items.order_id`
* `orders.order_id` → `payments.order_id`
* `products.product_id` → `order_items.product_id`

This structure supports:

* Customer-level analytics
* Order lifecycle analysis
* Revenue and payment breakdowns
* Product performance metrics

---

## Key Engineering Implications

* Timestamp parsing must be handled explicitly due to mixed date formats.
* Null handling rules differ by table and column role.
* Fact tables (`orders`, `order_items`, `payments`) require careful grain preservation during joins.
* Dimension tables (`customers`, `products`) require normalization and data quality enforcement before downstream use.

---
