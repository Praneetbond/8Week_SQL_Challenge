# Case Study #4 - Data Bank
### B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?
````sql
select 
  txn_type, 
  count(*), 
  sum(txn_amount) 
from 
  customer_transactions 
group by 
  txn_type;
````
### 2. What is the average total historical deposit counts and amounts for all customers?
````sql
select 
  round(
    count(txn_type)/ count(distinct customer_id)
  ) Avg_deposit, 
  round(
    sum(txn_amount)/ count(distinct customer_id), 
    2
  ) as Avg_deposit_amt 
from 
  customer_transactions 
where 
  txn_type = "deposit";
````
### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
````sql
With temp_cte as (
  select 
    customer_id, 
    month(txn_date) as txn_month, 
    sum(
      case when txn_type = "deposit" then 1 else 0 end
    ) as deposit_count, 
    sum(
      case when txn_type = "purchase" then 1 else 0 end
    ) as purchase_count, 
    sum(
      case when txn_type = "withdrawal" then 1 else 0 end
    ) as withdrawal_count 
  from 
    customer_transactions 
  group by 
    customer_id, 
    txn_month
) 
select 
  txn_month, 
  count(customer_id) 
from 
  temp_cte 
where 
  deposit_count > 1 
  and (
    purchase_count >= 1 
    or withdrawal_count >= 1
  ) 
group by 
  txn_month 
order by 
  txn_month;
````
### 4. What is the closing balance for each customer at the end of the month?
````sql
with recursive temp_cte as (
  select 
    customer_id, 
    date_of_txn, 
    txn_figure, 
    ifnull(
      lead(date_of_txn) over (
        partition by customer_id 
        order by 
          date_of_txn
      ), 
      date_of_txn
    ) as next_txn 
  from 
    (
      select 
        customer_id, 
        cast(
          date_format(txn_date, '%Y-%m-01') as date
        ) as date_of_txn, 
        sum(
          case when txn_type = "deposit" then txn_amount -txn_amount end
        ) as txn_figure 
      from 
        customer_transactions 
      group by 
        customer_id, 
        date_of_txn 
      order by 
        customer_id, 
        date_of_txn
    ) as temp 
  union all 
  select 
    customer_id, 
    date_add(date_of_txn, interval 1 month) as date_of_txn, 
    txn_figure, 
    next_txn 
  from 
    temp_cte 
  where 
    next_txn > date_add(date_of_txn, interval 1 month)
) 
select 
  customer_id, 
  last_day(date_of_txn) as date_of_txn, 
  txn_figure 
from 
  temp_cte 
order by 
  customer_id, 
  date_of_txn;
````
### 5. What is the percentage of customers who increase their closing balance by more than 5%?
````sql
with recursive temp_cte as (
  select 
    customer_id, 
    date_of_txn, 
    txn_figure, 
    ifnull(
      lead(date_of_txn) over (
        partition by customer_id 
        order by 
          date_of_txn
      ), 
      date_of_txn
    ) as next_txn 
  from 
    (
      select 
        customer_id, 
        cast(
          date_format(txn_date, '%Y-%m-01') as date
        ) as date_of_txn, 
        sum(
          case when txn_type = "deposit" then txn_amount else -txn_amount end
        ) as txn_figure 
      from 
        customer_transactions 
      group by 
        customer_id, 
        date_of_txn 
      order by 
        customer_id, 
        date_of_txn
    ) as temp 
  union all 
  select 
    customer_id, 
    date_add(date_of_txn, interval 1 month) as date_of_txn, 
    txn_figure, 
    next_txn 
  from 
    temp_cte 
  where 
    next_txn > date_add(date_of_txn, interval 1 month)
) 
select 
  round(
    100 * count(*)/(
      select 
        count(distinct customer_id) 
      from 
        customer_transactions
    ), 
    2
  ) as perc_of_cust 
from 
  (
    select 
      customer_id, 
      date_of_txn, 
      txn_figure, 
      (
        txn_figure - Lead(txn_figure) over (
          partition by customer_id 
          order by 
            date_of_txn
        )
      )/ txn_figure as percentage 
    from 
      temp_cte 
    where 
      txn_figure > 0 
    order by 
      customer_id, 
      date_of_txn
  ) as Balance 
where 
  percentage > 0.5;
````


