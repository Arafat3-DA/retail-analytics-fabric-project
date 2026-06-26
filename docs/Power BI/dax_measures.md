# 📐 DAX Measures Library

The Retail Analytics Power BI dashboard utilizes a comprehensive set of 37 DAX measures to calculate KPIs, time intelligence, and analytical metrics. These measures are organized into logical categories to support varying analytical needs.

---

## 1. Base Measures (7)

Core aggregation measures used directly in visuals or as foundation for complex calculations.

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Total Sales** | `SUM(fact_sales[SalesAmountUSD])` | Calculates total revenue in USD |
| **Gross Profit** | `SUM(fact_sales[GrossProfitUSD])` | Calculates total gross profit in USD |
| **Total Quantity Sold** | `SUM(fact_sales[Quantity])` | Calculates total units sold |
| **Total Orders** | `DISTINCTCOUNT(fact_sales[OrderKey])` | Counts unique orders placed |
| **Total Customers** | `DISTINCTCOUNT(fact_sales[CustomerKey])` | Counts unique purchasing customers |
| **Total Stores** | `DISTINCTCOUNT(fact_sales[StoreKey])` | Counts unique stores with sales activity |
| **Total Products Sold** | `DISTINCTCOUNT(fact_sales[ProductKey])` | Counts unique products sold |

---

## 2. Ratio & Margin Measures (7)

Measures calculating performance ratios and averages safely using the `DIVIDE()` function.

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Gross Profit Margin** | `DIVIDE([Gross Profit], [Total Sales])` | Profitability margin percentage |
| **Average Order Value** | `DIVIDE([Total Sales], [Total Orders])` | Average revenue per order (AOV) |
| **Average Quantity per Order** | `DIVIDE([Total Quantity Sold], [Total Orders])` | Average basket size per order |
| **Revenue per Customer** | `DIVIDE([Total Sales], [Total Customers])` | Average revenue generated per customer |
| **Revenue per Store** | `DIVIDE([Total Sales], [Total Stores])` | Average revenue generated per store |
| **Gross Profit per Customer** | `DIVIDE([Gross Profit], [Total Customers])` | Average profit generated per customer |
| **Orders per Customer** | `DIVIDE([Total Orders], [Total Customers])` | Average purchase frequency per customer |

---

## 3. Cost Measure (1)

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Cost of Sales** | `SUMX(fact_sales, fact_sales[Quantity] * fact_sales[UnitCost] / fact_sales[ExchangeRate])` | Calculates the total cost of goods sold (COGS) converted to USD |

---

## 4. Time Intelligence Measures (7)

Measures leveraging the `dim_date` dimension for period-to-date aggregations.

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Sales MTD** | `TOTALMTD([Total Sales], dim_date[Date])` | Month-to-date sales |
| **Sales QTD** | `TOTALQTD([Total Sales], dim_date[Date])` | Quarter-to-date sales |
| **Sales YTD** | `TOTALYTD([Total Sales], dim_date[Date])` | Year-to-date sales |
| **Gross Profit MTD** | `TOTALMTD([Gross Profit], dim_date[Date])` | Month-to-date gross profit |
| **Gross Profit YTD** | `TOTALYTD([Gross Profit], dim_date[Date])` | Year-to-date gross profit |
| **Orders YTD** | `TOTALYTD([Total Orders], dim_date[Date])` | Year-to-date orders |
| **Orders LYTD** | `CALCULATE([Orders YTD], DATEADD(dim_date[Date],-1,YEAR))` | Last year's Year-to-date orders |

---

## 5. Year-over-Year (YoY) Comparisons (4)

Measures calculating equivalent performance from the prior year.

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Total Sales LY** | `CALCULATE([Total Sales], SAMEPERIODLASTYEAR(dim_date[Date]))` | Total sales for the same period last year |
| **Total Gross Profit LY** | `CALCULATE([Gross Profit], SAMEPERIODLASTYEAR(dim_date[Date]))` | Total profit for the same period last year |
| **Total Orders LY** | `CALCULATE([Total Orders], SAMEPERIODLASTYEAR(dim_date[Date]))` | Total orders for the same period last year |
| **Total Quantity Sold LY** | `CALCULATE([Total Quantity Sold], SAMEPERIODLASTYEAR(dim_date[Date]))` | Total units sold for the same period last year |

---

## 6. Growth % Measures (4)

Measures calculating the percentage variance against prior periods.

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Sales YoY %** | `DIVIDE([Total Sales] - [Total Sales LY], [Total Sales LY])` | Sales growth percentage year-over-year |
| **Gross Profit YoY %** | `DIVIDE([Gross Profit] - [Total Gross Profit LY], [Total Gross Profit LY])` | Profit growth percentage year-over-year |
| **Orders YoY %** | `DIVIDE([Total Orders] - [Total Orders LY], [Total Orders LY])` | Order volume growth percentage year-over-year |
| **Quantity YoY %** | `DIVIDE([Total Quantity Sold] - [Total Quantity Sold LY], [Total Quantity Sold LY])` | Volume growth percentage year-over-year |

---

## 7. Additional Analytical & Variance Measures (4)

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Sales YTD vs LY %** | `DIVIDE([Sales YTD] - [Sales LYTD], [Sales LYTD])` | Sales YTD growth vs previous YTD |
| **Sales LYTD** | `CALCULATE([Sales YTD], SAMEPERIODLASTYEAR(dim_date[Date]))` | Sales Year-to-date for last year |
| **Sales LMTD** | `CALCULATE([Sales MTD], DATEADD(dim_date[Date],-1,MONTH))` | Sales Month-to-date for last month |
| **Sales Variance LY** | `[Total Sales] - [Total Sales LY]` | Absolute sales variance year-over-year |

---

## 8. Ranking Measures (2)

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Product Sales Rank** | `RANKX(ALL(dim_product[ProductName]), [Total Sales],,DESC,DENSE)` | Ranks products by total sales |
| **Store Sales Rank** | `RANKX(ALL(dim_store[Description]), [Total Sales],,DESC,Dense)` | Ranks stores by total sales |

---

## 9. Customer Measures (1)

| Measure | DAX Expression | Purpose |
|:---|:---|:---|
| **Avg Customer Age** | `AVERAGE(dim_customer[Age])` | Calculates the average age of purchasing customers |
