# Greenweez Sales
Greenweeez is an organic e-commerce website.

It is a B2C business selling health and lifestyle products.
Greenweez has many customers and it essential to keep this large customer base engaged by offering promotions and timely discounts.
Given this, it is essential to track the sales performance of different product categories and segment customers based on the categories of products purchased.

In this exercise, you will analyse Greenweez sales data. Your goal is to understand how sales are distributed among different product categories and customer segments.

## Sources 

## 1) Sales Aggregation
Apply aggregation functions to the gwz_sales table to display:
The total number of orders
The total number of products_id
The total number of customers_id
The sum of turnover
The sum of purchase cost
The sum of quantity
```
SELECT
COUNT(DISTINCT(orders_id)) AS nb_orders,
COUNT(DISTINCT(products_id)) AS nb_products,
COUNT(DISTINCT(customers_id)) AS nb_customers,
SUM(turnover) AS sum_turnover,
SUM(purchase_cost) AS sum_purchase_cost,
SUM(qty) AS sum_qty
FROM `course15.gwz_sales`
```

Calculate the same statistics for each category of category_1. Don’t forget to add the category_1 column in your SELECT clause to see which statistic belongs to which group.
```
SELECT
category_1,
COUNT(DISTINCT(orders_id)) AS nb_orders,
COUNT(DISTINCT(products_id)) AS nb_products,
COUNT(DISTINCT(customers_id)) AS nb_customers,
SUM(turnover) AS sum_turnover,
SUM(purchase_cost) AS sum_purchase_cost,
SUM(qty) AS sum_qty
FROM `course15.gwz_sales`
GROUP BY category_1
```

Order the categories of category_1 by the largest turnover.
```
SELECT
category_1,
COUNT(DISTINCT(orders_id)) AS nb_orders,
COUNT(DISTINCT(products_id)) AS nb_products,
COUNT(DISTINCT(customers_id)) AS nb_customers,
SUM(turnover) AS sum_turnover,
SUM(purchase_cost) AS sum_purchase_cost,
SUM(qty) AS sum_qty
FROM `course15.gwz_sales`
GROUP BY category_1
ORDER BY sum_turnover DESC
```

## 2) Delving deeper into the Product Categories
The sales team wants to know what the most sold product categories are in order to create new ads. Your job is to identify these top categories along with their number of customers.

Finding Top Categories
Focus on the category from category_1 with the largest turnover. Within this category, identify the categories from category_2 and category_3 with the largest turnover.
```
SELECT
category_2,category_3,
SUM(turnover) AS sum_turnover,
FROM `course15.gwz_sales`
WHERE category_1 = 'Bébé & Enfant'
GROUP BY category_2,category_3
ORDER BY sum_turnover DESC
```
Now that we have identified the most popular categories, we would like to understand if their popularity can be explained by many customers making one-time orders or by a high number of re-purchases.

If the category popularity is caused by a high number of re-purchases, we can assume that there is something in these products that is linked to customer satisfaction.

To run this analysis you should COUNT distinctly the number of orders and the number of customers in each category.
```
SELECT
category_2,category_3,
COUNT(DISTINCT orders_id) AS nb_orders,
COUNT(DISTINCT customers_id) AS nb_customers,
FROM `course15.gwz_sales`
WHERE category_1 = 'Bébé & Enfant'
GROUP BY category_2,category_3
```

Instead of displaying separate values for the number of orders and customers, compute the number of orders per customer instead. Order your results by the highest number of orders per customer.
```
SELECT
category_2,category_3,
COUNT(DISTINCT orders_id) AS nb_orders,
COUNT(DISTINCT customers_id) AS nb_customers,
(COUNT(DISTINCT orders_id)/COUNT(DISTINCT customers_id)) AS nb_orders_per_customer
FROM `course15.gwz_sales`
WHERE category_1 = 'Bébé & Enfant'
GROUP BY category_2,category_3
ORDER BY nb_orders_per_customer DESC
```
To conclude your analysis, the sales team would like to know the average purchase price and the number of distinct products for each sub-category of the category_1 = “Bébé & Enfant” (Baby & Children in 🇬🇧)
```
SELECT
category_2,category_3,
COUNT(DISTINCT products_id) AS nb_products,
AVG(purchase_cost) AS avg_purchase_cost
FROM `data-analytics-bootcamp-363212.course14.gwz_sales`
WHERE category_1 = 'Bébé & Enfant'
GROUP BY category_2,category_3
ORDER BY avg_purchase_cost DESC
```
























