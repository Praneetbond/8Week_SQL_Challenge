# Case Study #1 - Danny's Diner
## Bonus Case Study
### Join all the things
````sql
Select 
  S.customer_id, 
  S.order_date, 
  M.product_name, 
  M.price, 
  case when S.order_date >= Mem.join_date then 'Y' else 'N' end as member 
from 
  dannys_diner.sales S 
  join dannys_diner.menu M on S.product_id = M.product_id 
  left join dannys_diner.members Mem on S.customer_id = Mem.customer_id 
order by 
  S.customer_id, 
  S.order_date;
````

### Rank all the things
````sql
Select 
  *, 
  case when member = 'N' then null else rank() over (
    partition by customer_id, 
    member 
    order by 
      order_date
  ) end as Ranking 
from 
  (
    Select 
      S.customer_id, 
      S.order_date, 
      M.product_name, 
      M.price, 
      case when S.order_date >= Mem.join_date then 'Y' else 'N' end as member 
    from 
      dannys_diner.sales S 
      join dannys_diner.menu M on S.product_id = M.product_id 
      left join dannys_diner.members Mem on S.customer_id = Mem.customer_id 
    order by 
      1, 
      2
  ) as temp_1;
````
