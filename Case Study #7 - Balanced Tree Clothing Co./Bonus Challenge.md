# Case Study #7 - Balanced Tree Clothing Co.

### Bonus Challenge

#### Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.
#### Hint: you may want to consider using a recursive CTE to solve this problem!

````sql
select 
  product_id, 
  price, 
  concat(
    PH.level_text, " ", PH2.level_text, 
    " - ", PH3.level_text
  ) as product_name, 
  PH2.parent_id as category_id, 
  PH.parent_id as segment_id, 
  PH.id as style_id, 
  PH3.level_text as category_name, 
  PH2.level_text as segment_name, 
  PH.level_text as style_name 
from 
  product_prices PP 
  left join product_hierarchy PH on PP.id = PH.id 
  left join product_hierarchy PH2 on PH.parent_id = PH2.id 
  left join product_hierarchy PH3 on PH2.parent_id = PH3.id;
````
