# Case Study #3 - Foodie-Fi
## C. Challenge payment Question

#### The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
-	monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
-	upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
-	upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
-	once a customer churns they will no longer make payments

````sql
with RECURSIVE record_cte as (
  select 
    customer_id, 
    S.plan_id, 
    plan_name, 
    start_date, 
    ifnull(
      lead(start_date) over (
        partition by customer_id 
        order by 
          start_date
      ), 
      '2020-12-31'
    ) as plan_end_date, 
    price 
  from 
    subscriptions S 
    left join plans P on S.plan_id = P.plan_id 
  where 
    year(start_date) = 2020 
    and plan_name not in ('trial', 'churn') 
  union all 
  select 
    customer_id, 
    plan_id, 
    plan_name, 
    DATE_ADD(start_date, INTERVAL 1 MONTH) AS start_date, 
    plan_end_date, 
    price 
  FROM 
    record_cte 
  WHERE 
    plan_end_date > DATE_ADD(start_date, INTERVAL 1 MONTH) 
    AND plan_name != 'pro annual'
), 
record_cte2 as (
  SELECT 
    *, 
    LAG(plan_id, 1) OVER (
      PARTITION BY customer_id 
      ORDER BY 
        start_date
    ) AS previous_plan, 
    LAG(price, 1) OVER (
      PARTITION BY customer_id 
      ORDER BY 
        start_date
    ) AS prev_plan_amt, 
    RANK() OVER (
      PARTITION BY customer_id 
      ORDER BY 
        start_date
    ) AS payment_order 
  FROM 
    record_cte 
  ORDER BY 
    customer_id, 
    start_date
) 
SELECT 
  customer_id, 
  plan_id, 
  plan_name, 
  start_date AS payment_date, 
  CASE WHEN plan_name IN ('pro monthly', 'pro annual') 
  AND previous_plan = 1 THEN price - prev_plan_amt ELSE price END AS amount, 
  payment_order 
FROM 
  record_cte2;
````
