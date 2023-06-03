# Case Study #2 - Pizza Runner
## Pricing and Ratings

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```` sql 
select 
  sum(
    case when pizza_name = 'Meatlovers' then 12 else 10 end
  ) as Amount_earned 
from 
  new_customer_orders CO 
  left join new_runner_orders RO on CO.order_id = RO.order_id 
  left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
where 
  cancellation is null;
```` 

### 2. What if there was an additional $1 charge for any pizza extras?
#### Add cheese is $1 extra 
```` sql, 
  pizza_profit as (
    select 
      sum(
        case when pizza_name = 'Meatlovers' then 12 else 10 end
      ) as Amount_earned 
    from 
      new_customer_orders CO 
      left join new_runner_orders RO on CO.order_id = RO.order_id 
      left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
    where 
      cancellation is null
  ), 
  extras_profit as (
    select 
      count(*) as extra_adds 
    from 
      (
        select 
          cast(
            unnest(
              string_to_array(extras, ',')
            ) as int
          ) 
        from 
          new_customer_orders 
        where 
          order_id in (
            select 
              order_id 
            from 
              new_runner_orders 
            where 
              cancellation is null
          )
      ) as temp_1
  ) 
select 
  Amount_earned + extra_adds as total_profit 
from 
  pizza_profit, 
  extras_profit;
```` 

### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```` sql 
DROP TABLE IF EXISTS ratings;

CREATE TABLE ratings (
  "order_id" INTEGER, 
  "rating" INTEGER, 
  "submission_time" timestamp, 
  "comments" varchar(155)
);

INSERT INTO ratings ("order_id", "rating", "submission_time", "comments") 
VALUES 
        ('1','3','2020-01-01 18:57:02','Speed up the delivery'),
        ('2','5','2020-01-01 19:47:54','Yummy!!'),
        ('3','4','2020-01-03 00:42:37','Its good'),
        ('4','2','2020-01-04 14:43:03','Worst, no time managing'),
        ('5','5','2020-01-08 21:52:57','Just awesome'),
        ('7','5','2021-01-08 22:05:45','Great delivery and food'),
        ('8','5','2020-01-10 00:40:02','Its perfect'),
        ('10','4','2020-01-11 19:10:20','Rate little high');
```` 

### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id 
- order_id 
- runner_id 
- rating 
- order_time 
- pickup_time 
- Time between order and pickup 
- Delivery duration 
- Average speed
- Total number of pizzas 

```` sql 
select 
  customer_id, 
  CO.order_id, 
  runner_id, 
  rating, 
  order_time, 
  pickup_time, 
  (pickup_time - order_time) as time_bw_order_pickup, 
  duration || ' mins' as duration, 
  (
    (distance * 60)/ duration
  ) || ' Km/h' as speed, 
  count(*) as pizza_count 
from 
  new_customer_orders CO 
  left join new_runner_orders RO on CO.order_id = RO.order_id 
  left join pizza_runner.ratings R on CO.order_id = R.order_id 
where 
  cancellation is null 
group by 
  customer_id, 
  CO.order_id, 
  runner_id, 
  order_time, 
  pickup_time, 
  duration, 
  distance, 
  rating 
order by 
  order_id;
```` 

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```` sql 
select 
  sum(
    case when pizza_name = 'Meatlovers' then 12 else 10 end
  ) - runner_charges as net_profit 
from 
  (
    select 
      sum(distance * 0.30) as runner_charges 
    from 
      new_runner_orders 
    where 
      cancellation is null
  ) as expence, 
  new_customer_orders CO 
  left join new_runner_orders RO on CO.order_id = RO.order_id 
  left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
where 
  cancellation is null 
group by 
  runner_charges;
````
