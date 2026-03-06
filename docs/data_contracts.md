# DATA CONTRACTS

## **1. Customers Table**

* **Layer:** Silver
* **Table Name:** `customers`

### Columns

| Column                     | Data Type | Nullable | Key Constraints | Example Values | Business Rules / Transformations               |
| -------------------------- | --------- | -------- | --------------- | -------------- | ---------------------------------------------- |
| `customer_id`              | string    | No       | PK              | "06b8999e2fba1a1fbc88172c00ba8bc7"       | Unique identifier for each customer            |
| `customer_unique_id`       | string    | No       | None            | "861eff4711a542e4b93843c6dd7febb0"       | Unique identifier for each customer            |
| `customer_zip_code_prefix` | integer   | No       | None            | 14409, 9790   | Used for geographic joins; standardized format |
| `customer_city`            | string    | No       | None            | "São Paulo"    | Trimmed, cleaned, no duplicates                |
| `customer_state`           | string    | No       | None            | "SP"           | Two-letter state code                          |

### Known Exceptions / Caveats

* Some city names may have inconsistent capitalization or accents; standardization applied.
* No missing values after silver-layer cleaning.

---

## **2. Orders Table**

* **Layer:** Silver 
* **Table Name:** `orders`

### Columns

| Column                          | Data Type | Nullable | Key Constraints | Example Values      | Business Rules / Transformations                    |
| ------------------------------- | --------- | -------- | --------------- | ------------------- | --------------------------------------------------- |
| `order_id`                      | string    | No       | PK              | "e481f51cbdc54678b7cc49136f2d6af7"            | Unique order identifier                             |
| `customer_id`                   | string    | No       | FK → customers  | "9ef432eb6251297304e76186b10a928d"            | Foreign key to `customers.customer_id`              |
| `order_status`                  | string    | No       | None            | "delivered"         | Enum with allowed statuses: delivered, shipped, etc |
| `order_purchase_timestamp`      | timestamp | No       | None            | 2017-03-05 14:00:00 | Standardized timestamp format                       |
| `order_approved_at`             | timestamp | Yes      | None            | 2017-03-05 15:00:00 | May be null if approval delayed                     |
| `order_delivered_carrier_date`     | timestamp | Yes      | None            | 2017-03-10 10:00:00 | Null if not delivered yet                           |
| `order_delivered_customer_date`     | timestamp | Yes      | None            | 2017-03-10 10:00:00 | Null if not delivered yet                           |
| `order_estimated_delivery_date` | timestamp | Yes      | None            | 2017-03-12 18:00:00 | Used for SLA / delay calculations                   |

### Known Exceptions / Caveats

* Rows missing `order_id`, `customer_id`, or `order_purchase_timestamp` are removed during silver layer cleaning.

---

## **3. Order Items Table**

* **Layer:** Silver
* **Table Name:** `order_items`

### Columns

| Column             | Data Type     | Nullable | Key Constraints | Example Values | Business Rules / Transformations              |
| ------------------ | ------------- | -------- | --------------- | -------------- | --------------------------------------------- |
| `order_id`         | string        | No       | FK → orders     | "00010242fe8c5a6d1ba2dd792cb16214"       | Foreign key to `orders.order_id`              |
| `order_item_id`         | string        | No       | None     | "1"       | Composite key with `order_items.order_id`              |
| `product_id`       | integer        | No       | FK → products   | "4244733e06e7ecb4970a6e2683c13e61"       | Foreign key to `products.product_id`          |
| `seller_id`        | string        | No       | None            | "48436dade18ac8b2bce089ec2a041202"       | Identifier for the seller fulfilling the item |
| `price`            | decimal(10,2) | No       | None            | 120.50         | Standardized monetary value                   |
| `shipping_charges` | decimal(10,2) | No       | None            | 15.90          | Standardized monetary value                   |

### Known Exceptions / Caveats

* Negative prices or shipping charges are flagged during quality checks.

---

## **4. Products Table**

* **Layer:** Silver
* **Table Name:** `products`

### Columns

| Column                  | Data Type | Nullable | Key Constraints | Example Values   | Business Rules / Transformations                  |
| ----------------------- | --------- | -------- | --------------- | ---------------- | ------------------------------------------------- |
| `product_id`            | string    | No       | PK              | "1e9e8ef04dbcff4541ed26657ea517e5"         | Unique identifier for each product                |
| `product_category_name` | string    | Yes      | None            | "bed_bath_table" | Nullable; maps to `product_category_name_english` |
| `product_weight_g`      | int       | Yes      | None            | 500              | Nullable; used for shipping calculations          |
| `product_length_cm`     | int       | Yes      | None            | 25               | Used for shipping dimensions                      |
| `product_height_cm`     | int       | Yes      | None            | 10               | Used for shipping dimensions                      |
| `product_width_cm`      | int       | Yes      | None            | 15               | Used for shipping dimensions                      |

### Known Exceptions / Caveats

* Some category names may be missing.
* Original category names are in Portuguese; mapped using translation table.

---

## **5. Payments Table**

* **Layer:** Silver
* **Table Name:** `payments`

### Columns

| Column                 | Data Type     | Nullable | Key Constraints          | Example Values | Business Rules / Transformations              |
| ---------------------- | ------------- | -------- | ------------------------ | -------------- | --------------------------------------------- |
| `order_id`             | string        | No       | FK → orders, order_items | "b81ef226f3fe1789b1e8b2acac839d17"       | Ensures payments are linked to valid orders   |
| `payment_sequential`   | int           | No       | None                     | 1              | Identifies sequential payments per order      |
| `payment_type`         | string        | No       | None                     | "credit_card"  | Enum: credit_card, boleto, voucher, etc       |
| `payment_installments` | int           | No       | None                     | 5              | Must be ≥1; represents number of installments |
| `payment_value`        | decimal(10,2) | No       | None                     | 120.50         | Monetary; must be ≥0                          |

### Known Exceptions / Caveats

* Missing payments are flagged via referential integrity checks.

---

## **6. Product Category Translation Table**

* **Layer:** Gold
* **Table Name:** `product_category_reference`

### Columns

| Column                          | Data Type | Nullable | Key Constraints | Example Values    | Business Rules / Transformations  |
| ------------------------------- | --------- | -------- | --------------- | ----------------- | --------------------------------- |
| `product_category_name`         | string    | No       | PK              | "cama_mesa_banho" | Original Portuguese category name |
| `product_category_name_english` | string    | No       | None            | "bed_bath_table"  | Translated category for analytics |

### Known Exceptions / Caveats

* Some products may have missing categories.

---

## Notes / Conventions

* **Currency / Units:** All prices and shipping_charges are in Brazilian Real (BRL). Product weights in grams, dimensions in centimeters.
* **Referential Integrity:** All FK constraints are validated in silver layer, and orphan records are logged in metrics for monitoring.


