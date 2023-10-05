# Case Study #2 - Pizza Runner
![](https://8weeksqlchallenge.com/images/case-study-designs/2.png)
# Introduction
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Available Data
Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

He has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

All datasets exist within the pizza_runner database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

## Data Cleaning
Base on the information provided from the data, it was given that there so many errors in the data, hence we need to carry out some data cleaning before analysing.
- The data is available in pizza runner database which I duplicated the data so I don't tamper with the original file.
- After identifying the null values in the columns `extras` and column `exclusions`, I decided to use  `UPDATE` function to update the data with the following query
`
SELECT * FROM pizza_runner.customer_orders
SET exclusions =''
where exclusions ilike 'Null'
OR extras ilike 'Null'
`
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/ae0d5928-1c8b-4942-9cdd-ada86a7e655e)

- Looking at the `pizza_runner.runner_orders` table
- We could see that we have null value in the `pickup_time` column
- We also have null value in the `distance` column with some other errors like ending each row with **Km** of which we need to get rid of them
- Finally, we need to clean all the Null values in the cancellation column as well
Before cleaning, the table looks like this
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/369f911e-177e-4221-91ba-00a4c182cb2c)

- To clean the table, I did the following
```
CREATE TEMP TABLE runner_orders_temp AS
SELECT
	order_id,
	runner_id,
	CASE 
		WHEN pickup_time ilike 'Null' THEN ' '
		ELSE pickup_time
	END AS pickup_time,
	CASE
		WHEN distance ilike 'Null' THEN ' '
		WHEN distance ilike '%Km' THEN TRIM('Km' FROM distance)
		ELSE distance
	END AS distance,
	CASE
		WHEN duration ilike 'Null' THEN ' ' 
		WHEN duration ilike '%mins' THEN TRIM('mins' FROM duration)
		WHEN duration ilike '%minute' THEN TRIM('minute' FROM duration)
		WHEN duration ilike '%minutes' THEN TRIM('minutes' FROM duration)
		ELSE duration
	END AS duration,
	CASE 
		WHEN cancellation is NULL THEN ' '
		WHEN cancellation ilike 'Null' THEN ' '
		ELSE cancellation
	END AS cancellation
FROM pizza_runner.runner_orders
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/61bd01bf-9946-4fa7-aad2-87de285805e4)
- Just as confirmation, I want to confirm the data types by reformting each column to the normal data_type
```
UPDATE runner_orders_temp
SET distance = NULL WHERE distance = ' '

UPDATE runner_orders_temp
SET pickup_time = NULL WHERE pickup_time = ' '

UPDATE runner_orders_temp
SET duration = NULL WHERE duration = ' ';

ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time TYPE timestamp without time zone USING pickup_time::timestamp without time zone,
ALTER COLUMN distance TYPE double precision USING distance::double precision,
ALTER COLUMN duration TYPE integer USING duration::integer;

```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/f39b00dc-eb10-4202-a3a2-5ac025db495c)







## Solution to the Pizza Runner Case study, [question here](https://8weeksqlchallenge.com/case-study-2/)
### Section A. Pizza Metrics
1. How many pizzas were ordered?
```
select COUNT(order_id) as pizza_count
from pizza_runner.customer_orders
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/d67a4134-16b0-41ec-9a8b-9009679f205d)
- So far, 14 pizza has been ordered from pizza runner's company

2. How many unique customer orders were made?
```
select count(DISTINCT order_id) as order
from pizza_runner.customer_orders
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/0d642290-e157-420a-8ed4-7de05b30d264)


- 10 customers has ordered pizza from pizza runner before

3.How many successful orders were delivered by each runner?
```
Select runner_id,
count(*) as successful_order
FROM runner_orders_temp
where distance >= 1
group by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/e29f8526-6655-4fa3-9415-592d1622719a)
- Runner 1 has 4 successful delivered orders making him the best runner.
- Runner 2 has 3 successful delivered orders.
- Runner 3 has 1 successful delivered order.

4. How many of each type of pizza was delivered?
```
SELECT customer_orders.pizza_id,
	pizza_names.pizza_name,
	count(*) as pizza_cnt
FROM pizza_runner.customer_orders
JOIN runner_orders_temp USING (order_id)
JOIN pizza_runner.pizza_names on pizza_names.pizza_id = customer_orders.pizza_id
where distance >= 1
group by 1,2
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/566395f1-ea44-4305-971c-64701f974316)
- Vegetarian has been deliver 3 times with id `2`
- Meatlovers has been delivered 9 times with id `1`


5. How many Vegetarian and Meatlovers were ordered by each customer?
```
SELECT customer_orders.customer_id,
	pizza_names.pizza_name,
	count(*) as pizza_cnt
FROM pizza_runner.customer_orders
JOIN runner_orders_temp USING (order_id)
JOIN pizza_runner.pizza_names on pizza_names.pizza_id = customer_orders.pizza_id
--where distance >= 1
group by 1,2
order by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/2edcca30-6f9e-49be-92eb-edc78960aa79)

- Customer with id `101` ordered `2 Meatlovers`
- Customer with id `101` ordered `1 Vegetarians`
- Customer with id `102` ordered `2 Meatlovers`
- Customer with id `102` ordered `1 Vegetarians`
- Customer with id `103` ordered `3 Meatlovers`
- Customer with id `103` ordered `1 Vegetarians`
- Customer with id `104` ordered `3 Meatlovers`
- Customer with id `105` ordered `1 Vegetarians`


6. What was the maximum number of pizzas delivered in a single order?
```
SELECT
	order_id,
	count(pizza_id) as pizza_count
FROM runner_orders_temp
JOIN pizza_runner.customer_orders USING(order_id)
WHERE distance > 0 
GROUP BY 1
order by 2 DESC
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/1fad4ffe-a210-4128-8e03-6a76b50b10a9)
- order_id 4 has the highest number of pizza delivered

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```
SELECT
	customer_id,
	CASE
		WHEN exclusions <> '' OR extras <> '' THEN 'pizza_order_change'
		ELSE 'no_change'
	END as pizza_cahnge_type,
	COUNT(CASE
			WHEN exclusions <> '' OR extras <> '' THEN 'pizza_order_change'
			ELSE 'no_change'
		END) as pizza_cahnge_count
FROM pizza_runner.customer_orders
GROUP BY 1 , 2
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/39608367-1adb-425b-b01d-b7fac8eecf3f)
- Pizza id `101` has 3 orders and all the orders has no `extras` or `exclusions`
- Pizza id `102` also has 3 orders and all the orders has no `extras` or `exclusions`
- Pizza id `103` has 4 orders and all the orders has either an `extras` or an `exclusions`
- Pizza id `104` has 3 orders and 2 of the orders has either a change in their `extras` or `exclusions` and 1 order has no `extras` or `exclusions`

8. How many pizzas were delivered that had both exclusions and extras?
```
select
	count(*)
	FROM(
		SELECT
			pizza_id,
			CASE
				WHEN exclusions <> '' and extras <> '' THEN 'pizza_order_change'
				ELSE 'no_change'
			END as pizza_change_type
		FROM pizza_runner.customer_orders
		JOIN runner_orders_temp USING (order_id)
		where distance > 0
		)
	WHERE pizza_change_type = 'pizza_order_change'
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/5855b384-3ba6-4fc7-8114-ba57efb26c72)
- Only 1 order was delivered successfully and has both exclusions and extras

9. What was the total volume of pizzas ordered for each hour of the day?


