# DATA QUALITY RULES

---

# **Customer Table**

### **Keys**
- **Primary key:** `customer_id`  
- **Foreign key:** `customer_id` is referenced by **Orders** table  

### **Columns**

| Column                   | Rule              | Reason                                                   |
|--------------------------|-------------------|----------------------------------------------------------|
| `customer_id`            | string, non-null  | Primary key                                              |
| `customer_zip_code_prefix` | int, non-null     | Used for geographic aggregation and enrichment           |
| `customer_city`          | string, non-null  | Enables city‑level analysis and geo joins                |
| `customer_state`         | string, non-null  | Enables state‑level analysis and reporting               |

---

# **Order Items Table**

### **Keys**
- **Primary key:** `order_id`  
- **Foreign keys:**  
  - `order_id` → Orders, Payments  
  - `product_id` → Products  

### **Columns**

| Column            | Rule                   | Reason                                      |
|-------------------|------------------------|---------------------------------------------|
| `order_id`        | string, non-null       | Primary key; FK to Orders & Payments        |
| `product_id`      | string, non-null       | Foreign key to Products                     |
| `seller_id`       | string, non-null       | Identifies seller fulfilling the item       |
| `price`           | decimal(10,2), non-null | Monetary value                              |
| `shipping_charges`| decimal(10,2), non-null | Monetary value                              |

---

# **Orders Table**

### **Keys**
- **Primary key:** `order_id`  
- **Foreign keys:**  
  - `order_id` → Payments  
  - `customer_id` → Customers  

### **Columns**

| Column                        | Rule                    | Reason                                      |
|-------------------------------|-------------------------|---------------------------------------------|
| `order_id`                    | string, non-null        | Primary key; FK to Payments                 |
| `customer_id`                 | string, non-null        | Foreign key to Customers                    |
| `order_status`                | string (enum)           | Tracks order lifecycle stage                |
| `order_purchase_timestamp`    | timestamp, non-null     | Order creation event                        |
| `order_approved_at`           | timestamp, nullable     | Approval may not occur immediately          |
| `order_delivered_timestamp`   | timestamp, nullable     | Not all orders are delivered                |
| `order_estimated_delivery_date` | timestamp, nullable   | Used to measure delivery delays             |

---

# **Payments Table**

### **Keys**
- **Primary key:** `order_id`  
- **Foreign key:** `order_id` → Orders, Order Items  

### **Columns**

| Column               | Rule                      | Reason                                      |
|----------------------|---------------------------|---------------------------------------------|
| `order_id`           | string, non-null          | Primary key; FK to Orders & Order Items     |
| `payment_sequential` | int, non-null             | Distinguishes multiple payments per order   |
| `payment_type`       | string (enum), non-null   | Limited set of payment methods              |
| `payment_installments` | integer ≥ 1             | Installments must be positive               |
| `payment_value`      | decimal(10,2) ≥ 0         | Monetary; no negative payments              |

---

# **Products Table**

### **Keys**
- **Primary key:** `product_id`  
- **Foreign key:** `product_id` → Order Items  

### **Columns**

| Column                 | Rule                  | Reason                                      |
|------------------------|-----------------------|---------------------------------------------|
| `product_id`           | string, non-null      | Primary key; FK to Order Items              |
| `product_category_name`| string (enum), nullable | Category may be missing                    |
| `product_weight_g`     | int, nullable         | Physical attribute, not always provided                          |
| `product_length_cm`    | int, nullable         | Used for shipping calculations              |
| `product_height_cm`    | int, nullable         | Used for shipping calculations              |
| `product_width_cm`     | int, nullable         | Used for shipping calculations              |

---
