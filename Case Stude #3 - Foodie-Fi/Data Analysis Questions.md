# Case Study #3 - Foodie-Fi
### B. Data Analysis Questions

### 1. How many customers has Foodie-Fi ever had?
```` sql 
select 
  count(distinct customer_id) 
from 
  foodie_fi.subscriptions;
```` 

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```` sql 
select 
  date_format(start_date, '%Y-%m-01') as starts_date, 
  count(*) as cust_distribution 
from 
  foodie_fi.subscriptions 
where 
  plan_id = 0 
group by 
  starts_date 
order by 
  starts_date;
```` 

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```` sql 
select 
  plan_name, 
  count(*) 
from 
  foodie_fi.subscriptions S 
  left join foodie_fi.plans P on S.plan_id = P.plan_id 
where 
  year(start_date) > 2020 
group by 
  S.plan_id, 
  plan_name 
order by 
  S.plan_id;
```` 

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```` sql 
select 
  count(*) as No_of_cust, 
  round(
    (
      100 * count(*)
    )/(
      select 
        count(distinct customer_id) 
      from 
        foodie_fi.subscriptions
    ), 
    1
  ) as perc_cust 
from 
  (
    select 
      customer_id, 
      plan_id, 
      start_date, 
      row_number() over (
        partition by customer_id 
        order by 
          start_date desc
      ) as index_id 
    from 
      foodie_fi.subscriptions 
    order by 
      customer_id
  ) as temp_1 
where 
  index_id = 1 
  and plan_id = 4;
```` 

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```` sql 
select 
  count(*) as customer_count, 
  round(
    100 * count(*)/(
      select 
        count(distinct customer_id) 
      from 
        foodie_fi.subscriptions
    ), 
    0
  ) as cust_perc 
from 
  (
    select 
      customer_id, 
      plan_name, 
      start_date, 
      lead(plan_name) over (
        partition by customer_id 
        order by 
          start_date
      ) as next_plan 
    from 
      foodie_fi.subscriptions S 
      left join plans P on S.plan_id = P.plan_id
  ) as temp_1 
where 
  plan_name = 'trial' 
  and next_plan = 'churn';
```` 

### 6. What is the number and percentage of customer plans after their initial free trial?
```` sql 
select 
  next_plan as plan_name, 
  count(*) as customer_count, 
  round(
    100 * count(*)/(
      select 
        count(distinct customer_id) 
      from 
        foodie_fi.subscriptions
    ), 
    1
  ) as customer_perc 
from 
  (
    select 
      customer_id, 
      plan_name, 
      start_date, 
      lead(plan_name) over (
        partition by customer_id 
        order by 
          start_date
      ) as next_plan 
    from 
      foodie_fi.subscriptions S 
      left join plans P on S.plan_id = P.plan_id
  ) as temp_1 
where 
  plan_name = 'trial' 
group by 
  next_plan;
```` 

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```` sql 
select 
  plan_name, 
  count(*) as customer_count, 
  round(
    100 * count(*)/(
      select 
        count(distinct customer_id) 
      from 
        foodie_fi.subscriptions
    ), 
    1
  ) as customer_perc 
from 
  (
    select 
      customer_id, 
      plan_name, 
      start_date, 
      row_number() over (
        partition by customer_id 
        order by 
          start_date desc
      ) as index_id 
    from 
      foodie_fi.subscriptions S 
      left join foodie_fi.plans P on S.plan_id = P.plan_id 
    where 
      start_date <= '2020-12-31' 
    order by 
      customer_id
  ) as temp_1 
where 
  index_id = 1 
group by 
  plan_name;
```` 

### 8. How many customers have upgraded to an annual plan in 2020?
```` sql 
select 
  count(*) as Customers_upgraded 
from 
  (
    select 
      customer_id, 
      plan_name, 
      start_date, 
      lead(plan_name) over (
        partition by customer_id 
        order by 
          start_date
      ) as next_plan 
    from 
      foodie_fi.subscriptions S 
      left join foodie_fi.plans P on S.plan_id = P.plan_id 
    where 
      Year(start_date) = 2020
  ) as temp_1 
where 
  next_plan = 'pro annual';
```` 

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```` sql 
with update_plan as 
(
  select 
    customer_id, 
    plan_name, 
    start_date, 
    lead(start_date) over (
      partition by customer_id 
      order by 
        start_date
    ) as next_plan_date 
  from 
    foodie_fi.subscriptions S 
    left join foodie_fi.plans P on S.plan_id = P.plan_id 
  where 
    plan_name in ('trial', 'pro annual')
) 
select 
  round(
    avg(next_plan_date - start_date)
  ) as avg_days_for_update 
from 
  update_plan 
where 
  next_plan_date is not null;
```` 

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```` sql 
with update_plan as 
(
  select 
    customer_id, 
    plan_name, 
    datediff(
      lead(start_date) over (
        partition by customer_id 
        order by 
          start_date
      ), 
      start_date
    ) as date_diff 
  from 
    foodie_fi.subscriptions S 
    left join foodie_fi.plans P on S.plan_id = P.plan_id 
  where 
    plan_name in ('trial', 'pro annual')
) 
select 
  brackets, 
  count(*) 
from 
  (
    select 
      case  when date_diff <= 30 then '0-30' 
            when date_diff <= 60 then '30-60' 
            when date_diff <= 90 then '60-90' 
            when date_diff <= 120 then '90-120' 
            when date_diff <= 150 then '120-150' 
            when date_diff <= 180 then '150-180' 
            when date_diff <= 210 then '180-210' 
            when date_diff <= 240 then '210-240' 
            when date_diff <= 270 then '240-270' 
            when date_diff <= 300 then '270-300' 
            when date_diff <= 330 then '300-330' 
            when date_diff <= 360 then '330-360'
       end as brackets 
    from 
      update_plan 
    where 
      date_diff is not null
  ) as temp_1 
group by 
  brackets 
order by 
  case  when brackets = '0-30' then 1 
        when brackets = '30-60' then 2 
        when brackets = '60-90' then 3 
        when brackets = '90-120' then 4 
        when brackets = '120-150' then 5 
        when brackets = '150-180' then 6 
        when brackets = '180-210' then 7 
        when brackets = '210-240' then 8 
        when brackets = '240-270' then 9 
        when brackets = '270-300' then 10 
        when brackets = '300-330' then 11 
        when brackets = '330-360' then 13 
   end;
```` 

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```` sql 
with record_cte as 
  (
  select 
    customer_id, 
    plan_name, 
    start_date, 
    row_number() over (
      partition by customer_id 
      order by 
        start_date
    ) as index_id 
  from 
    foodie_fi.subscriptions S 
    join foodie_fi.plans P on S.plan_id = P.plan_id 
  where 
    plan_name in ('basic monthly', 'pro monthly') 
    and year(start_date) <= 2020
) 
select 
  count(*) as customer_count 
from 
  record_cte 
where 
  index_id = 2 
  and plan_name = 'basic_monthly';
````
