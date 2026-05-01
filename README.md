# Superstore-Sales-Profit-Analytics-Power-BI-Dashboard
This project is an end-to-end Business Intelligence dashboard built in Microsoft Power BI using the well-known Superstore dataset. It covers the full data analytics workflow — from raw data ingestion and cleaning, through data modeling, all the way to interactive visual storytelling.
# 📊 Superstore Sales & Profit Analytics — Power BI Dashboard

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Dataset](https://img.shields.io/badge/Dataset-Superstore-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

##  Project Description

This project is an end-to-end **Business Intelligence dashboard** built in **Microsoft Power BI** using the well-known **Superstore dataset**. It covers the full data analytics workflow — from raw data ingestion and cleaning, through data modeling, all the way to interactive visual storytelling.

The dashboard is designed to simulate a real-world retail analytics scenario, enabling business stakeholders to monitor sales performance, identify profitability drivers, track regional trends, and make data-informed decisions. It spans **four focused report pages**, each targeting a distinct analytical domain.

---

##  Objectives

- Provide a **360° view** of the business across revenue, profit, customers, and geography
- Enable **drill-down analysis** by region, product category, sub-category, segment, and time period
- Identify **top-performing customers and products** contributing most to profitability
- Uncover **underperforming areas** (e.g., loss-making sub-categories or regions) requiring strategic attention
- Track **monthly and yearly trends** to support forecasting and planning
- Demonstrate proficiency in the full Power BI development lifecycle for portfolio purposes

---

##  Dataset Description

| Property | Details |
|---|---|
| **Name** | Sample Superstore |
| **Source** | Tableau / Kaggle public dataset |
| **Domain** | Retail / E-commerce |
| **Records** | ~10,000 orders |
| **Time Range** | 2023 – 2027 |

### Key Fields

| Field | Description |
|---|---|
| `Order ID` | Unique order identifier |
| `Order Date / Ship Date` | Date dimensions for time intelligence |
| `Customer Name / ID` | Customer dimension |
| `Segment` | Consumer, Corporate, Home Office |
| `Region / State / City` | Geographic hierarchy |
| `Category / Sub-Category` | Product hierarchy |
| `Product Name` | Individual SKU |
| `Sales (Revenue)` | Gross revenue per line item |
| `Profit` | Net profit per line item |
| `Quantity` | Units sold |
| `Discount` | Discount applied |

---

##  Data Cleaning & Transformation

All data preparation was performed inside **Power Query Editor** before loading into the model.

### Steps Applied

1. **Removed duplicates** — Identified and dropped duplicate `Order ID` rows to ensure order-level accuracy.
2. **Handled null values** — Filled or removed missing entries in `Region`, `State`, and `City` columns; some nulls in `City` were retained as `(Blank)` for traceability.
3. **Corrected data types** — Enforced correct types: `Date` for order/ship dates, `Decimal` for Sales and Profit, `Integer` for Quantity.
4. **Standardized text fields** — Applied `Text.Trim` and `Text.Proper` to remove leading/trailing spaces and normalize casing in `Customer Name`, `Product Name`, and `Category`.
5. **Split date columns** — Extracted `Year`, `Month Number`, `Month Name`, and `Quarter` from `Order Date` for time-intelligence support.
6. **Calculated Profit Margin** — Added a custom column: `Profit Margin = Profit / Sales`.
7. **Filtered test/invalid records** — Removed rows where `Sales = 0` or `Quantity = 0` as these represented data entry errors.
8. **Renamed columns** — Standardized column names to PascalCase for consistency across the model (e.g., `ship date` → `ShipDate`).

---

##  Data Model — Star Schema

The data model follows a **Star Schema** design pattern, separating transactional facts from descriptive dimensions for optimal query performance.

```
                    ┌──────────────┐
                    │  DimDate     │
                    │  (Date Table)│
                    └──────┬───────┘
                           │
   ┌──────────────┐        │        ┌──────────────────┐
   │  DimCustomer │        │        │  DimProduct      │
   │  (Customer)  │        │        │  (Category/Sub)  │
   └──────┬───────┘        │        └────────┬─────────┘
          │                │                 │
          └────────────────┼─────────────────┘
                           │
                  ┌────────▼────────┐
                  │   FactOrders    │
                  │  (Order Lines)  │
                  └────────┬────────┘
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
   ┌──────▼───────┐ ┌──────▼───────┐ ┌──────▼───────┐
   │ DimGeography │ │ DimSegment   │ │ DimShipMode  │
   │(Region/State)│ │(Cust Segment)│ │(Shipping)    │
   └──────────────┘ └──────────────┘ └──────────────┘
```

### Tables

| Table | Type | Description |
|---|---|---|
| `FactOrders` | Fact | Core transactional table — one row per order line |
| `DimDate` | Dimension | Full date table with Year, Quarter, Month, Day attributes |
| `DimCustomer` | Dimension | Customer name, ID, segment |
| `DimProduct` | Dimension | Product name, category, sub-category |
| `DimGeography` | Dimension | Country, region, state, city |
| `DimSegment` | Dimension | Consumer, Corporate, Home Office |

All relationships are **one-to-many**, with the dimension table on the "one" side and `FactOrders` on the "many" side. Cross-filter direction is set to **Single** to maintain performance and prevent ambiguity.

---

##  DAX Calculations

### Measures

```dax
-- Total Revenue
Total Revenue = SUM(FactOrders[Sales])

-- Total Profit
Total Profit = SUM(FactOrders[Profit])

-- Profit Margin %
Profit Margin % = DIVIDE([Total Profit], [Total Revenue], 0)

-- Total Orders
Total Orders = DISTINCTCOUNT(FactOrders[Order ID])

-- Average Order Value
Avg Order Value = DIVIDE([Total Revenue], [Total Orders], 0)

-- YoY Revenue Growth
YoY Revenue Growth =
VAR CurrentYear = [Total Revenue]
VAR PreviousYear =
    CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(DimDate[Date]))
RETURN
    DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)

-- Revenue Goal Achievement %
Revenue vs Goal % =
DIVIDE([Total Revenue], [Revenue Goal], 0)

-- Cumulative Revenue (Running Total)
Cumulative Revenue =
CALCULATE(
    [Total Revenue],
    FILTER(
        ALLSELECTED(DimDate[Date]),
        DimDate[Date] <= MAX(DimDate[Date])
    )
)
```

### Calculated Columns

```dax
-- Profit Category (in FactOrders table)
Profit Category =
IF(FactOrders[Profit] > 0, "Profitable", "Loss")

-- Discount Band
Discount Band =
SWITCH(
    TRUE(),
    FactOrders[Discount] = 0,        "No Discount",
    FactOrders[Discount] <= 0.2,     "Low (≤20%)",
    FactOrders[Discount] <= 0.4,     "Medium (21–40%)",
    "High (>40%)"
)

-- Days to Ship
Days to Ship =
DATEDIFF(FactOrders[OrderDate], FactOrders[ShipDate], DAY)
```

---

##  Dashboard Pages Overview

### Page 1 — Executive Overview

> *High-level KPIs and geographic distribution*

This page serves as the **landing page** for executives and senior stakeholders. It opens with KPI cards for **Total Revenue (1.12M)** and **Total Profit (106.94K)**, providing an immediate snapshot of business health.

**Visuals included:**
- **Bar chart** — Sum of Profit and Revenue by Top Customers (Adrian Barton, Hunter Lopez, Tom Boeckenhauer, etc.)
- **Pie chart** — Profit distribution by City (Toronto 55.08%, Blank 35.54%, Houston 9.38%)
- **Treemap** — Revenue and Year by State/Province, showing California, Texas, Pennsylvania, Arizona, and more as dominant states
- **Goal KPI visual** — Revenue vs. target: 257.47K actual vs. 10.03K goal (+2,467.78% achievement)

**Key takeaway:** Toronto drives the majority of city-level profit; California and Texas are the top revenue-generating states.

---

### Page 2 — Product & Category Performance

> *Profitability breakdown by sub-category, segment, and time*

This page gives product managers and category heads a granular view of which product lines are driving or draining profitability.

**Visuals included:**
- **Stacked bar chart** — Profit and Revenue by Region (East and Central compared to Blank region)
- **Donut chart** — Profit by Ship Date Year (2023–2027), showing 2026 leading at 34.46% and 2025 at 31.84%
- **Small multiples (100% stacked bars)** — Revenue and Profit by Month and Sub-Category for Bookcases, Chairs, Envelopes, and Fasteners
- **Area/line chart** — Quantity and Profit by Sub-Category across 17 sub-categories (Fasteners → Copiers), revealing that Tables and Bookcases dip into negative profit territory

**Key takeaway:** Tables and Bookcases consistently generate losses — likely due to high discounting. Fasteners and Binders are lean, profitable sub-categories.

---

### Page 3 — Sales Trends & Segment Analysis

> *Monthly trends, segment revenue share, and top products*

This page is designed for sales teams and marketing analysts tracking performance over time.

**Visuals included:**
- **Horizontal bar chart** — Top Sub-Categories by Revenue (Chairs dominating at ~0.3M, followed by Tables, Bookcases)
- **Donut chart** — Revenue Share by Segment: Consumer 51.95% (582.16K), Corporate 32.11% (359.82K), Home Office 15.94% (178.62K)
- **Bar chart** — Top Products by Profit (Canon imageCLASS leads at ~5K, followed by Fellowes PB and GBC DocuBind)
- **Area line chart** — Monthly Revenue, Profit, and Sales Trend (Month 1–12), showing clear seasonality peaks in months 8–9 and 11–12

**Key takeaway:** The Consumer segment generates the majority of revenue. Revenue spikes in Q3 and Q4 suggest strong holiday-season and back-to-school purchasing cycles.

---

### Page 4 — Profitability Deep Dive

> *Segment waterfall, sub-category profit breakdown, and city trend lines*

This analytical page supports finance teams and business analysts in diagnosing where profit is being made and lost.

**Visuals included:**
- **KPI cards** — Total Profit: **106.94K** | Total Revenue: **1.12M**
- **Horizontal bar chart** — Profit by Sub-Category and Category: Chairs (Technology) leads at ~25K; Furnishings shows slight loss
- **Waterfall chart** — Profit by Segment: Consumer contributes ~55K, Corporate adds ~35K, Home Office ~17K, totaling 106.94K
- **Multi-line chart** — Profit Trend by Month and City (Blank, Houston, Toronto): Toronto peaks in November/December; Houston shows gradual decline toward year-end

**Key takeaway:** Home Office, while the smallest segment by revenue, contributes a disproportionately healthy profit margin. Toronto profit peaked in November/December, suggesting strong seasonal demand in that market.

---

##  Key Insights & Findings

| # | Insight | Business Implication |
|---|---|---|
| 1 | **Toronto contributes 55% of city-level profit** | Prioritize supply chain reliability and marketing investment in Toronto |
| 2 | **Tables sub-category is loss-making** | Review discount strategy — high discounting is likely eroding margin |
| 3 | **Consumer segment = 52% of total revenue** | Retention programs for consumer customers can significantly protect revenue |
| 4 | **Q4 revenue spikes ~50% above Q2 average** | Stock inventory and staff up ahead of Q4 to capture seasonal demand |
| 5 | **Canon imageCLASS is the most profitable single product** | Ensure consistent availability and consider bundling strategies |
| 6 | **2026 accounts for the highest profit year share at 34.46%** | Business is in a growth phase — projections should reflect upward trajectory |
| 7 | **Profit Margin is ~9.5% overall** | Healthy but below industry leaders — pricing optimization could add 2–3% |
| 8 | **Chairs drive the most revenue and profit among sub-categories** | Chairs should be a focal point for upselling and promotions |

---

##  Tools & Technologies

| Tool | Purpose |
|---|---|
| **Microsoft Power BI Desktop** | Dashboard development, visualizations |
| **Power Query (M Language)** | Data cleaning and transformation |
| **DAX (Data Analysis Expressions)** | Measures, KPIs, calculated columns |
| **Star Schema Modeling** | Data model architecture |
| **Microsoft Excel / CSV** | Source data format |
| **GitHub** | Version control and portfolio hosting |

---

## 📸 Screenshots

### Page 1 — Executive Overview
<img width="1485" height="814" alt="Screenshot 2026-05-01 210115" src="https://github.com/user-attachments/assets/b373de58-734d-41c7-bb9b-841c97dd5c6d" />

### Page 2 — Product & Category Performance
<img width="1437" height="812" alt="Screenshot 2026-05-01 210226" src="https://github.com/user-attachments/assets/5371f886-e782-4e9a-9f7d-af039cc95129" />



### Page 3 — Sales Trends & Segment Analysis

<img width="1284" height="724" alt="Screenshot 2026-05-01 210029" src="https://github.com/user-attachments/assets/0e040405-4768-4f6a-9f4f-e9faf853c3c8" />



### Page 4 — Profitability Deep Dive
<img width="1284" height="724" alt="Screenshot 2026-05-01 210029" src="https://github.com/user-attachments/assets/ea61fb27-c4c2-485e-be2c-f999fb9f1a92" />





## 👤 Author

**[Shahd Usama]**
Data Analyst | Python · SQL · Machine Learning

[![[LinkedIn](https://www.linkedin.com/in/shahdusama/)](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin)]
[![[GitHub](https://github.com/ShahdUsama24)](https://img.shields.io/badge/GitHub-Follow-181717?style=flat&logo=github)]


---

> *This project was built as part of a data analytics portfolio to demonstrate end-to-end Power BI development skills — from raw data to actionable business insights.*

---

⭐ *If you found this project useful, please consider giving it a star on GitHub!*
