# Introduction
In the previous exercise, you have transformed the circle_stock table into a clean and enhanced circle_stock_kpi table.

The enriched circle_stock_kpi table can now be used to perform an in-depth analysis of stock statistics (stock, shortage, stock_value) by model_name, model_type, product_name, size.

In this exercise, you will continue to build on the data cleaning and data enrichment you did in the previous exercise.

The objective is to address the needs of the purchasing team.
- Display stock and shortage statistics:
   - globally
   - by model_type
   - by model
   - by product
- Identify the top products
- Calculate the estimated number of days of stock remaining for the top products
To complete this analysis, we will perform data aggregation, which is the equivalent of the pivot table in Google Sheets.

### 1) Stock Aggregation
Run aggregation functions (COUNT(), SUM(), AVG()) on the circle_stock_kpi table to return the following values:
- the total number of products
- the total number of products in stock
- the shortage rate
- the total stock value
- the total stock

```
 SELECT
   COUNT(in_stock) AS nb_products
   ,SUM(in_stock) AS nb_products_in_stock
   ,ROUND(AVG(1-in_stock),3) AS shortage_rate
   ,SUM(stock_value) AS stock_value
   ,SUM(stock) AS total_stock
 FROM `course15.circle_stock_kpi`
```

Return the same metrics, aggregated by model_type, using the GROUP BY clause.
```
SELECT
  model_type
  ,COUNT(in_stock) AS nb_products
  ,SUM(in_stock) AS nb_products_in_stock
  ,ROUND(AVG(1-in_stock),3) AS shortage_rate
  ,SUM(stock_value) AS stock_value
  ,SUM(stock) AS total_stock
FROM `course15.circle_stock_kpi`
GROUP BY model_type
```

Finally, return the same metrics aggregated by model_type and model_name. Sort the results by total_stock_value, calculated previously, in descending order.
```
SELECT
  model_type
  ,model_name
  ,COUNT(in_stock) AS nb_products
  ,SUM(in_stock) AS nb_products_in_stock
  ,ROUND(AVG(1-in_stock),3) AS shortage_rate
  ,SUM(stock_value) AS stock_value
  ,SUM(stock) AS total_stock
FROM `course15.circle_stock_kpi`
GROUP BY
   model_type
  ,model_name
ORDER BY stock_value DESC
```

### 2) Sales Aggregation
The purchasing team would like to identify the top 10 products sold in order to monitor stock shortages for these products alongside the average sales volume last month for each product, giving us an estimate the remaining inventory by product in days until out of stock.

This could help better estimate future sales and reduce lost revenue.

#### 2.1) Determine top products
Using the circle_sales table, list the 10 products that were the most sold in terms of quantity. Save the result in a new top_products table.
```
SELECT
  product_id
  ,SUM(qty) AS qty
FROM `course15.circle_sales` AS s
GROUP BY product_id
ORDER BY qty DESC
LIMIT 10
```
Add a new column top_products to the circle_stock_kpi table that contains the value 0 in each row. Save the query results in a new circle_stock_kpi_top table.

```
SELECT
  *
  ,0 AS top_products
FROM `course15.circle_stock_kpi`
```

Update the top_products column of the circle_stock_kpi_top table by setting it to 1 if the product can be found in the top_products table.
Please use the following query to perform the update (you will be introduced to the detailed UPDATE concept in the Advanced SQL lecture next week)
Help
–> Make an update request using FROM between circle_stock_kpi_top and top_products <!– - Use a subquery in the WHERE clause of the update. It will filter out all the products_id from circle_stock_kpi_top which are in the top_products table.
```
UPDATE `course15.circle_stock_kpi_top` AS s
SET s.top_products = 1
FROM `course15.top_products` AS p
WHERE
  s.product_id = p.product_id
```

#### 2.2) Estimate daily product sales and days of stock remaining
Assuming it is October 1, 2021:
From the circle_sales table, calculate the total quantity sold for each product over the last 91 days and name the new column qty_91.

Calculate another column avg_daily_qty_91 that contains the average daily sales over the last 91 days for each of the products.

Store the result in a new table circle_sales_daily.
Hint: You can select the time range of 91 days by using the DATE_SUB function.
```
 SELECT
   product_id
   ,SUM(qty) AS qty_91
   ,ROUND(SUM(qty)/91,2) AS avg_daily_qty_91
 FROM `course15.circle_sales`
 WHERE TRUE
   AND date_date >= DATE_SUB('2021-10-01',INTERVAL 91 DAY)
 GROUP BY product_id
```

The purchasing team is particularly interested in top products with low stock (i.e., less than 50) to know if they need to reorder these products.
- Would you prefer to use stock or forecast_stock to assess low inventory?
- Identify these products from the circle_stock_kpi_top table and manually note their products_id.

We use forecast_stock because if the purchase team has already ordered products, there is no need to re-order them.

```
SELECT
  ### Key ###
  product_id
  ###########
  ,product_name
  ,top_products
  ,stock
  ,forecast_stock
FROM `course15.circle_stock_kpi_top`
WHERE TRUE
  AND top_products=1
  AND forecast_stock<50
ORDER BY product_id;
```


For the identified products:
- Compare the avg_daily_qty_91 from the circle_sales_daily table with the forecast_stock from the circle_stock_kpi_top table.
- Calculate the estimated nb_days of stock remaining.
To easily compare 2 queries on BigQuery, you can split one of them on the right pane of the screen and leave the other on the left pane - split tabs documentation
```
SELECT
  product_id
  ,product_name
  ,stock
  ,forecast_stock
FROM `course15.circle_stock_kpi`
WHERE TRUE
  AND product_id IN ("TS001BTB-MAM01_U_WH_XS", "TS001BTB-MAM01_U_BL_M", "TS001BTB-MAM01_U_WH_M")
ORDER BY product_id;
```

```
SELECT
  product_id
  ,qty_91
  ,avg_daily_qty_91
FROM `course15.circle_sales_daily` AS s
WHERE TRUE
  AND product_id IN ("TS001BTB-MAM01_U_WH_XS", "TS001BTB-MAM01_U_BL_M", "TS001BTB-MAM01_U_WH_M")
ORDER BY product_id;
```

![02-Aggregation-SQL-Pivot-Table-Circle-Inventory-Management-asset-5-Untitled](https://github.com/user-attachments/assets/44bf5ef8-75b9-4155-9542-e7c7bc78ad11)

We calculate nb_days_remaining for each product nb_day_estimated = stock / avg_daily_qty

- TS001BTB-MAM01_U_WH_XS —> 48 / 32.52 = 1.48 day
- TS001BTB-MAM01_U_BL_M —> 5 / 18.71 = 0.27 day
- TS001BTB-MAM01_U_WH_M —> 35 / 45.78 = 0.76 day The products have very low stock. The purchasing team has to reorder them very quickly.



