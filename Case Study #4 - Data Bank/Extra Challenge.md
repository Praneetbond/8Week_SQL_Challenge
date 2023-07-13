# Case Study #4 - Data Bank
## D. Extra Challenge (Half way done)

#### Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.
#### If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?
#### Special notes:
- Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation.
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
  customer_id, 
  last_day(date_of_txn) as mnth, 
  txn_figure as balance, 
  (
    case when txn_figure > 0 then (0.06 / 12)* txn_figure else 0 end
  ) as Yr_int 
from 
  temp_cte 
order by 
  customer_id, 
  date_of_txn;
````
