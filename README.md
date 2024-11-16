### GA4 BigQuery SQL Data Analysis Report

### Introduction:

This repository contains SQL queries and templates for analyzing Google Analytics 4 (GA4) data in BigQuery. It provides pre-built reports for user behavior, traffic sources, conversions, and more—helping analysts quickly extract valuable insights from their GA4 event data.

### Features:

1. Pre-built SQL queries for common GA4 analysis tasks.

2. Templates for key reports and metrics.
 
3. Easy integration with BigQuery projects.
  
4. Perfect for anyone looking to streamline GA4 data analysis in BigQuery.

### Custom SQL Query With Bigquery to get Better Actionable Insights:

#### ✅ New Vs Returning Users by Total users 
```sql
SELECT 
   case 
   when (select value.int_value from unnest(event_params) where event_name='session_start' and key='ga_session_number')= 1 then 'New User'
      when (select value.int_value from unnest(event_params) where event_name='session_start' and key='ga_session_number')> 1 then 'Returning_Users'
    else 'null' end  as User_Type,

    count(distinct user_pseudo_id) as total_user

 FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`

 group by User_Type;
```
##### Report: 
![Query 1 ](https://github.com/user-attachments/assets/5cc118a0-e8e0-4f95-87de-88db673a80ce)

#### ✅ Traffic Medium Name By Total user,Total Session,Unique Session, New session, engaged_sessions_per_user,number_of_sessions_per_user
```sql
select	

traffic_source.name,

count(user_pseudo_id) as total_user,

   count(concat(user_pseudo_id,(select value.int_value from unnest(event_params) where key='ga_session_id'))) as total_session,

   count(distinct concat(user_pseudo_id,(select value.int_value from unnest(event_params) where key='ga_session_id'))) as total_unique_session,

   count(distinct case when (select value.int_value from unnest(event_params) where key ='ga_session_number')=1 then concat(user_pseudo_id,(select value.int_value from unnest(event_params) where key='ga_session_id')) else null end) as new_session,

count(distinct case when (select value.string_value from unnest(event_params) where key = 'session_engaged') = '1' then concat(user_pseudo_id,(select value.int_value from unnest(event_params) where key = 'ga_session_id')) end) / count(distinct user_pseudo_id) as engaged_sessions_per_user,

count(distinct concat(user_pseudo_id,(select value.int_value from unnest(event_params) where key = 'ga_session_id'))) / count(distinct user_pseudo_id) as number_of_sessions_per_user

from  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`

group by 	traffic_source.name;
```
##### Report: 
![Query_3](https://github.com/user-attachments/assets/b8a049c2-8a5d-46ad-922d-3d9b1a269b26)

#### ✅ traffic_source.medium by total user, new User, New user(%), Returning User, and returning users (%) & active users 
```sql
select
  device.category,

  traffic_source.medium,

      count(user_pseudo_id) as total_user,

      count(case when (select value.int_value from unnest(event_params) where key ='ga_session_number')=1 then user_pseudo_id else null end)as new_users,

     round( count(case when (select value.int_value from unnest(event_params) where key ='ga_session_number')=1 then user_pseudo_id else null end) / count(user_pseudo_id),2) * 100 as new_users_pct,

          count(case when (select value.int_value from unnest(event_params) where key ='ga_session_number')>1 then user_pseudo_id else null end)as Returning_users,

     round( count(case when (select value.int_value from unnest(event_params) where key ='ga_session_number')>1 then user_pseudo_id else null end) / count(user_pseudo_id),2) * 100 as Returning_users_pct,

     count(case when (select value.int_value from unnest (event_params) where key ='engagement_time_msec') > 0 then user_pseudo_id else null end) as active_user

from  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
group by traffic_source.medium, device.category;
```
##### Report: 
![Query 2](https://github.com/user-attachments/assets/b392e79c-00ea-4d85-b150-4f426a804540)




#### ✅ Device Category Performance: Identify Conversion Differences by Device
This query provides insights into how users are converting across different devices and platforms (desktop, mobile, tablet), helping you determine if certain devices are underperforming.
```sql
SELECT 
     device.category as device_category,
     count( distinct user_pseudo_id) as total_user,
     countif(event_name='purchase') as total_purchase,
     sum(ecommerce.total_item_quantity) as total_item_sold,
     sum(ecommerce.purchase_revenue) as total_revenue,
     safe_divide(countif(event_name='purchase'),count(distinct user_pseudo_id)) as conversion_rate
 FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
 where event_name='purchase'
 group by device_category
 order by conversion_rate desc;
```
##### Report: 
![Screenshot_1](https://github.com/user-attachments/assets/2025570e-1864-4672-a2c1-7f3da8428e67)


#### ✅  Cart Abandonment rate  and checkout abandonment rate by Device Category
To improve Conversion Rate Optimization (CRO), especially around Average Order Value (AOV), Cart Abandonment, and Checkout Abandonment, understanding device-specific behavior is key.
Optimizing these areas across different devices can significantly impact overall conversion rates. Below are important queries to identify potential problems in these areas using GA4 data in BigQuery.
```sql
with dev_perform as ( 
  SELECT 
    device.category as device_category,
    user_pseudo_id, 
    countif(event_name='add_to_cart') as add_to_cart,
    countif(event_name='begin_checkout') as begin_checkout,
    countif(event_name='purchase') as purchase
 FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` 
 where event_name in ('add_to_cart','begin_checkout','purchase')
 group by device_category,user_pseudo_id
)

select
    device_category,
    count(distinct user_pseudo_id) as total_users,
    sum(add_to_cart) as total_add_to_cart,
    sum(begin_checkout) as total_begin_checkout,
    sum(purchase) as total_purchase,
    round(safe_divide(sum(add_to_cart)- sum(purchase),sum(add_to_cart)),2) as cart_abandonment_rate,
    round(safe_divide(sum(begin_checkout)- sum(purchase),sum(begin_checkout)),2) as checkout_abandonment_rate,
from dev_perform
group by device_category;
```
##### Report: 
![Screenshot_2](https://github.com/user-attachments/assets/20df8307-43a2-4601-9bed-ec34d2db1292)


#### ✅  Average Order Value (AOV) by Device Category
This query calculates the Average Order Value (AOV) for each device category. By understanding how much users spend on average across different devices, you can identify if certain devices are underperforming in terms of revenue generation.
```sql
WITH aov_data AS (
  SELECT
    user_pseudo_id,
    device.category AS device_category,
    SUM(event_params.value.int_value) AS total_purchase_value
  FROM
    `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*` AS events
  JOIN
    UNNEST(event_params) AS event_params
  WHERE
    event_name = 'purchase'
    AND event_params.key = 'value' -- assuming 'value' holds the transaction value
  GROUP BY
    user_pseudo_id, device.category
)

SELECT
  device_category,
  COUNT(DISTINCT user_pseudo_id) AS total_users,
  SUM(total_purchase_value) AS total_revenue,
  COUNT(DISTINCT user_pseudo_id) AS total_purchases,
  ROUND(SAFE_DIVIDE(SUM(total_purchase_value), COUNT(DISTINCT user_pseudo_id)),2) AS aov
FROM aov_data
GROUP BY device_category
ORDER BY aov DESC;
```
##### Report: 
![Screenshot_3](https://github.com/user-attachments/assets/724ad10f-f72a-40bb-8921-5ce85b2ad831)

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




