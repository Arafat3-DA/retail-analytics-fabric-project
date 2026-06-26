# 📂 Repository Structure Guide

This document explains the organization of the repository and the purpose of each major folder and file. The repository is structured to separate data engineering source code, detailed documentation, and presentation assets.

---

## 📁 Root Directory

| File/Folder | Purpose |
|:---|:---|
| `README.md` | The entry point to the project. Provides an executive summary, architectural overview, tech stack details, and project progress. |
| `LICENSE` | Contains the MIT License, defining the open-source usage rights for the project repository. |
| `.gitignore` | Standard Git ignore file to prevent pushing unnecessary local files or credentials to the remote repository. |

---

## 📂 notebooks/

Contains the PySpark notebook source code exported directly from Microsoft Fabric. These files encapsulate the data engineering transformation logic.

| File | Purpose |
|:---|:---|
| `silver_transformation.ipynb` | Handles the **Bronze to Silver** pipeline. Cleans raw Parquet data, casts data types, handles NULL values, standardizes strings, and writes the output as Delta tables. |
| `gold_modeling.ipynb` | Handles the **Silver to Gold** pipeline. Constructs the Star Schema (Fact and Dimension tables), performs USD currency conversions, and prepares the final Delta tables for reporting. |

---

## 📂 docs/

The centralized hub for all project documentation. It provides deep technical insights for engineers, analysts, and stakeholders.

| File/Folder | Purpose |
|:---|:---|
| `repository_structure.md` | This file; a guide to navigating the repository contents. |

### 📂 docs/Microsoft Fabric/

| File | Purpose |
|:---|:---|
| `architecture.md` | Details the Medallion Architecture workflow, system design, layer responsibilities, and key engineering decisions. |
| `data_dictionary.md` | A comprehensive schema definition covering every column, data type, and business description for both Silver and Gold tables. |
| `bronze_to_silver.md` | Documents the specific SQL transformations, filtering logic, and data quality rules applied to create the Silver layer. |
| `silver_to_gold.md` | Documents the dimensional modeling strategy, dropped columns, and the queries used to create the Gold Star Schema. |
| `performance_optimizations.md` | Highlights the engineering techniques used to optimize speed and compute across Fabric, Spark, Delta Lake, and Power BI. |

### 📂 docs/Power BI/

| File | Purpose |
|:---|:---|
| `powerbi_semantic_model.md` | Explains the Power BI data model, relationships, cardinalities, and Direct Lake integration decisions. |
| `dax_measures.md` | A complete library of the 37 DAX measures categorized by purpose (Time Intelligence, YoY, Ratios, etc.). |

---

## 📂 Power BI Dashboard/

Contains the version-controlled components of the Power BI report.

| File/Folder | Purpose |
|:---|:---|
| `Retail_Analytics_Dashboard.pbip` | The Power BI Project file. Allows Power BI Desktop to open the project in developer mode. |
| `Retail_Analytics_Dashboard.Report/` | A directory containing the extracted JSON definitions of the report layout, themes, visuals, and DAX metadata (useful for CI/CD and Git tracking). |

---

## 📂 screenshots/

Stores visual proof and documentation assets to support the README and LinkedIn portfolio posts.

| Folder | Purpose |
|:---|:---|
| `Lakehouse/` | Contains screenshots verifying the creation and structure of the Bronze, Silver, and Gold Lakehouses inside Microsoft Fabric. |
| `Notebooks/` | Contains screenshots of the PySpark notebook code executing successfully inside the Fabric environment. |
| `Power BI/` | Contains visual assets related to the dashboard. Includes the OneLake connection proof, relationship diagram, final report images, exported PDF, and the video walkthrough of the interactive dashboard. |
