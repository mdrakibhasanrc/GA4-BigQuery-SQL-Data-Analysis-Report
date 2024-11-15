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









