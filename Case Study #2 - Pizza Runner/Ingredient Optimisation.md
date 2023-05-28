# Case Study #2 - Pizza Runner
## C. Ingredient Optimisation
### 1. What are the standard ingredients for each pizza?
```` sql 
Select 
  min(
    case when TT.pizza_id = 1 then PT.topping_name else null end
  ) as Meat_lover_ingredients, 
  min(
    case when TT.pizza_id = 2 then PT.topping_name else null end
  ) as Vegetarian_ingredients 
from 
  (
    Select 
      pizza_id, 
      topping_ids, 
      row_number() over (
        partition by pizza_id 
        order by 
          topping_ids
      ) as ranking 
    from 
      toppings
  ) as TT 
  join pizza_runner.pizza_toppings PT on TT.topping_ids = PT.topping_id 
group by 
  ranking 
order by 
  ranking;
````
### 2. What was the most commonly added extra?
```` sql 
Select 
  PT.topping_name, 
  temp.extra_added 
from 
  pizza_runner.pizza_toppings PT, 
  (
    Select 
      cast(
        unnest(
          string_to_array(extras, ',')
        ) as int
      ) as topping_id, 
      count(order_id) as extra_added 
    from 
      new_customer_orders 
    where 
      extras is not null 
    group by 
      topping_id
  ) as temp 
where 
  PT.topping_id = temp.topping_id;
````

### 3. What was the most common exclusion?
````sql 
Select 
  PT.topping_name, 
  temp.Most_excl 
from 
  pizza_runner.pizza_toppings PT, 
  (
    Select 
      cast(
        unnest(
          string_to_array(exclusions, ',')
        ) as int
      ) as topping_id, 
      count(order_id) as Most_excl 
    from 
      new_customer_orders 
    where 
      exclusions is not null 
    group by 
      topping_id
  ) as temp 
where 
  PT.topping_id = temp.topping_id;
````

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
o  Meat Lovers
o  Meat Lovers - Exclude Beef
o  Meat Lovers - Extra Bacon
o  Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
````sql 
Select 
  concat(
    pizza_name, 
    case when Exc is not null then concat(' - Exclude ', Exc) else '' end, 
    case when extr is not null then concat(' - Extra ', Extr) else '' end
  ) as Order_table 
from 
  (
    Select 
      CO.order_id, 
      CO.index_id, 
      PN.pizza_name, 
      string_agg(distinct EC.excludes, ',') as Exc, 
      string_agg(distinct EX.extras, ',') as extr 
    from 
      (
        select 
          *, 
          row_number() over (partition by order_id) as index_id 
        from 
          new_customer_orders
      ) as CO 
      left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
      left join (
        select 
          order_id, 
          index_id, 
          case when exclude_id is not null then topping_name else null end as excludes 
        from 
          (
            Select 
              order_id, 
              row_number() over (partition by order_id) as index_id, 
              cast(
                unnest(
                  string_to_array(exclusions, ',')
                ) as int
              ) as exclude_id 
            from 
              new_customer_orders
          ) as temp 
          join pizza_runner.pizza_toppings PT on temp.exclude_id = PT.topping_id
      ) as EC on (
        CO.order_id = EC.order_id 
        and CO.index_id = EC.index_id
      ) 
      left join (
        select 
          order_id, 
          index_id, 
          case when extra_id is not null then topping_name else null end as extras 
        from 
          (
            Select 
              order_id, 
              row_number() over (partition by order_id) as index_id, 
              cast(
                unnest(
                  string_to_array(extras, ',')
                ) as int
              ) as extra_id 
            from 
              new_customer_orders
          ) as temp 
          join pizza_runner.pizza_toppings PT on temp.extra_id = PT.topping_id
      ) as EX on (
        CO.order_id = EX.order_id 
        and CO.index_id = EX.index_id
      ) 
    group by 
      CO.order_id, 
      CO.index_id, 
      PN.pizza_name
  ) as temp_2;
````

### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders  table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
````sql
, 
temp_1 as (
  select 
    order_id, 
    index_id, 
    pizza_name, 
    PT.topping_name, 
    PT.topping_id, 
    case when exclusions is not null 
    and topping_id in (
      select 
        cast(
          unnest(
            string_to_array(exclusions, ',')
          ) as int
        )
    ) then 1 else 0 end as excludes, 
    case when extras is not null 
    and topping_id in (
      select 
        cast(
          unnest(
            string_to_array(extras, ',')
          ) as int
        )
    ) then 1 else 0 end as Extras 
  from 
    pizza_runner.pizza_toppings PT, 
    (
      Select 
        *, 
        row_number() over () as index_id 
      from 
        new_customer_orders
    ) as CO 
    left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
    left join toppings TP on CO.pizza_id = TP.pizza_id 
  group by 
    order_id, 
    index_id, 
    pizza_name, 
    PT.topping_name, 
    PT.topping_id, 
    exclusions, 
    extras 
  order by 
    order_id, 
    index_id, 
    topping_id
), 
temp_2 as (
  select 
    order_id, 
    index_id, 
    topping_name, 
    topping_ids, 
    count(topping_name) as OG_tops 
  from 
    (
      Select 
        *, 
        row_number() over () as index_id 
      from 
        new_customer_orders
    ) as CO 
    left join toppings as TP on CO.pizza_id = TP.pizza_id 
    left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
  group by 
    order_id, 
    index_id, 
    topping_name, 
    topping_ids 
  order by 
    order_id, 
    index_id, 
    topping_ids
), 
temp_3 as (
  select 
    t1.order_id, 
    t1.index_id, 
    t1.pizza_name, 
    t1.topping_name, 
    t1.topping_id, 
    sum(
      (
        case when OG_tops is null then 0 else OG_tops end
      ) - t1.excludes + t1.extras
    ) as tops 
  from 
    temp_1 t1 
    left join temp_2 t2 on (
      t1.order_id = t2.order_id 
      and t1.index_id = t2.index_id 
      and t1.topping_id = t2.topping_ids
    ) 
  group by 
    t1.order_id, 
    t1.index_id, 
    t1.pizza_name, 
    t1.topping_name, 
    t1.topping_id 
  order by 
    t1.order_id, 
    t1.index_id, 
    t1.topping_id
) 
select 
  order_id, 
  concat(
    pizza_name, 
    ': ', 
    string_agg(data_1, ',')
  ) 
from 
  (
    select 
      order_id, 
      index_id, 
      pizza_name, 
      topping_id, 
      concat(
        case when tops > 1 then concat(tops, 'x') else '' end, 
        topping_name
      ) as data_1 
    from 
      temp_3 
    where 
      tops > 0
  ) as temp_4 
group by 
  order_id, 
  index_id, 
  pizza_name;
````

### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
````sql
, 
temp_1 as (
  select 
    order_id, 
    index_id, 
    pizza_name, 
    PT.topping_name, 
    PT.topping_id, 
    case when exclusions is not null 
    and topping_id in (
      select 
        cast(
          unnest(
            string_to_array(exclusions, ',')
          ) as int
        )
    ) then 1 else 0 end as excludes, 
    case when extras is not null 
    and topping_id in (
      select 
        cast(
          unnest(
            string_to_array(extras, ',')
          ) as int
        )
    ) then 1 else 0 end as Extras 
  from 
    pizza_runner.pizza_toppings PT, 
    (
      Select 
        CO.order_id, 
        CO.pizza_id, 
        exclusions, 
        extras, 
        row_number() over () as index_id 
      from 
        new_customer_orders CO 
        left join new_runner_orders RO on CO.order_id = RO.order_id 
      where 
        cancellation is null
    ) as CO 
    left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
    left join toppings TP on CO.pizza_id = TP.pizza_id 
  group by 
    order_id, 
    index_id, 
    pizza_name, 
    PT.topping_name, 
    PT.topping_id, 
    exclusions, 
    extras 
  order by 
    order_id, 
    index_id, 
    topping_id
), 
temp_2 as (
  select 
    order_id, 
    index_id, 
    topping_name, 
    topping_ids, 
    count(topping_name) as OG_tops 
  from 
    (
      Select 
        CO.order_id, 
        CO.pizza_id, 
        row_number() over () as index_id 
      from 
        new_customer_orders CO 
        left join new_runner_orders RO on CO.order_id = RO.order_id 
      where 
        cancellation is null
    ) as CO 
    left join toppings as TP on CO.pizza_id = TP.pizza_id 
    left join pizza_runner.pizza_names PN on CO.pizza_id = PN.pizza_id 
  group by 
    order_id, 
    index_id, 
    topping_name, 
    topping_ids 
  order by 
    order_id, 
    index_id, 
    topping_ids
) 
select 
  topping_name, 
  sum(tops) as Quantity 
from 
  (
    select 
      t1.order_id, 
      t1.index_id, 
      t1.pizza_name, 
      t1.topping_name, 
      t1.topping_id, 
      sum(
        (
          case when OG_tops is null then 0 else OG_tops end
        ) - t1.excludes + t1.extras
      ) as tops 
    from 
      temp_1 t1 
      left join temp_2 t2 on (
        t1.order_id = t2.order_id 
        and t1.index_id = t2.index_id 
        and t1.topping_id = t2.topping_ids
      ) 
    group by 
      t1.order_id, 
      t1.index_id, 
      t1.pizza_name, 
      t1.topping_name, 
      t1.topping_id 
    order by 
      t1.order_id, 
      t1.index_id, 
      t1.topping_id
  ) as temp_3 
group by 
  topping_name 
order by 
  Quantity DESC;
````
