# Case Study #8 - Fresh Segments
### Index Analysis

The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

#### 1. What is the top 10 interests by the average composition for each month?
````sql
select 
  month_year, 
  interest_name, 
  avg_composition 
from 
  (
    select 
      month_year, 
      interest_name, 
      round(composition / index_value, 2) as avg_composition, 
      dense_rank() over (
        partition by month_year 
        order by 
          composition / index_value desc
      ) as ranking 
    from 
      interest_metrics IM 
      left join interest_map IMp on IM.interest_id = IMp.id 
    where 
      month_year is not null 
    order by 
      1, 
      4
  ) as temp 
where 
  ranking between 1 
  and 10;
````

#### 2. For all of these top 10 interests - which interest appears the most often?
````sql
with top_interests as (
  select 
    interest_name, 
    count(*) as interest_count 
  from 
    (
      select 
        month_year, 
        interest_name, 
        avg_composition 
      from 
        (
          select 
            month_year, 
            interest_name, 
            round(composition / index_value, 2) as avg_composition, 
            dense_rank() over (
              partition by month_year 
              order by 
                composition / index_value desc
            ) as ranking 
          from 
            interest_metrics IM 
            left join interest_map IMp on IM.interest_id = IMp.id 
          where 
            month_year is not null 
          order by 
            1, 
            4
        ) as temp 
      where 
        ranking between 1 
        and 10
    ) as temp2 
  group by 
    interest_name 
  order by 
    interest_count desc
) 
select 
  interest_name 
from 
  top_interests 
where 
  interest_count = (
    select 
      max(interest_count) 
    from 
      top_interests
  );
````

#### 3. What is the average of the average composition for the top 10 interests for each month?
````sql
select 
  round(
    avg(avg_composition), 
    2
  ) as avg_of_avg_composition 
from 
  (
    select 
      month_year, 
      interest_name, 
      round(composition / index_value, 2) as avg_composition, 
      dense_rank() over (
        partition by month_year 
        order by 
          composition / index_value desc
      ) as ranking 
    from 
      interest_metrics IM 
      left join interest_map IMp on IM.interest_id = IMp.id 
    where 
      month_year is not null 
    order by 
      1, 
      4
  ) as temp 
where 
  ranking between 1 
  and 10;
````

#### 4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
````sql
with interest_data as (
  select 
    month_year, 
    interest_id, 
    round(
      sum(composition)/ sum(index_value), 
      2
    ) as avg_composition, 
    dense_rank() over (
      partition by month_year 
      order by 
        sum(composition)/ sum(index_value) desc
    ) as index_id 
  from 
    interest_metrics 
  where 
    month_year is not null 
  group by 
    month_year, 
    interest_id
) 
select 
  month_year, 
  interest_name, 
  3_month_moving_avg, 
  1_month_ago, 
  2_month_ago 
from 
  (
    select 
      month_year, 
      interest_name, 
      format(
        avg(avg_composition) over (
          rows between 2 preceding 
          and current row
        ), 
        2
      ) as 3_month_moving_avg, 
      concat(
        lag(interest_name) over (), 
        ": ", 
        lag(avg_composition) over ()
      ) as 1_month_ago, 
      concat(
        lag(interest_name) over (), 
        ": ", 
        lag(avg_composition, 2) over ()
      ) as 2_month_ago 
    from 
      interest_data ID 
      left join interest_map IM on ID.interest_id = IM.id 
    where 
      index_id = 1
  ) as temp 
where 
  month_year between "2018-09-01" 
  and "2019-08-31";
````
