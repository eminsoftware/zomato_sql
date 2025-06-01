# Zomato Data Analytics using SQL  
**Project Level**: Intermediate

## Introduction  
This project simulates the analytics side of a food delivery platform, using a dataset modeled after Zomato. It demonstrates how SQL can be used not only to clean and explore data, but also to derive insights relevant to user behavior, restaurant operations, and sales performance. The project is structured to showcase beginner to advanced SQL skills, including triggers, functions, and materialized views.

## Overview  
The dataset consists of five interrelated tables: `restaurant`, `users`, `orders`, `menu`, and `food`. Using PostgreSQL, various SQL queries were written to clean the data, analyze it, and answer meaningful business questions. Tasks are grouped into Beginner, Intermediate, and Advanced levels to reflect the growing complexity of operations performed throughout the project.

## Project Breakdown

- **Data Modeling & Setup**: Created normalized SQL tables and established appropriate foreign key constraints across the dataset.
- **Data Cleaning**: Standardized inconsistent data formats across columns such as city, cost, cuisine, currency, and restaurant addresses. Duplicate entries were removed and fields like income categories were reclassified.
- **Basic Analysis**: Ran SQL queries to count users, extract cities, identify age groups, and aggregate restaurant stats.
- **Intermediate Analysis**: Applied joins, subqueries, and grouping to calculate average sales, identify inactive users, analyze food popularity, and evaluate customer purchasing behavior.
- **Advanced SQL**: Developed stored functions for user lookups and restaurant insights, created materialized views for monthly revenue, and implemented triggers to log high-value orders and price changes.
- **Real-World Readiness**: Queries and logic simulate how a real food delivery company might structure and query its backend database for business reporting.

## Problems and Solutions

---

