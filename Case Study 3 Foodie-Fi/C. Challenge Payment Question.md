# C. Challenge Payment Question
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- Monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- Upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- Upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- Once a customer churns they will no longer make payments

Example outputs for this table might look like the following:
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/fda13b4e-713e-43b1-8ad6-777be55ba634)
---
Honestly i haven't figured out the solution to this 

```
SELECT *,
	COALESCE(LEAD("subscriptions".start_date) OVER (PARTITION BY customer_id ORDER BY "subscriptions".start_date), '2020-12-31') AS next_date
FROM foodie_fi."subscriptions"
JOIN foodie_fi."plans" USING (plan_id)
WHERE DATE_PART('year',start_date) = 2020 AND plan_id NOT IN (0,4)
ORDER BY 2
```
