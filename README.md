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
├── 📂 notebooks/               # All Fabric notebook exports (.ipynb)
│   ├── silver_transformation.ipynb
│   ├── gold_aggregation.ipynb
│   └── ...
│
├── 📂 docs/                    # Architecture notes and transformation logic
│   ├── bronze_to_silver.md
│   ├── silver_to_gold.md
│   └── data_dictionary.md
│
├── 📂 screenshots/             # Progress screenshots from Fabric and Power BI
│   ├── lakehouse_bronze.png
│   ├── silver_tables.png
│   └── powerbi_dashboard.png
│
├── LICENSE                     # MIT License
└── README.md                   # This file
```

---

## 🔄 Transformation Logic

### Bronze → Silver
- Cast all columns to correct data types
- Filter out null and duplicate records
- Add derived columns:
  - `GrossProfit = Revenue - Cost`
  - `OrderYear`, `OrderMonth` from date fields
- Write clean Delta tables to Silver Lakehouse

### Silver → Gold *(upcoming)*
- Build fact and dimension model
- Pre-aggregate KPIs by store, product, region, date
- Apply currency conversion using exchange rates
- Write final Gold tables for Power BI consumption

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
| Week 1 | Setup, GitHub, Bronze → Silver transformations | 🔄 In Progress |
| Week 2 | Silver → Gold aggregations | ⏳ Planned |
| Week 3 | Power BI dashboard development | ⏳ Planned |
| Week 4 | Final testing, documentation, LinkedIn posts | ⏳ Planned |

---

## 🔗 Follow the Build

This project is being built in public with weekly updates on LinkedIn.

**Author:** Arafat
**Email:** arafatsiddiqui3@gmail.com
**LinkedIn:** <a href="https://www.linkedin.com/in/arafat-siddiqui/">Arafat Siddiqui<a/>
**Portfolio:** https://arafat3-portfolio.netlify.app/
**GitHub:** [Arafat3-DA](https://github.com/Arafat3-DA)

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
