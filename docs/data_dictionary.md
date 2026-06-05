# 📖 Data Dictionary

This data dictionary defines the structure and metadata of both the **Silver (Cleaned & Standardized)** and **Gold (Dimensional Star Schema)** tables within the Microsoft Fabric Lakehouse.

---

## 🥈 Silver Layer Tables Summary

| Table Name | Table Type | Row Count | Description | Primary Key |
|:---|:---|:---|:---|:---|
| [`silver_sales`](#1-table-silver_sales) | Fact | `23,719,935` | Transactional sales records containing operational metrics | `(OrderKey, LineNumber)` |
| [`silver_orders`](#2-table-silver_orders) | Fact | `9,887,613` | Order header information detailing order dates and currency | `OrderKey` |
| [`silver_orderrows`](#3-table-silver_orderrows) | Fact | `23,719,935` | Line-item detail per order including pricing and cost metadata | `(OrderKey, RowNumber)` |
| [`silver_customer`](#4-table-silver_customer) | Dimension | `1,679,846` | Master customer profile data including locations and ages | `CustomerKey` |
| [`silver_product`](#5-table-silver_product) | Dimension | `2,517` | Catalog information including category hierarchies and weights | `ProductKey` |
| [`silver_store`](#6-table-silver_store) | Dimension | `74` | Active store locations and organizational metadata | `StoreKey` |
| [`silver_date`](#7-table-silver_date) | Dimension | `4,018` | Date dimension optimized for Time Intelligence calculations | `DateKey` |
| [`silver_currencyexchange`](#8-table-silver_currencyexchange) | Reference | `100,450` | Exchange rates for converting currencies | `(Date, FromCurrency)` |

---

## 🥇 Gold Layer Tables Summary

| Table Name | Table Type | Row Count | Description | Primary Key | Source Table |
|:---|:---|:---|:---|:---|:---|
| [`fact_sales`](#gold-1-table-fact_sales) | Fact | `23,719,935` | Central sales transactions in USD and local currencies | `(OrderKey, LineNumber)` | `silver_sales` |
| [`dim_customer`](#gold-2-table-dim_customer) | Dimension | `1,679,846` | Customer profiles (dropped Start/EndDate) | `CustomerKey` | `silver_customer` |
| [`dim_product`](#gold-3-table-dim_product) | Dimension | `2,517` | Product catalog with hierarchies and prices | `ProductKey` | `silver_product` |
| [`dim_store`](#gold-4-table-dim_store) | Dimension | `74` | Active store locations and organizational metadata | `StoreKey` | `silver_store` |
| [`dim_date`](#gold-5-table-dim_date) | Dimension | `4,018` | Calendar dimension for Time Intelligence | `DateKey` | `silver_date` |

---

## 🏛️ Silver Layer Schema Specifications

### 1. Table: `silver_sales`
*   **Type:** Fact
*   **Row Count:** `23,719,935` rows
*   **Description:** Core sales transaction records including pricing, unit cost, quantities, and calculated profit margins.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `OrderKey` | `long` | Composite PK / FK | No | Identifier for the order; references `silver_orders` and `silver_orderrows` |
| `LineNumber` | `integer` | Composite PK | No | Line item number within the specific order transaction |
| `OrderDate` | `DATE` | FK | No | Date order was placed; references `silver_date` |
| `DeliveryDate` | `DATE` | FK | Yes | Date order was fulfilled; references `silver_date` |
| `CustomerKey` | `integer` | FK | No | Unique customer identifier; references `silver_customer` |
| `StoreKey` | `integer` | FK | No | Unique store location identifier; references `silver_store` |
| `ProductKey` | `integer` | FK | No | Unique product catalog identifier; references `silver_product` |
| `Quantity` | `integer` | Measure | No | Count of units sold |
| `UnitPrice` | `DOUBLE` | Metric | No | Sales price per individual unit |
| `NetPrice` | `DOUBLE` | Metric | No | Net price received after adjustments |
| `UnitCost` | `DOUBLE` | Metric | No | Operational cost to produce or acquire a single unit |
| `CurrencyCode` | `string` | FK | No | Three-letter currency code (e.g. 'USD'); references `silver_currencyexchange` |
| `ExchangeRate` | `DOUBLE` | Rate | No | Exchange rate multiplier applied on the transaction day |
| `SalesAmount` | `DOUBLE` | Measure (New) | No | Derived amount: `ROUND(Quantity * UnitPrice, 2)` |
| `GrossProfit` | `DOUBLE` | Measure (New) | No | Derived profit: `ROUND((Quantity * UnitPrice) - (Quantity * UnitCost), 2)` |

---

### 2. Table: `silver_orders`
*   **Type:** Fact
*   **Row Count:** `9,887,613` rows
*   **Description:** Transaction header records indicating operational metadata around order timing, execution, and billing currency.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `OrderKey` | `long` | PK | No | Primary key identifier for the order |
| `CustomerKey` | `integer` | FK | No | Unique identifier for the customer; references `silver_customer` |
| `StoreKey` | `integer` | FK | No | Unique identifier for the store location; references `silver_store` |
| `OrderDate` | `DATE` | FK | No | Date the order was placed; references `silver_date` |
| `DeliveryDate` | `DATE` | FK | Yes | Date the order was fulfilled; references `silver_date` |
| `CurrencyCode` | `string` | FK | No | Transactional billing currency code |

---

### 3. Table: `silver_orderrows`
*   **Type:** Fact
*   **Row Count:** `23,719,935` rows
*   **Description:** Line-item transactional records tracking pricing structure breakdown, quantity, and manufacturing unit costs.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `OrderKey` | `long` | Composite PK / FK | No | Unique order header identifier; references `silver_orders` |
| `RowNumber` | `integer` | Composite PK | No | Line row sequence within the order |
| `ProductKey` | `integer` | FK | No | Catalog reference identifier; references `silver_product` |
| `Quantity` | `integer` | Measure | No | Number of products in this row line |
| `UnitPrice` | `DOUBLE` | Metric | No | Listed unit price for the product |
| `NetPrice` | `DOUBLE` | Metric | No | Net discount price applied |
| `UnitCost` | `DOUBLE` | Metric | No | Production cost per unit |

---

### 4. Table: `silver_customer`
*   **Type:** Dimension
*   **Row Count:** `1,679,846` rows
*   **Description:** Customer master database containing demographics, geography keys, birth dates, and contact addresses.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `CustomerKey` | `integer` | PK | No | Unique identifier for each customer |
| `GeoAreaKey` | `integer` | FK | Yes | Geographic location identifier key |
| `StartDate` | `DATE` | - | Yes | Active starting date for customer profile |
| `EndDate` | `DATE` | - | Yes | Active expiration date for customer profile (if applicable) |
| `Continent` | `string` | - | Yes | Geographic continent region |
| `Gender` | `string` | - | Yes | Casing standardized to Initcap ('Male', 'Female') |
| `Title` | `string` | - | Yes | Salutation title (e.g., 'Mr.', 'Ms.') |
| `FullName` | `string` | Attribute | Yes | Derived name: `GivenName` + `MiddleInitial` + `Surname` |
| `StreetAddress`| `string` | - | Yes | Physical street residence |
| `City` | `string` | - | Yes | Physical city residence |
| `State` | `string` | - | Yes | Physical state residence code |
| `StateFull` | `string` | - | Yes | Full physical state name |
| `ZipCode` | `string` | - | Yes | ZIP postal code |
| `Country` | `string` | - | Yes | ISO country code |
| `CountryFull` | `string` | - | Yes | Full country name |
| `Birthday` | `DATE` | - | Yes | Date of birth |
| `Age` | `integer` | Attribute | Yes | Current calculated age |
| `Occupation` | `string` | - | Yes | Customer's profession category |
| `Company` | `string` | - | Yes | Employer company name |

---

### 5. Table: `silver_product`
*   **Type:** Dimension
*   **Row Count:** `2,517` rows
*   **Description:** Detailed catalog lists for item descriptions, categories, manufacturers, pricing, and sizing parameters.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `ProductKey` | `integer` | PK | No | Unique item catalog identifier |
| `ProductCode` | `string` | Attribute | No | Standard code preserving leading zeroes (e.g. '0012') |
| `ProductName` | `string` | - | Yes | Full name of the catalog product |
| `Manufacturer` | `string` | - | Yes | Producer/Manufacturer |
| `Brand` | `string` | - | Yes | Core brand label |
| `Color` | `string` | - | Yes | Product color variant |
| `WeightUnit` | `string` | - | Yes | Weight unit measure standard (e.g. 'g', 'kg') |
| `Weight` | `DOUBLE` | Measure | Yes | Physical weight parameter |
| `Cost` | `DOUBLE` | Metric | No | Purchasing base cost |
| `Price` | `DOUBLE` | Metric | No | Listed standard retail pricing |
| `CategoryKey` | `integer` | FK | No | Top-level category class code |
| `CategoryName`| `string` | - | Yes | Category classification label |
| `SubCategoryKey`| `integer`| FK | No | Lower-level category class code |
| `SubCategoryName`| `string`| - | Yes | Subcategory classification label |

---

### 6. Table: `silver_store`
*   **Type:** Dimension
*   **Row Count:** `74` rows
*   **Description:** Operational metadata mapping store properties, geographical assignments, sizes, and open/closure states.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `StoreKey` | `integer` | PK | No | Unique store identifier |
| `StoreCode` | `integer` | Attribute | No | Operations code for physical branch identification |
| `GeoAreaKey` | `integer` | FK | No | Geographic region reference key |
| `CountryCode` | `string` | - | No | Standardized country abbreviation |
| `CountryName` | `string` | - | No | Country name label |
| `State` | `string` | - | Yes | Standardized state/province name |
| `OpenDate` | `DATE` | - | No | The official launching date of the physical branch |
| `CloseDate` | `DATE` | - | Yes | Closure date of branch (NULL indicates store is currently active) |
| `Description` | `string` | - | Yes | Internal store branch name descriptions |
| `SquareMeters` | `integer` | Measure | Yes | Gross operational store area dimensions |
| `Status` | `string` | Attribute | Yes | Store status code (NULL indicates active state) |

---

### 7. Table: `silver_date`
*   **Type:** Dimension
*   **Row Count:** `4,018` rows
*   **Description:** Calendar dimension table with calendar mappings, business working dates, and chronological divisions.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `DateKey` | `INTEGER` | PK | No | Integer calendar key (YYYYMMDD) |
| `Date` | `DATE` | Attribute | No | Standard full date value |
| `Year` | `integer` | Attribute | Yes | Calendar Year |
| `YearQuarter` | `string` | Attribute | Yes | Quarter formatted (e.g. '2026-Q1') |
| `YearQuarterNumber`|`integer`| - | Yes | Chronological quarter index |
| `Quarter` | `string` | Attribute | Yes | Quarter string label (e.g. 'Q1') |
| `YearMonth` | `string` | Attribute | Yes | Month code label (e.g. '2026-05') |
| `YearMonthShort`| `string` | - | Yes | Month abbreviation string |
| `YearMonthNumber`| `integer`| - | Yes | Unique chronological month sequential index |
| `Month` | `string` | Attribute | Yes | Full month name string |
| `MonthShort` | `string` | - | Yes | Three-letter month abbreviation |
| `MonthNumber` | `integer` | - | Yes | Calendar month index (1-12) |
| `DayofWeek` | `string` | Attribute | Yes | Weekday full name string |
| `DayofWeekShort`| `string` | - | Yes | Weekday three-letter code |
| `DayofWeekNumber`|`integer` | - | Yes | Weekday calendar number |
| `WorkingDay` | `integer` | Attribute | Yes | Workday flag (integer encoded) |
| `WorkingDayNumber`|`integer`| - | Yes | Chronological workday indexing count |

---

### 8. Table: `silver_currencyexchange`
*   **Type:** Reference
*   **Row Count:** `100,450` rows
*   **Description:** Currency rates dataset containing daily exchange rates for multi-currency conversion calculations.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `Date` | `DATE` | Composite PK / FK | No | Daily rate date; references `silver_date` |
| `FromCurrency`| `string` | Composite PK | No | Code representing originating currency |
| `ToCurrency` | `string` | - | No | Target base translation currency code |
| `ExchangeRate`| `DOUBLE` | Rate | No | Exchange factor applied on transaction date |

---

## 🥇 Gold Layer Schema Specifications

### Gold 1. Table: `fact_sales`
*   **Type:** Fact
*   **Row Count:** `23,719,935` rows
*   **Description:** Central sales fact table containing transaction records in USD and local currencies. No pre-aggregation is applied; all KPIs are dynamically aggregated via DAX in Power BI.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `OrderKey` | `long` | Composite PK / FK | No | Identifier for the order; references `dim_customer`, `dim_store` |
| `LineNumber` | `integer` | Composite PK | No | Line item number within the specific order transaction |
| `OrderDate` | `DATE` | FK | No | Date order was placed; references `dim_date` |
| `DeliveryDate` | `DATE` | FK | Yes | Date order was fulfilled; references `dim_date` |
| `CustomerKey` | `integer` | FK | No | Unique customer identifier; references `dim_customer` |
| `StoreKey` | `integer` | FK | No | Unique store location identifier; references `dim_store` |
| `ProductKey` | `integer` | FK | No | Unique product catalog identifier; references `dim_product` |
| `Quantity` | `integer` | Measure | No | Count of units sold |
| `UnitPrice` | `DOUBLE` | Metric | No | Sales price per individual unit in local currency |
| `NetPrice` | `DOUBLE` | Metric | No | Net price received after adjustments in local currency |
| `UnitCost` | `DOUBLE` | Metric | No | Operational cost to produce or acquire a single unit in local currency |
| `CurrencyCode` | `string` | FK | No | Three-letter currency code (e.g. 'USD') |
| `ExchangeRate` | `DOUBLE` | Rate | No | Exchange rate multiplier applied on the transaction day |
| `SalesAmount` | `DOUBLE` | Measure | No | Derived amount: `ROUND(Quantity * UnitPrice, 2)` (local currency) |
| `GrossProfit` | `DOUBLE` | Measure | No | Derived profit: `ROUND((Quantity * UnitPrice) - (Quantity * UnitCost), 2)` (local currency) |
| `SalesAmountUSD` | `DOUBLE` | Measure | No | USD converted sales: `ROUND(SalesAmount / ExchangeRate, 2)` |
| `GrossProfitUSD` | `DOUBLE` | Measure | No | USD converted profit: `ROUND(GrossProfit / ExchangeRate, 2)` |

---

### Gold 2. Table: `dim_customer`
*   **Type:** Dimension
*   **Row Count:** `1,679,846` rows
*   **Description:** Customer master database containing demographics, geography keys, birth dates, and contact addresses. `StartDate` and `EndDate` are dropped as they are not needed for analytics.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `CustomerKey` | `integer` | PK | No | Unique identifier for each customer |
| `GeoAreaKey` | `integer` | FK | Yes | Geographic location identifier key |
| `FullName` | `string` | Attribute | Yes | Full name: GivenName + MiddleInitial + Surname |
| `Gender` | `string` | - | Yes | Standardized casing (e.g. 'Male', 'Female') |
| `Title` | `string` | - | Yes | Salutation title (e.g., 'Mr.', 'Ms.') |
| `City` | `string` | - | Yes | Physical city residence |
| `State` | `string` | - | Yes | Physical state residence code |
| `StateFull` | `string` | - | Yes | Full physical state name |
| `ZipCode` | `string` | - | Yes | ZIP postal code |
| `Country` | `string` | - | Yes | ISO country code |
| `CountryFull` | `string` | - | Yes | Full country name |
| `Continent` | `string` | - | Yes | Geographic continent region |
| `Birthday` | `DATE` | - | Yes | Date of birth |
| `Age` | `integer` | Attribute | Yes | Current calculated age |
| `Occupation` | `string` | - | Yes | Customer's profession category |
| `Company` | `string` | - | Yes | Employer company name |

---

### Gold 3. Table: `dim_product`
*   **Type:** Dimension
*   **Row Count:** `2,517` rows
*   **Description:** Detailed product catalog lists for item descriptions, categories, manufacturers, pricing, and sizing parameters. All columns kept from `silver_product`.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `ProductKey` | `integer` | PK | No | Unique item catalog identifier |
| `ProductCode` | `string` | Attribute | No | Standard code preserving leading zeroes (e.g. '0012') |
| `ProductName` | `string` | - | Yes | Full name of the catalog product |
| `Manufacturer` | `string` | - | Yes | Producer/Manufacturer |
| `Brand` | `string` | - | Yes | Core brand label |
| `Color` | `string` | - | Yes | Product color variant |
| `WeightUnit` | `string` | - | Yes | Weight unit measure standard (e.g. 'g', 'kg') |
| `Weight` | `DOUBLE` | Measure | Yes | Physical weight parameter |
| `Cost` | `DOUBLE` | Metric | No | Purchasing base cost |
| `Price` | `DOUBLE` | Metric | No | Listed standard retail pricing |
| `CategoryKey` | `integer` | FK | No | Top-level category class code |
| `CategoryName`| `string` | - | Yes | Category classification label |
| `SubCategoryKey`| `integer`| FK | No | Lower-level category class code |
| `SubCategoryName`| `string`| - | Yes | Subcategory classification label |

---

### Gold 4. Table: `dim_store`
*   **Type:** Dimension
*   **Row Count:** `74` rows
*   **Description:** Operational metadata mapping store properties, geographical assignments, sizes, and open/closure states. All columns kept from `silver_store`.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `StoreKey` | `integer` | PK | No | Unique store identifier |
| `StoreCode` | `integer` | Attribute | No | Operations code for physical branch identification |
| `GeoAreaKey` | `integer` | FK | No | Geographic region reference key |
| `CountryCode` | `string` | - | No | Standardized country abbreviation |
| `CountryName` | `string` | - | No | Country name label |
| `State` | `string` | - | Yes | Standardized state/province name |
| `OpenDate` | `DATE` | - | No | The official launching date of the physical branch |
| `CloseDate` | `DATE` | - | Yes | Closure date of branch (NULL indicates store is currently active) |
| `Description` | `string` | - | Yes | Internal store branch name descriptions |
| `SquareMeters` | `integer` | Measure | Yes | Gross operational store area dimensions |
| `Status` | `string` | Attribute | Yes | Store status code (NULL indicates active state) |

---

### Gold 5. Table: `dim_date`
*   **Type:** Dimension
*   **Row Count:** `4,018` rows
*   **Description:** Calendar dimension table with calendar mappings, business working dates, and chronological divisions. All columns kept from `silver_date`.

| Column Name | Data Type | Key Type | Nullable | Description / Notes |
|:---|:---|:---|:---|:---|
| `DateKey` | `INTEGER` | PK | No | Integer calendar key (YYYYMMDD) |
| `Date` | `DATE` | Attribute | No | Standard full date value |
| `Year` | `integer` | Attribute | Yes | Calendar Year |
| `YearQuarter` | `string` | Attribute | Yes | Quarter formatted (e.g. '2026-Q1') |
| `YearQuarterNumber`|`integer`| - | Yes | Chronological quarter index |
| `Quarter` | `string` | Attribute | Yes | Quarter string label (e.g. 'Q1') |
| `YearMonth` | `string` | Attribute | Yes | Month code label (e.g. '2026-05') |
| `YearMonthShort`| `string` | - | Yes | Month abbreviation string |
| `YearMonthNumber`| `integer`| - | Yes | Unique chronological month sequential index |
| `Month` | `string` | Attribute | Yes | Full month name string |
| `MonthShort` | `string` | - | Yes | Three-letter month abbreviation |
| `MonthNumber` | `integer` | - | Yes | Calendar month index (1-12) |
| `DayofWeek` | `string` | Attribute | Yes | Weekday full name string |
| `DayofWeekShort`| `string` | - | Yes | Weekday three-letter code |
| `DayofWeekNumber`|`integer` | - | Yes | Weekday calendar number |
| `WorkingDay` | `integer` | Attribute | Yes | Workday flag (integer encoded) |
| `WorkingDayNumber`|`integer`| - | Yes | Chronological workday indexing count |
