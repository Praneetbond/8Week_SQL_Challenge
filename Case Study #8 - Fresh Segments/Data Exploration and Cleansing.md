# Case Study #8 - Fresh Segments

### Data Exploration and Cleansing

#### 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
````sql
alter table interest_metrics
add column month_year0 date;

update interest_metrics
set month_year0  = concat(right(month_year,4),"-", left(month_year,2),"-01");

alter table interest_metrics
drop column month_year;

alter table interest_metrics
change column month_year0 month_year date;

alter table interest_metrics
modify column month_year date after _year;
````

#### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
````sql
select 
  month_year, 
  count(*) as records_count 
from 
  interest_metrics 
group by 
  month_year 
order by 
  case when records_count is null then 1 else 2 end, 
  month_year;
````

#### 3. What do you think we should do with these null values in the fresh_segments.interest_metrics
##### There are 1193 null interest ids which can not be map with summary so we can drop the null rows

#### 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
````sql
select 
  concat(
    count(distinct interest_id), 
    " exist in interest_metrics but not interest_map"
  ) as interest_id 
from 
  interest_map IMp 
  left join interest_metrics IM on IMp.id = IM.interest_id;
````

#### 5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table
````sql
select 
  id, 
  count(interest_id) 
from 
  interest_map IM 
  left join interest_metrics NIM on IM.id = NIM.interest_id 
group by 
  id;
````

#### 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
````sql
Select 
  interest_id, 
  ranking, 
  percentile_ranking, 
  month_year, 
  composition, 
  index_value, 
  interest_name, 
  interest_summary, 
  created_at, 
  last_modified 
from 
  interest_map IM 
  left join interest_metrics NIM on IM.id = NIM.interest_id 
where 
  interest_id = 21246;
````

#### 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
````sql
select 
  count(*) as records_before_created_at 
from 
  interest_map IM 
  left join interest_metrics NIM on IM.id = NIM.interest_id 
where 
  month_year < created_at;
````
