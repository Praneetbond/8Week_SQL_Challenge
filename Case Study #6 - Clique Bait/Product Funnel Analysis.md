# Case Study #6 - Clique Bait
### 3. Product Funnel Analysis
#### Using a single SQL query - create a new output table which has the following details:
- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?
````sql
select 
  page_name, 
  sum(
    case when event_name = "Page View" then 1 else 0 end
  ) as view_count, 
  sum(
    case when event_name = "Add to Cart" then 1 else 0 end
  ) as cart_add_count, 
  sum(
    case when event_name = "Add to Cart" 
    and visit_id not in (
      select 
        visit_id 
      from 
        events E 
        left join event_identifier EI on E.event_type = EI.event_type 
      where 
        event_name = "Purchase"
    ) then 1 else 0 end
  ) as Not_purchased_count, 
  sum(
    case when event_name = "Add to Cart" 
    and visit_id in (
      select 
        visit_id 
      from 
        events E 
        left join event_identifier EI on E.event_type = EI.event_type 
      where 
        event_name = "Purchase"
    ) then 1 else 0 end
  ) as Purchase_count 
from 
  events E 
  left join event_identifier EI on E.event_type = EI.event_type 
  left join page_hierarchy PH on E.page_id = PH.page_id 
where 
  page_name not in (
    "Home Page", "All Products", "Checkout", 
    "Confirmation"
  ) 
group by 
  page_name;
````

#### Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
````sql
select 
  product_category, 
  sum(
    case when event_name = "Page View" then 1 else 0 end
  ) as view_count, 
  sum(
    case when event_name = "Add to Cart" then 1 else 0 end
  ) as cart_add_count, 
  sum(
    case when event_name = "Add to Cart" 
    and visit_id not in (
      select 
        visit_id 
      from 
        events E 
        left join event_identifier EI on E.event_type = EI.event_type 
      where 
        event_name = "Purchase"
    ) then 1 else 0 end
  ) as Not_purchased_count, 
  sum(
    case when event_name = "Add to Cart" 
    and visit_id in (
      select 
        visit_id 
      from 
        events E 
        left join event_identifier EI on E.event_type = EI.event_type 
      where 
        event_name = "Purchase"
    ) then 1 else 0 end
  ) as Purchase_count 
from 
  events E 
  left join event_identifier EI on E.event_type = EI.event_type 
  left join page_hierarchy PH on E.page_id = PH.page_id 
where 
  page_name not in (
    "Home Page", "All Products", "Checkout", 
    "Confirmation"
  ) 
group by 
  product_category;
````

#### Use your 2 new output tables - answer the following questions:

<details>
<summary>Click to expand SQL code for Product_table and Category_table </summary>

````sql
with product_table as (
  select 
    page_name as product, 
    sum(
      case when event_name = "Page View" then 1 else 0 end
    ) as view_count, 
    sum(
      case when event_name = "Add to Cart" then 1 else 0 end
    ) as cart_add_count, 
    sum(
      case when event_name = "Add to Cart" 
      and visit_id not in (
        select 
          visit_id 
        from 
          events E 
          left join event_identifier EI on E.event_type = EI.event_type 
        where 
          event_name = "Purchase"
      ) then 1 else 0 end
    ) as Not_purchased_count, 
    sum(
      case when event_name = "Add to Cart" 
      and visit_id in (
        select 
          visit_id 
        from 
          events E 
          left join event_identifier EI on E.event_type = EI.event_type 
        where 
          event_name = "Purchase"
      ) then 1 else 0 end
    ) as Purchase_count 
  from 
    events E 
    left join event_identifier EI on E.event_type = EI.event_type 
    left join page_hierarchy PH on E.page_id = PH.page_id 
  where 
    page_name not in (
      "Home Page", "All Products", "Checkout", 
      "Confirmation"
    ) 
  group by 
    page_name
), 

category_table as (
  select 
    product_category, 
    sum(
      case when event_name = "Page View" then 1 else 0 end
    ) as view_count, 
    sum(
      case when event_name = "Add to Cart" then 1 else 0 end
    ) as cart_add_count, 
    sum(
      case when event_name = "Add to Cart" 
      and visit_id not in (
        select 
          visit_id 
        from 
          events E 
          left join event_identifier EI on E.event_type = EI.event_type 
        where 
          event_name = "Purchase"
      ) then 1 else 0 end
    ) as Not_purchased_count, 
    sum(
      case when event_name = "Add to Cart" 
      and visit_id in (
        select 
          visit_id 
        from 
          events E 
          left join event_identifier EI on E.event_type = EI.event_type 
        where 
          event_name = "Purchase"
      ) then 1 else 0 end
    ) as Purchase_count 
  from 
    events E 
    left join event_identifier EI on E.event_type = EI.event_type 
    left join page_hierarchy PH on E.page_id = PH.page_id 
  where 
    page_name not in (
      "Home Page", "All Products", "Checkout", 
      "Confirmation"
    ) 
  group by 
    product_category
)
````
</details>

#### 1. Which product had the most views, cart adds and purchases?
````sql
select 
  max(
    case when view_count = (
      select 
        max(view_count) 
      from 
        product_table
    ) then product end
  ) as most_viewed_product, 
  max(
    case when cart_add_count = (
      select 
        max(cart_add_count) 
      from 
        product_table
    ) then product end
  ) as most_cart_add_product, 
  max(
    case when purchase_count = (
      select 
        max(purchase_count) 
      from 
        product_table
    ) then product end
  ) as most_purchased_product 
from 
  product_table;
````

#### 2. Which product was most likely to be abandoned?
````sql
select 
  product as most_abandoned_product 
from 
  product_table 
where 
  not_purchased_count = (
    select 
      max(not_purchased_count) 
    from 
      product_table
  );
````

#### 3. Which product had the highest view to purchase percentage?
````sql
select 
  product, 
  round(
    100 * purchase_count / view_count, 2
  ) as view_vs_purchase_percent 
from 
  product_table 
order by 
  view_vs_purchase_percent desc 
limit 
  1;
````

#### 4. What is the average conversion rate from view to cart add?
````sql
select 
  format(
    avg(conversion_rate), 
    4
  ) as avg_conversion_rate 
from 
  (
    select 
      product, 
      (view_count - cart_add_count)/ view_count as conversion_rate 
    from 
      product_table
  ) as temp;
````

#### 5. What is the average conversion rate from cart add to purchase?
````sql
select 
  format(
    avg(conversion_rate), 
    4
  ) as avg_conversion_rate 
from 
  (
    select 
      product, 
      (cart_add_count - purchase_count)/ cart_add_count as conversion_rate 
    from 
      product_table
  ) as temp;
````
