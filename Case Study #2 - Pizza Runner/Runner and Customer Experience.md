# Case Study #2 - Pizza Runner
## B: Runner and Customer Experience
### Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```` sql 
Select 
  date_part(
    'week', registration_date + interval '3 days'
  ) as week, 
  count(runner_id) regestration_count 
from 
  pizza_runner.runners 
group by 
  week 
order by 
  week;
````
### Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```` sql 
Select 
  avg(time_taken) || ' mins' as avg_travel_time 
from 
  (
    Select 
      RO.order_id, 
      date_part(
        'minutes', RO.pickup_time - CO.order_time
      ) as time_taken 
    from 
      new_runner_orders RO 
      left join new_customer_orders CO on RO.order_id = CO.order_id 
    where 
      RO.cancellation is null 
    group by 
      RO.order_id, 
      RO.pickup_time, 
      CO.order_time 
    order by 
      RO.order_id
  ) as temp_table;
````
### Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```` sql, 
Time_calc_cte as (
  Select 
    RO.order_id, 
    count(CO.pizza_id) as No_of_pizza, 
    date_part(
      'minutes', RO.pickup_time - CO.order_time
    ) as time_taken 
  from 
    new_runner_orders RO 
    left join new_customer_orders CO on RO.order_id = CO.order_id 
  where 
    RO.cancellation is null 
  group by 
    RO.order_id, 
    time_taken
) 
Select 
  No_of_pizza, 
  avg(time_taken) || ' mins' as time_to_prepare 
from 
  Time_calc_cte 
group by 
  No_of_pizza;
````
### Q4. What was the average distance travelled for each customer?
```` sql 
Select 
  CO.customer_id, 
  avg(RO.distance) || ' KMs' as Avg_travelled 
from 
  new_runner_orders RO 
  left join new_customer_orders CO on CO.order_id = RO.order_id 
where 
  RO.cancellation is null 
group by 
  CO.customer_id 
order by 
  CO.customer_id;
```` ### Q5. What was the difference between the longest and shortest delivery times for all orders?
```` sql 
Select 
  max(duration) - min(duration) || ' mins' as Time_diff 
from 
  new_runner_orders;
````
### Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```` sql 
Select 
  runner_id, 
  order_id, 
  (
    sum(distance * 60)/ sum(duration)
  ) || ' KM/h' as speed 
from 
  new_runner_orders 
where 
  cancellation is null 
group by 
  runner_id, 
  order_id 
order by 
  runner_id, 
  order_id;
````
### Q7. What is the successful delivery percentage for each runner?
```` sql 
Select 
  runner_id, 
  sum(
    case when cancellation is null then 1 else 0 end
  )* 100 / count(order_id) || '%' as Sucess_perc 
from 
  new_runner_orders 
group by 
  runner_id 
order by 
  runner_id;
````
