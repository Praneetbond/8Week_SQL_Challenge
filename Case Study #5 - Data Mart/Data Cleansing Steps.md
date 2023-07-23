# Case Study #5: Data Mart

## A. Data Cleansing Steps
#### In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:
- Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

Segment       | Age_band
------------- | -------------
1             | Young Adults
2             | Middle aged
3 or 4        | Retirees

- Add a new demographic column using the following mapping for the first letter in the segment values:

Segment       | Age_band
------------- | -------------
C             | Couples
F             | Families

- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
````sql
create view clean_weekly_sales as (
  select 
    txn_date, 
    week(txn_date) as week_number, 
    month(txn_date) as month_number, 
    year(txn_date) as calender_year, 
    region, 
    platform, 
    segment, 
    age_band, 
    demographic, 
    customer_type, 
    transactions, 
    sales, 
    avg_transation 
  from 
    (
      select 
        cast(
          concat(
            right(week_date, 2),
            "-", 
            left(right(week_date, 4), 
              1
            ), 
            "-", 
            left(week_date, length(week_date)-5
            )
          ) as date
        ) as txn_date, 
        region, 
        platform, 
        segment, 
        case
            when right(segment, 1) = 1 then "Young Adult"
            when right(segment, 1) = 2 then "Middle Aged"
            when right(segment, 1) in (3, 4) then "Retirees"
        else "unknown" end as age_band, 
        Case
            when left(segment, 1) = "C" then "Couples"
            when left(segment, 1) = "F" then "Families"
        else "unknown" end as demographic, 
        customer_type, 
        transactions, 
        sales, 
        round(sales / transactions, 2) as avg_transation 
      from 
        weekly_sales
    ) as temp 
  order by 
    week_number
);

````
