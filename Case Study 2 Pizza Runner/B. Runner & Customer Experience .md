# Runner and Customer Experience 
1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```
select 
	date_trunc('week',registration_date),
	count(runner_id)
from pizza_runner.runners
GROUP BY 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/0b6236f2-7c0f-4626-b322-1126e928bd74)
- Only one runner joined in the first `2_weeks` and 2 more runners joined in the third wek

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```
SELECT
    runner_id,
    AVG(EXTRACT(MINUTE FROM (pickup_time - order_time))) AS runner_time_avg
FROM
    runner_orders_temp
JOIN
    pizza_runner.customer_orders USING (order_id)
GROUP BY
    runner_id
HAVING
    AVG(EXTRACT(MINUTE FROM (pickup_time - order_time)))  IS NOT NULL;

```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/b85a2ee6-6196-4afa-872d-370b81b6ec8d)
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```
SELECT
	pizza_count,
	avg(_time_diff)
FROM(
	
	SELECT
		order_id,
		count(order_id) as pizza_count,
		AVG(EXTRACT (MINUTE FROM(pickup_time - order_time))) as _time_diff

	FROM
		runner_orders_temp
	JOIN
		pizza_runner.customer_orders USING (order_id)
	WHERE distance > 1
	GROUP BY 1
	)
	GROUP BY 1
	ORDER BY 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/e7a23e64-dd59-4620-8a34-2975167b15f2)
- Preparing one pizza takes an average time of 12 mins
- Preparing two pizza takes an average time of 18 mins
- Preparing one pizza takes an average time of 29 mins
- Hence, we can deduce that when pizza are been produced in bulk, it saves time than preparing them one by one, the more pizza we produce at a time, the more time that is been saved.

4. What was the average distance travelled for each runner?
```
SELECT
	runner_id,
	AVG(distance) as avg_distance
FROM runner_orders_temp
GROUP BY 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/76d02a41-e3a0-4b71-9517-8b4f89a7301d)
- I changed the question from `What was the average distance travelled for each customer?` to `What was the average distance travelled for each runner?` as the customer got their pizza delivered to thier desired location
5. What was the difference between the longest and shortest delivery times for all orders?
```
SELECT
	MAX(DURATION) - MIN(DURATION)
FROM (
	SELECT 
		order_id,
		duration
	FROM pizza_runner.customer_orders
	JOIN runner_orders_temp USING(order_id)
	WHERE distance > 0
	order by 2
		)
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/2d661f05-43b0-4eba-b6f1-cf6f20e9103b)
- There is a 30 mins diffrence between the longest delivery and the shortest delivery

6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```
SELECT 
	runner_id,
	order_id,
	count(*),
	avg(distance) as distance,
	AVG(distance/duration *60) as avg_speed
FROM runner_orders_temp
JOIN pizza_runner.customer_orders USING (order_id)
WHERE distance >1
GROUP BY 1,2
order by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/224f7346-c61a-40ee-8c26-4697271c1846)
- Runner with `id-1` has an avg speed betwen 37.5 and 60 covering between 10km and 20km
- Runner with `id_2` has an avg speed between 35 and 93.6 covering between 23km and 25km
- Runner with `id_3` has an avg speed of 40 covering 10km
- The distance dosen't really determine the speed of the runner because we have an avg speed for ruuner 1 with an avg speed of 60 over a distance of 10km and runner 3 covered thesame 10km with an avg speed of 40.

7. What is the successful delivery percentage for each runner?
```
SELECT
	runner_id,
	100 * SUM(CASE	
			WHEN distance = 0 THEN 0
			ELSE 1
		END) / COUNT(*) AS success_perc
FROM runner_orders_temp
GROUP BY 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/2a322caf-c555-4bf9-ba69-a62c53e93260)
-  Runner 1 has 100% successful delivery.
-  Runner 2 has 75% successful delivery.
-  Runner 3 has 50% successful delivery
