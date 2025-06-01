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


2)  /* restaurant names are formatted in a messy way: fully in upper or lowercase, in mixed case, etc.
	I fixed it to make it readable and clean */

update restaurant
set name = initcap(name)
where name is not null;


3) 	/* some addresses are corrupted, like: 
	"PALASH RESTAURANT AND BAR, S No 148A\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\..." */

update restaurant
set address = replace(address, '\\', '')
where address like '%\\%';

update restaurant
set address = initcap(address);

 
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


6)  -- duplicate deletion

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
---

