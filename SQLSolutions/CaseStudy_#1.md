
# 👷 CASE STUDY #1 - Job Search Analysis 

## 🤩 Queries

### 1. Calculate the number of jobs reviewed per hour per day for November 2020?

````sql
SELECT ds AS Jobs_Reviewed_Date, COUNT(job_id) AS Jobs_Count_per_day, (time_spent)/3600 AS Hours_Spent_per_day
FROM job_data
where ds >='2020-11-01'  and ds <='2020-11-30'  
GROUP BY Jobs_Reviewed_Date;
````

### 2. Calculate 7 day rolling average of throughput?

We will be calculating the rolling average of the time spent by the User with respect to the number of Jobs he/she searched for.


````sql
WITH throughput AS 
(SELECT ds AS Jobs_Reviewed_Date, COUNT(job_id) AS Jobs_Count_per_day, SUM(time_spent) AS Time_Spent
FROM job_data
WHERE ds >='2020-11-01'  and ds <='2020-11-30' 
GROUP BY ds)

SELECT *, 
SUM(Jobs_Count_per_day) OVER (ORDER BY Jobs_Reviewed_Date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) / SUM(Time_Spent) OVER (ORDER BY Jobs_Reviewed_Date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as 7_day_rolling
FROM throughput;
````

### 3. Calculate the percentage share of each language in the last 30 days?

````sql
with cte as
(SELECT language, count(job_id) AS number_of_jobs
FROM job_data
GROUP BY language)

SELECT *, ROUND(number_of_jobs*100/(SELECT sum(number_of_jobs) FROM cte),2) as Percent_Share
FROM cte;
````

### 4. Display duplicate rows present in the table.

We will solve this by two techniques:

Technique 1 - SIMPLE APPROACH
````sql
SELECT ds, job_id, actor_id, org, count(*) AS occurences
FROM job_data
GROUP BY ds, job_id, actor_id, org
HAVING occurences>1;
````

Technique 2 - WITH CLAUSE
````sql
WITH duplicate_check as
(SELECT *,
ROW_NUMBER() OVER (PARTITION BY ds, job_id, actor_id) as row_occurence 
FROM job_data)

SELECT * FROM duplicate_check WHERE row_occurence >1;
````
