# Case Study #7 - Balanced Tree Clothing Co.

### High Level Sales Analysis
#### 1.	What was the total quantity sold for all products?
````sql
select 
  sum(qty) as total_quantity 
from 
  sales;
````
#### 2.	What is the total generated revenue for all products before discounts?
````sql
select 
  sum(qty * price) as Total_revenue 
from 
  sales;
````
#### 3.	What was the total discount amount for all products?
````sql
select 
  round(
    sum(
      (qty * price * discount)/ 100
    ), 
    2
  ) as total_discount 
from 
  sales;
````
