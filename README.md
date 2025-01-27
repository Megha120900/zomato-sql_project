# Zomato Data Analysis using SQL
![Zomato logo](https://github.com/Megha120900/zomato-sql_project/blob/main/Zomato-Logo.png)

## Overview
### This project demonstrates my SQL problem-solving skills through the analysis of data for Zomato, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex sql queries

## Project Structure

#### • Database Setup: Creation of the zomato_d database and the required tables.
#### • Data Import: Inserting sample data into the tables.
#### • Data Cleaning: Handling null values and ensuring data integrity.
#### • Business Problems: Solving 16 specific business problems using SQL queries.


## Schema

```sql
CREATE TABLE customers
customer _id INT PRIMARY KEY,
customer_name VARCHAR (25),
reg_date DATE
) ;

CREATE TABLE restaurants
(
restaurant_id INT PRIMARY KEY,
restaurant_name VARCHAR (55),
city VARCHAR (15),
opening-hours VARCHAR (55)
);

CREATE TABLE orders
(
restaurant_id INT,
order_id INT PRIMARY KEY,
customer_id INT,
order_item VARCHAR (55) ,
order _date DATE,
order-time TIME,
order_status VARCHAR (55),
total_amount FLOAT
constraint fk_restaurants (restaurant_id) references restaurants(restaurant_id)
constraint fk_customers (customer_id) references customers(customer_id)
);


CREATE TABLE riders
(
rider_id INT PRIMARY KEY
rider_name VARCHAR (55) ,
sign_up DATE
);


CREATE TABLE deliveries
(
delivery_id INT PRINARY KEY,
order_id int,
delivery_status VARCHAR(35),
delivery_time TIME,
rider_id INT
constraint fk_orders (order_id) references orders(order_id)
constraint fk_riders (rider_id) references riders(rider_id)
);

```
## Business Questions and Analysis

### 1. Finding top ordered dish by a particular customer

```sql
Select c.customer_id,c.customer_name,trim(UNNEST(STRING_TO_ARRAY(o.order_item, ','))) as dish,count(*),
DENSE_RANK() OVER(order by count(*) desc) as rank
from customers c
inner join orders o on c.customer_id=o.customer_id
WHERE o.order_date >= current_date-interval '1 Year' and c.customer_id=1563
group by 1,3
order by 4 desc;

```

### 2. Identifying number of orders placed between each time slot, based on 2 hour window 

```sql

Select count(*) as order_count,
case WHEN extract(hour from order_time) between 0 and 1 then '00:00 - 2:00'
	WHEN extract(hour from order_time) between 2 and 3 then '02:00 - 4:00'
	WHEN extract(hour from order_time) between 4 and 5 then '04:00 - 6:00'
	WHEN extract(hour from order_time) between 6 and 7 then '06:00 - 8:00'
	WHEN EXTRACT (HOUR FROM order_time)BETWEEN 8 AND 9 THEN '08:00 - 10:00'
	WHEN EXTRACT (HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
 	WHEN EXTRACT (HOUR FROM order_time)BETWEEN 12 AND 13 THEN '12:00 - 14:00'
 	WHEN EXTRACT (HOUR FROM order_time)BETWEEN 14 AND 15 THEN '14:00 - 16:00'
 	WHEN EXTRACT (HOUR FROM order_time)BETWEEN 16 AND 17 THEN '16:00 - 18:00'
	WHEN EXTRACT (HOUR FROM order_time)BETWEEN 18 AND 19 THEN '18:00- 20:00'
	WHEN EXTRACT (HOUR FROM order_time)BETWEEN 20 AND 21 THEN '20:00 - 22:00'
	WHEN EXTRACT (HOUR FROM order_time)BETWEEN 22 AND 23 THEN '22:00 -00:00'
  END AS time_period
from orders
group by time_period
order by count(*) desc;

```

### 3. Average order value of customers who have placed more than 10 orders


```sql
Select c.customer_name,o.customer_id,avg(total_amount) as AOV
from orders o 
join 
customers c 
On
o.customer_id=c.customer_id
group by 2,1
HAVING COUNT(*)>10
order by 3 desc

```

### 4. List of users who have spent more than 10,000 Rs on orders in past 1 year

```sql

select customer_id,sum(total_amount) as total_spent from orders
where order_date >= current_date-interval '1 Year'
group by customer_id
having sum(total_amount)>10000
order by 2 desc;

```

### 5. Orders that were placed but not delivered
Returning restaurant_name, restaurant_id

```sql

Select r.restaurant_name,r.city,r.restaurant_id, count(*) as failed_delivery from orders o
join restaurants r 
on
o.restaurant_id=r.restaurant_id
join deliveries d
on 
o.order_id=d.order_id
where d.delivery_status='Failed'
group by r.restaurant_id
order by failed_delivery desc;

```

### 6. Ranking restaurants based on revenue earned till date within each city and overall

```sql

Select r.restaurant_name,r.city,r.restaurant_id,sum(total_amount) as Revenue,
dense_rank() over(partition by r.city order by sum(total_amount) desc) as withincity_rank,
dense_rank() over(order by sum(total_amount) desc) as overall_rank
from orders o
join restaurants r
on 
o.restaurant_id=r.restaurant_id
group by 2,3

```

### 7. Identifying most popular dish in each city

```sql

Select * from
(
Select TRIM(UNNEST(string_to_array(o.order_item,','))) AS dishes,
r.city,
count(*) as number_of_orders,
dense_rank() over(partition by r.city order by count(*) desc) as city_rank
from orders o
join restaurants r on
o.restaurant_id=r.restaurant_id
GROUP BY 1,2
) 
as t1
where city_rank = 1;

```

### 8. Customer churn - Customers who have not placed orders in past one year

```sql

Select distinct customer_id
from orders
where extract(year from order_date)=2024
and customer_id not in
	(
	Select distinct customer_id
	from orders
	where extract(year from order_date)=2025);

```


