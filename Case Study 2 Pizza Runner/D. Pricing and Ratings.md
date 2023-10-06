# D. Pricing and Ratings
1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```
SELECT sum(pizza_runner_net_sales) as networth
FROM
(
SELECT
    pizza_id,
    
        CASE
            WHEN pizza_id = 1 THEN COUNT(pizza_id) * 12
            ELSE COUNT(*) * 10
        END
     AS pizza_runner_net_sales
FROM
    pizza_runner.customer_orders
	GROUP BY 1
)
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/ab917cd6-f924-4bd3-bf33-e69d11b5053b)
Pizza runner has made $160 sofar

2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra
