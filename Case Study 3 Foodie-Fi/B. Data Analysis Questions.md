# B. Data Analysis Questions
1. How many customers has Foodie-Fi ever had?
```
SELECT 
	COUNT(DISTINCT customer_id) as customer
FROM foodie_fi.subscriptions
```

![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/dc7f3f89-d7fe-4f60-ab18-e734e2197968)
- Foodie_Fi has 1000 customers

2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
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
