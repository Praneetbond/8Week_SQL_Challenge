# Case Study #7 - Balanced Tree Clothing Co.
### Reporting Challenge
#### Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.
#### Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.
#### He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).
#### Feel free to split up your final outputs into as many tables as you need


> __Category Table__
>> _Details about Quantity, Discount, Revenue, Revenue Percent_
````sql
with sales_table as (
  select 
    style_name as product, 
    qty, 
    S.price, 
    S.discount, 
    format(
      sum(qty * S.price) - sum(qty * S.price * discount / 100), 
      2
    ) as revenue, 
    category_name, 
    segment_name, 
    txn_id, 
    start_txn_time 
  from 
    sales S 
    left join product_details PD on S.prod_id = PD.product_id 
  group by 
    product, 
    qty, 
    S.price, 
    S.discount, 
    category_name, 
    segment_name, 
    txn_id, 
    start_txn_time
) 
select 
  distinct category_name, 
  sum(qty) over (partition by category_name) as quantity, 
  format(
    sum(revenue) over (partition by category_name), 
    2
  ) as revenue, 
  format(
    sum(qty * price * discount / 100) over (partition by category_name), 
    2
  ) as discount, 
  format(
    sum(revenue) over (partition by category_name)/ sum(revenue) over (), 
    2
  ) as percent_revenue 
from 
  sales_table 
order by 
  percent_revenue desc;
````

> __Segment Table__
>> _Details about Quantity, Discount, Revenue, Revenue Percent_
````sql
with sales_table as (
  select 
    style_name as product, 
    qty, 
    S.price, 
    S.discount, 
    format(
      sum(qty * S.price) - sum(qty * S.price * discount / 100), 
      2
    ) as revenue, 
    category_name, 
    segment_name, 
    txn_id, 
    start_txn_time 
  from 
    sales S 
    left join product_details PD on S.prod_id = PD.product_id 
  group by 
    product, 
    qty, 
    S.price, 
    S.discount, 
    category_name, 
    segment_name, 
    txn_id, 
    start_txn_time
) 
select 
  distinct segment_name, 
  sum(qty) over (partition by segment_name) as quantity, 
  format(
    sum(revenue) over (partition by segment_name), 
    2
  ) as revenue, 
  format(
    sum(qty * price * discount / 100) over (partition by segment_name), 
    2
  ) as discount, 
  format(
    sum(revenue) over (partition by segment_name)/ sum(revenue) over (), 
    2
  ) as percent_revenue 
from 
  sales_table 
order by 
  percent_revenue desc;
````

> __Product Table__
>> _Details about Quantity, Discount, Revenue, Revenue Percent_
````sql
with sales_table as (
  select 
    style_name as product, 
    qty, 
    S.price, 
    S.discount, 
    format(
      sum(qty * S.price) - sum(qty * S.price * discount / 100), 
      2
    ) as revenue, 
    category_name, 
    segment_name, 
    txn_id, 
    start_txn_time 
  from 
    sales S 
    left join product_details PD on S.prod_id = PD.product_id 
  group by 
    product, 
    qty, 
    S.price, 
    S.discount, 
    category_name, 
    segment_name, 
    txn_id, 
    start_txn_time
) 
select 
  distinct product, 
  sum(qty) over (partition by product) as quantity, 
  format(
    sum(revenue) over (partition by product), 
    2
  ) as revenue, 
  format(
    sum(qty * price * discount / 100) over (partition by product), 
    2
  ) as discount, 
  format(
    sum(revenue) over (partition by product)/ sum(revenue) over (), 
    2
  ) as percent_revenue 
from 
  sales_table 
order by 
  percent_revenue desc;
````

> __Extra Table__
>> _Details about Transaction count, 25th + 50th + 75th Percentile, Discount, Average Product Count, Member Count_
````sql
select 
  count(txn_id) as unique_txns, 
  format(
    sum(
      case when index_id = 25 then revenue - discount else 0 end
    ), 
    2
  ) as 25th_percentile, 
  format(
    sum(
      case when index_id = 50 then revenue - discount else 0 end
    ), 
    2
  ) as 50th_percentile, 
  format(
    sum(
      case when index_id = 75 then revenue - discount else 0 end
    ), 
    2
  ) as 75th_percentile, 
  format(
    sum(discount)/ count(txn_id), 
    2
  ) as avg_discount, 
  round(
    avg(prod_count)
  ) as average_product_per_txn, 
  sum(case when member = 1 then 1 else 0 end) as member_count, 
  sum(case when member = 0 then 1 else 0 end) as non_member_count 
from 
  (
    select 
      txn_id, 
      sum(qty * price) as revenue, 
      sum(qty * price * discount / 100) as discount, 
      count(prod_id) as prod_count, 
      member, 
      ntile(100) over (
        order by 
          (
            sum(qty * price) - sum(qty * price * discount / 100)
          )
      ) as index_id 
    from 
      sales 
    group by 
      txn_id, 
      member
  ) as temp;
````

> __Member Table__
>> _Details about Transaction count and Percentage, Revenue and Percentage_
````sql
with temp as (
  select 
    *, 
    qty * price as revenue, 
    qty * price * discount / 100 as discount_amt 
  from 
    sales
) 
select 
  case when member = 1 then "member" else "non_member" end as "", 
  concat(
    count(distinct txn_id), 
    " - ", 
    round(
      100 * count(distinct txn_id)/(
        select 
          count(distinct txn_id) 
        from 
          temp
      ), 
      2
    ), 
    "%"
  ) as transaction, 
  format(
    sum(revenue - discount_amt), 
    2
  ) as revenue, 
  round(
    100 * sum(revenue - discount_amt)/(
      select 
        sum(revenue - discount) 
      from 
        temp
    ), 
    2
  ) as revenue_percent 
from 
  temp 
group by 
  member;
````

> ___Top Product by Category and Segment wise___
````sql
select 
  category_name, 
  segment_name, 
  product, 
  revenue 
from 
  (
    select 
      category_name, 
      segment_name, 
      style_name as product, 
      format(
        sum(qty * S.price)- sum(qty * S.price * discount / 100), 
        2
      ) as revenue, 
      dense_rank() over (
        partition by segment_name 
        order by 
          sum(qty * S.price) - sum(qty * S.price * discount / 100) desc
      ) as index_id 
    from 
      sales S 
      left join product_details PD on S.prod_id = PD.product_id 
    group by 
      category_name, 
      segment_name, 
      product
  ) as temp 
where 
  index_id = 1 
order by 
  revenue desc;
````
