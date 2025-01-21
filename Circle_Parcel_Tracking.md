Objective
🎯 The objective of this exercise is to query Circle’s parcel data in SQL to perform relevant analyses.

The different steps of the exercise are:
Data exploration: Understand the structure of the tables and define the primary keys.

Data enrichment: Add dimensions and KPIs needed to analyze parcel delivery

Data aggregation & analysis: Perform the aggregation to analyze parcel delivery

Data Import
Copy the 15 - circle_parcel Google Sheet on your drive

In the course15 dataset, create the cc_parcel table in your BigQuery project from parcel_sheet

In the course15 dataset, create the cc_parcel_product table in your BigQuery project from parcel_product_sheet

💁🏽 _Help_ - Google documentation to create a table from Google Sheet. In the advanced options, don’t forget to add 1 to the Header rows to skip option.

📚 Data dictionary
1) Data Exploration
Explore and understand the structure of both tables. How many columns are there? How many rows? Can you understand what data each of the columns contain?

What is the primary key of the cc_parcel table?

What is the primary key of cc_parcel_product table?

2) Data Enrichment
2.1) Parcel Status
The parcel may be at different stages of the delivery process:

If date_cancelled is not null, it means that the status of the parcel is Cancelled.

If date_shipping is null (and date_cancelled is also null), the parcel has not been shipped from the warehouse. It is in preparation in the warehouse. The status of the parcel is In progress.

If the date_delivery is null (and the date_shipping is not null and the date_cancelled is null), the parcel is in transit by the carrier. It has already been shipped but not yet delivered. The status of the parcel is In Transit.

If the date_delivery is not null, the parcel has already been delivered. The status of the parcel is Delivered.

In the cc_parcel table, add a new status column that contains the delivery status of the parcel by using the CASE WHEN clause. Save the results in a new table cc_parcel_kpi.
2.2) Delivery KPI
The Logistics & Shipping team wants to calculate and control:

expedition_time - time between purchase and shipping

transport_time - time between purchase and delivery

delivery_time - time between shipping and delivery

❓Refresher on KPI definitions for logistics


Let’s create some new fields!

First, we need to make sure that the date_purchase, date_shipping, date_delivery and date_cancelled columns are in the correct format.

Check the cc_parcel_kpi schema in BigQuery to confirm this. If the dates are not in date format, use the PARSE_DATE() function to format them. Save the data with the correctly formatted columns as cc_parcel_kpi. (This is the same name as the table we created in the previous step, so you will need to overwrite your existing table!)

💁🏽 Help - How to use PARSE_DATE()?
Add the 3 columns expedition_time, transport_time and delivery_time to the cc_parcel_kpi table and save the results as cc_parcel_kpi. Use DATE_DIFF(). (Same as before, you will need to overwrite your existing table!)

💁🏽 Help - How to use DATE_DIFF()?
3) Data Aggregation & Analysis
The logistics team would like to obtain statistics on shipments, transportation and delivery times in order to analyze them at different levels of granularity.

Run some aggregation functions on the cc_parcel_kpi table to obtain the following global statistics:

number of parcels

average shipping time with 2 digits after the decimal point

average transport time with 2 digits after the decimal point

average delivery time with 2 digits after the decimal point

Save the result in cc_parcel_kpi_global.

💁🏽 Help - Use ROUND() to limit the number of digits after decimal point

Calculate the same statistics for each carrier separately using the cc_parcel_kpi table. How do the different carriers compare against each other? Save the results in cc_parcel_kpi_transporter table.

Calculate the same statistics for each priority level separately using the cc_parcel_kpi table. How do the different times compare among different levels of parcel priority? Save the results in cc_parcel_kpi_priority.

We would like to see the ratio between the delivery time and the transport time. This way, we will know the impact of the transport time across the different levels of parcel priority. Re-use the same query as in the step above, but add a new column that uses SAFE_DIVIDE to compute the ratio. Save the results in cc_parcel_kpi_priority.

The logistics team would like to see the evolution of this global KPI over the months. Add a new month_purchase column to the cc_parcel_kpi table that extracts the month from the date_purchase column using EXTRACT() function. Save the results as cc_parcel_kpi. (Again, you will need to overwrite an existing table.)

💁🏽 Help - How to use EXTRACT()?
By querying the new cc_parcel_kpi table, display the evolution of the average shipping, transport and delivery times across all the months. How do the metrics change? Save the results in the cc_parcel_kpi_month table.

4) Delay
A parcel is considered late if the total delivery_time is longer than 5 days. The logistics team would like to analyze the delay_rate of parcels on a global level.

Using various transformation steps, create the analysis for the logistics team.

💁🏽 Help - step by step process
