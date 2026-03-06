## DATASET OVERVIEW

Dataset source: [Kaggle](https://www.kaggle.com/) \
Dataset name: [Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

---

## Overview

This dataset represents a Brazilian e-commerce marketplace containing customer, order, product, and payment information.

We are using the following subset of tables:

* customers
* orders
* order_items
* products
* product_category_name_translation
* payments

The dataset covers transactions from **2016 to 2018**, representing real marketplace activity across Brazil.

It is a classic **transactional star-schema-like model**, with:

| Entity / Table            | Type                     | Description                                      |
|---------------------------|---------------------------|--------------------------------------------------|
| Customers                 | Dimension                 | Master data describing each customer.            |
| Products                  | Dimension                 | Master data for all products.               |
| Orders                    | Fact (Header Level)       | High‑level order information such as order date, delivery date, and status. |
| Order Items               | Fact (Line Level)         | Detailed line‑item records for each product within an order. |
| Payments                  | Fact (Financial Events)   | Transaction-level payment records linked to orders. |
| Category Translation      | Lookup / Reference Table  | Small mapping table for category names. |


---

## Table: `customers`

### Structural Summary

* Rows: ~99K unique customers
* Columns: 5
* Primary Key: `customer_id`

**Columns & Types**

| Column Name                 | Data Type | Description                                   |
|-----------------------------|-----------|-----------------------------------------------|
| `customer_id`               | string    | Unique identifier for each order              |
| `customer_unique_id`        | string    | Unique identifier for each customer record.   |
| `customer_zip_code_prefix`  | integer   | ZIP/postal code prefix for the customer.      |
| `customer_city`             | string    | City where the customer is located.           |
| `customer_state`            | string    | State/region of the customer.                 |


### Business Meaning

* One row represents one **customer account instance**.
* `customer_unique_id` is an unique identifier of a customer.
* `customer_unique_id` can repeat across time for the same person (multiple orders).
* `customer_id` is unique for every order, even when multiple orders belong to the same `customer_unique_id`. This makes `customer_id` the primary key of the table.

This is **reference (dimension) data** used to enrich orders with geographic information.

### Data Quality Observations

* No null-heavy columns.
* `customer_id` is unique.
* `customer_unique_id` is not unique (expected).
* 27 Brazilian states represented.
* High city cardinality (thousands of cities).

### Grain

**One row per customer account record.**

---

## Table: `orders`

### Structural Summary

* Rows: ~99K orders
* Columns: 8
* Primary Key: `order_id`

**Columns & Types**

| Column Name                     | Data Type | Description                                      |
|---------------------------------|-----------|--------------------------------------------------|
| `order_id`                      | string    | Unique identifier for each order.                |
| `customer_id`                   | string    | Foreign key linking to the customers table.      |
| `order_status`                  | string    | Current status of the order.                     |
| `order_purchase_timestamp`      | timestamp | When the order was placed.                       |
| `order_approved_at`             | timestamp | When the order was approved.                     |
| `order_delivered_carrier_date`  | timestamp | When the order was handed to the carrier.        |
| `order_delivered_customer_date` | timestamp | When the order was delivered to the customer.    |
| `order_estimated_delivery_date` | timestamp | Expected delivery date at the time of purchase.  |


### Business Meaning

* One row represents one **placed order**.
* Transactional data.
* Connects customers to purchases.

### Data Quality Observations

* Some nulls in delivery-related timestamp columns (expected for cancelled/unshipped orders).
* Order statuses include delivered, shipped, canceled, invoiced, processing, unavailable, created, approved.
* `order_id` is unique.

### Grain

**One row per order.**

---

## Table: `order_items`

### Structural Summary

* Rows: ~98K
* Columns: 7
* Composite Key: (`order_id`, `order_item_id`)

**Columns & Types**

| Column Name             | Data Type | Description                                      |
|-------------------------|-----------|--------------------------------------------------|
| `order_id`              | string    | Identifier linking each item to its parent order. |
| `order_item_id`         | integer   | Line‑level identifier within an order.           |
| `product_id`            | string    | Identifier linking the item to the products table. |
| `seller_id`             | string    | Identifier of the seller fulfilling the item.    |
| `price`                 | decimal   | Item price charged to the customer.              |
| `freight_value`         | decimal   | Shipping cost allocated to the item.             |


### Business Meaning

* One row represents one **product within an order**.
* This is line-level transactional data.
* Multiple rows per order possible.

### Data Quality Observations

* No null-heavy columns.
* `order_item_id` resets per order (expected behavior).
* Price and freight (shipping) are non-negative.

### Grain

**One row per product per order.**

Note: This is the lowest transactional detail level in the purchasing flow.

---

## Table: `payments`

### Structural Summary

* Rows: ~99K
* Columns: 5
* Composite Key: (`order_id`, `payment_sequential`)

**Columns & Types**

| Column Name          | Data Type | Description                                      |
|----------------------|-----------|--------------------------------------------------|
| `order_id`           | string    | Identifier linking the payment to its order.     |
| `payment_sequential` | integer   | Sequence number for multiple payments per order. |
| `payment_type`       | string    | Method used for the payment.                     |
| `payment_installments` | integer | Number of installments selected by the customer. |
| `payment_value`      | decimal   | Amount paid in this specific transaction.        |


### Business Meaning

* One row represents one **payment event for an order**.
* An order can have multiple payments (split payments).
* Transactional financial data.

### Data Quality Observations

* Some orders have multiple payment rows.
* Payment types include:

  * credit card
  * debit card
  * boleto
  * voucher

* `payment_installments` sometimes > 1 (credit card).
* No null-heavy columns.

Potential checks:

* Negative payment_value?
* Payment sum not matching order item totals?

### Grain

**One row per payment attempt per order.**

---

## Table: `products`

### Structural Summary

* Rows: ~32K products
* Columns: 9
* Primary Key: `product_id`

**Columns & Types**

| Column Name          | Data Type | Description                                   |
|----------------------|-----------|-----------------------------------------------|
| `product_id`         | string    | Unique identifier for each product.           |
| `product_category_name` | string | Category label assigned to the product.       |
| `product_weight_g`   | integer   | Weight of the product in grams.               |
| `product_length_cm`  | integer   | Product length measured in centimeters.       |
| `product_height_cm`  | integer   | Product height measured in centimeters.       |
| `product_width_cm`   | integer   | Product width measured in centimeters.        |


### Business Meaning

* One row represents one product listing.
* Reference/dimension data.
* Used to enrich order_items table.

### Data Quality Observations

* Some nulls in product_category_name.
* Some products missing physical dimension info.
* Numerical attributes should be non-negative.
* Category names are in Portuguese.

### Grain

**One row per product.**

---

## Table: `product_category_name_translation`

### Structural Summary

* Rows: 71
* Columns: 2
* Primary Key: `product_category_name`

**Columns**

| Column Name                   | Data Type | Description                                      |
|-------------------------------|-----------|--------------------------------------------------|
| `product_category_name`       | string    | Original product category name in Portuguese.    |
| `product_category_name_english` | string  | Translated product category name in English.     |


### Business Meaning

* Lookup table translating product categories from Portuguese to English.
* Reference dimension table.

### Data Quality Observations

* Small static table.
* Should be fully unique on product_category_name.
* Should have no nulls.

### Grain

**One row per product category.**

---

## Overall Model Understanding

This is a **normalized transactional marketplace dataset** with:

* Header-level transactions (orders)
* Line-level detail (order_items)
* Financial events (order_payments)
* Dimension tables (customers, products, category translation)

The true analytical grain hierarchy is:

Customer
→ Order
→ Order Item
→ Payment

---

### Cross-Table Relationships (High-Level)

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

## Key Data Quality Signals to Monitor

* Duplicate primary keys
* Missing foreign key relationships
* Negative price or freight values
* Payment totals not matching item totals
* Invalid timestamp sequences
* Null-heavy product attributes
* Orphan products without category translation


---

## Key Engineering Implications

* Timestamp parsing must be handled explicitly due to mixed date formats.
* Null handling rules differ by table and column role.
* Fact tables (`orders`, `order_items`, `payments`) require careful grain preservation during joins.
* Dimension tables (`customers`, `products`) require normalization and data quality enforcement before downstream use.
