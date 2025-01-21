## Sales Team
These are the main missions of the sales team:

- Generate leads and reach sales goals
- Find users who align with their product and convert them into customers

These are the main KPIs for the sales team to monitor:
- L2O (Opportunities/ Leads)
- O2W (Customers/ Opportunities)
- L2C (Customers/ Leads)
- Sales cycle length
- Sales pipeline value


## Objective
The objective of this exercise is to query Circle’s acquisition funnel data in SQL and to perform a relevant analysis.

The different steps of the exercise are:
Overview:

[Data exploration](#data-exploration)

Understanding the structure of the tables and defining the primary keys.

[Data enrichment and analysis](#data-enrichment-and-analysis)

[Funnel analysis](#funnel-analysis)
- Display the current state of the funnel
- Calculate funnel statistics

## Sources
[Download Here](#https://docs.google.com/spreadsheets/d/1qhHVVdi6Z8PnD62QJtbnEjyX8d35sho573XwoasrYBk/edit?gid=0#gid=0)

## A) Data Exploration
Explore and understand the structure of the cc_funnel table. How many columns are there? How many rows? Can you understand what data each of the columns contain?

What is the primary key of the cc_funnel table? Feel free to correct any data anomalies if you find them.
---> Company column 
```
SELECT
  ### Key ###
  company
  ###########
  ,count(*) AS nb
FROM `course15.cc_funnel`
GROUP BY
  company
HAVING nb>=2
ORDER BY nb DESC
```

## B) Data Enrichment and Analysis
Using this example as a guide, perform the appropriate transformation, calculation, and aggregation steps to deliver the analysis relevant to the Circle Sales team.

### 1. Vision of the Current State of the Funnel
The sales team wants to know the current state of the funnel with the number of prospects at each stage. They are looking for:
- a global overview
- an overview grouped by prospect priority
- bonus an overview grouped by prospect priority with the 4 deal stages as columns

The possible stages of the funnel are Lead, Opportunity, Customer or Lost.

Add a column deal_stage to the cc_funnel table by using the CASE WHEN clause. Save the results in a table cc_funnel_kpi

![05-Circle-Acquisition-Funnel-asset-3-Untitled](https://github.com/user-attachments/assets/010b1de6-5c13-435f-be2e-9a264a66489c)

Check the primary key for cc_funnel_kpi.

Control that deal_stage is not NULL in cc_funnel_kpi.

Aggregate the number of prospects in each deal stage globally or by prospect priority level. use the GROUP BY clause on the cc_funnel_kpi table.

- Add a column deal_stage to the cc_funnel table by using the CASE WHEN clause. Save the results in a table cc_funnel_kpi
screenshot if necessary
- Check the primary key for cc_funnel_kpi.
- Control that deal_stage is not NULL in cc_funnel_kpi.
- Aggregate the number of prospects in each deal stage globally or by prospect priority level. use the GROUP BY clause on the cc_funnel_kpi table.

![05-Circle-Acquisition-Funnel-asset-4-Untitled](https://github.com/user-attachments/assets/e8f8551c-3423-4d5b-87bc-45cdbad1bdef)

Globally. screenshot if necessary
By priority. screenshot if necessary
To show the 4 deal stages as columns, query the cc_funnel_kpi table by using the COUNTIF() function.
screenshot if necessary


1 - Add the deal_stage column to cc_funnel and save the results in the cc_funnel_kpi table _cc_funnel_kpi_
```
 SELECT
   ### Key ###
   company
   ###########
   -- prospect infos --
   ,sector
   ,priority
   -- date --
   ,date_lead
   ,date_opportunity
   ,date_customer
   ,date_lost
   -- deal stage --
   ,CASE
     WHEN date_lost IS NOT NULL THEN "4 - Lost"
     WHEN date_customer IS NOT NULL THEN "3 - Customer"
     WHEN date_opportunity IS NOT NULL THEN "2 - Opportunity"
     WHEN date_lead IS NOT NULL THEN "1 - Lead"
     ELSE NULL
   END AS deal_stage
 FROM `course15.cc_funnel`
```

_Test - cc_funel_kpi_pk_
```
 SELECT
   ### Key ###
   company
   ###########
   ,count(*) AS nb
 FROM `course15.cc_funnel_kpi`
 GROUP BY
   company
 HAVING nb>=2
 ORDER BY nb DESC
```

_Test - cc_funnel_kpi_column-deal_stage_
```
SELECT
   ### Key ###
   company
   ###########
   ,deal_stage
 FROM `course15.cc_funnel_kpi`
 WHERE TRUE
   AND deal_stage IS NULL
 ORDER BY company
```

2 - Aggregate the results globally or by priority to obtain the number of prospects at each stage.
```
 SELECT
     ### Key ###
     deal_stage
     ###########
     ,COUNT(*) AS nb_prospect
   FROM `course15.cc_funnel_kpi`
   GROUP BY
     deal_stage
   ORDER BY
     deal_stage
```

By priority
```
   SELECT
     ### Key ###
     deal_stage
     ,priority
     ###########
     ,COUNT(*) AS nb_prospect
   FROM `course15.cc_funnel_kpi`
   GROUP BY
     deal_stage
     ,priority
   ORDER BY
     deal_stage
     ,priority
```

By priority with the different deal stages as columns
```
   SELECT
     ### Key ###
     priority
     ###########
     ,COUNTIF(deal_stage='1 - Lead') AS lead
     ,COUNTIF(deal_stage='2 - Opportunity') AS opportunity
     ,COUNTIF(deal_stage='3 - Customer') AS customer
     ,COUNTIF(deal_stage='4 - Lost') AS lost
   FROM `course15.cc_funnel_kpi`
   GROUP BY
     priority
   ORDER BY
     priority
```

### C) Funnel Statistics

Calculate the different statistics of the funnel:

- number of conversions
- conversion rate for each step of the funnel - Lead2Customers rate, Lead2Opportunity rate, Opportunity2Customers rate
- conversion time for each stage of the funnel - Lead2Customers time, Lead2Opportunity time, Opportunity2Customers time.

1. Add conversion step columns:

- a lead2conversion column: equal to 1 if the prospect has reached the customer stage, 0 if the prospect has reached the lost stage and null otherwise.
- a lead2opportunity column: equal to 1 if the prospect has reached the opportunity stage, 0 if the prospect has reached the lost stage and null otherwise.
- a opportunity2customer column: equal to 1 if the prospect has reached the customer stage, 0 if the lead has reached the lost stage after reaching the opportunity stage and null otherwise.

```
SELECT
   ### Key ###
   company
   ###########
   -- prospect infos --
   ,sector
   ,priority
   -- date --
   ,date_lead
   ,date_opportunity
   ,date_customer
   ,date_lost
   -- deal stage --
   ,CASE
     WHEN date_lost IS NOT NULL THEN "4 - Lost"
     WHEN date_customer IS NOT NULL THEN "3 - Customer"
     WHEN date_opportunity IS NOT NULL THEN "2 - Opportunity"
     WHEN date_lead IS NOT NULL THEN "1 - Lead"
     ELSE NULL
   END AS deal_stage
   -- rate --
   ,CASE
     WHEN date_lost IS NOT NULL THEN 0
     WHEN date_customer IS NOT NULL THEN 1
     ELSE NULL
   END AS lead2customer
   ,CASE
     WHEN date_lost IS NOT NULL THEN 0
     WHEN date_opportunity IS NOT NULL THEN 1
     ELSE NULL
   END AS lead2opportunity
   ,CASE
     WHEN date_lost IS NOT NULL AND date_opportunity IS NOT NULL THEN 0
     WHEN date_customer IS NOT NULL THEN 1
     ELSE NULL
   END AS opportunity2customer
 FROM `course15.cc_funnel`
```

2. - Add time conversion step columns:

- a lead2conversion_time column: equal date_diff between date_customers and date_prospect
- a lead2opportunity_time column: equal date_diff between date_opportunity and date_prospect
- a opportunity2customer_time column: equal date_diff between date_customers and date_opportunity

```
 SELECT
   ### Key ###
   company
   ###########
   -- prospect infos --
   ,sector
   ,priority
   -- date --
   ,date_lead
   ,date_opportunity
   ,date_customer
   ,date_lost
   -- deal stage --
   ,CASE
     WHEN date_lost IS NOT NULL THEN "4 - Lost"
     WHEN date_customer IS NOT NULL THEN "3 - Customer"
     WHEN date_opportunity IS NOT NULL THEN "2 - Opportunity"
     WHEN date_lead IS NOT NULL THEN "1 - Lead"
     ELSE NULL
   END AS deal_stage
   -- rate --
   ,CASE
     WHEN date_lost IS NOT NULL THEN 0
     WHEN date_customer IS NOT NULL THEN 1
     ELSE NULL
   END AS lead2customer
   ,CASE
     WHEN date_lost IS NOT NULL THEN 0
     WHEN date_opportunity IS NOT NULL THEN 1
     ELSE NULL
   END AS lead2opportunity
   ,CASE
     WHEN date_lost IS NOT NULL AND date_opportunity IS NOT NULL THEN 0
     WHEN date_customer IS NOT NULL THEN 1
     ELSE NULL
   END AS opportunity2customer
   -- time --
   ,DATE_DIFF(date_customer,date_lead,DAY) AS lead2customer_time
   ,DATE_DIFF(date_opportunity,date_lead,DAY) AS lead2opportunity_time
   ,DATE_DIFF(date_customer,date_opportunity,DAY) AS opportunity2customer_time
 FROM `course15.cc_funnel`
```


The sales team would like to see the previous KPI statistics aggregated:
- globally
- broken down by priority
- broken down by month

Aggregate rate and time KPIs at different levels of granularity - globally, by priority, by month.

```
SELECT
  -- prospect & conversion --
  COUNT(*) AS prospect
  ,COUNT(date_customer) AS customer
  -- rate --
  ,ROUND(AVG(lead2customer)*100,1) AS lead2customer_rate
  ,ROUND(AVG(lead2opportunity)*100,1) AS lead2opportunity_rate
  ,ROUND(AVG(opportunity2customer)*100,1) AS opportunity2customer_rate
  -- time
  ,ROUND(AVG(lead2customer_time),2) AS lead2customer_time
  ,ROUND(AVG(lead2opportunity_time),2) AS lead2opportunity_time
  ,ROUND(AVG(opportunity2customer_time),2) AS opportunity2customer_time
FROM `course15.cc_funnel_kpi`
```

_Priority_

```
SELECT
  priority
-- prospect & conversion --
  ,COUNT(*) AS prospect
  ,COUNT(date_customer) AS customer
  -- rate --
  ,ROUND(AVG(lead2customer)*100,1) AS lead2customer_rate
  ,ROUND(AVG(lead2opportunity)*100,1) AS lead2opportunity_rate
  ,ROUND(AVG(opportunity2customer)*100,1) AS opportunity2customer_rate
  -- time
  ,ROUND(AVG(lead2customer_time),2) AS lead2customer_time
  ,ROUND(AVG(lead2opportunity_time),2) AS lead2opportunity_time
  ,ROUND(AVG(opportunity2customer_time),2) AS opportunity2customer_time
FROM `course15.cc_funnel_kpi`
GROUP BY priority
```

_Month_
```
SELECT
  EXTRACT(MONTH FROM date_lead) AS month_lead
  -- prospect & conversion --
  ,COUNT(*) AS prospect
  ,COUNT(date_customer) AS customer
  ,SUM(lead2customer) AS customer
  -- rate --
  ,ROUND(AVG(lead2customer)*100,1) AS lead2customer_rate
  ,ROUND(AVG(lead2opportunity)*100,1) AS lead2opportunity_rate
  ,ROUND(AVG(opportunity2customer)*100,1) AS opportunity2customer_rate
  -- time
  ,ROUND(AVG(lead2customer_time),2) AS lead2customer_time
  ,ROUND(AVG(lead2opportunity_time),2) AS lead2opportunity_time
  ,ROUND(AVG(opportunity2customer_time),2) AS opportunity2customer_time
FROM `course15.cc_funnel_kpi`
GROUP BY month_lead
ORDER BY month_lead
```














