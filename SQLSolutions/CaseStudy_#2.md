
# 📈 CASE STUDY #2 - Job Search Analysis 

## 😉 Queries

### 1. Calculate the weekly user engagement

We will be extracting week from the event occur date and then will group it and will see the number of distinct users in a week asnd thus we will get the weekly engaged users count. 

````sql
SELECT EXTRACT(WEEK FROM occurred_at) AS Week_Number, COUNT(DISTINCT(user_id)) AS Weekly_Engagement_Number
FROM yammer_events
GROUP BY Week_Number;
````

### 2. Calculate the weekly user growth

We will be using a temporary table by using WITH Clause, by extracting the week and year from the profile creation date and will check only those users that are active.

````sql
With Weekly_data as (
SELECT EXTRACT(year from created_at) AS Year_Number,
       EXTRACT(week from created_at) AS Week_Number, 
       COUNT(DISTINCT(user_id)) AS Weekly_Active_Users
FROM yammer_users
WHERE state = 'active'
GROUP BY Year_Number, Week_Number)

SELECT *,
SUM(Weekly_Active_Users) OVER (ORDER BY Year_Number, Week_Number asc)  as Total_Users
FROM Weekly_data;
````

### 3. Calculate the weekly engagement per device
We will calculate this by again extracting the year and week from the event occuring date and GROUP it by the device with the condition that the event type is engagement.

````sql
SELECT EXTRACT(year FROM occurred_at) AS Year_Name,
       EXTRACT(week FROM occurred_at) AS Week_Num,
       device,
       COUNT(DISTINCT(user_id)) as Device_num 
FROM yammer_events
WHERE event_type = 'engagement'
GROUP BY 1,2,3
ORDER BY 1,2,3 ASC;
````

### 4. Calculate the email engagement metrics

Since we are not being told about any specific metric so we will calculate the Email Clicks Rate and Email Opening Rate


````sql
WITH Email_Data AS
(
SELECT *,
       CASE WHEN (action = 'email_clickthrough') THEN 'Email Clicked'
            WHEN (action = 'email_open') THEN 'Email Opened'
            ELSE 'Email Sent'
        END AS action_details
FROM yammer_emails)

SELECT
ROUND((100.0 *SUM(CASE WHEN action_details IN ('Email Clicked') THEN 1 ELSE 0 END)/SUM(CASE WHEN action_details IN ('Email Sent') THEN 1 ELSE 0 END)),2) AS Email_Click_Rate,
ROUND((100.0 *SUM(CASE WHEN action_details IN ('Email Opened') THEN 1 ELSE 0 END)/SUM(CASE WHEN action_details IN ('Email Sent') THEN 1 ELSE 0 END)),2) AS Email_Open_Rate
FROM Email_Data;
````

### 5. Calculate the Customer Retention post sign up

This is one of the most important metric of any company.

We will be using the concept of self join here, where we will tr to focus that whether a user has tried to interact with the platform post joining, i.e. to look for the occurence of the user in a month other than the sign up month.

````sql
SELECT COUNT(user_id) AS Total_Users,
       SUM(CASE WHEN Retention_Activity =1 THEN 1 ELSE 0 END) AS Total_Retained
FROM
(
SELECT  Account_Created_Users.user_id, Account_Created_Users.SignUp_Week, Engagement_By_Users.engagement_week,
        (Engagement_By_Users.engagement_week - Account_Created_Users.SignUp_Week) AS Retention_Activity
FROM
(
(SELECT DISTINCT(user_id), EXTRACT(week FROM occurred_at) AS SignUp_Week FROM yammer_events
WHERE event_type = 'signup_flow' 
AND event_name = 'complete_signup'
) AS Account_Created_Users
LEFT JOIN
(
SELECT user_id, DATE(occurred_at) AS Engagement_Date, EXTRACT(week FROM occurred_at) AS Engagement_Week
FROM yammer_events
WHERE event_type = 'engagement') Engagement_By_Users
ON Account_Created_Users.user_id = Engagement_By_Users.user_id
)
ORDER BY Account_Created_Users.user_id
) AS All_Data
````


## Great! It's done 🎉

