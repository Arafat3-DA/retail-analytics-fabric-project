# 🛒 Retail Analytics — Microsoft Fabric
> End-to-end data engineering and analytics project built on Microsoft Fabric using Medallion Architecture (Bronze → Silver → Gold), Apache Spark transformations, and Power BI dashboards.

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-Microsoft%20Fabric-purple)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📌 Project Overview

This project simulates a real-world retail analytics pipeline for a multi-store retail business operating across multiple countries. The goal is to transform raw transactional data into clean, structured, and insightful Power BI dashboards — following industry-standard data engineering practices.

The project is built entirely on **Microsoft Fabric**, using a **Medallion Architecture** that progressively refines data through three layers:

```
Raw Files → 🥉 Bronze Layer → 🥈 Silver Layer → 🥇 Gold Layer → 📊 Power BI
```

---

## 🎯 Business Questions This Project Answers

- What is the total revenue and gross profit by store, region, and country?
- Which products are top performers and which are underperforming?
- How do sales trends look month-over-month and year-over-year?
- What is the impact of currency exchange on international revenue?
- Which customer segments drive the most value?
- How do order volumes and fulfillment rates vary across stores?

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Microsoft Fabric                         │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │   🥉 BRONZE │───▶│  🥈 SILVER  │───▶│   🥇 GOLD   │   │
│  │  Raw Files   │    │  Cleaned &   │    │  Aggregated  │   │
│  │  (as-is)     │    │  Typed Data  │    │  Business    │   │
│  │              │    │              │    │  Models      │   │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
│                                                    │        │
│                                                    ▼        │
│                                           ┌──────────────┐  │
│                                           │   Power BI   │  │
│                                           │  Dashboards  │  │
│                                           └──────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Purpose | Format | Tool |
|---|---|---|---|
| **Bronze** | Store raw data exactly as received — no changes | Parquet | Fabric Lakehouse |
| **Silver** | Clean, cast types, remove nulls, add derived columns | Delta Tables | Spark Notebooks |
| **Gold** | Business-ready aggregations and fact/dim models | Delta Tables | Spark Notebooks |
| **Reporting** | Interactive dashboards and KPIs | Power BI | Power BI Desktop |

---

## 📦 Dataset

- **Size:** 59+ million rows of retail transactional data across 8 integrated tables
- **Source:** Loaded into Microsoft Fabric Lakehouse (Bronze layer)
- **Storage:** Raw_Data_Bronze folder in Lakehouse_Bronze

### Tables

| Table | Description | Type |
|---|---|---|
| `sales` | Transactional sales records | Fact |
| `orders` | Order header information | Fact |
| `orderrows` | Line-item detail per order | Fact |
| `customer` | Customer master data | Dimension |
| `product` | Product catalog | Dimension |
| `store` | Store locations and metadata | Dimension |
| `date` | Date dimension for time intelligence | Dimension |
| `currencyexchange` | Exchange rates for multi-currency support | Reference |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **Microsoft Fabric** | Unified analytics platform |
| **Lakehouse** | Data storage (Bronze, Silver, Gold) |
| **Apache Spark / PySpark** | Data transformation engine |
| **Spark SQL** | SQL-based transformations in notebooks |
| **Delta Lake** | Table format for ACID-compliant storage |
| **Power BI** | Business intelligence and dashboards |
| **Python** | Notebook scripting language |
| **GitHub** | Version control and documentation |

---

## 📁 Repository Structure

```
retail-analytics-fabric-project/
│
├── 📂 notebooks/               # Fabric notebook exports (.ipynb)
│   └── silver_transformation.ipynb
│
├── 📂 docs/                    # Architecture notes and transformation logic
│   ├── architecture.md
│   ├── bronze_to_silver.md
│   └── data_dictionary.md
│
├── 📂 screenshots/             # Progress screenshots from Fabric and Power BI
│   ├── Lakehouse_bronze/
│   │   └── Lakehouse_bronze.png
│   └── Lakehouse_silver/
│       └── Lakehouse_silver.png
│
├── LICENSE                     # MIT License
└── README.md                   # This file
```

---

## 🔄 Transformation Logic

### Bronze → Silver ✅ Complete
- Cast all columns to correct data types
- Filter NULL records on all critical columns
- Rename ambiguous columns — DT → OrderDate, Exchange → ExchangeRate
- Combine GivenName + MiddleInitial + Surname → FullName
- Drop irrelevant columns — Vehicle, Latitude, Longitude
- Add derived columns:
  - `SalesAmount = ROUND(Quantity × UnitPrice, 2)`
  - `GrossProfit = ROUND(SalesAmount - (Quantity × UnitCost), 2)`
- Write clean Delta tables to Lakehouse_Silver
- Total: 59,114,388 rows — zero data loss

### Silver → Gold 🔄 In Progress
- Build star schema — fact_sales and dimension tables
- Apply currency conversion — SalesAmount in USD
- Pre-aggregate KPIs for Power BI Direct Lake mode
- Write Gold tables to Lakehouse_Gold

### Gold → Power BI ⏳ Planned
- Connect Power BI via Direct Lake mode
- Build 5 dashboard pages
- Implement DAX measures and Row Level Security

---

## 📊 Power BI Dashboards *(upcoming)*

Planned dashboard pages:

- **Executive Summary** — Revenue, Profit, Orders KPIs
- **Sales Trends** — Monthly and yearly trend lines
- **Product Performance** — Top/bottom products by revenue and margin
- **Store Analysis** — Store-level comparison by region and country
- **Customer Insights** — Segmentation and lifetime value

---

## 📅 Project Progress

| Week | Focus | Status |
|---|---|---|
| Week 1 | Setup, GitHub, Bronze → Silver transformations | ✅ Complete |
| Week 2 | Silver → Gold aggregations | 🔄 In Progress |
| Week 3 | Power BI dashboard development | ⏳ Planned |
| Week 4 | Final testing, documentation, LinkedIn posts | ⏳ Planned |

---

## 🔗 Follow the Build

This project is being built in public with weekly updates on LinkedIn.

**Author:** Arafat Siddiqui

**Email:** arafatsiddiqui3@gmail.com

**LinkedIn:** [Arafat Siddiqui](https://www.linkedin.com/in/arafat-siddiqui/)

**Portfolio:** [arafat3-portfolio.netlify.app](https://arafat3-portfolio.netlify.app/)

**GitHub:** [Arafat3-DA](https://github.com/Arafat3-DA)

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
