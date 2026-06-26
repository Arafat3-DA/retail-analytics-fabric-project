# 📊 Power BI Semantic Model

This document outlines the design and structure of the Power BI semantic model used in the Retail Analytics dashboard. The model relies on a well-optimized **Star Schema** constructed in the Gold layer of the Microsoft Fabric Lakehouse.

---

## 🌟 Star Schema Design

The data model follows standard Kimball dimensional modeling principles. It features one central fact table surrounded by four dimension tables.

*   **Fact Table:** `fact_sales` (Records individual line items and transaction metrics)
*   **Dimension Tables:**
    *   `dim_customer` (Customer demographic and geographic data)
    *   `dim_product` (Product catalog and categorization)
    *   `dim_store` (Store location metadata)
    *   `dim_date` (Time-intelligence attributes)

By flattening snowflake structures during the Silver-to-Gold PySpark transformations (e.g., standardizing categories and subcategories directly into `dim_product`), the schema avoids complex multi-hop joins, dramatically improving query performance.

---

## 🔗 Relationships & Cardinality

Relationships in the Power BI Semantic Model are established to filter the `fact_sales` table based on user selections in the dimensions. 

| From Table (Many Side) | To Table (One Side) | Relationship Column | Cardinality | Filter Direction | Active |
|:---|:---|:---|:---|:---|:---|
| `fact_sales` | `dim_customer` | `CustomerKey` | Many-to-One ( \*:1 ) | Single | Yes |
| `fact_sales` | `dim_product` | `ProductKey` | Many-to-One ( \*:1 ) | Single | Yes |
| `fact_sales` | `dim_store` | `StoreKey` | Many-to-One ( \*:1 ) | Single | Yes |
| `fact_sales` | `dim_date` | `OrderDate` to `Date` | Many-to-One ( \*:1 ) | Single | Yes |
| `fact_sales` | `dim_date` | `DeliveryDate` to `Date` | Many-to-One ( \*:1 ) | Single | No (Inactive) |

*Note: The relationship using `DeliveryDate` is kept inactive by default to prioritize `OrderDate` for standard reporting. It can be activated via DAX (`USERELATIONSHIP`) for specific delivery-based metrics.*

---

## 🔑 Key Modeling Decisions

1. **Direct Lake Mode Integration:**
   * **Decision:** The semantic model operates in **Direct Lake** mode natively connected to OneLake Delta tables.
   * **Impact:** This removes the need for scheduled data refreshes and eliminates the memory duplication associated with Import mode, while providing performance that rivals in-memory cached models.

2. **Currency Conversion Shifted Upstream:**
   * **Decision:** Instead of calculating USD conversions dynamically row-by-row inside Power BI using DAX, `SalesAmountUSD` and `GrossProfitUSD` are pre-calculated during the Silver-to-Gold Spark pipeline.
   * **Impact:** This massively reduces CPU compute strain on the Power BI semantic engine when calculating major aggregations like `Total Sales` across millions of rows.

3. **Single Direction Filtering:**
   * **Decision:** All relationships enforce a **Single** filter direction (from Dimension to Fact).
   * **Impact:** Prevents ambiguous paths, avoids circular dependencies, and optimizes DAX evaluation times across all visuals.

4. **Dedicated Date Dimension:**
   * **Decision:** Utilizing a dedicated `dim_date` table generated in Spark instead of Power BI’s auto-date/time feature.
   * **Impact:** Reduces the hidden memory bloat of auto-date tables, enables custom fiscal hierarchies, and allows the seamless use of advanced DAX Time Intelligence functions (e.g., `TOTALYTD`, `SAMEPERIODLASTYEAR`).
