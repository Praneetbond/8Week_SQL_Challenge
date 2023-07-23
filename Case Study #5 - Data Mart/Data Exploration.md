# Case Study #5 - Data Exploration

## B. Data Exploration
### 1.What day of the week is used for each week_date value? 
```` sql 
select 
  distinct(
    dayname(txn_date)
  ) as Txn_day 
from 
  clean_weekly_sales 
order by 
  Txn_day;
````

### 2.What range of week numbers are missing from the dataset? 
````sql
With recursive week_series as (
    select 
      1 as week_number 
    union all 
    select 
      week_number + 1 
    from 
      week_series 
    where 
      week_number < 52
  ) 
select 
  week_number as missing_weeks 
from 
  week_series 
where 
  week_number not in (
    select 
      week_number 
    from 
      clean_weekly_sales 
    group by 
      calender_year, 
      week_number
  );
````

### 3.How many total transactions were there for each year in the dataset?
````sql 
select 
  calender_year, 
  format(
    sum(transactions), 
    0
  ) as num_of_txn 
from 
  clean_weekly_sales 
group by 
  calender_year;
````

### 4.What is the total sales for each region for each month? 
````sql 
select 
  region, 
  format(
    sum(sales), 
    2
  ) as total_sales 
from 
  clean_weekly_sales 
group by 
  region;
````

### 5.What is the total count of transactions for each platform
````sql 
select 
  platform, 
  format(
    sum(transactions), 
    0
  ) as num_of_txn 
from 
  clean_weekly_sales 
group by 
  platform;
````

### 6.What is the percentage of sales for Retail vs Shopify for each month?
````sql 
select 
  platform, 
  round(
    100 * sum(sales)/(
      select 
        sum(sales) 
      from 
        clean_weekly_sales
    ), 
    2
  ) as percent_sales 
from 
  clean_weekly_sales 
group by 
  platform;
````

### 7.What is the percentage of sales by demographic for each year in the dataset?
````sql 
select 
  calender_year, 
  demographic, 
  round(
    100 * sum(sales)/(
      select 
        sum(sales) 
      from 
        clean_weekly_sales
    ), 
    2
  ) as percent_sales 
from 
  clean_weekly_sales 
group by 
  calender_year, 
  demographic 
order by 
  calender_year, 
  percent_sales desc;
````

### 8.Which age_band and demographic values contribute the most to Retail sales?
````sql 
select 
  age_band, 
  demographic, 
  format(
    sum(sales), 
    2
  ) as total_sales 
from 
  clean_weekly_sales 
where 
  platform = "Retail" 
group by 
  age_band, 
  demographic;
````

#### 9.Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
````sql 
select 
  calender_year, 
  platform, 
  sum(sales)/ sum(transactions) as avg_transaction
from 
  clean_weekly_sales 
group by 
  calender_year, 
  platform 
order by 
  calender_year;
````
