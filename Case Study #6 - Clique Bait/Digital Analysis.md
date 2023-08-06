# Case Study #6 - Clique Bait
### 2. Digital Analysis
#### Using the available datasets - answer the following questions using a single query for each one:

#### 1. How many users are there?
````sql
select 
  count(distinct user_id) as num_of_users 
from 
  users;
````

#### 2. How many cookies does each user have on average?
````sql
select 
  count(cookie_id)/ count(distinct user_id) as avg_cookies 
from 
  users;
````

#### 3. What is the unique number of visits by all users per month?
````sql
select 
  monthname(event_time) as months, 
  count(distinct visit_id) as num_of_visits 
from 
  events 
group by 
  months, 
  month(event_time) 
order by 
  month(event_time);
````

#### 4. What is the number of events for each event type?
````sql
select 
  event_name, 
  count(*) as num_of_events 
from 
  events E 
  left join event_identifier EI on E.event_type = EI.event_type 
group by 
  event_name;
````

#### 5. What is the percentage of visits which have a purchase event?
````sql
select 
  round(
    100 * count(*)/(
      select 
        count(distinct visit_id) 
      from 
        events
    ), 
    2
  ) as percent_visits 
from 
  events E 
  left join event_identifier EI on E.event_type = EI.event_type 
where 
  event_name = "Purchase";
````

#### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
````sql
select 
  round(
    100 * count(*)/(
      select 
        count(distinct visit_id) 
      from 
        events
    ), 
    2
  ) as percent_visits 
from 
  (
    select 
      visit_id, 
      count(*) as txn 
    from 
      events E 
      left join page_hierarchy PH on E.page_id = PH.page_id 
      left join event_identifier EI on E.event_type = EI.event_type 
    where 
      event_name = "Purchase" 
      or page_name = "Checkout" 
    group by 
      visit_id 
    having 
      txn = 1
  ) as temp;
````

#### 7. What are the top 3 pages by number of views?
````sql
select 
  page_name, 
  count(*) as Num_of_views 
from 
  events E 
  left join page_hierarchy PH on E.page_id = PH.page_id 
  left join event_identifier EI on E.event_type = EI.event_type 
where 
  event_name = "Page View" 
group by 
  page_name 
order by 
  Num_of_views desc 
limit 
  3;
````

#### 8. What is the number of views and cart adds for each product category?
````sql
select 
  product_category, 
  sum(
    case when event_name = "Page View" then 1 else 0 end
  ) as View_count, 
  sum(
    case when event_name = "Add to Cart" then 1 else 0 end
  ) as Cart_count 
from 
  events E 
  left join page_hierarchy PH on E.page_id = PH.page_id 
  left join event_identifier EI on E.event_type = EI.event_type 
where 
  product_category is not null 
group by 
  product_category;
````

#### 9. What are the top 3 products by purchases?
````sql
select 
  page_name, 
  count(*) as purchase_count 
from 
  (
    select 
      page_name, 
      event_name 
    from 
      events E 
      left join page_hierarchy PH on E.page_id = PH.page_id 
      left join event_identifier EI on E.event_type = EI.event_type 
    where 
      visit_id in (
        select 
          visit_id 
        from 
          events E 
          left join page_hierarchy PH on E.page_id = PH.page_id 
        where 
          page_name = "Confirmation"
      ) 
      and event_name = "Add to Cart"
  ) as temp 
group by 
  page_name 
order by 
  purchase_count desc 
limit 
  3;
````
