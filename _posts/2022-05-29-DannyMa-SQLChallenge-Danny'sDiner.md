---
layout: post
title: The Column Subscriber Analysis
image: "/posts/thecolumn_share.png"
tags: [SQL, DannyMa, Danny's Diner]
---

---
1.What is the total amount each customer spent at the restaurant?

```ruby
SELECT s.customer_id, sum(m.price) as amount_spent
	FROM dannys_diner.sales as s 
	LEFT JOIN dannys_diner.menu as M 
	on s.product_id = m.product_id
	GROUP BY 1
	ORDER BY 1 

```

---
2.How many days has each customer visited the restaurant?

```ruby
SELECT  customer_id, count(DISTINCT(order_date)) As number_of_days
	FROM dannys_diner.sales
	GROUP BY 1

```

```ruby

--3.What was the first item from the menu purchased by each customer?
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

--4.What is the most purchased item on the menu and how many times was it purchased by all customers?
SELECT COUNT(s.product_id), M.product_name
FROM DANNYS_DINER.SALES as s
Join dannys_diner.menu as m 
using (product_id)
GROUP BY 2
ORDER BY COUNT DESC
LIMIT 1

-- 5. Which item was the most popular for each customer?
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
-- why can i not use product_id in the window function? 


-- 6. Which item was purchased first by the customer after they became a member?
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

-- 7.Which item was purchased just before the customer became a member?
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

-- 8.What is the total items and amount spent for each member before they became a member?

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

-- 9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

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

-- 10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, 
-- not just sushi - how many points do customer A and B have at the end of January?

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

*/


```


```ruby
ax = sns.stripplot(x='Hour', y='Opens', data=summary)
```


---
I notice that most of the time the newsletters are opened between 7-9, but the data corresponds to the time the newsletters are actually sent. I visualize a scatterplot of sends (subscriber count) and open and notice a linear relationship. This prompted me to perform a linear regression and visualize a basic model.

```ruby
y = summary['Opens']
x = summary['Sends']
np.polyfit(x,y, deg =1)

potential_Send = np.linspace(0,5000,100)
potential_Opens = 0.44547871*potential_Send + 183.80574801

sns.scatterplot(x = 'Sends', y = 'Opens', data = summary)
plt.plot(potential_Send,potential_Opens, color = 'red')
plt.show()
```

![alt text](/img/posts/Danny's Diner.png "Danny's Diner")
---
After performing more analysis, I did research and found that Mailchip suggests that 10 AM is the most optimal time to send out newsletters/emails to subscribers. I used that as  my recommendation to The Column.

![alt text](/img/posts/Opens_Analysis.jpg "Opens Analysis")

![alt text](/img/posts/Clicks_Analysis.jpg "Clicks Analysis")

![alt text](/img/posts/Lifetime_Column.jpg "Lifetime Performance")

