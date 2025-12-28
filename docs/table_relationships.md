# TABLE RELATIONSHIPS

---

# **Database Structure & Relationships**

## **Primary Keys**

| Sr No | Table        | Primary Key |
|-------|--------------|-------------|
| 1     | Customers    | customer_id |
| 2     | Orders       | order_id    |
| 3     | Products     | product_id  |
| 4     | Payments     | order_id    |
| 5     | Order Items  | order_id    |

---

## **Foreign Key Mapping**

| Table 1     | Table 2     | Foreign Key |
|-------------|-------------|-------------|
| Customers   | Orders      | customer_id |
| Customers   | Products    | None        |
| Customers   | Payments    | None        |
| Customers   | Order Items | None        |
| Orders      | Customers   | customer_id |
| Orders      | Products    | None        |
| Orders      | Payments    | order_id    |
| Orders      | Order Items | order_id    |
| Products    | Customers   | None        |
| Products    | Orders      | None        |
| Products    | Payments    | None        |
| Products    | Order Items | product_id  |
| Payments    | Customers   | None        |
| Payments    | Orders      | order_id    |
| Payments    | Products    | None        |
| Payments    | Order Items | order_id    |
| Order Items | Customers   | None        |
| Order Items | Orders      | order_id    |
| Order Items | Products    | product_id  |
| Order Items | Payments    | order_id    |

---

## **Cardinality**

1. **One Customer → One Order**  
2. **One Order → Many Products**  
3. **One Order → One Payment**  
4. **One Order → One Order Item**

---

## **Table Roles**

| Table       | Type       |
|-------------|------------|
| customers   | Dimension  |
| products    | Dimension  |
| orders      | Fact       |
| order_items | Fact       |
| payments    | Fact       |

---
