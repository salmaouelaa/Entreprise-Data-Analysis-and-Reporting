# Retail Sales Intelligence | SQL Server & Power BI

**End-to-end analytical reporting from retail transactions to decision-ready sales, customer, and product insights.**

**Microsoft SQL Server · T-SQL · Power BI · DAX · Dimensional Modeling · Sales Analytics · Customer Segmentation · KPI Reporting**

> **Project outcome:** Built a reusable SQL analytics layer and an interactive Power BI executive dashboard to monitor revenue, order activity, average order value, year-over-year performance, monthly sales trends, and product-category contribution.

## Executive overview

Retail sales data is only useful when decision-makers can translate transactions into consistent performance indicators. This project turns an existing retail data warehouse's **Gold-layer** sales, customer, and product tables into a reporting solution that supports both **executive monitoring** and **deeper customer/product analysis**.

The solution combines:

- **SQL analytics:** reusable queries and reporting views for sales trends, customer behavior, segmentation, and product performance.
- **Power BI semantic modeling:** a star-schema-style model with a dedicated calendar dimension, explicit relationships, and reusable DAX measures.
- **Interactive reporting:** an Executive Sales Overview page with KPI cards, time-series analysis, category contribution, and Year/Country slicers.

**Scope:** This repository demonstrates the analytics and BI reporting layers. The underlying Gold dataset is provided; this project does **not** claim to implement an upstream production ETL/ELT pipeline or to deliver measured commercial uplift.

## Business questions answered

| Business question | Analytical approach | Decision-making value |
| --- | --- | --- |
| How much revenue are we generating? | Total Revenue, Total Orders, Average Order Value | Track commercial performance using consistent KPIs |
| How are sales evolving over time? | Monthly revenue trend and previous-year comparison | Spot changes in sales momentum and investigate anomalies |
| Which product categories contribute most? | Category-level revenue breakdown and contribution analysis | Understand revenue concentration and product mix |
| Which customers are most valuable? | Customer-level spend, orders, recency, and behavioral segments | Support customer prioritization and retention analysis |
| Which products outperform or underperform? | Product revenue tiers and historical comparisons | Inform product portfolio and performance reviews |

## Power BI | Executive Sales Overview

The Power BI report brings core sales indicators into a single interactive view.

**Executive KPI layer**

| KPI | Definition | Why it matters |
| --- | --- | --- |
| **Total Revenue** | Sum of sales amount | Overall sales performance |
| **Total Orders** | Distinct count of order numbers | Transaction volume without double-counting order lines |
| **Average Order Value (AOV)** | Total Revenue / Total Orders | Revenue generated per order |
| **Revenue Growth YoY %** | (Current-period revenue − comparable prior-year revenue) / prior-year revenue | Direction and magnitude of annual change |

**Analytical visuals**

- **Monthly Revenue Evolution:** a line chart for monitoring sales over time, sorted chronologically.
- **Revenue by Product Category:** a horizontal bar chart for comparing category contribution.
- **Year and Country slicers:** interactive filtering across the executive metrics and visuals.

**Reporting improvements delivered:** standardized DAX definitions, a dedicated date table for time intelligence, clear fact/dimension relationships, consolidated KPI presentation, and interactive geographic/time filtering. These are **reporting and usability improvements**, not measured increases in revenue or productivity.

**Power BI report:** [`Sales_Performance_Dashboard.pbix`](reports/Sales_Performance_Dashboard.pbix) *(place your final `.pbix` file at this path before publishing).*

<!-- Optional: after adding a clean exported screenshot, uncomment:
![Executive Sales Overview](docs/sales_dashboard_preview.png)
-->

### DAX measures

The model uses explicit measures to keep metric definitions consistent across visuals and filters:

```dax
Total Revenue = SUM('gold fact_sales'[sales_amount])

Total Orders = DISTINCTCOUNT('gold fact_sales'[order_number])

Average Order Value = DIVIDE([Total Revenue], [Total Orders])

Revenue PY =
CALCULATE(
    [Total Revenue],
    SAMEPERIODLASTYEAR(DimDate[Date])
)

Revenue Growth YoY % =
DIVIDE(
    [Total Revenue] - [Revenue PY],
    [Revenue PY]
)
```

Other measures include **Total Customers** and **Total Quantity Sold**. Table names in DAX reflect the Power BI model and may differ from the SQL schema notation.

> **Interpretation note:** YoY measures depend on the selected date context and available prior-year records. The executive growth card should be validated against a Year / Revenue / Revenue PY comparison table before treating its displayed value as a verified business finding.

## Data architecture

```text
Existing retail Gold-layer data (SQL Server)
                |
                v
     +-----------------------+
     | gold.fact_sales       |<---- gold.dim_customers
     | transactions & KPIs   |<---- gold.dim_products
     +-----------------------+
                |
                +------> T-SQL exploratory analysis
                |        +--> gold.report_customers
                |        +--> gold.report_products
                |
                v
       Power BI (Import mode)
                |
      Star schema + DimDate
                |
       Reusable DAX measures
                |
     Executive Sales Overview
```

**Data model:** `gold.fact_sales` stores sales events; `gold.dim_customers` and `gold.dim_products` provide descriptive attributes. Power BI additionally uses a **DimDate** calendar table related to sales order dates. Customer and product reporting views are retained as separate analytical datasets rather than introducing unnecessary relationships into the core star schema.

## SQL analytics | Beyond the dashboard

### Sales trends and revenue contribution

- Revenue, active purchasing customers, and quantity by reporting period.
- Cumulative sales analysis and historical price benchmarks.
- Product-category share of total revenue.

**Scripts:** [`change_over_time_analysis.sql`](change_over_time_analysis.sql) · [`cumulative_analysis.sql`](cumulative_analysis.sql) · [`part_to_whole_analysis.sql`](part_to_whole_analysis.sql)

### Customer intelligence

The `gold.report_customers` view consolidates customer-level measures such as **total spend, orders, products purchased, recency, AOV, and average monthly spend**.

Behavioral segmentation uses project-defined rules:

| Segment | Rule |
| --- | --- |
| VIP | Observed purchase-history span ≥ 12 months and spend > 5,000 |
| Regular | Observed purchase-history span ≥ 12 months and spend ≤ 5,000 |
| New | Observed purchase-history span < 12 months |

These are descriptive segments, **not predictive churn scores**.

**Scripts:** [`data_segmentation_analysis.sql`](data_segmentation_analysis.sql) · [`report_customers.sql`](report_customers.sql)

### Product performance

The `gold.report_products` view supports product-level evaluation through **revenue, orders, customers, units, average selling price, and product performance tiers**. Historical comparison queries help flag products whose performance changes relative to prior observed periods.

**Scripts:** [`performance_analysis.sql`](performance_analysis.sql) · [`report_products.sql`](report_products.sql)

## Tools & skills demonstrated

| Area | Skills |
| --- | --- |
| **Business Intelligence** | KPI design, executive reporting, dashboard usability, interactive filtering |
| **Power BI** | Import mode, semantic modeling, relationships, slicers, card/line/bar visuals |
| **DAX** | `SUM`, `DISTINCTCOUNT`, `DIVIDE`, `CALCULATE`, `SAMEPERIODLASTYEAR`, filter context |
| **SQL Analytics** | T-SQL, CTEs, joins, aggregation, SQL views, window functions, `LAG` |
| **Data Modeling** | Fact and dimension tables, star-schema design, date dimension, grain awareness |
| **Business Analysis** | Sales trends, AOV, customer segmentation, product benchmarking, revenue mix |

## Repository structure

```text
.
├── 00_init_database.sql
├── change_over_time_analysis.sql
├── cumulative_analysis.sql
├── data_segmentation_analysis.sql
├── part_to_whole_analysis.sql
├── performance_analysis.sql
├── report_customers.sql
├── report_products.sql
├── reports/
│   └── Sales_Performance_Dashboard.pbix    # Add final Power BI report
├── docs/
│   └── sales_dashboard_preview.png         # Optional dashboard export
└── README.md
```

> If your SQL scripts are under `scripts/` in GitHub, update the links and tree to match the published repository.

## How to run

1. Open SQL Server and review `00_init_database.sql`. **Warning:** the initialization script drops and recreates `DataWarehouseAnalytics`; do not run it against a database you need to preserve.
2. Update the local `BULK INSERT` CSV paths and load the Gold tables.
3. Run the analytical SQL scripts and create the customer/product reporting views.
4. Open `reports/Sales_Performance_Dashboard.pbix` in Power BI Desktop.
5. In **Transform data → Data source settings**, point the report to your SQL Server instance, authenticate, and refresh.
6. Use the **Year** and **Country** slicers to explore the Executive Sales Overview.

**Requirements:** SQL Server 2022+ for scripts using `DATETRUNC`, a SQL client, the source CSV datasets, and Power BI Desktop.

## Validation & roadmap

**Validation still recommended before publication:** reconcile Power BI revenue/order totals against SQL queries; inspect the YoY calculation by year and selected country; verify date coverage and currency; test slicer interactions; and review missing/duplicate keys.

**Known SQL cleanup items:** fix the missing comma in `report_customers.sql` if it remains in your repository; review annual-versus-monthly comments in `cumulative_analysis.sql`; and guard against integer division and gaps in previous-period comparisons.

**Potential next iterations:** dedicated **Customer Intelligence**, **Product Performance**, and **Sales Trends** Power BI pages; documented metric definitions; automated data-quality tests; refresh documentation; and verified insights with actual numbers and date ranges.

---

**Portfolio focus:** Data Analytics · Business Intelligence · Power BI · SQL Server · DAX · Sales Performance Reporting
