# 🚀 Performance Optimizations

This project leverages Microsoft Fabric's integrated ecosystem to process and query nearly 60 million rows of data efficiently. Below are the key performance optimizations implemented across the technology stack.

---

## 1. Storage & Delta Lake Optimizations

*   **V-Order Implementation:** By default in Microsoft Fabric, all Delta tables are saved using V-Order. This drastically improves read times and compression ratios for analytical queries by sorting and organizing Parquet files at write-time.
*   **Parquet Columnar Format:** Utilizing Delta Lake's native Parquet backing ensures that queries only scan the necessary columns rather than entire rows, significantly minimizing I/O overhead.

## 2. Apache Spark (Data Engineering) Optimizations

*   **Data Type Casting (Precision Reduction):**
    *   **Action:** Highly precise SQL decimals (`decimal(20,5)`) from the Bronze layer were explicitly cast down to PySpark `DOUBLE` types in the Silver layer (e.g., `UnitPrice`, `UnitCost`, `ExchangeRate`).
    *   **Impact:** Reduces memory overhead on the Spark executors, mitigates out-of-memory errors on large shuffles, and accelerates distributed math operations during transformations.
*   **Early Null Filtering:**
    *   **Action:** Rows containing `NULL` values in critical composite keys (like `OrderKey` or `LineNumber`) were filtered out immediately during the Bronze-to-Silver transformation.
    *   **Impact:** Reduces the size of the dataset moving downstream, preventing skew and join failures in the analytical layer.
*   **Push-Down Computations:**
    *   **Action:** Complex row-by-row string operations (e.g., creating `FullName` from `GivenName`, `MiddleInitial`, and `Surname`) were performed in Spark rather than DAX.

## 3. Data Modeling (Gold Layer) Optimizations

*   **Star Schema Standardization:**
    *   **Action:** Data was explicitly modeled into 1 Fact and 4 Dimension tables, eliminating snowflake designs.
    *   **Impact:** Minimizes the number of hops (joins) the Power BI engine must execute to filter data, optimizing the speed of matrix visuals and cross-filtering.
*   **Pre-calculated Currency Conversions:**
    *   **Action:** The standard reporting values (`SalesAmountUSD` and `GrossProfitUSD`) were pre-calculated mathematically during the Spark job by dividing the local amount by the `ExchangeRate`.
    *   **Impact:** Saves Power BI from running intensive `SUMX` iterations across 23+ million rows for every dashboard interaction.

## 4. Power BI & Direct Lake Optimizations

*   **Direct Lake Mode:**
    *   **Action:** The semantic model connects to the Lakehouse using Direct Lake mode.
    *   **Impact:** Bypasses traditional Import mode (which requires memory duplication and scheduled refreshes) and DirectQuery (which suffers from high latency). Direct Lake loads Parquet data directly into the Power BI engine's memory, ensuring real-time performance without data movement.
*   **Safe DAX Division:**
    *   **Action:** All ratio and percentage measures utilize the native DAX `DIVIDE()` function.
    *   **Impact:** Gracefully handles divide-by-zero errors without requiring expensive `IF(ISBLANK())` logic, streamlining query evaluation.
*   **Single-Direction Relationships:**
    *   **Action:** Bi-directional cross-filtering is avoided; all relationships flow 1:* from dimension to fact.
    *   **Impact:** Reduces ambiguity in the VertiPaq engine, significantly speeding up measure calculation times.
