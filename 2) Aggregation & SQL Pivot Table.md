






1) Stock Aggregation
Run aggregation functions (COUNT(), SUM(), AVG()) on the circle_stock_kpi table to return the following values:

the total number of products

the total number of products in stock

the shortage rate

the total stock value

the total stock

Return the same metrics, aggregated by model_type, using the GROUP BY clause.

Finally, return the same metrics aggregated by model_type and model_name. Sort the results by total_stock_value, calculated previously, in descending order.

2) Sales Aggregation
The purchasing team would like to identify the top 10 products sold in order to monitor stock shortages for these products alongside the average sales volume last month for each product, giving us an estimate the remaining inventory by product in days until out of stock.

This could help better estimate future sales and reduce lost revenue.

2.1) Determine top products
Using the circle_sales table, list the 10 products that were the most sold in terms of quantity. Save the result in a new top_products table.

Add a new column top_products to the circle_stock_kpi table that contains the value 0 in each row. Save the query results in a new circle_stock_kpi_top table.

Update the top_products column of the circle_stock_kpi_top table by setting it to 1 if the product can be found in the top_products table.

Please use the following query to perform the update (you will be introduced to the detailed UPDATE concept in the Advanced SQL lecture next week)
🆘 Help
–> Make an update request using FROM between circle_stock_kpi_top and top_products <!– - Use a subquery in the WHERE clause of the update. It will filter out all the products_id from circle_stock_kpi_top which are in the top_products table.

2.2) Estimate daily product sales and days of stock remaining
Assuming it is October 1, 2021:
From the circle_sales table, calculate the total quantity sold for each product over the last 91 days and name the new column qty_91.

Calculate another column avg_daily_qty_91 that contains the average daily sales over the last 91 days for each of the products.

Store the result in a new table circle_sales_daily.

🆘 Help


The purchasing team is particularly interested in top products with low stock (i.e., less than 50) to know if they need to reorder these products.
Would you prefer to use stock or forecast_stock to assess low inventory?

Identify these products from the circle_stock_kpi_top table and manually note their products_id.

For the identified products:
Compare the avg_daily_qty_91 from the circle_sales_daily table with the forecast_stock from the circle_stock_kpi_top table.

Calculate the estimated nb_days of stock remaining.

_Help_- To easily compare 2 queries on BigQuery, you can split one of them on the right pane of the screen and leave the other on the left pane - split tabs documentation
