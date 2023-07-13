# Case Study #4 - Date Bank
## C. Data Allocation Challenge

#### To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:
- Option 1: data is allocated based off the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time
#### For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:
- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer
#### Using all of the data available - how much data would have been required for each option on a monthly basis?
````sql
with recursive temp_cte as (
  select 
    customer_id, 
    date_of_txn, 
    balance, 
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
          case when txn_type = "deposit" then txn_amount else - txn_amount end
        ) as balance 
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
    balance, 
    next_txn 
  from 
    temp_cte 
  where 
    next_txn > date_add(date_of_txn, interval 1 month)
), 
union_1 as (
  select 
    customer_id, 
    last_day(date_of_txn) as txn_date, 
    "--" as txn_type, 
    0 as transactions, 
    balance 
  from 
    temp_cte 
  order by 
    customer_id, 
    date_of_txn
), 
union_2 as (
  select 
    *, 
    ifnull(
      transactions + lag(transactions) over (
        partition by customer_id 
        order by 
          txn_date
      ), 
      transactions
    ) as balance 
  from 
    (
      select 
        customer_id, 
        txn_date, 
        txn_type, 
        case when txn_type = "deposit" then txn_amount else -(txn_amount) end as transactions 
      from 
        customer_transactions 
      order by 
        customer_id, 
        txn_date
    ) as temp
) 
select 
  *, 
  min(balance) over (partition by customer_id) as min_bal, 
  max(balance) over (partition by customer_id) as max_bal, 
  avg(balance) over (partition by customer_id) as avg_bal 
from 
  (
    select 
      * 
    from 
      union_1 
    union 
    select 
      * 
    from 
      union_2 
    order by 
      customer_id, 
      txn_date, 
      txn_type desc
  ) as temp;
````
