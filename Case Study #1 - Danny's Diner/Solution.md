# Case Study #1: Danny’s Diner
## Main Case study:

### Q1. What is the total amount each customer spent at the restaurant?
````sql
Select 
  S.customer_id, 
  sum(M.price) as Amt_spent 
from 
  dannys_diner.sales S 
  left join dannys_diner.menu M on S.product_id = M.product_id 
group by 
  S.customer_id 
order by 
  S.customer_id;
````

### Q2. How many days has each customer visited the restaurant?
````sql
Select 
  customer_id, 
  count(distinct order_date) as visit_count 
from 
  dannys_diner.sales 
group by 
  customer_id 
order by 
  customer_id;
````

### Q3. What was the first item from the menu purchased by each customer?
````sql
Select 
  customer_id, 
  product_name 
from 
  (
    Select 
      S.customer_id, 
      M.product_name, 
      dense_rank() over (
        partition by S.customer_id 
        order by 
          S.order_date
      ) as food_rank 
    from 
      dannys_diner.sales S 
      join dannys_diner.menu M on S.product_id = M.product_id
  ) as temp_1 
where 
  food_rank = 1 
group by 
  customer_id, 
  product_name 
order by 
  customer_id;
````

### Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
Select 
  M.product_name, 
  count(S.product_id) as sell_count 
from 
  dannys_diner.sales S 
  join dannys_diner.menu M on S.product_id = M.product_id 
group by 
  M.product_name 
order by 
  sell_count DESC 
limit 
  1;
````

### Q5. Which item was the most popular for each customer?
````sql
Select 
  customer_id, 
  product_name 
from 
  (
    Select 
      customer_id, 
      product_name, 
      product_purch, 
      dense_rank() over (
        partition by customer_id 
        order by 
          product_purch DESC
      ) as Ranking 
    from 
      (
        Select 
          S.customer_id, 
          M.product_name, 
          count(S.product_id) as product_purch 
        from 
          dannys_diner.sales S 
          join dannys_diner.menu M on S.product_id = M.product_id 
        group by 
          S.customer_id, 
          M.product_name
      ) as temp_1 
    group by 
      customer_id, 
      product_name, 
      product_purch
  ) as Record2 
where 
  Ranking = 1;
````

### Q6. Which item was purchased first by the customer after they became a member?
````sql
Select 
  customer_id, 
  product_name 
from 
  (
    Select 
      S.customer_id, 
      M.product_name, 
      dense_rank() over (
        partition by S.customer_id 
        order by 
          S.order_date
      ) as Ranking 
    from 
      dannys_diner.sales S 
      join dannys_diner.menu M on S.product_id = M.product_id 
      join dannys_diner.members Mem on S.customer_id = Mem.customer_id 
    where 
      Mem.join_date <= S.order_date 
    order by 
      Ranking
  ) as temp_1 
where 
  Ranking = 1 
order by 
  customer_id;
````

### Q7. Which item was purchased just before the customer became a member?
````sql
Select 
  customer_id, 
  product_name 
from 
  (
    Select 
      S.customer_id, 
      M.product_name, 
      dense_rank() over (
        partition by S.customer_id 
        order by 
          S.order_date DESC
      ) as Ranking 
    from 
      dannys_diner.sales S 
      join dannys_diner.menu M on S.product_id = M.product_id 
      join dannys_diner.members Mem on S.customer_id = Mem.customer_id 
    where 
      Mem.join_date > S.order_date
  ) as temp_1 
where 
  Ranking = 1 
order by 
  customer_id;
````
### Q8. What is the total items and amount spent for each member before they became a member?
````sql
Select 
  S.customer_id, 
  count(S.product_id) as Total_purchase_item, 
  sum(M.price) as Total_amt_spend 
from 
  dannys_diner.sales S 
  join dannys_diner.menu M on S.product_id = M.product_id 
  join dannys_diner.members Mem on S.customer_id = Mem.customer_id 
where 
  S.order_date < Mem.join_date 
group by 
  S.customer_id 
order by 
  S.customer_id;
````

### Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
Select 
  S.customer_id, 
  sum(
    case when S.product_id = 1 then M.price * 10 * 2 else M.price * 10 end
  ) as Points 
from 
  dannys_diner.sales S 
  join dannys_diner.menu M on S.product_id = M.product_id 
group by 
  S.customer_id 
order by 
  S.customer_id;
````

### Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
Select 
  S.customer_id, 
  sum(
    case when S.product_id = 1 then M.price * 10 * 2 when S.order_date between Mem.join_date 
    and Mem.join_date + interval '6 Days' then M.price * 10 * 2 else M.price * 10 end
  ) as Points 
from 
  dannys_diner.sales S 
  join dannys_diner.menu M on S.product_id = M.product_id 
  join dannys_diner.members Mem on S.customer_id = Mem.customer_id 
where 
  S.order_date <= '2021-01-31' 
group by 
  S.customer_id 
order by 
  S.customer_id;
````

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
