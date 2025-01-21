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

## B) Data Enrichment and Analysis
Using this example as a guide, perform the appropriate transformation, calculation, and aggregation steps to deliver the analysis relevant to the Circle Sales team.

### 1. Vision of the Current State of the Funnel
The sales team wants to know the current state of the funnel with the number of prospects at each stage. They are looking for:
- a global overview
- an overview grouped by prospect priority
- bonus an overview grouped by prospect priority with the 4 deal stages as columns

The possible stages of the funnel are Lead, Opportunity, Customer or Lost.

Add a column deal_stage to the cc_funnel table by using the CASE WHEN clause. Save the results in a table cc_funnel_kpi
screenshot if necessary
Check the primary key for cc_funnel_kpi.

Control that deal_stage is not NULL in cc_funnel_kpi.

Aggregate the number of prospects in each deal stage globally or by prospect priority level. use the GROUP BY clause on the cc_funnel_kpi table.

Globally. screenshot if necessary
By priority. screenshot if necessary
To show the 4 deal stages as columns, query the cc_funnel_kpi table by using the COUNTIF() function.
screenshot if necessary

### C) Funnel Statistics

Calculate the different statistics of the funnel:

- number of conversions
- conversion rate for each step of the funnel - Lead2Customers rate, Lead2Opportunity rate, Opportunity2Customers rate
- conversion time for each stage of the funnel - Lead2Customers time, Lead2Opportunity time, Opportunity2Customers time.

The sales team would like to see the previous KPI statistics aggregated:
- globally
- broken down by priority
- broken down by month

















