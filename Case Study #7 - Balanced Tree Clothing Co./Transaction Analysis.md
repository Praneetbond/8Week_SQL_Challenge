# Case Study #7 - Balanced Tree Clothing Co.

### Transaction Analysis
#### 1.	How many unique transactions were there?
````sql
select 
  count(distinct txn_id) as number_transactions 
from 
  sales;
````
#### 2.	What is the average unique products purchased in each transaction?
````sql
select 
  round(
    avg(product_count)
  ) as avg_product_count 
from 
  (
    select 
      txn_id, 
      count(distinct prod_id) as product_count 
    from 
      sales 
    group by 
      txn_id
  ) as temp;
````
#### 3.	What are the 25th, 50th and 75th percentile values for the revenue per transaction?
````sql
select 
  concat(percentile, "%") as percentile, 
  round(
    sum(revenue)/ count(*), 
    2
  ) as Value 
from 
  (
    select 
      txn_id, 
      sum(qty * price) as revenue, 
      ntile(100) over (
        order by 
          sum(qty * price)
      ) as percentile 
    from 
      sales 
    group by 
      txn_id
  ) as temp 
where 
  percentile in (25, 50, 75) 
group by 
  percentile;
````
#### 4.	What is the average discount value per transaction?
````sql
select 
  round(
    sum(
      (qty * price * discount)/ 100
    )/ count(distinct txn_id), 
    2
  ) as avg_discount 
from 
  sales;
````
#### 5.	What is the percentage split of all transactions for members vs non-members?
````sql
select 
  case when member = 0 then "False" else "True" end as members, 
  round(
    100 * count(distinct txn_id)/(
      select 
        count(distinct txn_id) 
      from 
        sales
    ), 
    2
  ) as txn_percentage 
from 
  sales 
group by 
  members;
````
#### 6.	What is the average revenue for member transactions and non-member transactions?
````sql
select 
  member, 
  round(
    (
      sum(qty * price)- sum(
        (qty * price * discount)/ 100
      )
    )/ count(distinct txn_id), 
    2
  ) as avg_revenue 
from 
  sales 
group by 
  member;
````
