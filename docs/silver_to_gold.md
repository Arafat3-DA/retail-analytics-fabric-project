# 🥇 Silver to Gold Transformation

This document details the transformation and dimensional modeling logic applied when converting standardized data from the **Silver (Cleaned & Standardized)** layer to the **Gold (Star Schema / Business)** layer in Microsoft Fabric.

---

## 📈 Processing Overview

The Silver-to-Gold pipeline processes the refined data into a star schema containing **1 central Fact table** and **4 Dimension tables**. All tables are saved as Delta tables using `saveAsTable()`, with no joins or pre-aggregations performed in Spark (aggregation and relationships are handled dynamically in Power BI via DAX).

*   **Fact Tables:** 1 (`fact_sales` - `23,719,935` rows)
*   **Dimension Tables:** 4 (`dim_product`, `dim_customer`, `dim_date`, `dim_store` - total `1,686,455` rows)
*   **Default Lakehouse:** `Lakehouse_Gold`
*   **Output Format:** Optimized Delta Lake tables

---

## 🔄 Tables Transformation Details

### 1. Dimension Table: `dim_product`
*   **Source Table:** `silver_product`
*   **Target Table:** `dim_product`
*   **Row Count:** `2,517` rows
*   **Transformations:** Selected all columns; no structural changes.

#### Spark SQL Implementation
```sql
SELECT
    ProductKey,
    ProductCode,
    ProductName,
    Manufacturer,
    Brand,
    Color,
    WeightUnit,
    Weight,
    Cost,
    Price,
    CategoryKey,
    CategoryName,
    SubCategoryKey,
    SubCategoryName
FROM silver_product
```

---

### 2. Dimension Table: `dim_customer`
*   **Source Table:** `silver_customer`
*   **Target Table:** `dim_customer`
*   **Row Count:** `1,679,846` rows
*   **Transformations:** Selected columns from `silver_customer` dropping `StartDate`, `EndDate`, and `StreetAddress` (not required for analytics).

#### Spark SQL Implementation
```sql
SELECT
    CustomerKey,
    GeoAreaKey,
    FullName,
    Gender,
    Title,
    City,
    State,
    StateFull,
    ZipCode,
    Country,
    CountryFull,
    Continent,
    Birthday,
    Age,
    Occupation,
    Company
FROM silver_customer
```

---

### 3. Dimension Table: `dim_date`
*   **Source Table:** `silver_date`
*   **Target Table:** `dim_date`
*   **Row Count:** `4,018` rows
*   **Transformations:** Selected all columns; no structural changes.

#### Spark SQL Implementation
```sql
SELECT
    DateKey,
    Date,
    Year,
    YearQuarter,
    YearQuarterNumber,
    Quarter,
    YearMonth,
    YearMonthShort,
    YearMonthNumber,
    Month,
    MonthShort,
    MonthNumber,
    DayofWeek,
    DayofWeekShort,
    DayofWeekNumber,
    WorkingDay,
    WorkingDayNumber
FROM silver_date
```

---

### 4. Dimension Table: `dim_store`
*   **Source Table:** `silver_store`
*   **Target Table:** `dim_store`
*   **Row Count:** `74` rows
*   **Transformations:** Selected all columns; no structural changes.

#### Spark SQL Implementation
```sql
SELECT
    StoreKey,
    StoreCode,
    GeoAreaKey,
    CountryCode,
    CountryName,
    State,
    OpenDate,
    CloseDate,
    Description,
    SquareMeters,
    Status
FROM silver_store
```

---

### 5. Fact Table: `fact_sales`
*   **Source Table:** `silver_sales`
*   **Target Table:** `fact_sales`
*   **Row Count:** `23,719,935` rows
*   **Transformations:**
    *   Filtered out records where `Quantity`, `UnitPrice`, or `UnitCost` is `NULL`.
    *   Performed multi-currency conversion to USD by dividing local currency values by daily `ExchangeRate`.
    *   Derived two new USD-reporting metrics:
        *   `SalesAmountUSD = ROUND(SalesAmount / ExchangeRate, 2)`
        *   `GrossProfitUSD = ROUND(GrossProfit / ExchangeRate, 2)`

#### Spark SQL Implementation
```sql
SELECT
    s.OrderKey,
    s.LineNumber,
    s.OrderDate,
    s.DeliveryDate,
    s.CustomerKey,
    s.StoreKey,
    s.ProductKey,
    s.Quantity,
    s.UnitPrice,
    s.NetPrice,
    s.UnitCost,
    s.CurrencyCode,
    s.ExchangeRate,
    s.SalesAmount,
    s.GrossProfit,
    ROUND(s.SalesAmount / s.ExchangeRate, 2)   AS SalesAmountUSD,
    ROUND(s.GrossProfit / s.ExchangeRate, 2)   AS GrossProfitUSD
FROM silver_sales s
WHERE s.Quantity IS NOT NULL
  AND s.UnitPrice IS NOT NULL
  AND s.UnitCost IS NOT NULL
```
