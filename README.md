
## Create and Populate Activity Table (DDL and DML)
```{sql}
CREATE table activity
(
  user_id varchar(20),
  event_name varchar(20),
  event_date date,
  country varchar(20)
);
DELETE FROM activity;
INSERT INTO activity VALUES 
  (1, 'app-installed', '2022-01-01', 'India'),
  (1, 'app-purchase', '2022-01-02', 'India'),
  (2, 'app-installed', '2022-01-01', 'USA'),
  (3, 'app-installed', '2022-01-01', 'USA'),
  (3, 'app-purchase', '2022-01-03', 'USA'),
  (4, 'app-installed', '2022-01-03', 'India'),
  (4, 'app-purchase', '2022-01-03', 'India'),
  (5, 'app-installed', '2022-01-03', 'SL'),
  (5, 'app-purchase', '2022-01-03', 'SL'),
  (6, 'app-installed', '2022-01-04', 'Pakistan'),
  (6, 'app-purchase', '2022-01-04', 'Pakistan');

SELECT * FROM activity;
```

## Total Active Users Per Day
This query calculates the total number of unique active users for each day.
```{sql}
SELECT event_date,
       COUNT(DISTINCT user_id) AS total_active_users
FROM activity
GROUP BY event_date;
```

## Weekly Active Users
This query calculates the total number of unique active users for each week.
```{sql}
SELECT DATEPART(WEEK, event_date) AS weeknumber,
       COUNT(DISTINCT user_id) AS total_active_users
FROM activity
GROUP BY DATEPART(WEEK, event_date);
```

## Date-Wise Total Number of Users Who Made a Purchase the Same Day They Installed the App
This query finds the total number of users who installed and made a purchase on the same day.
```{sql}
WITH cte AS (
  SELECT a1.event_date, COUNT(DISTINCT a1.user_id) AS total_users
  FROM activity a1
  JOIN activity a2 
    ON a1.user_id = a2.user_id 
   AND a1.event_date = a2.event_date
   AND a1.event_name <> a2.event_name
  GROUP BY a1.event_date
), cte2 AS (
  SELECT DISTINCT event_date
  FROM activity
)
SELECT c2.event_date, COALESCE(c1.total_users, 0) AS no_of_users
FROM cte c1
RIGHT JOIN cte2 c2
  ON c1.event_date = c2.event_date;
```

## Percentage of Paid Users in India, USA, and Other Countries
This query calculates the percentage of users who made a purchase, categorized by country.
```{sql}
WITH cte AS (
  SELECT CASE 
           WHEN (country = 'India' AND event_name = 'app-purchase') OR (country = 'USA' AND event_name = 'app-purchase') THEN country 
           WHEN (country <> 'India' OR country <> 'USA') AND event_name = 'app-purchase' THEN 'Others'
         END AS Country
  FROM activity
)
SELECT Country, COUNT(*) * 100 / (SELECT COUNT(*) FROM activity WHERE event_name = 'app-purchase') AS Percentage_Users
FROM cte 
WHERE Country IS NOT NULL
GROUP BY Country;
```

## Users Who Installed on a Given Day and Purchased on the Next Day
This query finds the number of users who installed the app on one day and made a purchase the next day.
```{sql}
WITH cte AS (
  SELECT a2.event_date, COUNT(a1.user_id) AS cnt
  FROM activity a1
  JOIN activity a2 
    ON a1.user_id = a2.user_id
   AND DATEPART(DAY, a1.event_date) = DATEPART(DAY, a2.event_date) - 1 
  GROUP BY a2.event_date
), cte1 AS (
  SELECT DISTINCT event_date 
  FROM activity
)
SELECT c2.event_date, COALESCE(c1.cnt, 0) AS cnt
FROM cte c1
RIGHT JOIN cte1 c2
  ON c1.event_date = c2.event_date;
```

## Explanation of Queries
1. **Create and Populate Activity Table**: This section creates a table named `activity` and populates it with sample data representing user activities.
2. **Total Active Users Per Day**: This query calculates the number of unique users active on each day by counting distinct `user_id`s for each `event_date`.
3. **Weekly Active Users**: This query calculates the number of unique users active each week using the `DATEPART(WEEK, event_date)` function to group by week number.
4. **Users Making Purchase on Installation Day**: This query uses Common Table Expressions (CTEs) to join activities of the same user on the same day but with different events (installation and purchase), and counts these occurrences per day.
5. **Percentage of Paid Users by Country**: This query classifies countries into 'India', 'USA', and 'Others', and calculates the percentage of users who made purchases in each category.
6. **Users Installing and Purchasing on Consecutive Days**: This query identifies users who installed the app on one day and made a purchase the next day by joining activity records offset by one day.
