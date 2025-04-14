# Explore a Relational Data Warehouse with Azure Synapse Analytics

This lab demonstrates how to use a **dedicated SQL pool** in **Azure Synapse Analytics** to store and query data in a **relational data warehouse**. You’ll explore the star and snowflake schema models and write queries that aggregate data across fact and dimension tables.

---

## Prerequisites

- An **Azure subscription** with **administrative access**

---

## Provision an Azure Synapse Analytics Workspace

1. Sign into the Azure portal: https://portal.azure.com  
2. Open the **Cloud Shell** using the `[>_]` button next to the search bar.
3. Choose **PowerShell** and create storage if prompted.
4. Run the following commands in the Cloud Shell to clone the required repository:

   ```powershell
   rm -r dp203 -f
   git clone https://github.com/MicrosoftLearning/Dp-203-azure-data-engineer dp203
   cd dp203/Allfiles/labs/08
   ./setup.ps1
   ```
5. Select your subscription if prompted and provide a password when requested.
6. Wait for the script to complete (approx. 15 minutes).

## Start the Dedicated SQL Pool
1.In the Azure portal, navigate to the dp203-xxxxxxx resource group.
2.Select your Synapse workspace and click Open Synapse Studio.
3.In Synapse Studio, go to the Manage page and start the sqlxxxxxxx SQL pool.

## Explore the Data Warehouse Schema
On the Data page in Synapse Studio, expand:

  Workspace ➜ SQL database ➜ sqlxxxxxxx ➜ Tables

### Key Tables
-FactInternetSales: Contains numeric metrics and foreign keys.
-DimPromotion: Includes a surrogate PromotionKey and business AlternateKey.
-DimProduct ➜ DimProductSubcategory ➜ DimProductCategory: Example of a snowflake schema.
-DimDate: Time dimension with attributes like month, year, day name, etc.
## Query the Data Warehouse
### Analyze Internet Sales
Create a new SQL script called Analyze Internet Sales.
```sql
SELECT d.CalendarYear AS Year,
       SUM(i.SalesAmount) AS InternetSalesAmount
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
GROUP BY d.CalendarYear
ORDER BY Year;
```
Add month breakdown:
```sql
SELECT d.CalendarYear AS Year,
       d.MonthNumberOfYear AS Month,
       SUM(i.SalesAmount) AS InternetSalesAmount
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
GROUP BY d.CalendarYear, d.MonthNumberOfYear
ORDER BY Year, Month;
```
Add geographic region:
```sql
SELECT d.CalendarYear AS Year,
       g.EnglishCountryRegionName AS Region,
       SUM(i.SalesAmount) AS InternetSalesAmount
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
GROUP BY d.CalendarYear, g.EnglishCountryRegionName
ORDER BY Year, Region;
```
Add product category:
```sql
SELECT d.CalendarYear AS Year,
       pc.EnglishProductCategoryName AS ProductCategory,
       g.EnglishCountryRegionName AS Region,
       SUM(i.SalesAmount) AS InternetSalesAmount
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
JOIN DimProduct AS p ON i.ProductKey = p.ProductKey
JOIN DimProductSubcategory AS ps ON p.ProductSubcategoryKey = ps.ProductSubcategoryKey
JOIN DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey
GROUP BY d.CalendarYear, pc.EnglishProductCategoryName, g.EnglishCountryRegionName
ORDER BY Year, ProductCategory, Region;
```
## Use Ranking Functions
### Row Number and Aggregates by Region
```sql
SELECT g.EnglishCountryRegionName AS Region,
       ROW_NUMBER() OVER(PARTITION BY g.EnglishCountryRegionName ORDER BY i.SalesAmount ASC) AS RowNumber,
       i.SalesOrderNumber AS OrderNo,
       i.SalesOrderLineNumber AS LineItem,
       i.SalesAmount AS SalesAmount,
       SUM(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
       AVG(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionAverage
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
WHERE d.CalendarYear = 2022
ORDER BY Region;
```
### Rank Cities by Sales
```sql
SELECT g.EnglishCountryRegionName AS Region,
       g.City,
       SUM(i.SalesAmount) AS CityTotal,
       SUM(SUM(i.SalesAmount)) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
       RANK() OVER(PARTITION BY g.EnglishCountryRegionName ORDER BY SUM(i.SalesAmount) DESC) AS RegionalRank
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
GROUP BY g.EnglishCountryRegionName, g.City
ORDER BY Region;
```
## Use Approximate Count
### Exact count per year
```sql
SELECT d.CalendarYear AS CalendarYear,
       COUNT(DISTINCT i.SalesOrderNumber) AS Orders
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
GROUP BY d.CalendarYear
ORDER BY CalendarYear;
```
### Approximate count per year
```sql
SELECT d.CalendarYear AS CalendarYear,
       APPROX_COUNT_DISTINCT(i.SalesOrderNumber) AS Orders
FROM FactInternetSales AS i
JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
GROUP BY d.CalendarYear
ORDER BY CalendarYear;
```
## Wrap-up
When finished, return to the Manage page in Synapse Studio and pause the sqlxxxxxxx dedicated SQL pool to avoid unnecessary charges.
## Resources
-Azure Synapse Analytics Documentation
-Ranking Functions (T-SQL)
-APPROX_COUNT_DISTINCT Function




