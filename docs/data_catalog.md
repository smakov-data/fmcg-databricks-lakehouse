# Data Catalog — Gold Layer

## Overview

The Gold Layer represents the semantic business model of the FMCG Lakehouse project.
All objects are optimized for analytical workloads, reporting, and BI consumption.

The model follows a classic star schema pattern consisting of dimension tables and a central fact table, exposed via a denormalized analytical view.

## 1. gold.dim_date

### Purpose

Calendar dimension providing standardized time attributes for reporting and trend analysis.

### Columns

| Column Name | Data Type | Description |
| --- | --- | --- |
| date_key | INT | Surrogate date key used for analytical joins |
| date | DATE | Calendar date |
| month_start_date | DATE | First day of the corresponding month |
| year | INT | Calendar year |
| quarter | INT | Calendar quarter |
| month_name | STRING | Full month name |
| month_short_name | STRING | Abbreviated month name |
| year_quarter | STRING | Year and quarter representation |

---

## 2. gold.dim_customers

### Purpose

Business dimension storing customer master data enriched with market, platform, and channel attributes.

### Columns

| Column Name | Data Type | Description |
| --- | --- | --- |
| customer_code | STRING | Natural business key of the customer |
| customer | STRING | Customer name |
| market | STRING | Market or region of the customer |
| platform | STRING | Sales platform |
| channel | STRING | Distribution or sales channel |

---

## 3. gold.dim_products

### Purpose

Product dimension providing standardized product hierarchy and classification attributes.

### Columns

| Column Name | Data Type | Description |
| --- | --- | --- |
| product_code | STRING | Natural business key of the product |
| division | STRING | High-level product division |
| category | STRING | Product category |
| product | STRING | Product name |
| variant | STRING | Product variant |

---

## 4. gold.dim_gross_price

### Purpose

Pricing dimension storing year-based product prices used for revenue calculation.

### Columns

| Column Name | Data Type | Description |
| --- | --- | --- |
| product_code | STRING | Product business key |
| year | INT | Pricing year |
| price_inr | DECIMAL | Gross price in INR |

---

## 5. gold.fact_orders

### Purpose

Core fact table representing aggregated daily sales transactions.
Stores numeric metrics and business keys linking to dimension tables.

### Grain

- Date × Product × Customer

### Columns

| Column Name | Data Type | Description |
| --- | --- | --- |
| date | DATE | Transaction date |
| product_code | STRING | Product business key |
| customer_code | STRING | Customer business key |
| sold_quantity | INT | Quantity of units sold |

---

## 6. gold.vw_fact_orders_enriched

### Purpose

Denormalized analytical view combining fact_orders with all related dimensions.
Designed for direct consumption by BI tools and analytical queries.

### Joins

- fact_orders → dim_date (by month_start_date)
- fact_orders → dim_customers
- fact_orders → dim_products
- fact_orders → dim_gross_price (by product and year)

### Metrics

| Metric Name | Description |
| --- | --- |
| sold_quantity | Units sold |
| price_inr | Unit price |
| total_amount_inr | Derived revenue metric (sold_quantity × price_inr) |

---

## Notes

- Gold Layer objects are implemented as Delta tables and views.
- Incremental loading logic is applied to fact tables.
- The Gold Layer exposes business-friendly, analytics-ready datasets.
