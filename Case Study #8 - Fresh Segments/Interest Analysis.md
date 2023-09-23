# Case Study #8 - Fresh Segments

### Interest Analysis

#### 1. Which interests have been present in all month_year dates in our dataset?
````sql
select 
  interest_name 
from 
  interest_metrics IM 
  left join interest_map IMp on IM.interest_id = IMp.id 
group by 
  interest_name 
having 
  count(month_year) = (
    select 
      count(distinct month_year) 
    from 
      interest_metrics
  ) 
order by 
  interest_name;
````

#### 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?
````sql
with record as (
  select 
    months, 
    count(interest_id) as interest_count 
  from 
    (
      select 
        interest_id, 
        count(month_year) as months 
      from 
        interest_metrics IM 
        left join interest_map IMp on IM.interest_id = IMp.id 
      where 
        month_year is not null 
      group by 
        interest_id
    ) as temp 
  group by 
    months
) 
select 
  * 
from 
  (
    select 
      months, 
      interest_count, 
      round(
        100 * sum(interest_count) over (
          order by 
            months desc rows unbounded preceding
        )/(
          sum(interest_count) over ()
        ), 
        2
      ) as percentile 
    from 
      record
  ) as temp1 
where 
  percentile <= 90.99;

````
#### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?
````sql
select 
  count(*) as number_of_data 
from 
  (
    select 
      interest_id, 
      count(month_year) as months 
    from 
      interest_metrics IM 
      left join interest_map IMp on IM.interest_id = IMp.id 
    where 
      month_year is not null 
    group by 
      interest_id 
    having 
      months < 6
  ) as temp;
````

#### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.
````sql
select 
  month_year, 
  int_count, 
  int_ids, 
  round(100 * int_ids / int_count, 2) as percent 
from 
  (
    select 
      month_year, 
      count(interest_id) as int_count, 
      sum(
        case when interest_id in (
          select 
            interest_id 
          from 
            interest_metrics 
          where 
            month_year is not null 
          group by 
            interest_id 
          having 
            count(month_year) < 6
        ) then 1 else 0 end
      ) as int_ids 
    from 
      interest_metrics 
    where 
      month_year is not null 
    group by 
      month_year
  ) as temp;

````

#### 5. After removing these interests - how many unique interests are there for each month?
````sql
select 
  month_year, 
  sum(
    case when interest_id in (
      select 
        interest_id 
      from 
        interest_metrics 
      where 
        month_year is not null 
      group by 
        interest_id 
      having 
        count(month_year) > 5
    ) then 1 else 0 end
  ) as int_count 
from 
  interest_metrics 
where 
  month_year is not null 
group by 
  month_year;
````
