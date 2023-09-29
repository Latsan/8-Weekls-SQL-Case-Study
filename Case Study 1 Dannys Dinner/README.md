# Case Study #1: Danny's Dinner
![Danny's diner](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/adfb5fe7-4d4f-4b84-8676-c6137d1100c7)

## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
### Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

### Solution to Danny's Dinner problem
1. What is the total amount each customer spent at the restaurant?
 ```
SELECT
     sales.customer_id,
     SUM(menu.price) AS total_sales
   FROM dannys_diner.sales
     INNER JOIN dannys_diner.menu USING(product_id)
   GROUP BY sales.customer_id;
```
![question1_1](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/d262ccaf-85b0-4611-8add-845c7e0d5c0d)

2. How many days has each customer visited the restaurant?
```
SELECT 
  customer_id, 
  COUNT(DISTINCT order_date) AS usr_cnt
FROM dannys_diner.sales
GROUP BY customer_id;

```

![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/1350309e-e363-4fb2-a178-550b81c8d5ec)

3. What was the first item from the menu purchased by each customer?

```
SELECT
	customer_id,
	product_name
FROM (
	  SELECT 
		sales.customer_id, 
		sales.order_date, 
		menu.product_name,
		DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS _rank
	  FROM dannys_diner.sales
	  INNER JOIN dannys_diner.menu USING (product_id)
)

WHERE _rank = 1
GROUP BY 1, 2;
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/b6fb07b7-2800-4b41-89b8-b85d3d16a9d4)

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```
select customer_id,
	product_name,
	_count
from (
	SELECT 
		customer_id,
		product_name,
		count(order_date) as _count,
		dense_rank() over(partition by customer_id order by count(customer_id) DESC) as _rank
	 FROM dannys_diner.sales
	JOIN dannys_diner.menu USING(product_id)
	--where product_name = 'ramen'
	group by 1,2
	)
	WHERE _rank = 1 
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/0e5fa6a3-f82e-4560-b8fa-649a147359a2)

5. Which item was the most popular for each customer?
```
WITH most_popular AS (
  SELECT 
    sales.customer_id, 
    menu.product_name, 
    COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT 
  customer_id, 
  product_name, 
  order_count
FROM most_popular 
WHERE rank = 1;
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/c887a6a9-b6bb-4265-9eab-de748b16b663)


6. Which item was purchased first by the customer after they became a member?
```
SELECT
	customer_id,
	product_name
	
FROM (
	SELECT members.customer_id,
		sales.product_id,
		ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date) as _rank
	FROM dannys_diner.members 
	INNER JOIN dannys_diner.sales 
		ON sales.customer_id = members.customer_id
		and  join_date < order_date 
)
JOIN dannys_diner.menu USING(product_id)
WHERE _rank = 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/50daef29-19ff-4139-9ae7-da12bcfe37c5)

7. Which item was purchased just before the customer became a member?
```
SELECT
	customer_id,
	product_name
	
FROM (
	SELECT members.customer_id,
		sales.product_id,
		ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date DESC) as _rank
	FROM dannys_diner.members 
	INNER JOIN dannys_diner.sales 
		ON sales.customer_id = members.customer_id
		and  join_date > order_date 
)
JOIN dannys_diner.menu USING(product_id)
WHERE _rank = 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/3cdd7913-e6b9-44b8-9cba-a813b06ea3ec)

8. What is the total items and amount spent for each member before they became a member?
```
SELECT
	customer_id,
	COUNT(product_name) as amt_product,
	SUM(price) as amt_spent
	
FROM (
	SELECT members.customer_id,
		sales.product_id,
		ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date DESC) as _rank
	FROM dannys_diner.members 
	INNER JOIN dannys_diner.sales 
		ON sales.customer_id = members.customer_id
		and  join_date > order_date 
)
JOIN dannys_diner.menu USING(product_id)
--WHERE _rank = 1
GROUP BY 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/77db6159-6115-4805-b57b-3a0406381a1c)

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
For question number 9, I will find calculate the point danny's customer had before becoming a member, after becoming a member and the whole customers points
- Before becoming a member
```
WITH 
	temp_1 as 
		(
		SELECT
			customer_id,
			product_name,
			price,
			CASE
				WHEN product_name <> 'sushi' THEN price*10
				else price * 20
			end as points

		FROM (
			SELECT members.customer_id,
				sales.product_id,
				ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date DESC) as _rank
			FROM dannys_diner.members 
			INNER JOIN dannys_diner.sales 
				ON sales.customer_id = members.customer_id
				and  join_date > order_date 
		)
		JOIN dannys_diner.menu USING(product_id)
)
select customer_id,sum(points) as points
FROM temp_1
Group by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/597dd086-2d43-462a-bb4a-816e84359afb)

- After becoming a member
```
WITH 
	temp_1 as 
		(
		SELECT
			customer_id,
			product_name,
			price,
			CASE
				WHEN product_name <> 'sushi' THEN price*10
				else price * 20
			end as points

		FROM (
			SELECT members.customer_id,
				sales.product_id,
				ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date DESC) as _rank
			FROM dannys_diner.members 
			INNER JOIN dannys_diner.sales 
				ON sales.customer_id = members.customer_id
				and  join_date < order_date 
		)
		JOIN dannys_diner.menu USING(product_id)
)
select customer_id,sum(points) as points
FROM temp_1
Group by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/c00f4eaf-b060-4758-9117-1a8b58ec8598)

- Total Customer's points
```
select
	customer_id,
	sum(points) as points
FROM (
	SELECT 
		customer_id,
		CASE
					WHEN product_name <> 'sushi' THEN price*10
					else price * 20
				end as points
	FROM dannys_diner.menu
	JOIN dannys_diner.sales USING (product_id)
	)
GROUP BY 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/f930454b-ca9d-4d35-817f-073bc89bb652)

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?





