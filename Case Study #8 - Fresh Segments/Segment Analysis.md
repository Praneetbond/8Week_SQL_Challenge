# Case Study #8 - Fresh Segments

### Segment Analysis

#### 1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
````sql
with eligible_interest_ids as (
  select 
    interest_id 
  from 
    interest_metrics 
  group by 
    interest_id 
  having 
    count(month_year) > 5
), 
top10_composition as (
  select 
    composition 
  from 
    interest_metrics 
  where 
    interest_id in (
      select 
        * 
      from 
        eligible_interest_ids
    ) 
  order by 
    1 desc 
  limit 
    10
), 
bottom10_composition as (
  select 
    composition 
  from 
    interest_metrics 
  where 
    interest_id in (
      select 
        * 
      from 
        eligible_interest_ids
    ) 
  order by 
    1 
  limit 
    10
) 
select 
  interest_id, 
  month_year, 
  composition 
from 
  interest_metrics 
where 
  composition in (
    select 
      * 
    from 
      top10_composition
  ) 
  or composition in (
    select 
      * 
    from 
      bottom10_composition
  ) 
  and month_year is not null 
  and interest_id in (
    select 
      * 
    from 
      eligible_interest_ids
  ) 
order by 
  composition desc;
````

#### 2. Which 5 interests had the lowest average ranking value?
````sql
select 
  interest_name, 
  round(
    avg(ranking), 
    2
  ) as avg_ranking 
from 
  interest_metrics IM 
  left join interest_map IMp on IM.interest_id = IMp.id 
where 
  interest_id in (
    select 
      interest_id 
    from 
      interest_metrics 
    group by 
      interest_id 
    having 
      count(month_year) > 5
  ) 
group by 
  interest_name 
order by 
  avg(ranking) 
limit 5;
````

#### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?
````sql
select 
  interest_name, 
  round(
    stddev(percentile_ranking), 
    2
  ) as std_dev 
from 
  interest_metrics IM 
  left join interest_map IMp on IM.interest_id = IMp.id 
where 
  interest_id in (
    select 
      interest_id 
    from 
      interest_metrics 
    group by 
      interest_id 
    having 
      count(month_year) > 5
  ) 
group by 
  interest_name 
order by 
  2 desc 
limit 
  5;
````

#### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
````sql
with top_percentile as (
  select 
    interest_id 
  from 
    interest_metrics 
  where 
    interest_id in (
      select 
        interest_id 
      from 
        interest_metrics 
      group by 
        interest_id 
      having 
        count(month_year) > 5
    ) 
  group by 
    interest_id 
  order by 
    round(
      stddev(percentile_ranking), 
      2
    ) desc 
  limit 
    5
) 
select 
  * 
from 
  (
    select 
      distinct interest_id, 
      interest_name, 
      case when percentile_ranking in (
        max(percentile_ranking) over (partition by interest_id), 
        min(percentile_ranking) over (partition by interest_id)
      ) then month_year end as month_year, 
      case when percentile_ranking in (
        max(percentile_ranking) over (partition by interest_id), 
        min(percentile_ranking) over (partition by interest_id)
      ) then percentile_ranking end as percentile 
    from 
      interest_metrics IM 
      left join interest_map IMp on IM.interest_id = IMp.id 
    where 
      interest_id in (
        select 
          * 
        from 
          top_percentile
      )
  ) as temp 
where 
  month_year is not null 
order by 
  interest_id;
````
