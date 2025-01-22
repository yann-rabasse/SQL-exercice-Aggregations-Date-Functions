## Introduction

Using aggregation, data can be analysed at different granularity levels.
When analysing data, we can start from a big picture analysis to understand the lay of the land, and where to focus our analysis further.
When aggregating, it is important to pay attention to what columns we’re using on our SELECT clause, as well as our GROUP BY clause.

## Case Study

Today, we’re going to conduct a profitability analysis for Superstore, an office stationery store based in the United States that sell office supplies and furniture.

You’ve been hired as a consultant to understand:

- Where are Superstore’s top-selling states?
- Are there any non-profitable regions that are doing well in sales?
- If so, what are causing these losses?

[Download Here](#https://docs.google.com/spreadsheets/d/1ucXDeeKSDdOXqyPy0u4RMNWUhEtpP5BT3pIVe6JIPoY/edit?gid=976735188#gid=976735188)

You’ve been provided with Superstore’s sales data in 2014 to 2017.

Each row in this data set represents an item in an order. In other words, when there are multiple items in an order, the table will contain multiple rows with the same Order ID.

Source: https://www.kaggle.com/datasets/vivek468/superstore-dataset-final

## Task 1: Top Selling States
We’ll first start by understanding where Superstore’s top selling states are.
As not all top-selling states may be profitable, we also want to include profits derived from each state.



💡 *The goal here is to prompt students to interpret the data output.

One glaring insight here is that 2 of the Top 5 states are not profitable, which can be problematic.

The second insight here is that while California is ~30% larger than New York in sales, it is about equal in profit. In other words, they’re not as efficient in making profits for every $ revenue generated.

Prompt students to start applying a drill-down analysis thinking, i.e. now that we’ve seen a high-level view of the business’ revenue and profitability, what can we zoom into to help Superstore better understand their revenue and profitability.*

```
SELECT State,
       ROUND(SUM(Sales), 2) AS total_sales,
       ROUND(SUM(Profit), 2) AS total_profit
  FROM `recap.superstore`
GROUP BY State
ORDER BY total_sales DESC
```

### Result
![image](https://github.com/user-attachments/assets/14fa8c6a-f380-4191-961e-0dfd54d08cf1)

## Task 2: Diagnose the biggest non-profitable state
Uh oh! It turns out that 2 of the Top 5 selling states are turning losses.

This is concerning, as it can mean that for each $ Superstore is gaining in revenue, they’re actually losing money.

As a consultant, you want to make recommendations on what Superstore can do to fix this issue. We’re going to first understand whether this issue is isolated to a particular:

Category
Specific cities
Even though there are multiple State candidates for our analysis, we’ll first start with diagnosing profitability issues in Texas.

💡 *The goal here is to prompt students to think through their hypothesis, and quickly iterate through those hypothesis.

In terms of SQL, we’re going to leverage the same SQL structure, and this is also their chance to practice that changing non-aggregated columns in the SELECT clause will also require changes to the GROUP BY clause.*

### Step 1: Diagnosing product categories in Texas
Your first hypothesis is that there are certain product categories that are profitable in Texas. Hence, Superstore could consider shifting their focus to sell primarily those categories
```
SELECT Category,
       ROUND(SUM(Profit), 2) AS total_profit,
       ROUND(SUM(Sales), 2) AS total_sales
  FROM `recap.superstore`
 WHERE State = 'Texas'
GROUP BY Category
ORDER BY total_sales DESC;
```
#### Result
<img width="511" alt="o50ou14i44b2vqluhq3gdwkxtoit" src="https://github.com/user-attachments/assets/71d00c4c-3126-4108-a417-a0397dbd9661" />


### Step 2: Diagnosing regions in Texas

```
SELECT Category,
       ROUND(SUM(Profit), 2) AS total_profit,
       ROUND(SUM(Sales), 2) AS total_sales
  FROM `recap.superstore`
 WHERE State = 'Texas'
GROUP BY Category
ORDER BY total_sales DESC;
```
<img width="511" alt="o50ou14i44b2vqluhq3gdwkxtoit" src="https://github.com/user-attachments/assets/9f50326f-4a26-40b4-a3eb-3637165a78ae" />


Well, this looks problematic.

From the Top 5 selling cities, total losses equate to ~$20k, which far outweighs Fort Worth’s profit of $300.

There seems to be a wider spread issue in the state that’s worth looking into.

### Step 3: Investigate discounts
On the dataset, there was a column called Discount that specifies the % discount applied to an order.

Could it be that the state is giving outrageous discounts to customers, which are driving sales, but are hurting profitability?

Let’s test that hypothesis.
```
SELECT Discount,
       ROUND(SUM(Profit), 2) AS total_profit,
       ROUND(SUM(Sales), 2) AS total_sales
  FROM `recap.superstore`
 WHERE State = 'Texas'
GROUP BY Discount
ORDER BY total_sales DESC;
```
<img width="436" alt="bih22zy2k31hwf2olkyi1kjoypyw" src="https://github.com/user-attachments/assets/acc869d7-997a-412e-82c6-104648706d49" />

We may be onto something here.

Sales with 30% or larger discounts seems to be selling at a lost consistently. Let’s check how the golden state and the big apple as done it.

Benchmarking with California

```
SELECT State,
       Discount,
       ROUND(SUM(Profit), 2) AS total_profit,
       ROUND(SUM(Sales), 2) AS total_sales
  FROM `recap.superstore`
 WHERE State IN ('California', 'Texas', 'New York')
GROUP BY State, Discount
ORDER BY State, Discount;
```
<img width="634" alt="uto3n177ahxv8j0pkxtxcwujmokh" src="https://github.com/user-attachments/assets/f768087b-0426-49fb-a086-5ac8aaa8bdf2" />

Aha! We’re onto something.

We can see that the big profitable states have seemed to avoid offering discounts more than 20% because they attract losses.

So, there goes our first recommendation, that is to review the discount structure offered in the state of Texas. While they may attract a lot of sales, the business is losing money for each $ it’s selling.

## Task 3: Analysis extension
That was a good start to this analysis! Well done!

How would you continue from here? What would you drill down into further to formulate your recommendations to the board of Superstore?

💡 *The goal here is to prompt students to think through their hypothesis, and quickly iterate through those hypothesis.

In terms of SQL, we’re going to leverage the same SQL structure, and this is also their chance to practice that changing non-aggregated columns in the SELECT clause will also require changes to the GROUP BY clause.*






