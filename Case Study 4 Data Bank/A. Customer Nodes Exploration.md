# Case Study #4 - Data Bank

![4](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/be2583b6-0803-454e-a6ad-1c08925d7454)
## Introduction

There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

## Available Data

The Data Bank team have prepared a data model for this case study as well as a few example rows from the complete dataset below to get you familiar with their tables.

![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/29040c8a-de4c-47b7-8463-5916640bdf79)

## Table 1: Regions

Just like popular cryptocurrency platforms - Data Bank is also run off a network of nodes where both money and data is stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world.

This regions table contains the region_id and their respective region_name values
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/b0b169ca-d90b-489f-b2d8-da937ea4b1e4)

## Table 2: Customer Nodes
Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.

This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

Below is a sample of the top 10 rows of the data_bank.customer_nodes
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/ede432c2-15c5-400c-9b9f-38afdf8db7a5)

## Table 3: Customer Transactions
This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.
![image](https://github.com/Latsan/8-Weekls-SQL-Case-Study/assets/78388641/42a5e020-a7cf-423e-8fe4-86981d060b27)








# Section A. Customer Nodes Exploration

### 1.How many unique nodes are there on the Data Bank system?











### 2. What is the number of nodes per region?






### 3. How many customers are allocated to each region?






### 4. How many days on average are customers reallocated to a different node?





### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
