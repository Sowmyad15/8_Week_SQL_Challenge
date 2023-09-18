# Case Study 1: Danny's Diner

## Dataset

### Sales Table ###

| customer_id | order_date               | product_id |
| ----------- | ------------------------ | ---------- |
| A           | 2021-01-01T00:00:00.000Z | 1          |
| A           | 2021-01-01T00:00:00.000Z | 2          |
| A           | 2021-01-07T00:00:00.000Z | 2          |
| A           | 2021-01-10T00:00:00.000Z | 3          |
| A           | 2021-01-11T00:00:00.000Z | 3          |
| A           | 2021-01-11T00:00:00.000Z | 3          |
| B           | 2021-01-01T00:00:00.000Z | 2          |
| B           | 2021-01-02T00:00:00.000Z | 2          |
| B           | 2021-01-04T00:00:00.000Z | 1          |
| B           | 2021-01-11T00:00:00.000Z | 1          |
| B           | 2021-01-16T00:00:00.000Z | 3          |
| B           | 2021-02-01T00:00:00.000Z | 3          |
| C           | 2021-01-01T00:00:00.000Z | 3          |
| C           | 2021-01-01T00:00:00.000Z | 3          |
| C           | 2021-01-07T00:00:00.000Z | 3          |

---
### Menu Table ###

| product_id | product_name | price |
| ---------- | ------------ | ----- |
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |

---
### Members Table ###

| customer_id | join_date                |
| ----------- | ------------------------ |
| A           | 2021-01-07T00:00:00.000Z |
| B           | 2021-01-09T00:00:00.000Z |

---

## Solution

***

### 1. What is the total amount each customer spent at the restaurant?

````sql
Select c.customer_id,sum(m.price) as total_price
from dannys_diner.sales c
inner join
dannys_diner.menu m
on c.product_id=m.product_id
group by c.customer_id
order by c.customer_id;
````

#### Answer:
| customer_id | total_price |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A, B and C spent $76, $74 and $36 respectivly.

***

### 2. How many days has each customer visited the restaurant?

````sql
Select customer_id,count(distinct order_date) as days_visited
from dannys_diner.sales
group by customer_id;
````

#### Answer:
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

- Customer A, B and C visited 4, 6 and 2 times respectivly.

***

### 3. What was the first item from the menu purchased by each customer?

````sql
With first_ordered as
(Select c.customer_id,c.order_date,m.product_name,
dense_rank() over (partition by c.customer_id order by c.order_date asc) as ordered_products
from dannys_diner.sales c inner join dannys_diner.menu m
on c.product_id=m.product_id)

Select customer_id,product_name from first_ordered
where ordered_products=1
group by customer_id,product_name;
````

#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

- Customer A's first order is curry and sushi.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
Select c.product_id,m.product_name,count(c.product_id) as mostly_ordered 
from dannys_diner.sales c inner join dannys_diner.menu m 
on c.product_id=m.product_id
group by c.product_id,m.product_name
order by mostly_ordered desc
limit 1
````



#### Answer:
| product_id | product_name | mostly_ordered |
| ---------- | ------------ | -------------- |
| 3          | ramen        | 8              |

- Most purchased item on the menu is ramen which is 8 times.

***

### 5. Which item was the most popular for each customer?

````sql
With ordered_menu as 
(Select c.customer_id,m.product_name,count(c.product_id),
dense_rank() over (partition by c.customer_id order by count(c.product_id) desc) as ordered_products
from dannys_diner.sales c inner join dannys_diner.menu m
on c.product_id=m.product_id
group by c.customer_id,m.product_name)

Select customer_id,product_name from ordered_menu
where ordered_products=1;
````

#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| B           | curry        |
| B           | sushi        |
| C           | ramen        |

- Customer A and C's favourite item is ramen while customer B savours all items on the menu. 

***

### 6. Which item was purchased first by the customer after they became a member?

````sql
With member_data as 
(Select c.customer_id,c.product_id,m.join_date,
row_number() over (partition by c.customer_id order by c.order_date) as date_joined
from dannys_diner.sales c inner join dannys_diner.members m
on c.customer_id=m.customer_id
and c.order_date>m.join_date
)

Select c.customer_id,m.product_name from member_data c
inner join dannys_diner.menu m on c.product_id=m.product_id
where c.date_joined=1 order by c.customer_id;
````


#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

After becoming a member 
- Customer A's first order was curry.
- Customer B's first order was sushi.

***

### 7. Which item was purchased just before the customer became a member?

````sql
With member_data as 
(Select c.customer_id,c.product_id,m.join_date,
row_number() over (partition by c.customer_id order by c.order_date) as before_joined
from dannys_diner.sales c inner join dannys_diner.members m
on c.customer_id=m.customer_id
and c.order_date<m.join_date
)

select c.customer_id,m.product_name 
from member_data c
inner join dannys_diner.menu m 
on c.product_id=m.product_id
where c.before_joined=1 order by c.customer_id;
````

#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | curry        |
Before becoming a member 
- Customer A’s and B’s last order was sushi and curry.

***

### 8. What is the total items and amount spent for each member before they became a member?

````sql
Select c.customer_id,count(c.product_id),sum(m.price)
from dannys_diner.sales c inner join dannys_diner.menu m
on c.product_id=m.product_id
inner join dannys_diner.members mem
on c.customer_id=mem.customer_id
and c.order_date<mem.join_date
group by c.customer_id
order by c.customer_id asc ;

````


#### Answer:
| customer_id | count | sum |
| ----------- | ----- | --- |
| A           | 2     | 25  |
| B           | 3     | 40  |

Before becoming a member
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
With points_on_items as (select c.customer_id,
case
when c.product_id=1 then count(c.product_id)*m.price*20
else count(c.product_id)*m.price*10
end as total
from dannys_diner.sales c
inner join dannys_diner.menu m on c.product_id=m.product_id
group by c.customer_id,c.product_id,m.price
)

Select customer_id,sum(total) as total_points 
from points_on_items
group by customer_id order by customer_id ;
````


#### Answer:
| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

- Total points for customer A, B and C are 860, 940 and 360 respectivly.

***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql
With valid_data as 
(SELECT customer_id, join_date, join_date + 6 as valid_date 
FROM dannys_diner.members)

Select c.customer_id,
sum(
	case
	when c.order_date between v.join_date and v.valid_date
	then p.price*20
	when p.product_id=1
	then p.price*20
	else
	p.price*10
	end) as total
from
dannys_diner.sales c inner join valid_data v on
c.customer_id=v.customer_id
and c.order_date<'2021-02-01'
inner join dannys_diner.menu p on
c.product_id=p.product_id
group by c.customer_id;
````

#### Answer:
| customer_id | total |
| ----------- | ----- |
| A           | 1370  |
| B           | 820   |

- Total points for Customer A and B are 1370 and 820 respectivly.

***
