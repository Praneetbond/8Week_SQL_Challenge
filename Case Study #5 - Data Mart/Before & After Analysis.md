# Case Study #5 - Data Exploration

## 3. Before & After Analysis

#### This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.
#### Taking the week_date value of 2020 - 06 - 15 as the baseline week where the Data Mart sustainable packaging changes came into effect.
#### We would include all week_date values for 2020 - 06 - 15 as the start of the period after the change and the previous week_date values would be before 
#### Using this analysis approach - answer the following questions:-

#### 1. What is the total sales for the 4 weeks before and after 2020 - 06 - 15? What is the growth or reduction rate in actual values and percentage of sales?
````sql 
select 
  before_sales, 
  after_sales, 
  after_sales - before_sales as Actual_growth, 
  round(
    100 *(after_sales - before_sales)/ after_sales, 
    2
  ) as percent_growth 
from 
  (
    select 
      sum(
        case when txn_date between '2020-06-15' 
        and '2020-06-15' + interval 4 week then sales else 0 end
      ) after_sales, 
      sum(
        case when txn_date between '2020-06-15' - interval 4 week 
        and '2020-06-15' then sales else 0 end
      ) as before_sales 
    from 
      clean_weekly_sales
  ) as temp;
````

#### 2. What about the entire 12 weeks before and after?
````sql 
select 
  before_sales, 
  after_sales, 
  after_sales - before_sales as Actual_growth, 
  round(
    100 *(after_sales - before_sales)/ after_sales, 
    2
  ) as percent_growth 
from 
  (
    select 
      sum(
        case when txn_date between '2020-06-15' 
        and '2020-06-15' + interval 12 week then sales else 0 end
      ) after_sales, 
      sum(
        case when txn_date between '2020-06-15' - interval 12 week 
        and '2020-06-15' then sales else 0 end
      ) as before_sales 
    from 
      clean_weekly_sales
  ) as temp;
````

#### 3.How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
```` sql
with temp_cte as (
    select 
      week('2018-06-15') as 18_base_week, 
      week('2019-06-15') as 19_base_week, 
      week('2020-06-15') as 20_base_week
  ) 
select 
  calender_year, 
  sum(before_sales), 
  sum(after_sales), 
  sum(after_sales)- sum(before_sales) as actual_sales, 
  round(
    100 *(
      sum(after_sales)- sum(before_sales)
    )/ sum(after_sales), 
    2
  ) as percent_sales 
from 
  (
    select 
      calender_year, 
      (
        case when calender_year = 2018 
        and week_number between 18_base_week 
        and 18_base_week + 12 then sales when calender_year = 2019 
        and week_number between 19_base_week 
        and 19_base_week + 12 then sales when calender_year = 2020 
        and week_number between 20_base_week 
        and 20_base_week + 12 then sales else 0 end
      ) as after_sales, 
      (
        case when calender_year = 2018 
        and week_number between 18_base_week - 12 
        and 18_base_week then sales when calender_year = 2019 
        and week_number between 19_base_week - 12 
        and 19_base_week then sales when calender_year = 2020 
        and week_number between 20_base_week - 12 
        and 20_base_week then sales else 0 end
      ) as before_sales 
    from 
      clean_weekly_sales, 
      temp_cte
  ) as temp 
group by 
  calender_year;
````
