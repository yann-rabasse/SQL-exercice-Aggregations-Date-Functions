# Data Exploration and Cleaning

## Objective: 
The objective of this exercise is to query Circle’s stock data in SQL.

- [Data exploration and cleaning](#data-exploration-and-cleaning)
Understanding the structure of the tables,
Defining the primary keys,
Cleaning the data.

- [Data Enrichment](#data-enrichment)
adding new columns,
Descriptive dimensions,
Descriptive categories,
KPIs,
Metrics.

## Sources:
[Download Here](https://docs.google.com/spreadsheets/d/19cDDybWRQrWkGpGJfL6Yp63zkHXEtQRn_SNXySUm6sI/edit?gid=0#gid=0)


### 1) Data Exploration and Cleaning

1.1) Primary Key Identification
The first step is to understand the structure of the table and identify the primary key.

First, run SELECT queries to display the circle_stock and circle_sales tables.

Explore and understand the structure of these tables.
- How many columns are there?
- How many rows?
- Can you understand what data each of the columns contain?
```
SELECT
  *
FROM `course15.circle_stock`;

SELECT
  *
FROM `course15.circle_sales`
```


1.2) What is the primary key for the circle_sales table? Use a GROUP BY clause to confirm it is the primary key.

Reminder - What is a primary key?
A primary key is a UNIQUE identifier for each row in a table. It means that the primary key column (or a combination of columns) should not have any duplicate values.

Previously you were using the SELECT DISTINCT clause to search for duplicate values and WHERE column is NOT NULL to confirm that the column does not have any missing values. You can also achieve this by using the GROUP BY and HAVING clauses, namely, by grouping all the potential primary key values, then counting how many values there are in each of the groups, and finally filtering those groups that have 2 or more values (HAVING number_of_values_in_group >= 2).

Your goal is to have no primary key value groups that would have 2 or more occurrences, because that signals duplication of values in the column.

The query counts any duplicates when grouping by both the date_date and product_id columns. There are no duplicates, so they can be used as a primary key for the table.
```
 SELECT
   ### Key ###
   s.date_date
   ,s.product_id
   ###########
   ,count(*) AS nb
 FROM `course15.circle_sales` AS s
 GROUP BY
   date_date
   ,product_id
 HAVING nb>=2
 ORDER BY nb DESC
```

Now, use the same process to check if model is the primary key of the circle_stock table.
```
SELECT
  ### Key ###
  t.model
  ###########
  ,count(*) AS nb
FROM `course15.circle_stock` AS t
GROUP BY
  model
HAVING nb>=2
ORDER BY nb DESC
```
Let’s consider a specific model with duplicates, e.g. SH003P02-SL01-M. Select all the rows in the circle_stock table matching this model and order them by color and size.

Why there are duplicates in this model?
What is the primary key?

Stock

For this model, we see several rows corresponding to different colors and sizes. We can potentially assume that the combination of model, color and size is the primary key.
```
SELECT
  *
FROM `course15.circle_stock`
WHERE
  model = "SH003P02-SL01-M"
ORDER BY
  color
  ,size
```

Could the combination of model, color and size columns be the primary key of the circle_stock table?

The query counts possible duplicates in the combination of model, color and size columns. There are no duplicates. The model, color and size combination is a primary key for the circle_stock table. However, it is not very practical to have three columns to identify a product and a row.
```
SELECT
  ### Key ###
  t.model
  ,t.color
  ,t.size
  ###########
  ,count(*) AS nb
FROM `course15.circle_stock` AS t
GROUP BY
  model
  ,color
  ,size
HAVING nb>=2
ORDER BY nb DESC
```

Note</u> *- primary_key process

Check that there are no duplicates for the primary key (use GROUP BY)

If you have duplicates in some potential primary key columns, analyze specific cases to understand why:

Should you add another column to the primary key?

Is it just a bug in the raw data that we should fix, e.g., missing values in one of the columns?

Is it just a junk table without a good structure and an identifiable primary key?

The primary key can be composed of one or more columns. In general, it is a good practice to use the ID or a combination of IDs as the primary key (unique identifier of an object) rather than a name that is more likely to have duplicate values.

It is a good practice to put the primary key as the first column of the table and to identify it with comment lines in queries.

Use # or -- to comment line

### 1.2) Table Cleaning

Now that we have identified the primary key let’s clean up the table!

Return a new product_id column from the circle_stock table by concatenating the model, color and size columns. Use the “AS” statement to give this new column a name- product_id. Your concatenated product_id should follow this pattern - BR001P1-JL02-W_BL_S. Double check the returned results before you follow with the next steps.
```
SELECT
### Key ###
  t.model AS product_model
  ,t.color AS product_color
  ,t.size AS product_size
  ###########
  ,CONCAT(t.model,"_",t.color,"_",t.size) AS product_id
  -- name
  ,t.model_name
  ,t.color_name AS color_description
  -- product info --
  ,t.new AS product_new
  ,t.price AS product_price
  -- stock metrics --
  ,t.forecast_stock AS forecasted_stock_level
  ,t.stock AS current_stock_level
FROM `course15.circle_stock` AS t
```

Check that product_id is a primary key for the circle_stock table. What can you tell?
```
SELECT
  ### Key ###
  CONCAT(t.model,"_",t.color,"_",t.size) AS product_id
  ###########
  ,count(*) AS nb
FROM `course15.circle_stock` AS t
GROUP BY
  product_id
HAVING nb>=2
ORDER BY nb DESC
```
We have 6 rows with a NULL value as products_id. Let’s understand why.

Identify the rows where product_id IS NULL.
```
SELECT
  ### Key ###
  t.model AS product_model
  ,t.color AS product_color
  ,t.size AS product_size
  ###########
  ,CONCAT(t.model,"_",t.color, "_",t.size) AS product_id
  -- name
  ,t.model_name
  ,t.color_name AS color_description
  -- product info --
  ,t.new AS product_new
  ,t.price AS product_price
  -- stock metrics --
  ,t.forecast_stock AS forecasted_stock_level
  ,t.stock AS current_stock_level
FROM `course15.circle_stock` AS t
WHERE TRUE
  AND CONCAT(t.model,"_",t.color, "_",t.size) IS NULL
```
We have 6 rows with a NULL value as products_id. This is due to the size value being NULL for products that have no size information. When we use CONCAT() on a NULL value, the result is also NULL. To correct this, we should give a default value for products that have no size information.

We want to fill NULL values in the size column so that there are no NULL values in our new product_id column. Use the IFNULL function and set “no-size” to be the default value for the size column. You should not have any more NULL values in product_id. Now, is the product_id column a primary key for the circle_stock table?

💁🏽 Note - If you don’t know the IFNULL() function, Google it 😉

```
SELECT
  ### Key ###
  t.model AS product_model
  ,t.color AS product_color
  ,t.size AS product_size
  ###########
  ,CONCAT(t.model,"_",t.color,"_",IFNULL(t.size,"no-size")) AS product_id
  -- name
  ,t.model_name
  ,t.color_name AS color_description
  -- product info --
  ,t.new AS product_new
  ,t.price AS product_price
  -- stock metrics --
  ,t.forecast_stock AS forecasted_stock_level
  ,t.stock AS current_stock_level
FROM `course15.circle_stock` AS t
WHERE TRUE
  AND CONCAT(t.model,"_",t.color, "_",t.size) IS NULL
```
It worked as we don’t have any NULL product_id values anymore. Let’s see if this corrected product_id column is a primary key.
```
SELECT
  ### Key ###
  CONCAT(t.model,"_",t.color,"_",IFNULL(t.size,"no-size")) AS product_id
  ###########
  ,count(*) AS nb
FROM `course15.circle_stock` AS t
GROUP BY
  product_id
HAVING nb>=2
ORDER BY nb DESC
```
No duplication. New product_id is a primary key

### 2) Data Enrichment
In this part of the exercise, the objective is to transform the raw circle_stock table into a clean and enriched circle_stock_kpi table.

Typically, we would add all of our new columns in one SQL query without lots of intermediate tables, but for practice we’re going to add and save them one by one.

2.1) Add a product_name column
We want to add the product_name column which is the concatenation of the model_name, the color_name and size in the following format: model_name + " " + color_name + " - Size " + size (e.g. Brassière Level Up Black - Size L).

Select all the columns from the circle_stock table.

Add the product_id column.

Add the new product_name column using the CONCAT() function. (Don’t forget to handle NULL values in the size column 😉).

Check your product_name column follows the above format before you continue with the next steps.

Run the query and save the result of the query in a new circle_stock_name table. (For this, you can use the BigQuery Save Results to Table functionality).

```
 SELECT
   ### Key ###
   CONCAT(t.model,"_",t.color,"_",IFNULL(t.size,"no-size")) AS product_id
   ###########
   ,t.model AS product_model
   ,t.color AS product_color
   ,t.size AS product_size
   -- name
   ,t.model_name
   ,t.color_name AS color_description
   ,CONCAT(t.model_name," ",t.color_name,IF(t.size IS NULL,"",CONCAT(" - Size ",t.size))) AS product_name
   -- product info --
   ,t.new AS product_new
   ,t.price AS product_price
   -- stock metrics --
   ,t.forecast_stock AS forecasted_stock_level
   ,t.stock AS current_stock_level
 FROM `course15.circle_stock` AS t
 WHERE TRUE
 ORDER BY product_id
```
2.2) Add a model_type column

The purchase team wants to analyze the stock and filter it by different model types. They want to classify the models into the following model_type:

Accessories for “Tour de cou” (neckband in 🇬🇧), “Tapis” (mat in 🇬🇧) or “Gourde” (flask in 🇬🇧)

T-shirt for “T-shirt”

Crop-top for “Brassière” (bra in 🇬🇧) and “Crop-top”

Legging for “Legging”

Short for “Short”

Top for “Débardeur” or “Haut” (tank top in 🇬🇧)

Using CASE WHEN and REGEXP_CONTAINS(), create a new model_type column from the circle_stock_name table

Before we store this result, check that each product has an associated model_type. If not, you’ll need to find a solution!

Help - You can use the REPLACE() function to remove the accent of è in model_name.

```
 SELECT
   *
 FROM `course15.circle_stock_cat`
 WHERE
   model_type IS NULL
```
Store the result in a new circle_stock_cat table containing all the same columns as the circle_stock_name table + the new model_type column.

Help - To avoid typographical problems due to capitalization, you should also use the LOWER() function.
BRASSIERE Yvette has no accent on the è. There are several ways to correct this. Either by adding brassiere without è in the REGEXP formula, or by using REPLACE to replace è with e in model_name.
```
 SELECT
   ### Key ###
   product_id
   ###########
   ,product_model
   ,product_color
   ,product_size
    -- category
    ,CASE
      WHEN REGEXP_CONTAINS(LOWER(model_name),'t-shirt') THEN 'T-shirt'
      WHEN REGEXP_CONTAINS(LOWER(model_name),'short') THEN 'Short'
      WHEN REGEXP_CONTAINS(LOWER(model_name),'legging') THEN 'Legging'
      WHEN REGEXP_CONTAINS(REPLACE(LOWER(model_name),'è','e'), 'brassiere|crop-top') THEN 'Crop-top'
      WHEN REGEXP_CONTAINS(LOWER(model_name),'débardeur|haut') THEN 'Top'
      WHEN REGEXP_CONTAINS(LOWER(model_name),'tour de cou|tapis|gourde') THEN 'Accessories'
     ELSE NULL
   END AS model_type
   -- name
   ,model_name
   ,color_description
   ,product_name
   -- product info --
   ,product_new
   ,product_price
   -- stock metrics --
   ,forecasted_stock_level
   ,current_stock_level
 FROM `course15.circle_stock_name`
 WHERE TRUE
 ORDER BY product_id
```

2.3) Add an in_stock column for the shortage rate
The purchase team wants to analyze the shortage rate and the stock value.

Return a column in_stock from circle_stock_cat which is 0 if the product is out of stock and 1 when in stock.

This time let’s not save it immediately to a new table, instead we’re going to add this column to our next sql query (at the same time as we add stock_value).

```
 SELECT
   ### Key ###
   product_id
   ###########
   ,product_model
   ,product_color
   ,product_size
   -- category
   ,model_type
   -- name
   ,model_name
   ,color_description
   ,product_name
   -- product info --
   ,product_new
   ,product_price
   -- stock metrics --
   ,forecasted_stock_level
   ,current_stock_level
   ,IF(stock>0,1,0) AS in_stock
 FROM `course15.circle_stock_cat`
```

2.4) Add a stock_value column
From circle_stock_cat add a new column stock_value and save the results in a new circle_stock_kpi table. Don’t forget to add the in_stock column as well.

Your new circle_stock_kpi table should include all the columns from circle_stock_cat + the in_stock and stock_value columns.

Note - You can use the ROUND() function to limit the number of digits and make the result easier to read. However, the result will be approximate.
```
 SELECT
   ### Key ###
   product_id
   ###########
   ,product_model
   ,product_color
   ,product_size
   -- category
   ,model_type
   -- name
   ,model_name
   ,color_description
   ,product_name
   -- product info --
   ,product_new
   -- stock metrics --
   ,forecasted_stock_level
   ,current_stock_level
   ,IF(stock>0,1,0) AS in_stock
   -- value
   ,product_price
   ,ROUND(stock*price,2) AS stock_value
 FROM `course15.circle_stock_cat`
 ORDER BY product_id
```
























