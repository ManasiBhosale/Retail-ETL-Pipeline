# DATA QUALITY RULES

---

## **Customers Table**

#### **Keys**

* **Primary key:** `customer_id`
* **Foreign key:** `customer_id` is referenced by **Orders** table

#### **Columns**

| Column                     | Rule             | Reason                                         |
| -------------------------- | ---------------- | ---------------------------------------------- |
| `customer_id`              | string, non-null | Primary key                                    |
| `customer_unique_id`       | string, non-null | Unique identifier for each customer record     |
| `customer_zip_code_prefix` | int, non-null    | Used for geographic aggregation and enrichment |
| `customer_city`            | string, non-null | Enables city‑level analysis and geo joins      |
| `customer_state`           | string, non-null | Enables state‑level analysis and reporting     |

---

## **Orders Table**

#### **Keys**

* **Primary key:** `order_id`
* **Foreign keys:**

  * `customer_id` → Customers
  * `order_id` → Order Items, Payments

#### **Columns**

| Column                          | Rule                    | Reason                                    |
| ------------------------------- | ----------------------- | ----------------------------------------- |
| `order_id`                      | string, non-null        | Primary key; FK to Order Items & Payments |
| `customer_id`                   | string, non-null        | Foreign key to Customers                  |
| `order_status`                  | string (enum), non-null | Tracks order lifecycle stage              |
| `order_purchase_timestamp`      | timestamp, non-null     | Order creation event                      |
| `order_approved_at`             | timestamp, nullable     | Approval may not occur immediately        |
| `order_delivered_carrier_date`  | timestamp, nullable     | Carrier delivery date                     |
| `order_delivered_customer_date` | timestamp, nullable     | Customer delivery date                    |
| `order_estimated_delivery_date` | timestamp, nullable     | Used to measure delivery delays           |

---

## **Order Items Table**

#### **Keys**

* **Primary key:** `order_id, order_item_id`
* **Foreign keys:**

  * `order_id` → Orders
  * `product_id` → Products

#### **Columns**

| Column          | Rule                    | Reason                                 |
| --------------- | ----------------------- | -------------------------------------- |
| `order_id`      | string, non-null        | PK component; FK to Orders             |
| `order_item_id` | int, non-null           | PK component; identifies item sequence |
| `product_id`    | string, non-null        | Foreign key to Products                |
| `seller_id`     | string, non-null        | Identifies seller fulfilling the item  |
| `price`         | decimal(10,2), non-null | Monetary value                         |
| `freight_value` | decimal(10,2), non-null | Shipping charges                       |

---

## **Payments Table**

#### **Keys**

* **Primary key:** `order_id, payment_sequential`
* **Foreign key:** `order_id` → Orders

#### **Columns**

| Column                 | Rule                    | Reason                                                  |
| ---------------------- | ----------------------- | ------------------------------------------------------- |
| `order_id`             | string, non-null        | PK component; FK to Orders                              |
| `payment_sequential`   | int, non-null           | PK component; distinguishes multiple payments per order |
| `payment_type`         | string (enum), non-null | Limited set of payment methods                          |
| `payment_installments` | int ≥ 1, non-null       | Installments must be positive                           |
| `payment_value`        | decimal(10,2) ≥ 0       | Monetary; no negative payments                          |

---

## **Products Table**

#### **Keys**

* **Primary key:** `product_id`
* **Foreign key:** `product_id` → Order Items

#### **Columns**

| Column                  | Rule                    | Reason                                  |
| ----------------------- | ----------------------- | --------------------------------------- |
| `product_id`            | string, non-null        | Primary key; FK to Order Items          |
| `product_category_name` | string (enum), nullable | Category may be missing                 |
| `product_weight_g`      | int, nullable           | Physical attribute, not always provided |
| `product_length_cm`     | int, nullable           | Used for shipping calculations          |
| `product_height_cm`     | int, nullable           | Used for shipping calculations          |
| `product_width_cm`      | int, nullable           | Used for shipping calculations          |

---

## **Product Category Name Translation Table**

#### **Keys**

* **Primary key:** `product_category_name`
* **Foreign key:** `product_category_name` → Products

#### **Columns**

| Column                          | Rule             | Reason                                 |
| ------------------------------- | ---------------- | -------------------------------------- |
| `product_category_name`         | string, non-null | Primary key; used for translation      |
| `product_category_name_english` | string, non-null | Standardized English name for analysis |

---

