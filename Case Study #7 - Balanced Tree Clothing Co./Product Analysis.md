# Case Study #7 - Balanced Tree Clothing Co.

### Product Analysis
#### 1. What are the top 3 products by total revenue before discount?
````sql
select 
  product_name, 
  sum(qty * S.price) as revenue 
from 
  sales S 
  left join product_details PD on S.prod_id = PD.product_id 
group by 
  product_name 
order by 
  revenue desc 
limit 
  3;
````
#### 2. What is the total quantity, revenue and discount for each segment?
````sql
select 
  segment_name, 
  sum(qty) as quentity, 
  sum(qty * S.price) as revenue, 
  round(
    sum(
      (qty * S.price * discount)/ 100
    ), 
    2
  ) as discount 
from 
  sales S 
  left join product_details PD on S.prod_id = PD.product_id 
group by 
  segment_name;
````
#### 3. What is the top selling product for each segment?
````sql
select 
  segment_name, 
  product_name, 
  format(revenue, 2) as revenue 
from 
  (
    select 
      segment_name, 
      product_name, 
      sum(qty * S.price)- sum(qty * S.price * discount / 100) as revenue, 
      dense_rank() over (
        partition by segment_name 
        order by 
          (
            sum(qty * S.price)- sum(qty * S.price * discount / 100)
          ) desc
      ) as index_id 
    from 
      sales S 
      left join product_details PD on S.prod_id = PD.product_id 
    group by 
      segment_name, 
      product_name
  ) as temp 
where 
  index_id = 1;
````
#### 4. What is the total quantity, revenue and discount for each category?
````sql
select 
  category_name, 
  sum(qty) as quentity, 
  sum(qty * S.price) as revenue, 
  round(
    sum(qty * S.price * discount / 100), 
    2
  ) as discount 
from 
  sales S 
  left join product_details PD on S.prod_id = PD.product_id 
group by 
  category_name;
````
#### 5. What is the top selling product for each category?
````sql
select 
  category_name, 
  product_name, 
  format(revenue, 2) as revenue 
from 
  (
    select 
      category_name, 
      product_name, 
      sum(qty * S.price)- sum(qty * S.price * discount / 100) as revenue, 
      dense_rank() over (
        partition by category_name 
        order by 
          sum(qty * S.price)- sum(qty * S.price * discount / 100) desc
      ) as index_id 
    from 
      sales S 
      left join product_details PD on S.prod_id = PD.product_id 
    group by 
      category_name, 
      product_name
  ) as temp 
where 
  index_id = 1;
````
#### 6. What is the percentage split of revenue by product for each segment?
````sql
select 
  segment_name, 
  product_name, 
  round(
    100 *(
      sum(qty * S.price)- sum(qty * S.price * discount / 100)
    )/(
      select 
        sum(qty * price)- sum(qty * price * discount / 100) 
      from 
        sales
    ), 
    2
  ) as percent_revenue 
from 
  sales S 
  left join product_details PD on S.prod_id = PD.product_id 
group by 
  segment_name, 
  product_name 
order by 
  percent_revenue desc;
````
#### 7. What is the percentage split of revenue by segment for each category?
````sql
select 
  category_name, 
  segment_name, 
  round(
    100 *(
      sum(qty * S.price)- sum(qty * S.price * discount / 100)
    )/(
      Select 
        sum(qty * price)- sum(qty * price * discount / 100) 
      from 
        sales
    ), 
    2
  ) as percent_revenue 
from 
  sales S 
  left join product_details PD on S.prod_id = PD.product_id 
group by 
  category_name, 
  segment_name 
order by 
  category_name, 
  percent_revenue desc;
````
#### 8. What is the percentage split of total revenue by category?
````sql
select 
  category_name, 
  round(
    100 *(
      sum(qty * S.price)- sum(qty * S.price * discount / 100)
    )/(
      select 
        sum(qty * price)- sum(qty * price * discount / 100) 
      from 
        sales
    ), 
    2
  ) as percent_revenue 
from 
  sales S 
  left join product_details PD on S.prod_id = PD.product_id 
group by 
  category_name;
````
#### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
````sql
select 
  product_name, 
  count(txn_id)/(
    select 
      count(distinct txn_id) 
    from 
      sales
  ) as penetration 
from 
  sales S 
  left join product_details PD on S.prod_id = PD.product_id 
group by 
  product_name;
````
#### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
````sql
with prod as (
  select 
    txn_id, 
    product_name 
  from 
    sales S 
    left join product_details PD on S.prod_id = PD.product_id
) 
select 
  prod1.product_name as product_1, 
  prod2.product_name as product_2, 
  prod3.product_name as product_3, 
  count(*) as purchase_count 
from 
  prod as prod1 
  join prod as prod2 on prod1.txn_id = prod2.txn_id 
  and prod1.product_name != prod2.product_name 
  and prod.product_name < prod2.product_name 
  join prod as prod3 on prod1.txn_id = prod3.txn_id 
  and prod1.product_name != prod3.product_name 
  and prod2.product_name != prod3.product_name 
  and prod1.product_name < prod3.product_name 
  and prod2.product_name < prod3.product_name 
group by 
  product_1, 
  product_2, 
  product_3 
order by 
  purchase_count desc 
limit 
  1;
````
