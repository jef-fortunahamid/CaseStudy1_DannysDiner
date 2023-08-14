# Case Study #1 Danny's Diner
*Note: All information and data related to the case study were obtained from [here](https://8weeksqlchallenge.com/case-study-1/).*

## Business Task
Danny wants to understand his customers' preferences, visit patterns, and spending habits. He believes this information will enhance  the customer experience. Furthermore, he's considering exopanding the customer loyalty program and requires datasets for his team to review without SQL. Howver, due to privacy issues, Danny only provided with a sample of his overall customer data. Hoping that these sample data are enough to write fully functioning SQL queries to help him answer his questions.

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
  6. Distinct Values (DISTINCT)

## Questions and Solutions
> What is the total amount each customer spent at the restaurant?
```sql
SELECT
    sales.customer_id
  , SUM(menu.price) AS total_purchase
FROM dannys_diner.sales 
INNER JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_purchase DESC;
```
-'sales.customer_id' and 'menu.price' columns are from 'dannys_dinner.sales' and 'dannys_diner.menu, repectively. So we need to merge the tables using INNER JOIN (connecting them based on the 'product_id').
-We're selecting the record of each customer through 'sales.customer_id' and we're adding (summing) the prices of all the products, 'SUM(menu.price) AS total_purchase'.
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/af990388-739d-4c18-85d6-d6ff3b7ee0db)

> How many days has each customer visited the restaurant?
```sql
SELECT
    customer_id
  , COUNT(DISTINCT order_date) AS total_purchasing_days
FROM dannys_diner.sales
GROUP BY customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/72c44518-ad9b-4a3e-b22c-77220ce7cc37)

> What was the first item(s) from the menu purchased by each customer?
```sql
WITH first_order AS (
  SELECT
      sales.customer_id
    , sales.order_date
    , RANK() OVER (
          PARTITION BY sales.customer_id 
          ORDER BY sales.order_date
          ) AS sales_date_rank -- to rank the days starting from the first day they are a customer
    , menu.product_name
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)
SELECT DISTINCT --to select all unique menu items purchased on the same day 
    customer_id
  , product_name
FROM first_order
WHERE sales_date_rank = 1;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/013f05b9-ece4-4d6d-b6d9-a36697222724)

> What is the most purchased item on the menu and how many times was it ourchased by all customers?
```sql
SELECT
    menu.product_name
  , COUNT(sales.*) AS total_purchases
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY total_purchases DESC
LIMIT 1;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/00dfc7dd-04e4-455c-b28d-94ef6cd86ecc)

> Which item(s) was(were) the most popular for each customer?
```sql
WITH customer_menu_count AS (  
  SELECT
      sales.customer_id
    , menu.product_name
    , COUNT(sales.product_id) AS menu_count
    , RANK() OVER (
            PARTITION BY sales.customer_id 
            ORDER BY COUNT(sales.product_id) DESC --the highest count comes first
          ) AS menu_rank 
        --to give us the same rank for menu items that have the same count
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  GROUP BY 
      sales.customer_id
    , menu.product_name
)
SELECT
    customer_id
  , product_name
  , menu_count
FROM customer_menu_count
WHERE menu_rank = 1; --we only need the top rank
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/bf207c33-0b3b-477f-a6b0-316ed018f7c9)

> Which Item was pourchased first by the customer after they became a member and what date was it? (including the date they joined)
```sql
WITH first_purchase_after_membership AS (
  SELECT
      sales.customer_id
    , sales.order_date 
    , menu.product_name
    , RANK() OVER (
              PARTITION BY sales.customer_id 
              ORDER BY sales.order_date
              ) AS purchase_rank
            --to rank order date in chronological order from or after join date
    , members.join_date
  FROM dannys_diner.sales 
  LEFT JOIN dannys_diner.members
    ON members.customer_id = sales.customer_id
  LEFT JOIN dannys_diner.menu 
    ON sales.product_id = menu.product_id
  WHERE sales.order_date >= members.join_date -- to select dates on or after the join date
)
SELECT DISTINCT --to select all unique menu items purchased on the same day
    customer_id
  , join_date
  , order_date AS first_order_date_after_membership
  , product_name
FROM first_purchase_after_membership
WHERE purchase_rank = 1;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/3a1d483e-495f-457e-9422-154c0416c7c2)

> Which menu item(s) was(were) purchased just before the customer became a member and when?
```sql
WITH purchase_before_membership AS (
  SELECT
      sales.customer_id
    , sales.order_date
    , menu.product_name
    , RANK() OVER (
              PARTITION BY sales.customer_id 
              ORDER BY sales.order_date DESC --need to rank date from latest to earliest
              ) AS purchase_rank
    , members.join_date
  FROM dannys_diner.sales 
  LEFT JOIN dannys_diner.members
    ON members.customer_id = sales.customer_id
  LEFT JOIN dannys_diner.menu 
    ON sales.product_id = menu.product_id
  WHERE sales.order_date < members.join_date -- to select dates before the join date
)
SELECT DISTINCT --to select all unique menu items purchased on the same day
    customer_id
  , order_date
  , product_name
FROM purchase_before_membership
WHERE purchase_rank = 1;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/14401117-4b3a-4b41-b128-cfea53b73277)

> WHat is the number of unique menu items and total amount spent for each member before they became a member?
```sql
SELECT
    sales.customer_id
  , COUNT(DISTINCT sales.product_id) AS unique_menu --count unique menu items
  , SUM(menu.price) AS total_price --get the total amount spent
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON members.customer_id = sales.customer_id
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
WHERE sales.order_date < members.join_date --get the date before member's join date
GROUP BY
  sales.customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/7dbd59f6-302f-4139-8617-26577b3659d6)

> If each $1 spent equates to 10 points and sushi has 2x points multiplier - how many points would each customer have?
```sql
SELECT
    sales.customer_id
  , SUM (CASE WHEN menu.product_name = 'sushi' THEN (menu.price * 10 * 2)
          ELSE (price * 10)
          END) AS total_points
FROM dannys_diner.sales AS sales
LEFT JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_points DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/bb78487f-98ab-4397-8820-1384b92549f8)

> In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at bthe end of January?
```sql
SELECT
    sales.customer_id
  , SUM (CASE
          WHEN sales.order_date BETWEEN members.join_date AND (members.join_date + INTERVAL '6 days')
              THEN (menu.price * 10 * 2)
          WHEN menu.product_name = 'sushi' 
              THEN (menu.price * 10 * 2)
          ELSE (menu.price * 10)
          END) AS total_points
FROM dannys_diner.sales AS sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
WHERE 
    sales.order_date <= '2021-01-31'
GROUP BY sales.customer_id
ORDER BY total_points DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/a5da89ed-0c87-4be8-b442-2c08ef624ec3)

> Bonus Questions

> Danny and his team can use to quickly derive insights without needing to join the tables using SQL. Recreate the following table output using the available data.

| customer_id | order_date |product_name |	price |	member |
|-------------|------------|-------------|--------|-------|
| A |	2021-01-01 |	curry |	15 |	N |
| A	| 2021-01-01 |	sushi	| 10 |	N |
| A	| 2021-01-07 |	curry	| 15 |	Y |
| A	| 2021-01-10 |  ramen | 12 |  Y |
| A	| 2021-01-11 |	ramen | 12 |	Y |
| A	| 2021-01-11 | 	ramen |	12 |	Y |
| B	| 2021-01-01 | 	curry |	15 |	N |
| B	| 2021-01-02 |	curry |	15 |	N |
| B	| 2021-01-04 |	sushi |	10 |	N |
| B	| 2021-01-11 |	sushi |	10 |	Y |
| B	| 2021-01-16 |	ramen |	12 |	Y |
| B	| 2021-02-01 |	ramen |	12 |	Y |
| C	| 2021-01-01 |	ramen |	12 |	N |
| C	| 2021-01-01 |	ramen |	12 |	N |
| C	| 2021-01-07 |	ramen |	12 |	N |
```sql
SELECT
    sales.customer_id
  , sales.order_date
  , menu.product_name
  , menu.price
  , CASE 
        WHEN sales.order_date >= members.join_date THEN 'Y'
        ELSE 'N'
      END AS member
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
LEFT JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
ORDER BY
    sales.customer_id
  , sales.order_date
;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/2d104ff9-2a0f-46a3-b0e5-887609a757ea)

> Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

| customer_id	| order_date	| product_name	| price	| member	| ranking |
| ------------| ------------| --------------| ------| --------| --------| 
| A	| 2021-01-01	| curry	| 15	| N	| null | 
| A	| 2021-01-01	| sushi	| 10	| N	| null | 
| A	| 2021-01-07	| curry	| 15	| Y	| 1 | 
| A	| 2021-01-10	| ramen	| 12	| Y	| 2 | 
| A	| 2021-01-11	| ramen	| 12	| Y	| 3 | 
| A	| 2021-01-11	| ramen	| 12	| Y	| 3 | 
| B	| 2021-01-01	| curry	| 15	| N	| null | 
| B	| 2021-01-02	| curry	| 15	| N	| null | 
| B	| 2021-01-04	| sushi	| 10	| N	| null | 
| B	| 2021-01-11	| sushi	| 10	| Y	| 1 | 
| B	| 2021-01-16	| ramen	| 12	| Y	| 2 | 
| B	| 2021-02-01	| ramen	| 12	| Y	| 3 | 
| C	| 2021-01-01	| ramen	| 12	| N	| null | 
| C	| 2021-01-01	| ramen	| 12	| N	| null | 
| C	| 2021-01-07	| ramen	| 12	| N	| null | 
```sql
WITH joint_sales AS (
	SELECT
	    sales.customer_id
	   , sales.order_date
	   , menu.product_name
	   , menu.price
	   , CASE 
  	    WHEN sales.order_date >= members.join_date THEN 'Y'
  	    ELSE 'N'
  	    END AS member
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.members
	  ON sales.customer_id = members.customer_id
	LEFT JOIN dannys_diner.menu
	  ON sales.product_id = menu.product_id
	ORDER BY
	     sales.customer_id
	   , sales.order_date
)
SELECT
    customer_id
  , order_date
  , product_name
  , price
  , member
  , CASE
        WHEN member = 'N' THEN null
        ELSE
          RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
      END AS ranking
FROM joint_sales
ORDER BY
    customer_id
  , order_date
;
```
![image](https://github.com/jef-fortunahamid/CaseStudy1_DannysDiner/assets/125134025/f8613629-3c51-483e-9d22-982701d5f894)
