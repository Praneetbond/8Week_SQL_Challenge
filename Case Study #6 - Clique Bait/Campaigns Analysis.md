# Case Study #6 - Clique Bait
### 4. Campaigns Analysis
#### Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
````sql
select 
  user_id, 
  visit_id, 
  visit_time, 
  sum(views) as view_count, 
  sum(Add_to_cart) as Cart_add_count, 
  sum(purchases) as purchase_flag, 
  min(campaign_name) as campaign_name, 
  sum(Ad_Impression) as ad_impression_count, 
  sum(Ad_click) as click_count, 
  group_concat(products separator ', ') as products 
from 
  (
    select 
      user_id, 
      E.visit_id, 
      min(event_time) over (
        partition by E.visit_id 
        order by 
          sequence_number
      ) as visit_time, 
      case when event_name = "Page View" then 1 else 0 end as views, 
      case when event_name = "Add to Cart" then 1 else 0 end as Add_to_cart, 
      case when event_name = "Purchase" then 1 else 0 end as purchases, 
      campaign_name, 
      case when event_name = "Ad Impression" then 1 else 0 end as Ad_Impression, 
      case when event_name = "Ad Click" then 1 else 0 end as Ad_click, 
      case when event_name = "Add to Cart" then page_name else null end as products 
    from 
      events E 
      left join event_identifier EI on E.event_type = EI.event_type 
      left join page_hierarchy PH on E.page_id = PH.page_id 
      left join users U on E.cookie_id = U.cookie_id 
      left join campaign_identifier CI on PH.product_id between left(CI.products, 1) 
      and right(CI.products, 1) 
      and (
        event_time between CI.start_date 
        and CI.end_date
      )
  ) as temp 
group by 
  user_id, 
  visit_id, 
  visit_time 
order by 
  user_id, 
  visit_time;
````

#### Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.
#### Some ideas you might want to investigate further include:
- Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
````sql
select 
  round(
    100 * sum(
      case when impression_count = 0 then 1 else 0 end
    )/ count(*), 
    2
  ) as campaign_without_impression, 
  round(
    100 * sum(
      case when impression_count >= 1 then 1 else 0 end
    )/ count(*), 
    2
  ) as campaign_with_impression 
from 
  (
    select 
      user_id, 
      campaign_name, 
      sum(ad_impression_count) as impression_count 
    from 
      clique_bait_data 
    where 
      campaign_name is not null 
    group by 
      user_id, 
      campaign_name
  ) as temp;
````

- Does clicking on an impression lead to higher purchase rates?
````sql
select 
  purchase_flag, 
  round(
    100 * sum(ad_impression_count)/(
      select 
        sum(ad_impression_count) 
      from 
        clique_bait_data
    ), 
    2
  ) as purchase_rate 
from 
  clique_bait_data 
group by 
  purchase_flag;
````

- What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
````sql
select 
  ad_impression_count, 
  click_count, 
  round(
    100 * sum(purchase_flag)/(
      select 
        sum(purchase_flag) 
      from 
        clique_bait_data
    ), 
    2
  ) as purchase_rate 
from 
  clique_bait_data 
group by 
  ad_impression_count, 
  click_count;
````

- What metrics can you use to quantify the success or failure of each campaign compared to each other?
````sql
select 
  campaign_name, 
  sum(purchase_flag) as purchase_count, 
  sum(cart_add_count) as Product_count 
from 
  clique_bait_data 
where 
  campaign_name is not null 
group by 
  campaign_name;
````
