## Objective
The objective of this exercise is to query Circle’s parcel data in SQL to perform relevant analyses.

The different steps of the exercise are:
Data exploration: Understand the structure of the tables and define the primary keys.
Data enrichment: Add dimensions and KPIs needed to analyze parcel delivery
Data aggregation & analysis: Perform the aggregation to analyze parcel delivery

## Sources:
[Download Here](#https://docs.google.com/spreadsheets/d/1qhHVVdi6Z8PnD62QJtbnEjyX8d35sho573XwoasrYBk/edit?gid=0#gid=0)

## 1) Data Exploration
Explore and understand the structure of both tables. How many columns are there? How many rows? Can you understand what data each of the columns contain?
```
SELECT
  *
FROM `course15.cc_parcel`;

SELECT
  *
FROM `course15.cc_parcel_product`
```

What is the primary key of the cc_parcel table?
```
SELECT
  ### Key ###
  parcel_id
  ###########
 ,count(*) AS nb
FROM `course15.cc_parcel`
GROUP BY
  parcel_id
HAVING nb>=2
ORDER BY nb DESC
```

What is the primary key of cc_parcel_product table?
```
SELECT
  ### Key ###
  parcel_id
  ,model_name
  ###########
  ,count(*) AS nb
FROM `course15.cc_parcel_product`
GROUP BY
  parcel_id
  ,model_name
HAVING nb>=2
ORDER BY nb DESC
```
Note: The primary key is parcel_id + model_name. It is not ideal to have model_name as a string. It would have been better to have the model_id but it is not available in the table.


## 2) Data Enrichment
### 2.1) Parcel Status
The parcel may be at different stages of the delivery process:

If date_cancelled is not null, it means that the status of the parcel is Cancelled.

If date_shipping is null (and date_cancelled is also null), the parcel has not been shipped from the warehouse. It is in preparation in the warehouse. The status of the parcel is In progress.

If the date_delivery is null (and the date_shipping is not null and the date_cancelled is null), the parcel is in transit by the carrier. It has already been shipped but not yet delivered. The status of the parcel is In Transit.

If the date_delivery is not null, the parcel has already been delivered. The status of the parcel is Delivered.

In the cc_parcel table, add a new status column that contains the delivery status of the parcel by using the CASE WHEN clause. Save the results in a new table cc_parcel_kpi.
```
SELECT
  ### Key ###
  parcel_id
  ###########
  -- parcel infos
  ,parcel_tracking
  ,transporter
  ,priority
  -- date --
  ,date_purchase
  ,date_shipping
  ,date_delivery
  ,date_cancelled
  -- status --
  ,CASE
    WHEN date_cancelled IS NOT NULL THEN 'Cancelled'
    WHEN date_shipping IS NULL THEN 'In Progress'
    WHEN date_delivery IS NULL THEN 'In Transit'
    WHEN date_delivery IS NOT NULL THEN 'Delivered'
    ELSE NULL
  END AS status
FROM
  `course15.cc_parcel`
```


### 2.2) Delivery KPI
The Logistics & Shipping team wants to calculate and control:

- expedition_time - time between purchase and shipping
- transport_time - time between purchase and delivery
- delivery_time - time between shipping and delivery

#### KPI Definitions
List of Parcels
Number of parcels by status type. Shows the current status of shipping operations or delivery.

Shipping Time: 
measures supply-chain performance, time between order reception date and shipping date.
DATE_DIFF(order_date, shipping_date)

Delivery Time: 
measures carrier performance, time between shipping date and delivery date.
DATE_DIFF(shipping_date, delivery_date)

Refund Rate
The ratio between the refund amount and the total turnover. The ratio can also be calculated on the number of parcels and other error metrics (delay, broken/missing products, etc.)
SUM(refund_amount)/SUM(turnover)

Let’s create some new fields!

First, we need to make sure that the date_purchase, date_shipping, date_delivery and date_cancelled columns are in the correct format.

Check the cc_parcel_kpi schema in BigQuery to confirm this. If the dates are not in date format, use the PARSE_DATE() function to format them. Save the data with the correctly formatted columns as cc_parcel_kpi. (This is the same name as the table we created in the previous step, so you will need to overwrite your existing table!)

Help - How to use PARSE_DATE()?
[DOC HERE](#https://cloud.google.com/bigquery/docs/reference/standard-sql/date_functions#parse_date)
[FORMAT_STRING](#https://cloud.google.com/bigquery/docs/reference/standard-sql/format-elements#format_elements_date_time)
documentation to create the correct format string required as the first argument to PARSE_DATE()
```
SELECT
  ### Key ###
  parcel_id
  ###########
  -- parcel infos
  ,parcel_tracking
  ,transporter
  ,priority
  -- date --
  ,PARSE_DATE("%B %e, %Y", date_purchase) AS date_purchase
  ,PARSE_DATE("%B %e, %Y", date_shipping) AS date_shipping
  ,PARSE_DATE("%B %e, %Y", date_delivery) AS date_delivery
  ,PARSE_DATE("%B %e, %Y", date_cancelled) AS date_cancelled
  -- status --
  ,CASE
    WHEN date_cancelled IS NOT NULL THEN 'Cancelled'
    WHEN date_shipping IS NULL THEN 'In Progress'
    WHEN date_delivery IS NULL THEN 'In Transit'
    WHEN date_delivery IS NOT NULL THEN 'Delivered'
    ELSE NULL
  END AS status
FROM `course15.cc_parcel`
```


Add the 3 columns expedition_time, transport_time and delivery_time to the cc_parcel_kpi table and save the results as cc_parcel_kpi. Use DATE_DIFF(). (Same as before, you will need to overwrite your existing table!)

Help - How to use DATE_DIFF()?

[DOC HERE](#https://cloud.google.com/bigquery/docs/reference/standard-sql/date_functions#date_diff)
To use DATE_DIFF, the arguments must be in date format. Use the previous question to correctly format the date in the date_diff function.
```
SELECT
  ### Key ###
  parcel_id
  ###########
  -- parcel infos
  ,parcel_tracking
  ,transporter
  ,priority
  -- date --
  ,PARSE_DATE("%B %e, %Y", date_purchase) AS date_purchase
  ,PARSE_DATE("%B %e, %Y", date_shipping) AS date_shipping
  ,PARSE_DATE("%B %e, %Y", date_delivery) AS date_delivery
  ,PARSE_DATE("%B %e, %Y", date_cancelled) AS date_cancelled
  -- status --
  ,CASE
    WHEN date_cancelled IS NOT NULL THEN 'Cancelled'
    WHEN date_shipping IS NULL THEN 'In Progress'
    WHEN date_delivery IS NULL THEN 'In Transit'
    WHEN date_delivery IS NOT NULL THEN 'Delivered'
    ELSE NULL
  END AS status
  -- time --
  ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_shipping),PARSE_DATE("%B %e, %Y", date_purchase),DAY) AS expedition_time
  ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_delivery),PARSE_DATE("%B %e, %Y", date_shipping),DAY) AS transport_time
  ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_delivery),PARSE_DATE("%B %e, %Y", date_purchase),DAY) AS delivery_time
FROM `course15.cc_parcel`
```

## 3) Data Aggregation & Analysis
The logistics team would like to obtain statistics on shipments, transportation and delivery times in order to analyze them at different levels of granularity.

Run some aggregation functions on the cc_parcel_kpi table to obtain the following global statistics:
- number of parcels
- average shipping time with 2 digits after the decimal point
- average transport time with 2 digits after the decimal point
- average delivery time with 2 digits after the decimal point

Save the result in cc_parcel_kpi_global.
Help - Use ROUND() to limit the number of digits after decimal point
```
SELECT
  COUNT(*) AS nb_parcel
  ,ROUND(AVG(expedition_time),2) AS expedition_time
  ,ROUND(AVG(transport_time),2) AS transport_time
  ,ROUND(AVG(delivery_time),2) AS delivery_time
FROM `course15.cc_parcel_kpi`
```

Calculate the same statistics for each carrier separately using the cc_parcel_kpi table. How do the different carriers compare against each other? Save the results in cc_parcel_kpi_transporter table.
```
SELECT
  transporter
  ,COUNT(*) AS nb_parcel
  ,ROUND(AVG(expedition_time),2) AS expedition_time
  ,ROUND(AVG(transport_time),2) AS transport_time
  ,ROUND(AVG(delivery_time),2) AS delivery_time
FROM `course15.cc_parcel_kpi`
GROUP BY transporter
```
The preparation time is equivalent between the two carriers, which seems logical since the carrier does not intervene in the preparation of the parcel; it is the role of the warehouse. However, there is a difference with UPS, which has a lower delivery time and a lower number of parcels. This can be explained by UPS being the premium carrier used for express delivery while Colissimo is the standard carrier without express delivery.

Calculate the same statistics for each priority level separately using the cc_parcel_kpi table. How do the different times compare among different levels of parcel priority? Save the results in cc_parcel_kpi_priority.
```
SELECT
  priority
  ,COUNT(*) AS nb_parcel
  ,ROUND(AVG(expedition_time),2) AS expedition_time
  ,ROUND(AVG(transport_time),2) AS transport_time
  ,ROUND(AVG(delivery_time),2) AS delivery_time
FROM `course15.cc_parcel_kpi`
GROUP BY priority
```
The shipping time is considerably shorter for high and medium priority parcels than for low priority parcels. This means that at the warehouse, important parcels are processed faster. However, there is not much difference in transportation time. The carrier cannot prioritize the parcels based on their importance or simply doesn’t have the information.

We would like to see the ratio between the delivery time and the transport time. This way, we will know the impact of the transport time across the different levels of parcel priority. Re-use the same query as in the step above, but add a new column that uses SAFE_DIVIDE to compute the ratio. Save the results in cc_parcel_kpi_priority.
```
SELECT
  priority
  ,COUNT(*) AS nb_parcel
  ,ROUND(AVG(expedition_time),2) AS expedition_time
  ,ROUND(AVG(transport_time),2) AS transport_time
  ,ROUND(AVG(delivery_time),2) AS delivery_time
  ,SAFE_DIVIDE(ROUND(AVG(transport_time),2),ROUND(AVG(delivery_time),2)) AS ratio_transport_delivery
FROM `course15.cc_parcel_kpi`
GROUP BY priority
```

The logistics team would like to see the evolution of this global KPI over the months. Add a new month_purchase column to the cc_parcel_kpi table that extracts the month from the date_purchase column using EXTRACT() function. Save the results as cc_parcel_kpi. (Again, you will need to overwrite an existing table.)

Help - How to use EXTRACT()?
- To find the BigQuery documentation about EXTRACT —> Google it!
- To use EXTRACT(), the function argument has to be in the date format
```
SELECT
   ### Key ###
   parcel_id
   ###########
   -- parcel infos
   ,parcel_tracking
   ,transporter
   ,priority
   -- date --
   ,PARSE_DATE("%B %e, %Y", date_purchase) AS date_purchase
   ,PARSE_DATE("%B %e, %Y", date_shipping) AS date_shipping
   ,PARSE_DATE("%B %e, %Y", date_delivery) AS date_delivery
   ,PARSE_DATE("%B %e, %Y", date_cancelled) AS date_cancelled
   -- month --
   ,EXTRACT(MONTH FROM PARSE_DATE("%B %e, %Y", date_purchase)) AS month_purchase
   -- status --
   ,CASE
     WHEN date_cancelled IS NOT NULL THEN 'Cancelled'
     WHEN date_shipping IS NULL THEN 'In Progress'
     WHEN date_delivery IS NULL THEN 'In Transit'
     WHEN date_delivery IS NOT NULL THEN 'Delivered'
     ELSE NULL
   END AS status
   -- time --
   ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_shipping),PARSE_DATE("%B %e, %Y", date_purchase),DAY) AS expedition_time
   ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_delivery),PARSE_DATE("%B %e, %Y", date_shipping),DAY) AS transport_time
   ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_delivery),PARSE_DATE("%B %e, %Y", date_purchase),DAY) AS delivery_time
 FROM `course15.cc_parcel`
```

By querying the new cc_parcel_kpi table, display the evolution of the average shipping, transport and delivery times across all the months. How do the metrics change? Save the results in the cc_parcel_kpi_month table.
```
 SELECT
   month_purchase
   ,COUNT(*) AS nb_parcel
   ,ROUND(AVG(expedition_time),2) AS expedition_time
   ,ROUND(AVG(transport_time),2) AS transport_time
   ,ROUND(AVG(delivery_time),2) AS delivery_time
 FROM `course15.cc_parcel_kpi`
 GROUP BY month_purchase
 ORDER BY month_purchase
```

## 4) Delay
A parcel is considered late if the total delivery_time is longer than 5 days. The logistics team would like to analyze the delay_rate of parcels on a global level.

Using various transformation steps, create the analysis for the logistics team.

Help - step by step process
- Edit the query you wrote to create the cc_parcel_kpi table. Add the code for a new delay column.
- Use DATE_DIFF() function.
- NULL value in delay column if date_delivery is NULL.
- Add a new delay_rate column (average global delay) to the cc_parcel_kpi_global table.
  - First we add the delay column to cc_parcel_kpi table
  - If there is no delivery date, there is no delivery_time, so we want NULL for the delay.
```
SELECT
  ### Key ###
  parcel_id
  ###########
  -- parcel infos
  ,parcel_tracking
  ,transporter
  ,priority
  -- date --
  ,PARSE_DATE("%B %e, %Y", date_purchase) AS date_purchase
  ,PARSE_DATE("%B %e, %Y", date_shipping) AS date_shipping
  ,PARSE_DATE("%B %e, %Y", date_delivery) AS date_delivery
  ,PARSE_DATE("%B %e, %Y", date_cancelled) AS date_cancelled
  -- month --
  ,EXTRACT(MONTH FROM PARSE_DATE("%B %e, %Y", date_purchase)) AS month_purchase
  -- status --
  ,CASE
    WHEN date_cancelled IS NOT NULL THEN 'Cancelled'
    WHEN date_shipping IS NULL THEN 'In Progress'
    WHEN date_delivery IS NULL THEN 'In Transit'
    WHEN date_delivery IS NOT NULL THEN 'Delivered'
    ELSE NULL
  END AS status
  -- time --
  ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_shipping),PARSE_DATE("%B %e, %Y", date_purchase),DAY) AS expedition_time
  ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_delivery),PARSE_DATE("%B %e, %Y", date_shipping),DAY) AS transport_time
  ,DATE_DIFF(PARSE_DATE("%B %e, %Y", date_delivery),PARSE_DATE("%B %e, %Y", date_purchase),DAY) AS delivery_time
  -- delay
  ,IF(date_delivery IS NULL,NULL,IF(DATE_DIFF(PARSE_DATE("%B %e, %Y", date_delivery),PARSE_DATE("%B %e, %Y", date_purchase),DAY)>5,1,0))    AS delay
FROM `course15.cc_parcel`
```
Then we add the delay_rate to the cc_parcel_kpi_global table
```
SELECT
  COUNT(*) AS nb_parcel
  ,ROUND(AVG(expedition_time),2) AS expedition_time
  ,ROUND(AVG(transport_time),2) AS transport_time
  ,ROUND(AVG(delivery_time),2) AS delivery_time
  ,ROUND(AVG(delay),2) AS delay_rate
FROM `course15.cc_parcel_kpi`
```
The delay_rate is 12%. If we had more data, it could be interesting to analyze the evolution of the delay_rate over months to see if it improves or not.




