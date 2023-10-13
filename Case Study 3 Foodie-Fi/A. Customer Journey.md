# 8 Week SQL Challenge: Case Study #3 Foodie-Fi Solution
<img width="621" alt="129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0" src="https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/56d30be8-8153-4c69-90f5-b9fda0b8e016">

## Introduction
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.
### Available Data
Danny has shared the data design for Foodie-Fi and also short descriptions on each of the database tables - our case study focuses on only 2 tables but there will be a challenge to create a new table for the Foodie-Fi team.

All datasets exist within the foodie_fi database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/8b0f0b0b-2fc9-404a-8bc9-b29c20f0277f)
### Table 1: plans

Customers can choose which plans to join Foodie-Fi when they first sign up.

Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90

Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.

Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.

When customers cancel their Foodie-Fi service - they will have a churn plan record with a null price but their plan will continue until the end of the billing period.

### Table 2: subscriptions

Customer subscriptions show the exact date where their specific plan_id starts.

If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the start_date in the subscriptions table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.

When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.

### Interactive SQL Instance
You can use the embedded DB Fiddle below to easily access these example datasets - this interactive session has everything you need to start solving these questions using SQL.

You can click on the Edit on DB Fiddle link on the top right hand corner of the embedded session below and it will take you to a fully functional SQL editor where you can write your own queries to analyse the data.

You can feel free to choose any SQL dialect you’d like to use, the existing Fiddle is using PostgreSQL 13 as default.

Serious SQL students will have access to the same relevant schema SQL and example solutions which they can use with their Docker setup from within the course player!

## Case Study Questions
This case study is split into an initial data understanding question before diving straight into data analysis questions before finishing with 1 single extension challenge.

## A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

---
- This Case study has 2 tables located in the `foodie_fi` schema
- I joined the two tables with the primary key together to have a view of what's going both tables
- I counted to total numbers of customers that has patronised `foodie_fi`
```
SELECT COUNT(DISTINCT customer_id) 
FROM foodie_fi.subscriptions
JOIN foodie_fi.plans USING(plan_id)
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/62202455-de83-40d9-b38c-7f571c542836)
- The dataset has 1000 customers that has subscriped to `foodie_fi` platform whether they used the `trial` plan , `basic monthly` plan, `pro monthly` plan, `pro annual` plan or they have `churn` their subscrptions.

```
SELECT *
FROM (
	SELECT *,
		row_number(*) OVER (PARTITION BY customer_id ORDER BY start_date) AS _rank
	FROM foodie_fi.subscriptions
	JOIN foodie_fi.plans USING(plan_id)
	)
WHERE _rank = 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/fc4a2b01-53f8-40e2-84bc-575b28853555)
- From the dataset, we could see that every customers started with a free trail i.e there was no customer that paid for a premium plan 

```
SELECT plan_name,
	count(*)
FROM (
	SELECT *,
		row_number(*) OVER (PARTITION BY customer_id ORDER BY start_date) AS _rank
	FROM foodie_fi.subscriptions
	JOIN foodie_fi.plans USING(plan_id)
	)
WHERE _rank = 2
Group by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/43e3575c-5df6-4e83-9915-ec1aaf444212)
I disoveed the following checking for the customer next activities after using thier free trial and found out that 

- 546 customers subscribed for the `Basic monthly` plan
- 325 customers subscribed for the `Pro monthly` plan
- 37 customers subscribed for the `Pro annual` plan
- 92 customers churn their subscription

Hence we can say that `90.8%` customers subscribed to a plan after the free 7 days trial which is a good starting

---
Analysing we further, 37 customers subscribed for the `Pro annual` plan so we don't need to worry much about them again, we only need to ensure that they are getting the quality content of what they paid.
- Hence we want to focus on the remainng **963** customers

```
SELECT plan_name,
	count(*)
FROM (
	SELECT *,
		row_number(*) OVER (PARTITION BY customer_id ORDER BY start_date) AS _rank
	FROM foodie_fi.subscriptions
	JOIN foodie_fi.plans USING(plan_id)
	)
WHERE _rank = 3
Group by 1
```
- 214 customers subscribed for the `Pro monthly` plan
- 186 customers subscribed for the `Pro annual` plan
- 170 customers churn their subscription

From this stats, we could say that people don't really enjoy the basic plan as they have limited access and can only stream videos, hence no customer subscribed for the plan this time around.
- 400 customers subscribed out of 963
- 186 pro annual plan making 223 annual pro plan. Foodie_fi is improving
- After the third check only 437 customers has an ongoing subscription plan which means 563 customers has checked out.


---
Let's go for a fianl check
```
SELECT plan_name,
	count(*)
FROM (
	SELECT *,
		row_number(*) OVER (PARTITION BY customer_id ORDER BY start_date) AS _rank
	FROM foodie_fi.subscriptions
	JOIN foodie_fi.plans USING(plan_id)
	)
WHERE _rank = 4
Group by 1
```
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/bfab215d-ca21-4d59-b56f-32ec6067840d)

- 45 customers joined the annual plan and sofar we have 268 customers has an annual plan


## B. Data Analysis Questions
























