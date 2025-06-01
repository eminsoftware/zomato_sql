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
---

