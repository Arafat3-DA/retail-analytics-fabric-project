# 🥈 Bronze to Silver Transformation

This document details the exact transformation logic applied when moving raw transactional data from the **Bronze (Raw)** layer to the **Silver (Cleaned & Standardized)** layer in Microsoft Fabric. 

---

## 📈 Processing Overview

The Bronze-to-Silver ingestion pipeline processed all **8 core tables** with zero data loss, standardizing data types, cleaning missing values, and generating optimized Delta tables.

*   **Total Rows Processed:** `59,114,388` rows
*   **Total Processing Time:** Under 5 minutes
*   **Data Loss:** `0%` (All tables matched Bronze source row counts exactly)
*   **Output Format:** Delta Lake tables (ACID compliant, optimized for analytical queries)

---

## 🔄 Tables Transformation Details

### 1. Table: `currencyexchange`
*   **Source View:** `bronze_currencyexchange`
*   **Target Table:** `silver_currencyexchange`
*   **Row Count:** `100,450` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `Date` | `timestamp` | `DATE` | `CAST` to standard date format | No |
| `FromCurrency` | `string` | `string` | Kept as is | No |
| `ToCurrency` | `string` | `string` | Kept as is | No |
| `ExchangeRate` | `decimal(20,5)` | `DOUBLE` | `CAST` to double; renamed from `Exchange` | No |

#### Filter Condition
```sql
WHERE Date IS NOT NULL 
  AND Exchange IS NOT NULL
```

#### Spark SQL Implementation
```sql
SELECT
    CAST(Date AS DATE)        AS Date,
    FromCurrency,
    ToCurrency,
    CAST(Exchange AS DOUBLE)  AS ExchangeRate
FROM bronze_currencyexchange
WHERE Date IS NOT NULL
  AND Exchange IS NOT NULL
```

---

### 2. Table: `customer`
*   **Source View:** `bronze_customer`
*   **Target Table:** `silver_customer`
*   **Row Count:** `1,679,846` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `CustomerKey` | `integer` | `integer` | Kept as is | No |
| `GeoAreaKey` | `integer` | `integer` | Kept as is | Yes |
| `StartDate` | `timestamp` | `DATE` | `CAST` to date; renamed from `StartDT` | Yes |
| `EndDate` | `timestamp` | `DATE` | `CAST` to date; renamed from `EndDT` | Yes |
| `Continent` | `string` | `string` | Kept as is | Yes |
| `Gender` | `string` | `string` | `INITCAP` applied for standardized casing (e.g., 'Male', 'Female') | Yes |
| `Title` | `string` | `string` | Kept as is | Yes |
| `FullName` | `string` | `string` | `CONCAT` of `GivenName`, `MiddleInitial`, and `Surname` with `COALESCE` handling | Yes |
| `StreetAddress` | `string` | `string` | Kept as is | Yes |
| `City` | `string` | `string` | Kept as is | Yes |
| `State` | `string` | `string` | Kept as is | Yes |
| `StateFull` | `string` | `string` | Kept as is | Yes |
| `ZipCode` | `string` | `string` | Kept as is | Yes |
| `Country` | `string` | `string` | Kept as is | Yes |
| `CountryFull` | `string` | `string` | Kept as is | Yes |
| `Birthday` | `timestamp` | `DATE` | `CAST` to standard date format | Yes |
| `Age` | `integer` | `integer` | Kept as is | Yes |
| `Occupation` | `string` | `string` | Kept as is | Yes |
| `Company` | `string` | `string` | Kept as is | Yes |

#### Dropped Columns
*   `Vehicle` (analytical irrelevance)
*   `Latitude` (analytical irrelevance)
*   `Longitude` (analytical irrelevance)

#### Filter Condition
```sql
WHERE CustomerKey IS NOT NULL
```

#### Spark SQL Implementation
```sql
SELECT
    CustomerKey,
    GeoAreaKey,
    CAST(StartDT AS DATE) AS StartDate,
    CAST(EndDT AS DATE)   AS EndDate,
    Continent,
    INITCAP(Gender)       AS Gender,
    Title,
    CONCAT(GivenName, ' ',
           COALESCE(MiddleInitial, ''), ' ',
           Surname)       AS FullName,
    StreetAddress,
    City,
    State,
    StateFull,
    ZipCode,
    Country,
    CountryFull,
    CAST(Birthday AS DATE) AS Birthday,
    Age,
    Occupation,
    Company
FROM bronze_customer
WHERE CustomerKey IS NOT NULL
```

---

### 3. Table: `date`
*   **Source View:** `bronze_date`
*   **Target Table:** `silver_date`
*   **Row Count:** `4,018` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `Date` | `timestamp` | `DATE` | `CAST` to standard date format | No |
| `DateKey` | `string` | `INTEGER` | `CAST` to standard integer key | No |
| `Year` ... `WorkingDayNumber` | Various | Various | 15 columns kept as is (already clean) | Yes |

#### Filter Condition
```sql
WHERE Date IS NOT NULL
```

#### Spark SQL Implementation
```sql
SELECT
    CAST(Date AS DATE)       AS Date,
    CAST(DateKey AS INTEGER) AS DateKey,
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
FROM bronze_date
WHERE Date IS NOT NULL
```

---

### 4. Table: `orderrows`
*   **Source View:** `bronze_orderrows`
*   **Target Table:** `silver_orderrows`
*   **Row Count:** `23,719,935` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `OrderKey` | `long` | `long` | Kept as is | No |
| `RowNumber` | `integer` | `integer` | Kept as is | No |
| `ProductKey` | `integer` | `integer` | Kept as is | No |
| `Quantity` | `integer` | `integer` | Kept as is | No |
| `UnitPrice` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision decimal | No |
| `NetPrice` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision decimal | No |
| `UnitCost` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision decimal | No |

#### Filter Condition
```sql
WHERE OrderKey IS NOT NULL 
  AND Quantity IS NOT NULL
```

#### Spark SQL Implementation
```sql
SELECT
    OrderKey,
    RowNumber,
    ProductKey,
    Quantity,
    CAST(UnitPrice AS DOUBLE)  AS UnitPrice,
    CAST(NetPrice AS DOUBLE)   AS NetPrice,
    CAST(UnitCost AS DOUBLE)   AS UnitCost
FROM bronze_orderrows
WHERE OrderKey IS NOT NULL
  AND Quantity IS NOT NULL
```

---

### 5. Table: `orders`
*   **Source View:** `bronze_orders`
*   **Target Table:** `silver_orders`
*   **Row Count:** `9,887,613` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `OrderKey` | `long` | `long` | Kept as is | No |
| `CustomerKey` | `integer` | `integer` | Kept as is | No |
| `StoreKey` | `integer` | `integer` | Kept as is | No |
| `OrderDate` | `timestamp` | `DATE` | `CAST` to date; renamed from `DT` | No |
| `DeliveryDate` | `timestamp` | `DATE` | `CAST` to standard date format | Yes |
| `CurrencyCode` | `string` | `string` | Kept as is | No |

#### Filter Condition
```sql
WHERE OrderKey IS NOT NULL 
  AND CustomerKey IS NOT NULL
```

#### Spark SQL Implementation
```sql
SELECT
    OrderKey,
    CustomerKey,
    StoreKey,
    CAST(DT AS DATE)           AS OrderDate,
    CAST(DeliveryDate AS DATE) AS DeliveryDate,
    CurrencyCode
FROM bronze_orders
WHERE OrderKey IS NOT NULL
  AND CustomerKey IS NOT NULL
```

---

### 6. Table: `product`
*   **Source View:** `bronze_product`
*   **Target Table:** `silver_product`
*   **Row Count:** `2,517` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `ProductKey` | `integer` | `integer` | Kept as is | No |
| `ProductCode` | `string` | `string` | Kept as is (preserved as string for leading zeros) | No |
| `ProductName` | `string` | `string` | Kept as is | Yes |
| `Manufacturer` | `string` | `string` | Kept as is | Yes |
| `Brand` | `string` | `string` | Kept as is | Yes |
| `Color` | `string` | `string` | Kept as is | Yes |
| `WeightUnit` | `string` | `string` | Kept as is | Yes |
| `Weight` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision | Yes |
| `Cost` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision | No |
| `Price` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision | No |
| `CategoryKey` | `integer` | `integer` | Kept as is | No |
| `CategoryName` | `string` | `string` | Kept as is | Yes |
| `SubCategoryKey` | `integer` | `integer` | Kept as is | No |
| `SubCategoryName` | `string` | `string` | Kept as is | Yes |

#### Filter Condition
```sql
WHERE ProductKey IS NOT NULL
```

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
    CAST(Weight AS DOUBLE)  AS Weight,
    CAST(Cost AS DOUBLE)    AS Cost,
    CAST(Price AS DOUBLE)   AS Price,
    CategoryKey,
    CategoryName,
    SubCategoryKey,
    SubCategoryName
FROM bronze_product
WHERE ProductKey IS NOT NULL
```

---

### 7. Table: `store`
*   **Source View:** `bronze_store`
*   **Target Table:** `silver_store`
*   **Row Count:** `74` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `StoreKey` | `integer` | `integer` | Kept as is | No |
| `StoreCode` | `integer` | `integer` | Kept as is | No |
| `GeoAreaKey` | `integer` | `integer` | Kept as is | No |
| `CountryCode` | `string` | `string` | Kept as is | No |
| `CountryName` | `string` | `string` | Kept as is | No |
| `State` | `string` | `string` | Kept as is | Yes |
| `OpenDate` | `timestamp` | `DATE` | `CAST` to standard date format | No |
| `CloseDate` | `timestamp` | `DATE` | `CAST` to standard date (NULL = store is currently open) | Yes |
| `Description` | `string` | `string` | Kept as is | Yes |
| `SquareMeters` | `integer` | `integer` | Kept as is | Yes |
| `Status` | `string` | `string` | Kept as is (NULL = active) | Yes |

#### Filter Condition
```sql
WHERE StoreKey IS NOT NULL
```

#### Spark SQL Implementation
```sql
SELECT
    StoreKey,
    StoreCode,
    GeoAreaKey,
    CountryCode,
    CountryName,
    State,
    CAST(OpenDate AS DATE)  AS OpenDate,
    CAST(CloseDate AS DATE) AS CloseDate,
    Description,
    SquareMeters,
    Status
FROM bronze_store
WHERE StoreKey IS NOT NULL
```

---

### 8. Table: `sales`
*   **Source View:** `bronze_sales`
*   **Target Table:** `silver_sales`
*   **Row Count:** `23,719,935` rows

#### Schema Mapping
| Column Name | Bronze Type | Silver Type | Action / Transformation | Nullable |
|:---|:---|:---|:---|:---|
| `OrderKey` | `long` | `long` | Kept as is | No |
| `LineNumber` | `integer` | `integer` | Kept as is | No |
| `OrderDate` | `timestamp` | `DATE` | `CAST` to standard date format | No |
| `DeliveryDate` | `timestamp` | `DATE` | `CAST` to standard date format | Yes |
| `CustomerKey` | `integer` | `integer` | Kept as is | No |
| `StoreKey` | `integer` | `integer` | Kept as is | No |
| `ProductKey` | `integer` | `integer` | Kept as is | No |
| `Quantity` | `integer` | `integer` | Kept as is | No |
| `UnitPrice` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision decimal | No |
| `NetPrice` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision decimal | No |
| `UnitCost` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision decimal | No |
| `CurrencyCode` | `string` | `string` | Kept as is | No |
| `ExchangeRate` | `decimal(20,5)` | `DOUBLE` | `CAST` to double precision decimal | No |
| `SalesAmount` | **New Column** | `DOUBLE` | Computed: `ROUND(Quantity * UnitPrice, 2)` | No |
| `GrossProfit` | **New Column** | `DOUBLE` | Computed: `ROUND((Quantity * UnitPrice) - (Quantity * UnitCost), 2)` | No |

#### Filter Condition
```sql
WHERE Quantity IS NOT NULL 
  AND UnitPrice IS NOT NULL 
  AND UnitCost IS NOT NULL
```

#### Spark SQL Implementation
```sql
SELECT
    OrderKey,
    LineNumber,
    CAST(OrderDate AS DATE)       AS OrderDate,
    CAST(DeliveryDate AS DATE)    AS DeliveryDate,
    CustomerKey,
    StoreKey,
    ProductKey,
    Quantity,
    CAST(UnitPrice AS DOUBLE)     AS UnitPrice,
    CAST(NetPrice AS DOUBLE)      AS NetPrice,
    CAST(UnitCost AS DOUBLE)      AS UnitCost,
    CurrencyCode,
    CAST(ExchangeRate AS DOUBLE)  AS ExchangeRate,
    ROUND(Quantity * CAST(UnitPrice AS DOUBLE), 2)    AS SalesAmount,
    ROUND((Quantity * CAST(UnitPrice AS DOUBLE)) -
          (Quantity * CAST(UnitCost AS DOUBLE)), 2)   AS GrossProfit
FROM bronze_sales
WHERE Quantity IS NOT NULL
  AND UnitPrice IS NOT NULL
  AND UnitCost IS NOT NULL
```
