# QUALITY CHECKS

### Overview

This document summarizes all automated data quality validations enforced in **Step 4** of the pipeline (Silver validation + Referential integrity + Metrics consolidation).

All checks are:

* Configuration-driven (`dq_config`)
* Executed via reusable functions (`04_Function_Book`)
* Aggregated into a centralized Delta table: `quality_metrics`
* Cross-referenced with:

  * `pipeline_overview.md`
  * `data_contracts.md`

---

## 1️⃣ Per-Table Quality Checks

---

#### 🟦 Customers (`04_customers_quality_checks`)

| Table     | Check Type   | Columns                  | Rule                          | Notes / Action        |
| --------- | ------------ | ------------------------ | ----------------------------- | --------------------- |
| customers | Completeness | customer_id              | NOT NULL                      | Nulls flagged         |
| customers | Completeness | customer_unique_id       | NOT NULL                      | Nulls flagged         |
| customers | Completeness | city, state, zip         | NOT NULL                      | Nulls flagged         |
| customers | Format       | customer_id              | Regex validation              | Invalid IDs flagged   |
| customers | Format       | customer_unique_id       | Regex validation              | Invalid IDs flagged   |
| customers | Enum         | customer_state           | Must be valid Brazilian state | Unknown state flagged |
| customers | Range        | customer_zip_code_prefix | Valid zip range               | Out-of-range flagged  |
| customers | Uniqueness   | customer_id              | Primary key unique            | Duplicates flagged    |

#### Example Violation

* Invalid state code: `"XX"`
* Zip outside allowed range

---

#### 🟦 Orders (`04_orders_quality_checks`)

| Table  | Check Type   | Columns                  | Rule                              | Notes / Action                 |
| ------ | ------------ | ------------------------ | --------------------------------- | ------------------------------ |
| orders | Completeness | order_id, customer_id    | NOT NULL                          | Critical nulls removed earlier |
| orders | Completeness | purchase_timestamp       | NOT NULL                          | Required field                 |
| orders | Format       | order_id                 | Regex validation                  | Invalid ID flagged             |
| orders | Date Range   | order_purchase_timestamp | Between 2016-09-04 and 2018-10-18 | Outside window flagged         |
| orders | Enum         | order_status             | Allowed values only               | Invalid status flagged         |
| orders | Uniqueness   | order_id                 | Primary key unique                | Duplicates flagged             |

#### Example Violation

* Timestamp outside dataset period
* Duplicate order_id

---

#### 🟦 Order Items (`04_order_items_quality_checks`)

| Table       | Check Type   | Columns                   | Rule                      | Notes / Action         |
| ----------- | ------------ | ------------------------- | ------------------------- | ---------------------- |
| order_items | Completeness | All columns               | ZERO NULLS allowed        | Strict enforcement     |
| order_items | Format       | order_id, product_id      | Regex validation          | Invalid format flagged |
| order_items | Format       | order_item_id             | Length + pattern enforced | Invalid flagged        |
| order_items | Uniqueness   | (order_id, order_item_id) | Composite PK unique       | Duplicates flagged     |

#### Example Violation

* Same `(order_id, order_item_id)` appearing twice

---

#### 🟦 Products (`04_products_quality_checks`)

| Table    | Check Type   | Columns                       | Rule               | Notes / Action        |
| -------- | ------------ | ----------------------------- | ------------------ | --------------------- |
| products | Completeness | product_id                    | NOT NULL           | Required              |
| products | Completeness | weight, length, height, width | NOT NULL           | Strict enforcement    |
| products | Format       | product_id                    | Regex validation   | Invalid flagged       |
| products | Data Type    | dimensions                    | Must be integer    | Type mismatch flagged |
| products | Uniqueness   | product_id                    | Primary key unique | Duplicates flagged    |

#### Example Violation

* Non-integer dimension
* Duplicate product_id

---

#### 🟦 Payments (`04_payments_quality_checks`)

| Table    | Check Type   | Columns                        | Rule                                                | Notes / Action     |
| -------- | ------------ | ------------------------------ | --------------------------------------------------- | ------------------ |
| payments | Completeness | order_id, payment_type, value  | NOT NULL                                            | Required           |
| payments | Format       | order_id                       | Regex validation                                    | Invalid flagged    |
| payments | Enum         | payment_type                   | Must be in allowed list (boleto, credit_card, etc.) | Invalid flagged    |
| payments | Range        | payment_value                  | ≥ 0                                                 | Negative flagged   |
| payments | Uniqueness   | (order_id, payment_sequential) | Composite PK unique                                 | Duplicates flagged |

#### Example Violation

* payment_value < 0
* payment_type = "crypto" (invalid enum)

---

## 2️⃣ Referential Integrity Checks

(`04_referential_integrity_checks`)

Strict zero-tolerance validation using left anti joins.

| Child Table | Parent Table | Key         | Rule                    | Action          |
| ----------- | ------------ | ----------- | ----------------------- | --------------- |
| orders      | customers    | customer_id | Must exist in customers | Orphans flagged |
| order_items | orders       | order_id    | Must exist in orders    | Orphans flagged |
| payments    | orders       | order_id    | Must exist in orders    | Orphans flagged |
| order_items | products     | product_id  | Must exist in products  | Orphans flagged |

#### Enforcement Logic

* Null keys removed before join
* Left anti join identifies orphan rows
* Threshold = 0 (any violation → Fail)

---

## 3️⃣ Conflict Resolution & Business Rule Flags

These checks validate cross-table logical consistency.

| Category            | Rule                                              | Example           |
| ------------------- | ------------------------------------------------- | ----------------- |
| Status vs Payment   | Delivered order without payment                   | Flag anomaly      |
| Timestamp Conflicts | Delivery before purchase                          | Flag anomaly      |
| Missing Payments    | Order exists but no payment record                | Flag via RI check |
| Payment Gap         | Sum(order_items.price + shipping) ≠ payment_value | Flag gap          |

---

## 4️⃣ Output Metrics

All quality notebooks return a `metrics_df` containing:

* table_name
* check_type
* column_name
* violation_count
* threshold
* status (Pass / Fail)
* execution_timestamp

---

### Consolidation (`04_0_Quality_Metrics_Writer`)

* Executes all table-specific quality notebooks
* Collects:

  * `metrics_customers`
  * `metrics_orders`
  * `metrics_order_items`
  * `metrics_products`
  * `metrics_payments`
  * `metrics_referential`
* Combines via `unionByName`
* Writes to Delta table: **`quality_metrics`**


This enables:

* SLA monitoring
* Dashboard reporting
* Historical audit trail
* Automated alerting

---

## 5️⃣ Automation vs Manual Checks

#### ✅ Fully Automated

* Null checks
* Regex format checks
* Data type validation
* Enum validation
* Range checks
* Date range checks
* Primary key uniqueness
* Referential integrity

#### ⚠️ Semi-Automated / Logical Monitoring

* Status-payment anomalies
* Timestamp conflicts
* Payment vs item value mismatches

#### 🔎 Known Gaps

* No real-time source validation
* No external third-party address validation
* No fraud scoring
* No duplicate customer fuzzy matching

---

## 6️⃣ Architecture Flow Reference

```
pipeline_overview.md
        ↓
data_contracts.md
        ↓
quality_checks.md
        ↓
quality_metrics (Delta table)
```

* `pipeline_overview.md` → explains layers
* `data_contracts.md` → defines schema & rules
* `quality_checks.md` → enforces validation logic
* `quality_metrics` → operational monitoring output

---

## Final Summary

This quality framework ensures:

* Structural correctness
* Domain validity
* Referential consistency
* Business rule enforcement
* Production-grade auditability

The system is:

* Modular
* Configurable
* Reusable
* Scalable
* Delta Lake compatible
