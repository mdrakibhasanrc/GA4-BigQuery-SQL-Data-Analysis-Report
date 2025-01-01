### GA4 BigQuery SQL Data Analysis for Conversion Rate Optimization Report


### Introduction:

This repository contains SQL queries and templates for analyzing Google Analytics 4 (GA4) data in BigQuery. It provides pre-built reports for user behavior, traffic sources, conversions, and more—helping analysts quickly extract valuable insights for Conversion Rate Optimization.

### Features:

1. Pre-built SQL queries for common GA4 analysis tasks.

2. Templates for key reports 
 
3. Easy integration with BigQuery projects.
  
4. Perfect for anyone looking to streamline GA4 data analysis in BigQuery.

### Data Collection & Setup (Integrate GA4 & BigQuery):

1. Configured Google Analytics 4 (GA4) to export data to BigQuery, enabling more detailed and granular analysis.

2. Set up automated queries to track product views, add-to-cart events, revenue, and user demographics to gather insights across different user segments.

### Custom SQL Query With Bigquery to get Better Actionable Insights:

#### ✅ Overall Site Performance Important Metrics:

```sql
with flat_data as (
  SELECT  
     user_pseudo_id,
     sum(ecommerce.purchase_revenue) as total_revenue,
     countif(event_name='page_view') as page_view,
     countif(event_name='add_to_cart') as add_to_cart,
     countif(event_name='begin_checkout') as begin_checkout,
     countif(event_name='purchase') as purchase
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
where event_name in ('page_view','add_to_cart','begin_checkout','purchase')
group by user_pseudo_id
)

select
    count(distinct user_pseudo_id) as total_user,
    sum(page_view) as total_page_view,
    sum(total_revenue) as total_sales,
    round(safe_divide(sum(add_to_cart)-sum(purchase),sum(add_to_cart))*100,2) as cart_abandonment_rate,
    round(safe_divide(sum(begin_checkout)-sum(purchase),sum(begin_checkout))*100,2) as checkout_abandonment_rate,
    safe_divide(sum(total_revenue),sum(purchase)) as average_order_value,
    round(safe_divide(sum(purchase),count(distinct user_pseudo_id))*100,2) as conversion_rate
from flat_data;

```

Report:

![Screenshot_4](https://github.com/user-attachments/assets/e804542e-f42f-4473-8c48-886fae5e3dcb)


Summary Insights:

       I) High abandonment at both the cart (90.28%) and checkout (85.31%) stages points to potential 
       friction in the purchase journey. Addressing issues like user experience, ease of 
       checkout, or offering incentives could reduce abandonment rates and improve conversions.

      II) Conversion rate (2.11%) is low, which might be expected given the high abandonment rates, 
      but it presents a clear opportunity for optimization in the conversion process
      (e.g.,  ptimizing for mobile users, improving site speed, or offering incentives for completing purchases).

      III) The Average Order Value of $63.63 suggests that, while revenue is being generated,
      higher AOV could be achievable through strategies like cross-selling, up-selling, or 
       introducing higher-value products.


##### ✅ Device Category Performance: Identify Conversion Rate Differences by Device:

This query provides insights into how users are converting across different devices and
platforms (desktop, mobile, tablet), helping you determine if certain devices are underperforming.
```sql
SELECT  
    device.category AS device_category,
    COUNT(DISTINCT user_pseudo_id) AS total_users,
    COUNTIF(event_name = 'purchase') AS total_purchase,
    SUM(CAST(ecommerce.total_item_quantity AS INT64)) AS total_sold, 
    SUM(CAST(ecommerce.purchase_revenue AS FLOAT64)) AS total_revenue, 
    ROUND(SAFE_DIVIDE(COUNT(DISTINCT IF(event_name = 'purchase', user_pseudo_id, NULL)), COUNT(DISTINCT IF(event_name = 'page_view', user_pseudo_id, NULL)))*100,2) AS conversion_rate  -- Correct conversion rate calculation
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE event_name IN ('purchase', 'page_view')  -- Consider both purchase and page view events
GROUP BY device_category
ORDER BY conversion_rate DESC;
```
Report:

![Screenshot_1](https://github.com/user-attachments/assets/c9e0f9cf-5fc3-4184-954a-ff59d326bae9)

Summary & Insights: 

        I) Mobile users have a higher conversion rate (1.7) than desktop (1.6), suggesting better purchase likelihood on mobile.
 
        II) Desktop users contribute more revenue ($208,815) and purchases (13,182 items) than mobile ($146,768 and 9,200 items).

        II) Tablet users underperform with the lowest revenue ($6,582) and purchases (111), indicating a need for optimization.


   ##### ✅ Cart Abandonment rate and checkout abandonment rate by Device Category:

To improve Conversion Rate Optimization (CRO), especially around Average Order Value (AOV), Cart Abandonment, 
and Checkout Abandonment, understanding device-specific behavior is key. Optimizing these areas across different
devices can significantly impact overall conversion rates. Below are important queries to identify potential 
problems in these areas using GA4 data in BigQuery.

```sql
with flat_data as (
  SELECT  
     device.category as device_category,
     count(distinct user_pseudo_id) as total_users,
     countif(event_name='add_to_cart') as add_to_cart,
     countif(event_name='begin_checkout') as begin_checkout,
     countif(event_name='purchase') as purchase,
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
where event_name in ('add_to_cart','begin_checkout','purchase')
group by device_category
)

select
   device_category,
   total_users,
   sum(add_to_cart) as total_add_to_cart,
   sum(begin_checkout) as total_begin_checkout,
   sum(purchase) as total_purchase,
   round(safe_divide(sum(add_to_cart)-sum(purchase),sum(add_to_cart))*100,2) as cart_abandonment_rate,
   round(safe_divide(sum(begin_checkout)-sum(purchase),sum(begin_checkout))*100,2) as checkout_abandonment_rate
from  flat_data
group by device_category,total_users;

```

Report:


![Screenshot_2](https://github.com/user-attachments/assets/1458ec8b-457c-4b56-93d8-63148a785916)


Summary & Insight: 

      I) Mobile vs. Desktop: Both have similar abandonment rates, but desktop users convert better, with more total purchases (3,226 vs. 2,355).

     II) Mobile vs. Tablet: Mobile has lower abandonment rates and higher purchases (2,355 vs. 111), while tablet users have the highest abandonment rates.

     III) Desktop vs. Tablet: Desktop users convert much better (3,226 vs. 111), with tablet users showing the highest abandonment rates and lowest purchases.

##### ✅ Average Order Value (AOV) by Device Category:

This query calculates the Average Order Value (AOV) for each device category. By understanding how much users spend
on average across different devices, you can identify if certain devices are underperforming in terms of revenue generation.

```sql
with flat_data as (
  SELECT  
     device.category as device_category,
     count(distinct user_pseudo_id) as total_users,
     sum(ecommerce.purchase_revenue) as total_sales
FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
where event_name ='purchase'
group by device_category
)

select
   device_category,
   total_users,
   total_sales,
   round(safe_divide(total_sales,total_users),2) as average_order_value
from  flat_data
group by device_category,total_users,total_sales;

```

Report:


![Screenshot_3](https://github.com/user-attachments/assets/faa7da23-adce-4d18-9862-3d76dd9f74e4)


Summary & Insight: 

       I) Desktop users have the highest total sales ($208,815) and AOV ($82.18), indicating larger purchases.

      II) Mobile users contribute significant sales ($146,768) with a slightly lower AOV ($79.29), suggesting smaller purchases.

      III) Tablet users have the lowest total sales ($6,582) and AOV ($67.86), indicating a need for optimization.


#### ✅ New Vs Returning Users by Total users and conversion rate
```sql
with user_sessions as (
  select
    user_pseudo_id,
    case
      when (select value.int_value from unnest(event_params) where key='ga_session_number')=1 then 'new_users'
      when (select value.int_value from unnest(event_params) where key='ga_session_number')>1 then 'returning_user'
      else null end as user_type
  from  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  where event_name='session_start'
),

user_purchase as (
  select
     user_pseudo_id,
     countif(event_name='purchase') as total_purchase
  from  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  where event_name='purchase'
  group by user_pseudo_id
)
select
   us.user_type,
   count(distinct us.user_pseudo_id) as total_users,
   count(distinct case when up.total_purchase>0 then us.user_pseudo_id end) as Purchase,
   round(safe_divide(count(distinct case when up.total_purchase>0 then up.user_pseudo_id end),count(distinct us.user_pseudo_id))*100,2) as conversion_rate
from user_sessions us
left join user_purchase up on up.user_pseudo_id=us.user_pseudo_id  
group by us.user_type
ORDER BY us.user_type;
```
##### Report: 


![Screenshot_1](https://github.com/user-attachments/assets/d126fd75-e395-4959-b2e3-af60f2c7082f)


Summary:
        New Users: 257,314 users, 3,511 made a purchase, with a 1.36% conversion rate.
        
        Returning Users: 51,395 users, 3,447 made a purchase, with a 6.71% conversion rate.
        
Key Findings:

        Despite having many more new users, returning users have a much higher conversion rate (6.71% vs. 1.36%).
        
        This suggests that retaining customers or improving engagement for new users could boost purchases.




#### ✅  Device Conversion Funnel Analysis
The Device Conversion Funnel Analysis helps you understand how users behave across different device types (e.g., mobile, desktop, tablet) as they progress through the conversion stages, from viewing products to completing a purchase. By tracking the device category (mobile, desktop, or tablet), you can identify where users drop off in the conversion funnel for each device type and take targeted actions to improve your site's performance.
This query calculates the Average Order Value (AOV) for each device category. By understanding how much users spend on average across different devices, you can identify if certain devices are underperforming in terms of revenue generation.
```sql
WITH device_conversion_funnel AS (
  SELECT 
    device.category AS device_category,
    user_pseudo_id,
    COUNTIF(event_name = 'view_item') AS product_views,
    COUNTIF(event_name = 'add_to_cart') AS add_to_cart,
    COUNTIF(event_name = 'begin_checkout') AS begin_checkout,
    COUNTIF(event_name = 'purchase') AS purchases
  FROM 
    `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  WHERE 
    event_name IN ('view_item', 'add_to_cart', 'begin_checkout', 'purchase')
  GROUP BY 
    device_category, user_pseudo_id
)

SELECT
    device_category,
    COUNT(DISTINCT user_pseudo_id) AS total_users,
    SUM(product_views) AS total_product_views,
    SUM(add_to_cart) AS total_add_to_cart,
    SUM(begin_checkout) AS total_begin_checkout,
    SUM(purchases) AS total_purchases,
    SAFE_DIVIDE(SUM(add_to_cart), SUM(product_views)) AS add_to_cart_rate,
    SAFE_DIVIDE(SUM(begin_checkout), SUM(add_to_cart)) AS checkout_initiation_rate,
    SAFE_DIVIDE(SUM(purchases), SUM(begin_checkout)) AS purchase_conversion_rate,
    SAFE_DIVIDE(SUM(purchases), SUM(product_views)) AS overall_conversion_rate
FROM 
    device_conversion_funnel
GROUP BY 
    device_category
ORDER BY 
    overall_conversion_rate DESC;
```
##### Report: 
![Screenshot_4](https://github.com/user-attachments/assets/2f1651e5-a1aa-4809-bec0-190f99d745f9)
![Screenshot_5](https://github.com/user-attachments/assets/c2ac19f8-a611-42da-9fa5-8c8cb1e1b4d5)


#### ✅  Traffic Channel Channel By

E-commerce platforms face challenges with high cart and checkout abandonment rates, leading to low conversion rates and missed revenue opportunities. Optimizing traffic sources and user journeys is crucial to improve performance metrics and drive growth.

```sql
with flat_data as (
  SELECT 
    traffic_source.medium as traffic_medium,
    user_pseudo_id,
    countif(event_name='page_view') as page_view,
    sum(ecommerce.purchase_revenue) as total_sales,
    countif(event_name='add_to_cart') as add_to_cart,
    countif(event_name='begin_checkout') as begin_checkout,
    countif(event_name='purchase') as purchase
 FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
 where event_name in ('page_view','add_to_cart','begin_checkout','purchase')
 group by traffic_source.medium, user_pseudo_id)

 select
     traffic_medium,
     count(distinct user_pseudo_id) as total_users,
     sum(page_view) as total_page_view,
     sum(add_to_cart) as total_add_to_cart,
     sum(begin_checkout) as total_begin_checkout,
     sum(purchase) as total_purchase,
     sum(total_sales) as total_revenue,
     round(safe_divide(sum(add_to_cart)-sum(purchase),sum(add_to_cart))*100,2) as cart_abandonment_rate,
     round(safe_divide(sum(begin_checkout)-sum(purchase),sum(begin_checkout))*100,2) as chekout_abandonment_rate,
     round(safe_divide(sum(total_sales),sum(purchase)),2) as AOV,
     round(safe_divide(sum(purchase),count(distinct user_pseudo_id))*100,2) as conversion_rate
from flat_data
group by traffic_medium
order by conversion_rate desc;    
```
##### Report: 

![Screenshot_1](https://github.com/user-attachments/assets/e9e10c79-8e0f-451e-bd5f-7d55c919fc07)

![Screenshot_2](https://github.com/user-attachments/assets/e001206d-0467-42ba-85fb-ac5529cd64c8)



#### ✅   Analyzing Conversion Funnel Data

Identifying drop-off points in your ecommerce store is crucial for understanding why users abandon their journey and how you can improve conversions. By analyzing these key stages, you can enhance the shopping experience and drive more sales. 

```sql
with dataset as (
SELECT 
    user_pseudo_id,
    event_name,
    parse_date('%Y%m%d',event_date) as event_date,
 FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
 where event_name in ('view_item','add_to_cart','begin_checkout','purchase')
),

view_item as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name='view_item'
),

add_to_cart as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name='add_to_cart'
),

begin_checkout as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name='begin_checkout'
),

purchase as (
  select
      user_pseudo_id,
      event_name,
      event_date
  from dataset
  where event_name='purchase'
),

funnel as (
  select
      vi.event_date,
      count(distinct vi.user_pseudo_id) as view_item_count,
      count(distinct atc.user_pseudo_id) as add_to_cart_count,
      count(distinct bc.user_pseudo_id) as begin_checkout_count,
      count(distinct p.user_pseudo_id) as purchase_count
  from view_item vi
  left join add_to_cart atc on vi.user_pseudo_id=atc.user_pseudo_id and vi.event_date=atc.event_date
  left join begin_checkout bc on atc.user_pseudo_id=bc.user_pseudo_id and atc.event_date=bc.event_date
  left join purchase p on bc.user_pseudo_id=p.user_pseudo_id and bc.event_date=p.event_date
  group by vi.event_date
)

select
   sum(view_item_count) as total_view_item_count,
   sum(add_to_cart_count) as total_add_to_cart_count,
   sum(begin_checkout_count) as total_begin_checkout_count,
   sum(purchase_count) as total_purchase_count,
   round(safe_divide( sum(add_to_cart_count),nullif(sum(view_item_count),0))*100,2) as add_to_cart_rate,
   round(safe_divide(sum(begin_checkout_count),nullif(sum(add_to_cart_count),0))*100,2) as begin_checkout_rate,
   round(safe_divide(sum(purchase_count),nullif(sum(begin_checkout_count),0))*100,2) as purchase_rate,
   round(safe_divide(sum(view_item_count) - sum(add_to_cart_count), nullif(sum(view_item_count), 0)) * 100, 2) as drop_off_after_view_item,
round(safe_divide(sum(add_to_cart_count) - sum(begin_checkout_count), nullif(sum(add_to_cart_count), 0)) * 100, 2) as drop_off_after_add_to_cart,
round(safe_divide(sum(begin_checkout_count) - sum(purchase_count), nullif(sum(begin_checkout_count), 0)) * 100, 2) as drop_off_after_begin_checkout
from funnel;   
```
##### Report: 

![Screenshot_1](https://github.com/user-attachments/assets/a6530845-7245-4367-b1bc-0a9b5c9c70b1)

![Screenshot_2](https://github.com/user-attachments/assets/2ca0a39e-a7e4-4798-963b-3217b038a6c9)


Key Insights from Conversion Funnel Analysis:

    Total View Items: 72,533 users viewed products in the store.
    
    Total Add to Cart: 14,382 users added products to their carts (19.83% conversion from view to cart).
    
    Total Begin Checkout: 5,846 users began the checkout process (40.65% conversion from cart to checkout).
    
    Total Purchases: 2,888 users completed their purchase (49.4% conversion from checkout to purchase).
        
Drop-Off Analysis:

    Drop-Off After View Item: 80.17% of users left after viewing products, indicating a need for improvement in product pages or motivation to add to cart.
  
    Drop-Off After Add to Cart: 59.35% abandoned their carts, highlighting potential issues in the cart or checkout process.
   
    Drop-Off After Begin Checkout: 50.6% of users abandoned checkout, suggesting areas for optimization in the final purchasing steps.
