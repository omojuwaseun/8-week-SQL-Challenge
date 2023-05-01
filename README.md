# 8-week-SQL-Challenge
### Data analysis for insights
## Introduction

 I recently decided to challenge myself and participate in the 8-week sql challenge hosted by Danny Ma. For the first case we were tasked with some business questions that could help provide insights into the restaurant business that will help make recommendations needed to grow 
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen. His Diner is in need of assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they've spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalized experience for his loyal customers.
He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues and has shared 3 key datasets for this case study: Sales, Menu, Members


## Case Study Questions

## What is the total amount each customer spent at the restaurant?
`SELECT sales.customer_id, sum(price) as total_spent
FROM sales
left join menu
on sales.product_id = menu.product_id
group by 1;`

## How many days has each customer visited the restaurant?
`select customer_id, count(distinct order_date) as number_of_visits
from sales
group by 1;`

## What was the first item from the menu purchased by each customer?
`WITH first_order AS 
  (SELECT customer_id, product_id, order_date, 
     DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS order_rank
   FROM sales)
select customer_id, order_date, m.product_name
from first_order f
join menu m
on f.product_id = m.product_id
where order_rank = 1
order by 1;`

## What is the most purchased item on the menu and how many times was it purchased by all customers?
`SELECT product_name, COUNT(*) as total_purchases
FROM sales
JOIN menu ON sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY total_purchases DESC
LIMIT 1;`

## Which item was the most popular for each customer?
`with most_pop as (
select customer_id, product_id, count(sales.product_id) as quantity,
DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(sales.product_id)) as order_rank
from sales
group by 1,2)
select customer_id, menu.product_name, quantity
from most_pop
join menu
on most_pop.product_id = menu.product_id
where order_rank = 1 
order by customer_id;`

## Which item was purchased just before the customer became a member?
`with item1 as (
select customer_id, order_date, product_id
from sales
), item2 as (
select customer_id, order_date, menu.product_name
from item1 
join menu
on item1.product_id = menu.product_id
), item3 as (
select item2.customer_id, max(item2.order_date) as max_order_date
from item2
join members
on item2.customer_id = members.customer_id
where item2.order_date < members.join_date
group by item2.customer_id
)
select item2.customer_id, item2.order_date, item2.product_name
from item2
join item3
on item2.customer_id = item3.customer_id and item2.order_date = item3.max_order_date;`


## What is the total items and amount spent for each member before they became a member?
`select s.customer_id, count(s.product_id) as items_bought, sum(m.price) as total_spent
from sales s
join menu m
on s.product_id = m.product_id
join members me
on s.customer_id = me.customer_id
where order_date < join_date
group by 1;`

## If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
`with sub as (
select s.customer_id, m.product_name, m.price,
	CASE WHEN product_name = 'sushi'
	THEN m.price * 2*10
	ELSE m.price *10
	END AS points
From sales s
join menu m 
using(product_id)
)
SELECT customer_id, sum(price) as total_spent, sum(points) as total_points
from sub
group by 1
order by 1;`
