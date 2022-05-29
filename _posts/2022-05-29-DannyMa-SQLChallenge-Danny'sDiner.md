---
layout: post
title: Danny's Diner Week 1 SQL Challenge
image: "/posts/Danny_Diner.png"
tags: [SQL, DannyMa, Danny's Diner]
---

---
What is the total amount each customer spent at the restaurant?

```ruby
SELECT s.customer_id, sum(m.price) as amount_spent
	FROM dannys_diner.sales as s 
	LEFT JOIN dannys_diner.menu as M 
	on s.product_id = m.product_id
	GROUP BY 1
	ORDER BY 1 

```

---
How many days has each customer visited the restaurant?

```ruby
SELECT  customer_id, count(DISTINCT(order_date)) As number_of_days
	FROM dannys_diner.sales
	GROUP BY 1

```


---
What was the first item from the menu purchased by each customer?

```ruby
with CTE AS 
(
      SELECT S.order_date, S.customer_id, M.product_name,
      RANK ()  OVER (ORDER BY S.order_date) 
      FROM DANNYS_DINER.SALES AS S
     JOIN DANNYS_DINER.MENU AS M
     USING (product_id)
     )
SELECT customer_id, product_name
FROM CTE
WHERE RANK = 1 
order BY customer_id

```

---
What is the most purchased item on the menu and how many times was it purchased by all customers?

```ruby

SELECT COUNT(s.product_id), M.product_name
FROM DANNYS_DINER.SALES as s
Join dannys_diner.menu as m 
using (product_id)
GROUP BY 2
ORDER BY COUNT DESC
LIMIT 1

```

--- 
Which item was the most popular for each customer?

```ruby

WITH CTE AS (
  SELECT s.customer_id, m.product_name,
RANK () OVER( PARTITION BY s.customer_id ORDER BY COUNT(m.product_name) desc)
FROM DANNYS_DINER.SALES AS S
  LEFT JOIN DANNYS_DINER.MENU AS m
  ON S.product_id = m.product_id
GROUP BY s.customer_id, m.product_name, s.product_id)

SELECT customer_id, product_name
from CTE
WHERE RANK = 1

```

--- 
Which item was purchased first by the customer after they became a member?

```ruby

with CTE AS (
  SELECT s.customer_id, menu.product_name, s.order_date, m.join_date,
DENSE_RANK () over (partition by s.customer_id order by s.order_date) as ordered_first
FROM dannys_diner.members as m 
LEFT JOIN dannys_diner.sales as s 
on m.customer_id = s.customer_id
  JOIN DANNYS_DINER.MENU AS menu
  USING (product_id)
WHERE s.order_date >= m.join_date )

SELECT customer_id, product_name
FROM CTE
WHERE ordered_first = 1

```

--- 
Which item was purchased just before the customer became a member?

```ruby

WITH CTE AS (
  SELECT S.customer_id, menu.product_name, m.join_date, s.order_date,
	DENSE_RANK () over (partition by s.customer_id order by s.order_date desc) as ordered_before_membership
FROM dannys_diner.members as m
  Left Join dannys_diner.sales as s 
  on m.customer_id = s.customer_id
	JOIN dannys_diner.menu as menu
	using (product_id)
where s.order_date < m.join_date
)
SELECT customer_id, product_name
FROM CTE
WHERE ordered_before_membership = 1 

```

--- 

What is the total items and amount spent for each member before they became a member?

```ruby

CREATE VIEW  ITEMS_AMOUNT_SPENT AS
(SELECT S.CUSTOMER_ID, COUNT(S.CUSTOMER_ID) AS TOTAL_ITEMS, SUM(MENU.PRICE), S.product_id

FROM DANNYS_DINER.MEMBERS AS M 
LEFT JOIN DANNYS_DINER.SALES AS S
ON M.CUSTOMER_ID = S.CUSTOMER_ID
 JOIN DANNYS_DINER.MENU AS MENU
 USING (product_id)

WHERE S.ORDER_DATE < M.JOIN_DATE
GROUP BY 1, MENU.PRICE, S.product_id
);

SELECT customer_id, SUM(SUM) AS AMOUNT_SPENT, SUM(TOTAL_ITEMS) AS TOTAL_ITEMS
FROM items_amount_spent
GROUP BY 1 
ORDER BY customer_id

-- OR USING CTE

with CTE AS 
(SELECT S.CUSTOMER_ID, COUNT(S.CUSTOMER_ID) AS TOTAL_ITEMS, SUM(MENU.PRICE), S.product_id

FROM DANNYS_DINER.MEMBERS AS M 
LEFT JOIN DANNYS_DINER.SALES AS S
ON M.CUSTOMER_ID = S.CUSTOMER_ID
 JOIN DANNYS_DINER.MENU AS MENU
 USING (product_id)

WHERE S.ORDER_DATE < M.JOIN_DATE
GROUP BY 1, MENU.PRICE, S.product_id
)

SELECT customer_id, SUM(SUM) AS AMOUNT_SPENT, SUM(TOTAL_ITEMS) AS TOTAL_ITEMS
FROM CTE
GROUP BY 1
ORDER BY customer_id

```

--- 
If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```ruby

SELECT S.customer_id, sum(
case 
	when  m.product_name = 'sushi' then m.price*20 
    else m.price*10
    end ) as points
    FROM dannys_diner.sales AS S
    JOIN dannys_diner.menu AS M 
    USING (product_id)

group by 1
order by s.customer_id

```
---

In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```ruby

with CTE AS 
(select s.customer_id, s.order_date, menu.product_name,
case 
	when s.order_date between m.join_date and m.join_date+7 
      	then 20*menu.price
    	else menu.price*10
      	end as points

from dannys_diner.sales as s 
join dannys_diner.menu as menu
using (product_id)
join dannys_diner.members as m
using (customer_id)
where s.order_date <= '2021-01-31'
)

SELECT customer_id, SUM(
CASE 
	WHEN product_name = 'sushi' 
    then points*2 
    else points*1
    END) AS TOTAL_POINTS
FROM CTE
GROUP BY 1

```

![alt text](/img/posts/Danny_Diner.png "Danny's Diner")
---

