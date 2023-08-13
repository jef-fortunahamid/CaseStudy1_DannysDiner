# Case Study #1 Danny's Diner

## Business Task

## Entity Relationship Diagram
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/655e02ca-a3dc-4e94-bb32-e45fa198cefe)


## General Insights:
- *Customer Spending and Visits:* The queries aimed to understand customer behaviours, such as total amounts spent, frequency of visits, and preferences before and after becoming members.
- *Menu Item Popularity:* Several questions focused on identifying popular menu items, both overall and specific ot each member.
- *Membership Impact:* The impact of membership on purchasing behaviour was a recurring theme, with insights into purchases made before and after joining the loyalty program.
- *Points System:* The potential introduction of a points system was explored, considering various multipliers for specific items and timeframes.

## Key SQL Syntax and Functions
These are the syntax and functions that have been used to explore and answer the problems given.
  1. Joins (LEFT JOIN, INNER JOIN)
  2. Aggregation Functions (SUM, COUNT)
  3. Window Functions (RANK(), ROW_NUMBER() DENSED_RANK())
  4. Common Table Expressions (CTE)
  5. Conditional Logic (CASE WHEN)
  6. Date Functions
  7. Grouping and Ordering (GROUP BY, ORDER BY)
  8. Limiting Results (LIMIT)
  9. Distinct Values (DISTINCT)

## Questions and Solutions
> What is the total amount each customer spent at the restaurant?

```sql
SELECT
    sales.customer_id
  , SUM(menu.price) AS total_purchase
FROM dannys_diner.sales 
LEFT JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_purchase DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/af990388-739d-4c18-85d6-d6ff3b7ee0db)
