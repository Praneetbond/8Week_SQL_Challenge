# Case Study #4 - Data Bank
### A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?
````sql
select 
  count(distinct node_id) as unique_nodes 
from 
  customer_nodes;
````
### 2. What is the number of nodes per region?
````sql
select 
  region_name, 
  count(node_id) as total_nodes 
from 
  regions R 
  left join customer_nodes CN on R.region_id = CN.region_id 
group by 
  region_name;
````
### 3. How many customers are allocated to each region?
````sql
select 
  region_name, 
  count(distinct customer_id) as total_customers 
from 
  regions R 
  left join customer_nodes CN on R.region_id = CN.region_id 
group by 
  region_name;
````
### 4. How many days on average are customers reallocated to a different node?
````sql
select 
  round(
    avg(
      datediff(end_date, start_date)
    )
  ) as day_diff 
from 
  customer_nodes 
where 
  end_date != '9999-12-31';
````
### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
````sql
with temp_cte as (
  select 
    *, 
    sum(
      datediff(end_date, start_date)
    ) as day_diff, 
    ntile(100) over (
      partition by region_id 
      order by 
        sum(
          datediff(end_date, start_date)
        )
    ) as percentile 
  from 
    customer_nodes 
  where 
    end_date != '9999-12-31' 
  group by 
    customer_id, 
    region_id, 
    node_id, 
    start_date, 
    end_date
) 
select 
  region_name, 
  max(
    if(percentile = 50, day_diff, null)
  ) as median, 
  max(
    if(percentile = 80, day_diff, null)
  ) as 80_percentile, 
  max(
    if(percentile = 95, day_diff, null)
  ) as 95_percentile 
from 
  (
    select 
      region_id, 
      day_diff, 
      percentile 
    from 
      temp_cte 
    where 
      percentile in (50, 80, 95) 
    group by 
      region_id, 
      day_diff, 
      percentile
  ) as temp 
  left join regions on temp.region_id = regions.region_id 
group by 
  region_name;
````
