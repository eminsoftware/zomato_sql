# Zomato Data Analytics using SQL  
**Project Level**: Advanced

## Overview  
The dataset consists of five interrelated tables: `restaurant`, `users`, `orders`, `menu`, and `food`. Using PostgreSQL, various SQL queries were written to clean the data, analyze it, and answer meaningful business questions. Tasks are grouped into Beginner, Intermediate, and Advanced levels to reflect the growing complexity of operations performed throughout the project.

## Project Breakdown

- **Data Modeling & Setup**: Created normalized SQL tables and established appropriate foreign key constraints across the dataset.
- **Data Cleaning**: Standardized inconsistent data formats across columns such as city, cost, cuisine, currency, and restaurant addresses. Duplicate entries were removed and fields like income categories were reclassified.
- **Basic Analysis**: Ran SQL queries to count users, extract cities, identify age groups, and aggregate restaurant stats.
- **Intermediate Analysis**: Applied joins, subqueries, and grouping to calculate average sales, identify inactive users, analyze food popularity, and evaluate customer purchasing behavior.
- **Advanced SQL**: Developed stored functions for user lookups and restaurant insights, created materialized views for monthly revenue, and implemented triggers to log high-value orders and price changes.
- **Real-World Readiness**: Queries and logic simulate how a real food delivery company might structure and query its backend database for business reporting.

## Tasks and Solutions

### Creating Tables
```sql
create table restaurant (
    id text primary key,
    name text,
    city text,
    rating text,
    rating_count text,
    cost text,
    cuisine text,
    lic_no text,
    link text,
    address text,
    menu text
);

create table users (
    user_id integer primary key,
    name text,
    email text,
    password text,
    age text,
    gender text,
    marital_status text,
    occupation text,
    monthly_income text,
    educational_qualifications text,
    family_size text
);

create table food (
    f_id text primary key,
    item text,
    veg_or_non_veg text
);

create table menu (
    menu_id text,
    r_id text,
    f_id text,
    cuisine text,
    price varchar(20),
	primary key (menu_id, r_id, f_id)
);

create table orders (
    order_id serial primary key,
    order_date date,
    sales_qty integer,
    sales_amount integer,
    currency text,
    user_id integer,
    r_id text
);
```

### Assigning Foreign Keys
```sql
alter table orders
add constraint fk_user_id
foreign key (user_id) references users(user_id);

alter table menu
add constraint fk_food_id
foreign key (f_id) references food(f_id);

--

/* 
issue: 
	 fk_restaurant_id constraint failed because in 'id' table value is formatted like '567335',
	 but in 'r_id' like '567335.0' despite both columns having 'text' data type.
solution: 
     I updated 'r_id' and formatted the values exactly like in 'id'
	 Then, I successfully created the constraint.
*/

update orders
set r_id = regexp_replace(r_id, '\.0$', '');

alter table orders
add constraint fk_restaurant_id
foreign key (r_id) references restaurant(id);

--

/*
issue:
	 fk_restaurant_id2 constraint failed because some values were present in r_id but not in id
solution:
	 as the number of unmatching rows wasn't much (only 273),
	 I decided to delete those values to enforce referential integrity
     Then, I successfully created the constraint.
*/

delete from menu
where r_id not in 
(
select distinct id
from restaurant
);

alter table menu
add constraint fk_restaurant_id2
foreign key (r_id) references restaurant(id)
```

### Data Cleaning
```sql
1)	/* in rows of cuisine / city columns where there are multiple values, there is no space after comma between those values
	example: Indian,Rajasthani instead of Indian, Rajasthani
	I fixed it because this reduces the readability
	'g': global flag to apply it to all occurrences in a row */

update restaurant
set cuisine = regexp_replace(cuisine, ',', ', ', 'g')
where cuisine like '%,%';

update restaurant
set city = regexp_replace(city, ',', ', ', 'g')
where city like '%,%';

update menu
set cuisine = regexp_replace(cuisine, ',', ', ', 'g')
where cuisine like '%,%';
```
```sql
2)	/* restaurant names are formatted in a messy way: fully in upper or lowercase, in mixed case, etc.
	I fixed it to make it readable and clean */

update restaurant
set name = initcap(name)
where name is not null;
```
```sql
3)	/* some addresses are corrupted, like: 
       "PALASH RESTAURANT AND BAR, S No 148A\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\..." */

update restaurant
set address = replace(address, '\\', '')
where address like '%\\%';

update restaurant
set address = initcap(address);
```
```sql
4)	-- updating the categorizations within the monthly_income column 

update users
set monthly_income = case
	when monthly_income = 'More than 50000' then '₹50,000+'
	when monthly_income = 'Below Rs.10000' then 'up to ₹10,000'
	when monthly_income = '25001 to 50000' then '₹25,001-50,000'
	when monthly_income = '10001 to 25000' then '₹10,001-25,000'
	else monthly_income
end
where monthly_income in ('More than 50000', 'Below Rs.10000', '25001 to 50000', '10001 to 25000');
```
```sql
5)	/* distinct currency outputted INR, INR, USD. 
	I found out that one of INR has a length of 3, while the other one 4
	this meant that the INR with a length of 4 has a hidden charachter that differentiates it
	I filtered that INR: where length(currency) = 4, and queried the ASCII code of each charachter in the word 'INR' 
	c4 had the code 13, and ASCII code of 13 stands for \r (carriage return)
	so, I replaced it with a space. consequently, now distinct currency outputs only 1 INR */

select distinct currency, 
	   length(currency) 
from orders;

select currency,
       ASCII(substring(currency from 1 for 1)) as c1,
       ASCII(substring(currency from 2 for 1)) as c2,
       ASCII(substring(currency from 3 for 1)) as c3,
       ASCII(substring(currency from 4 for 1)) as c4
from orders
where length(currency) = 4
limit 5;

update orders
set currency = replace(currency, CHR(13), '')
where length(currency) = 4;
```
```sql
6)	-- duplicate deletion

create table menu_backup as select * from menu;

with ranked_menu as (
  select *,
         row_number() over (
           partition by r_id, f_id, cuisine, price order by menu_id
         ) as rn
  from menu
)
delete from menu
where menu_id in (
  select menu_id from ranked_menu where rn > 1
);
```

### Basic Tasks (1-8)
```sql
1) -- List all distinct cities where restaurants are located.

select distinct city 
from restaurant;
```
```sql
2) -- Count how many users are there in total.

select count(user_id)
from users;
```
```sql
3) -- List all users whose age is below 25.

select * 
from users
where age < 25;
```
```sql
4) -- Find total number of menu items for each restaurant.

select name, 
	   r_id, 
	   count(menu_id)
from menu m
join restaurant r
on m.r_id = r.id
group by 1, 2;
```
```sql
5) -- List top 3 most common occupations among users.

select occupation, 
	   count(*)
from users
group by 1
limit 3;
```
```sql
6) -- Show all restaurants with cost greater than ₹500.

select *
from restaurant
where substring(cost,2)::int > 500
and cost != '';
```
```sql
7) -- Count the number of restaurants per city.
      /* some city names are inputted like 'city x, city' and some of them like 'city x & city y'
      replace function converts '&' to a comma. then, unnest splits by a comma */

select trim(unnest(string_to_array(replace(city, '&', ','), ','))) as city, 
	   count(id) as cnt_restaurants
from restaurant
group by 1
order by 2 desc;
```
```sql
8) -- List all non-veg items from the food table.

select item 
from food 
where veg_or_non_veg = 'Non-veg';
```

### Intermediate Tasks (9-17)
```sql
9) -- Find the average number of items sold per restaurant.

select r_id, 
       name, 
	   round(avg(sales_qty)) as avg_sales
from orders o
join restaurant r
on o.r_id = r.id
group by 1, 2
order by 3 desc;
```
```sql
10) -- Identify users who have never placed an order.

select * 
from users
where user_id not in
(
select user_id
from orders
);
```
```sql
11) -- Determine which food item appears most frequently in the menu.

select f_id, 
	   food_name, 
	   veg_or_non_veg, 
	   count(f_id)
from
(
select m.f_id, 
	   item as food_name, 
	   veg_or_non_veg
from menu m
join food f
on m.f_id = f.f_id
)
where food_name is not null
group by 1, 2, 3
order by 4 desc;
```
```sql
12) -- Find 5 users with the highest total sales_amount.

select name, 
	   sales_qty, 
	   u.user_id
from orders o
join users u
on o.user_id = u.user_id
order by 2 desc
limit 5;
```
```sql
13) -- Get a count of orders per month using order_date.

select extract(month from order_date) as month, 
       count(user_id) as cnt_orders
from orders
group by 1
order by 1;
```
```sql
14) -- Find restaurants that serve a specific cuisine (e.g., 'Chinese').

select * 
from menu
where cuisine like '%Chinese%';
```
```sql
15) -- Find the average age of users by city.

select trim(unnest(string_to_array(city, ','))) as city_name, 
       round(avg(age)) as avg_age
from restaurant r
join orders o
on r.id = o.r_id
join users u
on o.user_id = u.user_id
group by 1
order by 1;
```
```sql
16) -- Count total number of veg vs non-veg items.

select veg_or_non_veg, 
       count(f_id)
from food
where veg_or_non_veg is not null
group by 1;
```
```sql
17) -- Create a report showing each user's total and average spending.

select user_id, 
       name, 
	   sum(sales_amount) as sum_sales_amount, 
	   round(avg(sales_amount)) as avg_sales_amount
from
(
select u.user_id, name, sales_amount
from users u
join orders o
on u.user_id = o.user_id
)
group by 1, 2
order by user_id;
```

### Advanced Tasks (18-22)
```sql
18) --  Create the following function:
        /* input: r_id
        output: Top 3 food items by sales from that restaurant. */

create or replace function fn_top_items(p_rid int)
returns text
as $$

declare
	top_items text;
	
begin

	with x_cte as
	(
	select 
		o.r_id, 
		r.name as restaurant_name, 
		o.sales_amount as total_sales, 
		item as food_name, 
		row_number() over(partition by r.name order by sum(sales_amount)) as rn
	from orders o
	join restaurant r on o.r_id = r.id
	join menu m on r.id = m.r_id
	join food f on f.f_id = m.f_id
	where o.r_id::int = p_rid
	group by 1, 2, 3, 4
	)
	select string_agg(food_name, ', ') into top_items
	from x_cte
	where rn <= 3;

	return top_items;
	
end;

$$
language plpgsql;

select fn_top_items('564436');
```
```sql
19) -- Create the following function:
       /* input: user_id
       output: name, age, monthly income of the given user */

create or replace function fn_user_info(p_userid int)
returns table
(
user_id int,
name text,
age int,
monthly_income text
)

as $$
	
begin

	return query
	select users.user_id, 
	       users.name, 
		   users.age, 
		   users.monthly_income 
	from users
	where users.user_id = p_userid;

end;
$$
language plpgsql

select * 
from fn_user_info('76543');
```
```sql
20) -- Create a materialized view that shows monthly revenue for each restaurant.

create materialized view mn_monthly_revenue as
select 
	r.id as restaurant_id, 
	r.name as restaurant_name, 
	extract(month from o.order_date) as month,
	sum(o.sales_amount) as total_monthly_revenue
from restaurant r
join orders o
on r.id = o.r_id
group by 1, 2, 3
order by 3;

select * 
from mn_monthly_revenue;

refresh materialized view mv_monthly_revenue;
```
```sql
21) -- Create a trigger to automatically log every order over ₹1000 into a table called high_value_orders
    -- whenever an order is inserted into the orders table.

create table high_value_orders
(
  order_id serial,
  order_date date,
  sales_qty integer,
  sales_amount integer,
  currency text,
  user_id integer,
  r_id text,
  logged_at TIMESTAMP default now()
);

create or replace function tr_fn_high_value_orders()
returns trigger
as $$
begin

	if
		new.sales_amount > 1000 then
		insert into high_value_orders (order_id, order_date, sales_qty, sales_amount, currency, user_id, r_id, logged_at)
		values (new.order_id, new.order_date, new.sales_qty, new.sales_amount, new.currency, new.user_id, new.r_id, now());

	end if;

	return new;

end;
$$
language plpgsql;

create trigger tr_high_value_orders
after insert on orders
for each row
execute function tr_fn_high_value_orders();

insert into orders(order_id, order_date, sales_qty, sales_amount, currency, user_id, r_id)
values(1235762, current_date, 2, 1400, 'INR', 74, 484807);

select * 
from high_value_orders;
```
```sql
22) -- Create a trigger to automatically log old and new price into table called price_change
    -- after an update on menu table

create table price_change
(
menu_id text,
r_id text,
f_id text,
cuisine text,
old_price varchar(20),
new_price varchar(20),
updated_at timestamp default now()
);

create or replace function tr_fn_price_change()
returns trigger 
as $$
begin
	if 
		new.price <> old.price then
		insert into price_change(menu_id, r_id, f_id, cuisine, old_price, new_price, updated_at)
		values(old.menu_id, old.r_id, old.f_id, old.cuisine, old.price, new.price, now());

	end if;

	return new;

end;
$$
language plpgsql;

create trigger tr_price_change
after update on menu
for each row
execute function tr_fn_price_change();

update menu
set price = 70.0
where menu_id = 'mn328'; 

select * 
from price_change;
```

## Conclusion  
This project provided a well-rounded opportunity to work with a simulated real-world relational database, covering everything from foundational queries to complex logic implementations. Through methodical data cleaning and analysis, I extracted meaningful insights about users, restaurants, and sales. The implementation of PostgreSQL functions, materialized views, and triggers also allowed me to practice advanced SQL features that are crucial for robust data engineering and analytics tasks.  



