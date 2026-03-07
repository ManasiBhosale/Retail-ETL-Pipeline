# 🛒 Retail ETL Data Pipeline

An end-to-end **data engineering pipeline** that ingests, validates, and transforms retail e-commerce data into analytics-ready datasets using a **Medallion Architecture (Bronze → Silver → Gold)**.

The project demonstrates production-style **data quality validation, referential integrity enforcement, and modular ETL design** using PySpark and Delta Lake principles.



---

# 📊 Project Overview

Retail companies generate massive amounts of transactional data across orders, customers, products, and payments. However, raw data is often **incomplete, inconsistent, or unreliable**.

This project builds a robust ETL pipeline that:

1️⃣ Ingests raw retail datasets <br>
2️⃣ Cleans and standardizes the data <br>
3️⃣ Enforces data quality rules <br>
4️⃣ Validates cross-table relationships <br>
5️⃣ Produces analytics-ready datasets <br>

The pipeline ensures that downstream analytics and dashboards are built on **trusted, validated data**.

---

# 🏗 Architecture

The pipeline follows a **Medallion Data Architecture**, a common design pattern in modern data engineering systems.

<img src="https://github.com/ManasiBhosale/Retail-ETL-Pipeline/blob/c9b3666e11fe71398436e60f2d8f87fc707b8d5d/Images/Pipeline%20Architecture1.png" width="80%">
<p><em>Figure: Pipeline Architecture Diagram</em></p>



### Layer Responsibilities

| Layer                     | Purpose                                                     |
| ------------------------- | ----------------------------------------------------------- |
| **Bronze**                | Raw ingestion of source datasets                            |
| **Silver**                | Data cleaning, schema enforcement, and transformation       |
| **Quality Checks**        | Validation of completeness, format, ranges, and constraints |
| **Referential Integrity** | Validation of table relationships                           |
| **Gold**                  | Analytics-ready datasets for reporting                      |

This architecture is widely used in **modern lakehouse systems** such as Databricks and Delta Lake.

---

# 🧰 Tech Stack

| Technology               | Purpose                              |
| ------------------------ | ------------------------------------ |
| **Python**               | Exploratory Data Analysis (EDA)      |
| **PySpark**              | ETL data transformations             |
| **Delta Lake**           | Storage format and table management  |
| **Databricks Notebooks** | Development environment              |
| **SQL**                  | Data transformations and validations |
| **Power BI**             | Business Intelligence and dashboards |


---

# 🔍 Data Quality Framework

The pipeline includes a **configuration-driven data quality validation framework**.

Quality checks are defined through a **`dq_config` object**, allowing reusable and scalable validation rules across datasets.

## Supported Validations

| Check Type            | Description                          |
| --------------------- | ------------------------------------ |
| Null Checks           | Ensure required fields are populated |
| Format Checks         | Regex validation for IDs             |
| Range Checks          | Validate numeric ranges              |
| Enum Checks           | Validate allowed categorical values  |
| Date Range Checks     | Validate timestamp ranges            |
| Primary Key Checks    | Detect duplicate keys                |
| Referential Integrity | Detect orphan records                |

---

# 📈 Quality Monitoring

All validation notebooks produce a **metrics DataFrame**, which is consolidated into a centralized table:  `quality_metrics`


Metrics captured include:

| Metric              | Description               |
| ------------------- | ------------------------- |
| violation_count     | Number of rule violations |
| threshold           | Allowed threshold         |
| status              | Pass / Fail               |
| execution_timestamp | Time of check execution   |

This enables:

* automated monitoring
* pipeline observability
* data quality SLAs

<img src="https://github.com/ManasiBhosale/Retail-ETL-Pipeline/blob/c9b3666e11fe71398436e60f2d8f87fc707b8d5d/Images/Data%20Quality%20Monitoring.png" width="70%">
<p><em>Figure: Quality Metrics Snapshot</em></p>
---

# 🔗 Referential Integrity Validation

The pipeline validates critical relationships across datasets.

| Child Table | Parent Table | Key         |
| ----------- | ------------ | ----------- |
| orders      | customers    | customer_id |
| order_items | orders       | order_id    |
| payments    | orders       | order_id    |
| order_items | products     | product_id  |

Validation is performed using **left anti joins** to detect orphan records.

---

# 🗂 Data Model

The Gold Layer follows a **dimensional modelling approach** designed for analytics workloads.


<img src="https://github.com/ManasiBhosale/Retail-ETL-Pipeline/blob/57756fec241deb84a3370b942aacd385cdacd95e/Images/Data%20Model%20Diagram1.png" width="70%">
<p><em>Figure: Data Model Diagram</em></p>

### Core Entities

Dimension Tables:

- `dim_customers`
- `dim_products`

Fact Tables:

- `fact_orders`
- `fact_order_items`
- `fact_payments`

This schema enables efficient aggregation and BI reporting.

---

# 📊 Business Intelligence Report

To demonstrate the analytical value of the ETL pipeline, a **Power BI report** was built on top of the **Gold layer datasets**.

The report enables stakeholders to analyze **sales performance, customer behavior, product trends, and operational metrics** through interactive visualizations.

All reports include a **Year Slicer (2016, 2017, 2018)** allowing dynamic filtering across the report.

---

# 📊1 — Sales Performance

This report provides an overview of **overall business performance and sales metrics**.


<img src="https://github.com/ManasiBhosale/Retail-ETL-Pipeline/blob/cabfaf8542c2c2d98695779cfdff8c618137c131/Images/ETL-project_brazil_page-0001.jpg" width="80%">
<p><em>Figure: Sales report</em></p>

### Key KPIs

| Metric        | Description                |
| ------------- | -------------------------- |
| Revenue       | Total revenue generated    |
| Profit        | Net profit from all orders |
| Orders        | Total number of orders     |
| Shipping Cost | Total shipping expenses    |

Each KPI includes **comparison vs business targets**.

### Visualizations

| Visualization                 | Insight                                     |
| ----------------------------- | ------------------------------------------- |
| Revenue Over Time (Bar Chart) | Shows revenue growth trend                  |
| Profit by Region (Table)      | Displays revenue and profitability by state |
| Orders Delivered (Card)       | Total successfully delivered orders         |
| Order Completion Rate         | % of orders successfully completed          |
| Payment Success Rate          | % of successful payments                    |
| Average Order Value           | Average value per order                     |

---

# 👥 2 — Customer Analysis

This report analyzes **customer segmentation and geographic distribution**.


<img src="https://github.com/ManasiBhosale/Retail-ETL-Pipeline/blob/cabfaf8542c2c2d98695779cfdff8c618137c131/Images/ETL-project_brazil_page-0002.jpg" width="80%">
<p><em>Figure: Customer Analysis</em></p>


### Key Metrics

| Metric        | Description               |
| ------------- | ------------------------- |
| Customer Base | Total number of customers |

### Visualizations

| Visualization                            | Insight                                          |
| ---------------------------------------- | ------------------------------------------------ |
| Customer Type Distribution (Donut Chart) | Proportion of new vs returning customers         |
| Customer Type Performance                | Orders and revenue contribution by customer type |
| State-Level Order Value Map              | Geographic revenue distribution                  |
| City Distribution Map                    | Customer concentration by city                   |

This helps identify **high-value regions and customer segments**.

---

# 📦 3 — Product Analysis

This report focuses on **product category performance and logistics insights**.

<img src="https://github.com/ManasiBhosale/Retail-ETL-Pipeline/blob/cabfaf8542c2c2d98695779cfdff8c618137c131/Images/ETL-project_brazil_page-0003.jpg" width="80%">
<p><em>Figure: Product Analysis</em></p>

### Visualizations

| Visualization                     | Insight                                         |
| --------------------------------- | ----------------------------------------------- |
| Revenue by Category (Bar Chart)   | Top performing product categories               |
| Average Shipping Cost by Category | Logistics cost comparison                       |
| Value vs Volume by Product Weight | Relationship between product weight and revenue |
| Product Performance Table         | Category, sub-category, price, and revenue      |

These insights help optimize **product strategy and shipping efficiency**.

---

# 📑 4 — Order Analysis

This report analyzes **order trends and operational efficiency**.

<img src="https://github.com/ManasiBhosale/Retail-ETL-Pipeline/blob/cabfaf8542c2c2d98695779cfdff8c618137c131/Images/ETL-project_brazil_page-0004.jpg" width="80%">
<p><em>Figure: Order Analysis</em></p>

### Visualizations

| Visualization                   | Insight                             |
| ------------------------------- | ----------------------------------- |
| Quarterly Order Trend           | Seasonal order patterns             |
| Monthly Delivery Volume         | Delivery operations performance     |
| Order Trend by Month            | Order growth trends                 |
| Revenue Trend by Month          | Monthly revenue performance         |
| Revenue vs Last Month           | Month-over-month revenue comparison |
| Orders by Status (Funnel Chart) | Order pipeline stages               |

This enables monitoring of **order lifecycle performance and operational bottlenecks**.

---

# 🎯 Key Business Insights Enabled

The dashboards allow stakeholders to quickly answer questions such as:

* Which **regions generate the most profit?**
* Which **product categories drive the highest revenue?**
* What is the **customer distribution across cities and states?**
* Are **orders being delivered successfully?**
* How does **revenue change month-to-month?**

---

# 🚀 Running the Pipeline

Clone the repository:

```bash
git clone https://github.com/ManasiBhosale/Retail-ETL-Pipeline.git
````

Open the notebooks in **Databricks** or a compatible PySpark environment.

Execute the pipeline in order:

```
1. Bronze ingestion
2. Silver transformations
3. Data quality checks
4. Referential integrity validation
5. Gold layer creation
```

---
## 🙏 Credits

- **Dataset:** Brazilian E-Commerce Public Dataset by Olist ([Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce))
- **Platforms used:** [Databricks](https://www.databricks.com/) for ETL and data engineering, and [Power BI](https://powerbi.microsoft.com/) for dashboards
- **Dashboard theme inspiration:** [Superstore Sales Theme](https://community.fabric.microsoft.com/t5/Themes-Gallery/Superstore-Sales/td-p/3231442)

- **Icon Credits:**<br>
  <sub><em>(Below icons were used in the PowerBI report)</em></sub>
  
  <sub>
  <a href="https://www.flaticon.com/free-icons/whiteboard" title="whiteboard icons">Whiteboard icons created by Freepik - Flaticon</a><br>
  <a href="https://www.flaticon.com/free-icons/commerce-and-shopping" title="commerce and shopping icons">Commerce and shopping icons created by Freepik - Flaticon</a><br>
  <a href="https://www.flaticon.com/free-icons/graph" title="graph icons">Graph icons created by bukeicon - Flaticon</a><br>
  <a href="https://www.flaticon.com/free-icons/business-and-finance" title="business and finance icons">Business and finance icons created by bukeicon - Flaticon</a>
  </sub>
---
