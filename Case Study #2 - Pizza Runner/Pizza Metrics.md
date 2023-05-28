# Case Study #2 - Pizza Runner
## A: Pizza Metrics
### Q1. How many pizzas were ordered?
````sql
Select 
  count(pizza_id) as pizza_ordered_count 
from 
  new_customer_orders;
````

### Q2. How many unique customer orders were made?
````sql
Select 
  count(distinct order_id) as Unique_cust 
from 
  new_customer_orders;
````

### Q3. How many successful orders were delivered by each runner?
````sql
Select 
  runner_id, 
  count(order_id) as Succ_order_count 
from 
  new_runner_orders 
where 
  cancellation is null 
group by 
  runner_id 
order by 
  runner_id;
````

### Q4. How many of each type of pizza was delivered?
````sql
Select 
  PN.pizza_name, 
  count(CO.pizza_id) as Pizza_delivered_count 
from 
  new_customer_orders CO 
  join new_runner_orders RO on CO.order_id = RO.order_id 
  left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
where 
  RO.cancellation is null 
group by 
  PN.pizza_name;
````

### Q5. How many Vegetarian and Meatlovers were ordered by each customer?
```` sql 
Select 
  CO.customer_id, 
  PN.pizza_name, 
  count(CO.pizza_id) as num_of_orders 
from 
  new_customer_orders CO 
  join pizza_runner.pizza_names PN on PN.pizza_id = CO.pizza_id 
group by 
  CO.customer_id, 
  PN.pizza_name 
order by 
  CO.customer_id, 
  PN.pizza_name DESC;
```` 

### Q6. What was the maximum number of pizzas delivered in a single order?
```` sql 
Select 
  CO.order_id, 
  count(CO.pizza_id) as max_no_of_pizza_delivered 
from 
  new_customer_orders CO 
  join new_runner_orders RO on CO.order_id = RO.order_id 
where 
  RO.cancellation is null 
group by 
  CO.order_id 
order by 
  max_no_of_pizza_delivered DESC 
limit 
  1;
````
### Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```` sql 
Select 
  CO.customer_id, 
  sum(
    case when exclusions is null 
    and extras is null then 1 else 0 end
  ) as no_change, 
  sum(
    case when exclusions is not null 
    or extras is not null then 1 else 0 end
  ) as least_one_change 
from 
  new_customer_orders CO 
  join new_runner_orders RO on CO.order_id = RO.order_id 
where 
  RO.cancellation is null 
group by 
  CO.customer_id 
order by 
  CO.customer_id;
````

### Q8. How many pizzas were delivered that had both exclusions and extras?
```` sql 
Select 
  sum(
    case when exclusions is not null 
    and extras is not null then 1 else 0 end
  ) as Pizza_count 
from 
  new_customer_orders CO 
  join new_runner_orders RO on CO.order_id = RO.order_id 
where 
  RO.cancellation is null;
```` ### Q9. What was the total volume of pizzas ordered for each hour of the day?
```` sql 
Select 
  date_part('hour', order_time) as Hours, 
  count(pizza_id) as pizzas_ordered 
from 
  new_customer_orders 
group by 
  Hours 
order by 
  pizzas_ordered;
````

### Q10. What was the volume of orders for each day of the week?
```` sql 
Select 
  to_char(order_time, 'day') as days, 
  count(pizza_id) as pizzas_ordered 
from 
  new_customer_orders 
group by 
  days 
order by 
  pizzas_ordered;
````
