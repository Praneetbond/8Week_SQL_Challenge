## Data Cleaning
### New Customer Order Table
````sql
With new_customer_orders as (
  Select 
    order_id, 
    customer_id, 
    pizza_id, 
    case when lower(exclusions) like 'null' 
    or exclusions like '' then null else exclusions end as exclusions, 
    case when lower(extras) like 'null' 
    or extras like '' then null else extras end extras, 
    order_time 
  from 
    pizza_runner.customer_orders 
  order by 
    order_id
),
````
### New Runner Order Table
````sql
new_runner_orders as (
  SELECT 
    order_id, 
    runner_id, 
    CASE WHEN LOWER(pickup_time) = 'null' THEN null ELSE cast (pickup_time as timestamp) END AS pickup_time, 
    CASE WHEN LOWER(distance) = 'null' THEN null WHEN LOWER(distance) LIKE '%km' THEN cast(
      TRIM(
        'kKmM ' 
        FROM 
          distance
      ) as float
    ) ELSE cast (distance as float) END AS distance, 
    CASE WHEN LOWER(duration) = 'null' THEN null WHEN LOWER(duration) LIKE '%min%' THEN cast (
      TRIM(
        'minutes ' 
        FROM 
          duration
      ) as int
    ) ELSE cast (duration as int) END AS duration, 
    CASE WHEN LOWER(cancellation) = 'null' 
    OR cancellation = '' THEN null ELSE cancellation END AS cancellation 
  FROM 
    pizza_runner.runner_orders
),
````
### Topping Table (Pizza Recipes)
````sql
toppings as (
  Select 
    pizza_id, 
    topping_ids, 
    topping_name 
  from 
    (
      SELECT 
        pizza_id, 
        CAST(
          unnest(
            string_to_array(toppings, ',')
          ) AS INTEGER
        ) AS topping_ids 
      FROM 
        pizza_runner.pizza_recipes
    ) as PR 
    left join pizza_runner.pizza_toppings PT on PR.topping_ids = PT.topping_id 
  order by 
    pizza_id, 
    topping_ids
)
````
