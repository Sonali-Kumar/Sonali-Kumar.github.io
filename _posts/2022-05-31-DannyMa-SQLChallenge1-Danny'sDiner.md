---
layout: post
title: "Case Study #1 - Danny's Diner"
image: "/posts/Danny_Diner.png"
tags: [SQL, DannyMa, Danny's Diner]
---


## Learning Objective 

The following topics relevant to the Danny's Diner case study are covered lots of depth in the Serious SQL course:

* Common Table Expressions
* Group By Aggregates
* Window Functions for ranking
* Case When
* Table Joins

## Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny's Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they've spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

* `sales`
* `menu`
* `members`

You can inspect the entity relationship diagram and example data below.

## Entity Relationship Diagram

[https://dbdiagram.io/embed/608d07e4b29a09603d12edbd]


## Example Datasets

All datasets exist within the **`dannys_diner`** database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

### Table 1: sales

The `sales` table captures all `customer_id` level purchases with an corresponding `order_date` and `product_id` information for when and what menu items were ordered.

<div class="responsive-table" markdown="block">

| customer_id | order_date | product_id |
| ----------- | ---------- | ---------- |
| A           | 2021-01-01 |          1 |
| A           | 2021-01-01 |          2 |
| A           | 2021-01-07 |          2 |
| A           | 2021-01-10 |          3 |
| A           | 2021-01-11 |          3 |
| A           | 2021-01-11 |          3 |
| B           | 2021-01-01 |          2 |
| B           | 2021-01-02 |          2 |
| B           | 2021-01-04 |          1 |
| B           | 2021-01-11 |          1 |
| B           | 2021-01-16 |          3 |
| B           | 2021-02-01 |          3 |
| C           | 2021-01-01 |          3 |
| C           | 2021-01-01 |          3 |
| C           | 2021-01-07 |          3 |

</div>

### Table 2: menu

The `menu` table maps the `product_id` to the actual `product_name` and `price` of each menu item.

<div class="responsive-table" markdown="block">

| product_id | product_name | price |
| ---------- | ------------ | ----- |
|          1 | sushi        |    10 |
|          2 | curry        |    15 |
|          3 | ramen        |    12 |

</div>

### Table 3: members

The final `members` table captures the `join_date` when a `customer_id` joined the beta version of the Danny's Diner loyalty program.

<div class="responsive-table" markdown="block">

| customer_id | join_date  |
| ----------- | ---------- |
| A           | 2021-01-07 |
| B           | 2021-01-09 |

</div>

## Interactive SQL Session

You can use the embedded DB Fiddle below to easily access these example datasets - this interactive session has everything you need to start solving these questions using SQL.

You can click on the `Edit on DB Fiddle` link on the top right hand corner of the embedded session below and it will take you to a fully functional SQL editor where you can write your own queries to analyse the data.

You can feel free to choose any SQL dialect you'd like to use, the existing Fiddle is using PostgreSQL 13 as default.

Serious SQL students have access to a dedicated SQL script in the 8 Week SQL Challenge section of the course which they can use to generate relevant temporary tables like we've done throughout the entire course!

<div class="sqlfiddle-container">
  <iframe src="https://embed.db-fiddle.com/912b55b7-0c69-4f19-906f-aaef8ece6088" frameborder="0" allowfullscreen="" title="Embedded fiddle"></iframe>
</div>

## Case Study Questions & My Solutions

Each of the following case study questions can be answered using a single SQL statement:

* What is the total amount each customer spent at the restaurant?

```ruby

SELECT S.CUSTOMER_ID, SUM(M.PRICE) AS MONEY_SPENT
   from DANNYS_DINER.SALES AS S 
   LEFT JOIN DANNYS_DINER.MENU AS M 
   USING (PRODUCT_ID)

GROUP BY 1
ORDER BY 1

```

![alt text](/img/posts/DannyDiner_1.PNG "Purchase Amount by Customer!")

---

* How many days has each customer visited the restaurant?

```ruby
SELECT customer_id, COUNT(DISTINCT(order_date)) AS NUMBER_OF_VISITS
	FROM DANNYS_DINER.SALES

GROUP BY 1
ORDER BY 1
```

![alt text](/img/posts/dannysdiner/DannyDiner_2.PNG "Customer Visits")

---
* What was the first item from the menu purchased by each customer?

```ruby
with first_item_purchase as (
  
    SELECT S.customer_id, M.product_name, S.order_date,
    DENSE_RANK () OVER (PARTITION BY S.CUSTOMER_ID ORDER BY S.ORDER_DATE) as rank_by_orderdate
    	FROM DANNYS_DINER.SALES AS S  
        LEFT JOIN DANNYS_DINER.MENU AS M 
        USING (product_id)
)
Select customer_id, product_name as first_order
	from first_item_purchase	
	where rank_by_orderdate = 1

```

![alt text](/img/posts/dannysdiner/DannyDiner_3.PNG "First Purchase by Customer")

* What is the most purchased item on the menu and how many times was it purchased by all customers?

```ruby
SELECT M.product_name as most_purchased_item, COUNT(S.product_id) AS count_of_orders
	FROM DANNYS_DINER.SALES AS S 
    LEFT JOIN DANNYS_DINER.MENU AS M 
    USING (product_id)
   
GROUP by 1
order by COUNT(S.product_id) DESC
limit 1
```

![alt text](/img/posts/dannysdiner/DannyDiner_4.PNG "Most Purchased Item and Order Count")

* 5. Which item was the most popular for each customer?


```ruby
WITH MOST_POPULAR_ITEM AS (
  
select S.customer_id, M.PRODUCT_NAME, COUNT(S.product_id) AS PURCHASE_COUNT,
	RANK () OVER (PARTITION BY S.customer_id ORDER BY count(S.product_id) DESC) 
	from DANNYS_DINER.SALES AS S 
  	LEFT JOIN DANNYS_DINER.MENU AS M 
  	USING (product_id)
    
GROUP BY 1, 2
)

SELECT customer_id, product_name, purchase_count
	FROM MOST_POPULAR_ITEM
   	WHERE RANK = 1
```

![alt text](/img/posts/dannysdiner/DannyDiner_5.PNG "Most Popular Item by Customer")

* 6. Which item was purchased first by the customer after they became a member?


```ruby
WITH FIRST_MEMBER_PURCHASE AS (
  
SELECT S.CUSTOMER_ID, S.ORDER_DATE, M.PRODUCT_NAME,
DENSE_RANK () OVER (PARTITION BY S.customer_id ORDER BY S.ORDER_DATE) AS RANK_BY_ORDER_DATE
	FROM DANNYS_DINER.SALES AS S 
    LEFT JOIN DANNYS_DINER.MEMBERS AS MEM 
   	USING (customer_id)
        LEFT JOIN DANNYS_DINER.MENU AS M 
        USING (product_id)
    WHERE S.ORDER_DATE >= MEM.JOIN_DATE
)

SELECT customer_id, product_name
	FROM FIRST_MEMBER_PURCHASE
	WHERE RANK_BY_ORDER_DATE = 1
```

![alt text](/img/posts/dannysdiner/DannyDiner_6.PNG "Member's First Purchase")

* 7. Which item was purchased just before the customer became a member?

```ruby
WITH LAST_NON_MEMBER_PURCHASE AS (
  
SELECT S.CUSTOMER_ID, S.ORDER_DATE, M.PRODUCT_NAME,
DENSE_RANK () OVER (PARTITION BY S.customer_id ORDER BY S.ORDER_DATE DESC) AS RANK_BY_ORDER_DATE
	FROM DANNYS_DINER.SALES AS S 
    LEFT JOIN DANNYS_DINER.MEMBERS AS MEM 
   	USING (customer_id)
        LEFT JOIN DANNYS_DINER.MENU AS M 
        USING (product_id)
    WHERE S.ORDER_DATE < MEM.JOIN_DATE
)

SELECT customer_id, product_name
	FROM LAST_NON_MEMBER_PURCHASE
	WHERE RANK_BY_ORDER_DATE = 1
```

![alt text](/img/posts/dannysdiner/DannyDiner_7.PNG "Last Non-Member Purchase")

* 8. What is the total items and amount spent for each member before they became a member?

```ruby
SELECT S.customer_id, COUNT(S.PRODUCT_ID) AS TOTAL_ITEMS, SUM(M.PRICE) AS AMOUNT_SPENT
	FROM DANNYS_DINER.SALES AS S
    LEFT JOIN DANNYS_DINER.MENU AS M 
    USING (product_id)
    	LEFT JOIN DANNYS_DINER.MEMBERS AS MEM 
        USING (customer_id)
WHERE S.ORDER_DATE < MEM.JOIN_DATE
GROUP BY S.CUSTOMER_ID
```

![alt text](img/posts/dannysdiner/DannyDiner_8.PNG "Customer's purchase details before becoming memebers")

* 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```ruby
SELECT S.customer_id,
sum(CASE WHEN M.PRODUCT_NAME = 'sushi' then m.price*20 
	else m.price*10
    end) as points
      FROM DANNYS_DINER.SALES AS S 
      LEFT JOIN DANNYS_DINER.MENU AS M 
      USING (product_id)
    
group by 1
order by 1
```

![alt text](/img/posts/dannysdiner/DannyDiner_9.PNG "Points Earned")

* 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```ruby
SELECT S.CUSTOMER_ID, 
SUM(CASE WHEN S.ORDER_DATE BETWEEN MEM.JOIN_DATE AND MEM.JOIN_DATE+6 THEN M.PRICE*20
    when m.product_name = 'sushi' then m.price*20
	ELSE M.PRICE*10
    END) AS total_points
      FROM DANNYS_DINER.members AS MEM 
      LEFT JOIN DANNYS_DINER.SALES AS S 
      USING (customer_id)
      LEFT JOIN DANNYS_DINER.MENU AS M
      USING (product_id)
GROUP BY 1
ORDER BY 1
```

![alt text](/img/posts/dannysdiner/DannyDiner_10.PNG "Points Earned by Members")


----

### Final Thoughts

This is my first SQL Case Study, absolutely loved it! Thank you Danny for creating these case studies to help SQL-learners *(like myself)* apply their theoratical knowledge into pratice.

---

## For those who would like to join the challenge, here is the link: [https://8weeksqlchallenge.com/]
