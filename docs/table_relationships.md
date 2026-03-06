# TABLE RELATIONSHIPS

---

# **Database Structure & Relationships**

## **Primary Keys**

| Sr No | Table                             | Primary Key                  |
| ----- | --------------------------------- | ---------------------------- |
| 1     | customers                         | customer_id                  |
| 2     | orders                            | order_id                     |
| 3     | products                          | product_id                   |
| 4     | order_payments                    | order_id, payment_sequential |
| 5     | order_items                       | order_id, order_item_id      |
| 6     | product_category_name_translation | product_category_name        |

---

## **Foreign Key Mapping**

| Table 1        | Table 2                           | Foreign Key           |
| -------------- | --------------------------------- | --------------------- |
| customers      | orders                            | customer_id           |
| customers      | products                          | None                  |
| customers      | order_payments                    | None                  |
| customers      | order_items                       | None                  |
| orders         | customers                         | customer_id           |
| orders         | products                          | None                  |
| orders         | order_payments                    | order_id              |
| orders         | order_items                       | order_id              |
| products       | customers                         | None                  |
| products       | orders                            | None                  |
| products       | order_payments                    | None                  |
| products       | order_items                       | product_id            |
| order_payments | customers                         | None                  |
| order_payments | orders                            | order_id              |
| order_payments | products                          | None                  |
| order_payments | order_items                       | order_id              |
| order_items    | customers                         | None                  |
| order_items    | orders                            | order_id              |
| order_items    | products                          | product_id            |
| order_items    | order_payments                    | order_id              |
| products       | product_category_name_translation | product_category_name |

---

## **Cardinality**

1. **One Customer → Many Orders**
2. **One Order → Many Order Items**
3. **One Order → Many Payments** (split payments possible)
4. **One Product → Many Order Items**
5. **One Product Category → Many Products**

---

## **Table Roles**

| Table                             | Type      |
| --------------------------------- | --------- |
| customers                         | Dimension |
| products                          | Dimension |
| product_category_name_translation | Dimension |
| orders                            | Fact      |
| order_items                       | Fact      |
| order_payments                    | Fact      |

---

