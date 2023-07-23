# Case Study #5 - Data Exploration

## 4. Bonus Question

#### Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

- Region
````sql 
select 
  *, 
  (after_sales - before_sales) as actual_sales, 
  round(
    100 *(after_sales - before_sales)/ after_sales, 
    2
  ) as percent_sales 
from 
  (
    select 
      region, 
      sum(
        case when txn_date between '2020-06-15' - interval 12 week 
        and '2020-06-15' then sales else 0 end
      ) as before_sales, 
      sum(
        case when txn_date between '2020-06-15' 
        and '2020-06-15' + interval 12 week then sales else 0 end
      ) as after_sales 
    from 
      clean_weekly_sales 
    group by 
      region
  ) as temp 
order by 
  percent_sales;
````

- Platform
````sql 
select 
  *, 
  (after_sales - before_sales) as actual_sales, 
  round(
    100 *(after_sales - before_sales)/ after_sales, 
    2
  ) as percent_sales 
from 
  (
    select 
      platform, 
      sum(
        case when txn_date between '2020-06-15' - interval 12 week 
        and '2020-06-15' then sales else 0 end
      ) as before_sales, 
      sum(
        case when txn_date between '2020-06-15' 
        and '2020-06-15' + interval 12 week then sales else 0 end
      ) as after_sales 
    from 
      clean_weekly_sales 
    group by 
      platform
  ) as temp 
order by 
  percent_sales;
````

- age_band
````sql 
select 
  *, 
  (after_sales - before_sales) as actual_sales, 
  round(
    100 *(after_sales - before_sales)/ after_sales, 
    2
  ) as percent_sales 
from 
  (
    select 
      age_band, 
      sum(
        case when txn_date between '2020-06-15' - interval 12 week 
        and '2020-06-15' then sales else 0 end
      ) as before_sales, 
      sum(
        case when txn_date between '2020-06-15' 
        and '2020-06-15' + interval 12 week then sales else 0 end
      ) as after_sales 
    from 
      clean_weekly_sales 
    group by 
      age_band
  ) as temp 
order by 
  percent_sales;
````

- demographic
````sql 
select 
  *, 
  (after_sales - before_sales) as actual_sales, 
  round(
    100 *(after_sales - before_sales)/ after_sales, 
    2
  ) as percent_sales 
from 
  (
    select 
      age_band, 
      sum(
        case when txn_date between '2020-06-15' - interval 12 week 
        and '2020-06-15' then sales else 0 end
      ) as before_sales, 
      sum(
        case when txn_date between '2020-06-15' 
        and '2020-06-15' + interval 12 week then sales else 0 end
      ) as after_sales 
    from 
      clean_weekly_sales 
    group by 
      age_band
  ) as temp 
order by 
  percent_sales;
````

- customer_type
````sql 
select 
  *, 
  (after_sales - before_sales) as actual_sales, 
  round(
    100 *(after_sales - before_sales)/ after_sales, 
    2
  ) as percent_sales 
from 
  (
    select 
      demographic, 
      sum(
        case when txn_date between '2020-06-15' - interval 12 week 
        and '2020-06-15' then sales else 0 end
      ) as before_sales, 
      sum(
        case when txn_date between '2020-06-15' 
        and '2020-06-15' + interval 12 week then sales else 0 end
      ) as after_sales 
    from 
      clean_weekly_sales 
    group by 
      demographic
  ) as temp 
order by 
  percent_sales;
````
