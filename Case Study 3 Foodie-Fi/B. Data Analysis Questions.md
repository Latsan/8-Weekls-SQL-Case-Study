# B. Data Analysis Questions
### 1. How many customers has Foodie-Fi ever had?
```
SELECT 
	COUNT(DISTINCT customer_id) as customer
FROM foodie_fi.subscriptions
```

![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/dc7f3f89-d7fe-4f60-ab18-e734e2197968)
- Foodie_Fi has 1000 customers

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```
SELECT 
	DATE_PART('month',start_date) as month_number,
	count(*) as cnt
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
WHERE plan_name = 'trial'
group by 1
order by 1
```

![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/e2166bdb-cba4-4761-b8bd-477ebc59e628)
- March has the highest onboarding rate
- February has the least onboarding rate
### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```
SELECT 
	plan_id,
	plan_name,
	count(*) as cnt
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
WHERE  start_date > '2020-12-31'
group by 1,2
order by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/7721cee2-9581-4d9a-b207-76e11995ff23)

After year 2022, the `foodie_fi` has
- 8 basic monthly users
- 60 pro montly users
- 63 pro annual users
- 71 users churned

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```
SELECT 
	count(distinct customer_id) as churn_customers,
 	CONCAT(ROUND(100 * count(distinct customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),1),' %') as "% churned"
	
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
WHERE  plan_name = 'churn'  
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/2d566e47-9e85-424d-b42d-1f0799038287)
- 30.7% customers has churned from foodie_fi project

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```
SELECT  
	count(distinct customer_id) as cus_cnt,
	CONCAT(ROUND(100 * count(distinct customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),1),'%') as "%_cnt"
FROM (
	SELECT *,
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) as _rank

	FROM foodie_fi.subscriptions
	JOIN foodie_fi.plans USING(plan_id)
)
WHERE _rank = 2 AND plan_name = 'churn'
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/3d02f15e-d91b-451a-94f4-a8a9e7c6cdb3)

- 9.2% of the total users churned immdediately after the first free trial pf 7 days
### 6. What is the number and percentage of customer plans after their initial free trial?

```
SELECT  
	plan_id,
	plan_name,
	count(distinct customer_id) as cus_cnt,
	CONCAT(ROUND(100 * count(distinct customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),1),'%') as "%_cnt"
FROM (
	SELECT *,
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) as _rank

	FROM foodie_fi.subscriptions
	JOIN foodie_fi.plans USING(plan_id)
)
WHERE _rank = 2 
GROUP BY 1,2
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/9f86011e-ddd2-4b38-b3dd-a3c1f6c2fd23)

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```
SELECT plan_id,
	count(*),
	CONCAT(100* COUNT(*)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),'%') as "%_count"
FROM(

	SELECT customer_id,
		plan_id,
		LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) as next_date
	FROM foodie_fi.subscriptions
	WHERE start_date <= '2020-12-31'
	)
WHERE next_date IS NULL
GROUP BY 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/5ad8d244-828e-43c8-85f5-4827fa863db2)

### 8. How many customers have upgraded to an annual plan in 2020?

```
SELECT COUNT(DISTINCT customer_id)
FROM foodie_fi.subscriptions
WHERE start_date BETWEEN '2020-01-01' AND '2020-12-31'
	AND plan_id = 3
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/fe0d0b49-7562-4ade-a08f-99cfa5c9e0fd)
- 195 customers upgraded to a `pro annual plan` in the yeal 2020

### 9.How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
- I calculated the numbers of users with `trial` subscription in a CTE, I also calculated for `annual` subscription and I subtracted the diffrence between the two CTEs and then find the average
```
WITH 
	trial as(
		SELECT *
		FROM foodie_fi.subscriptions
		WHERE plan_id = 0
),
annual as(
	SELECT *
	FROM foodie_fi.subscriptions
	WHERE plan_id = 3
)
SELECT 
	ROUND(AVG(annual.start_date - trial.start_date))
FROM trial
JOIN annual USING (customer_id)
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/5e7197b8-7fc0-49a2-a725-a1756ca3f101)

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
- To get this solved, I introduce a function called `WIDTH_BUCKET` to help be calculate the width bucket of number of days in 30 days interval
```
WITH 
	trial as(
		SELECT *
		FROM foodie_fi.subscriptions
		WHERE plan_id = 0
),
annual as(
	SELECT *
	FROM foodie_fi.subscriptions
	WHERE plan_id = 3
),
bins AS (
	SELECT WIDTH_BUCKET(annual.start_date - trial.start_date,0,365,12) as width
	FROM trial
	JOIN annual USING(customer_id)
	
)

SELECT 
	CONCAT((width - 1 ) * 30 || ' - ', width * 30 || ' days') as width_class,
	COUNT(*)
FROM bins
GROUP BY width
order by width
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/370918ab-3436-428b-b96e-e858e3c540b4)

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?



















  
