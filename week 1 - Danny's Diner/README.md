# Week 1 - Danny’s Diner

![dannys_diner](https://github.com/venkatasaikarthikeya/8weeks-sql-challenge/assets/34571463/5e4e6b33-b38a-4add-81aa-bb38a2703ce3)


### **Introduction**:

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favorite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

### **Problem Statement:**

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favorite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

- `sales`
- `menu`
- `members`

You can inspect the entity relationship diagram and example data below:

<img width="711" alt="Screenshot 2024-01-08 at 1 07 57 PM" src="https://github.com/venkatasaikarthikeya/8weeks-sql-challenge/assets/34571463/c3da50c1-87a5-4c38-95d1-929f50075804">



**Dataset Code**:

```jsx
CREATE DATABASE IF NOT EXISTS dannys_diner;

USE dannys_diner;

CREATE TABLE sales (
  customer_id VARCHAR(1),
  order_date DATE,
  product_id INTEGER
);

INSERT INTO sales
  (customer_id, order_date, product_id)
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');

SELECT *
FROM sales;
+-------------+------------+------------+
| customer_id | order_date | product_id |
+-------------+------------+------------+
| A           | 2021-01-01 |          1 |
| A           | 2021-01-01 |          2 |
| A           | 2021-01-07 |          2 |
| A           | 2021-01-10 |          3 |
| A           | 2021-01-11 |          3 |
| A           | 2021-01-11 |          3 |
| B           | 2021-01-01 |          2 |
| B           | 2021-01-02 |          2 |
| B           | 2021-01-04 |          1 |
| B           | 2021-01-11 |          1 |
| B           | 2021-01-16 |          3 |
| B           | 2021-02-01 |          3 |
| C           | 2021-01-01 |          3 |
| C           | 2021-01-01 |          3 |
| C           | 2021-01-07 |          3 |
+-------------+------------+------------+

CREATE TABLE menu (
  product_id INTEGER,
  product_name VARCHAR(5),
  price INTEGER
);

INSERT INTO menu
  (product_id, product_name, price)
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');

SELECT *
FROM menu;
+------------+--------------+-------+
| product_id | product_name | price |
+------------+--------------+-------+
|          1 | sushi        |    10 |
|          2 | curry        |    15 |
|          3 | ramen        |    12 |
+------------+--------------+-------+

CREATE TABLE members (
  customer_id VARCHAR(1),
  join_date DATE
);

INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');

SELECT *
FROM members;
+-------------+------------+
| customer_id | join_date  |
+-------------+------------+
| A           | 2021-01-07 |
| B           | 2021-01-09 |
+-------------+------------+
```


**Case Study Questions**:

```
1. What is the total amount each customer spent at the restaurant? - DONE
2. How many days has each customer visited the restaurant? - DONE
3. What was the first item from the menu purchased by each customer? - DONE
4. What is the most purchased item on the menu and how many times was it purchased by all customers? - DONE
5. Which item was the most popular for each customer? - DONE
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi how many points do customer A and B have at the end of January?
```

**Solutions:**

```sql
-- 1. What is the total amount each customer spent at the restaurant?
mysql> SELECT S.customer_id AS customer, SUM(M.price) AS amount_spent
    -> FROM
    ->     sales AS S
    ->     INNER JOIN
    ->     menu AS M
    -> ON S.product_id = M.product_id
    -> GROUP BY S.customer_id
    -> ORDER BY S.customer_id;
+----------+--------------+
| customer | amount_spent |
+----------+--------------+
| A        |           76 |
| B        |           74 |
| C        |           36 |
+----------+--------------+
3 rows in set (0.01 sec)

-- 2. How many days has each customer visited the restaurant?
mysql> SELECT customer_id AS customer, COUNT(DISTINCT order_date) AS visited_days
    -> FROM sales
    -> GROUP BY customer_id
    -> ORDER BY customer_id;
+----------+--------------+
| customer | visited_days |
+----------+--------------+
| A        |            4 |
| B        |            6 |
| C        |            2 |
+----------+--------------+
3 rows in set (0.00 sec)

-- 3. What was the first item from the menu purchased by each customer?
mysql> SELECT DISTINCT A.customer_id AS customer, M.product_name AS item_name
    -> FROM
    ->     (
    ->         SELECT R.customer_id, R.product_id
    ->         FROM
    ->             (
    ->                 SELECT *,
    ->                        RANK() over (PARTITION BY customer_id ORDER BY order_date) AS rnk
    ->                 FROM sales
    ->             ) AS R
    ->         WHERE R.rnk = 1
    ->     ) AS A
    ->     INNER JOIN
    ->     menu AS M
    -> ON A.product_id = M.product_id
    -> ORDER BY A.customer_id;
+----------+-----------+
| customer | item_name |
+----------+-----------+
| A        | sushi     |
| A        | curry     |
| B        | curry     |
| C        | ramen     |
+----------+-----------+
4 rows in set (0.00 sec)

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
mysql> SELECT R.product_id, M.product_name AS product, R.sold_count AS purchase_count
    -> FROM
    ->     (
    ->         SELECT product_id, COUNT(product_id) AS sold_count
    ->         FROM sales
    ->         GROUP BY product_id
    ->         ORDER BY sold_count DESC
    ->         LIMIT 1
    ->     ) AS R
    ->     INNER JOIN
    ->     menu AS M
    -> ON R.product_id = M.product_id;
+------------+---------+----------------+
| product_id | product | purchase_count |
+------------+---------+----------------+
|          3 | ramen   |              8 |
+------------+---------+----------------+
1 row in set (0.00 sec)

-- 5. Which item was the most popular for each customer?
mysql> SELECT R.customer_id, M.product_name, R.bought
    -> FROM
    ->     (
    ->         SELECT B.customer_id, B.product_id, B.bought
    ->         FROM
    ->             (
    ->                 SELECT *,
    ->                        RANK() over (PARTITION BY A.customer_id ORDER BY A.bought DESC) AS most_bought
    ->                 FROM
    ->                     (
    ->                         SELECT customer_id, product_id, COUNT(product_id) AS bought
    ->                         FROM sales
    ->                         GROUP BY customer_id, product_id
    ->                     ) AS A
    ->             ) AS B
    ->         WHERE B.most_bought = 1
    ->     ) AS R
    ->     INNER JOIN
    ->     menu AS M
    -> ON R.product_id = M.product_id
    -> ORDER BY R.customer_id;
+-------------+--------------+--------+
| customer_id | product_name | bought |
+-------------+--------------+--------+
| A           | ramen        |      3 |
| B           | sushi        |      2 |
| B           | curry        |      2 |
| B           | ramen        |      2 |
| C           | ramen        |      3 |
+-------------+--------------+--------+
5 rows in set (0.00 sec)

-- 6. Which item was purchased first by the customer after they became a member?
mysql> SELECT A.customer_id, M.product_name
    -> FROM
    ->     (
    ->         SELECT R.product_id, R.customer_id
    ->         FROM
    ->             (
    ->                 SELECT S.customer_id, S.product_id, S.order_date,
    ->                        RANK() over (PARTITION BY S.customer_id ORDER BY S.order_date) AS rnk
    ->                 FROM sales AS S LEFT JOIN members AS M
    ->                 ON S.customer_id = M.customer_id
    ->                 WHERE S.order_date >= M.join_date
    ->             ) AS R
    ->         WHERE R.rnk = 1
    ->     ) AS A
    ->     INNER JOIN
    ->     menu AS M
    -> ON A.product_id = M.product_id
    -> ORDER BY A.customer_id;
+-------------+--------------+
| customer_id | product_name |
+-------------+--------------+
| A           | curry        |
| B           | sushi        |
+-------------+--------------+
2 rows in set (0.00 sec)

-- 7. Which item was purchased just before the customer became a member?
mysql> SELECT R.customer_id, M.product_name
    -> FROM
    ->     (
    ->         SELECT A.customer_id, A.order_date, A.product_id, A.rnk
    ->         FROM
    ->             (
    ->                 SELECT S.customer_id, S.order_date, S.product_id,
    ->                        RANK() over (PARTITION BY S.customer_id ORDER BY S.order_date DESC) AS rnk
    ->                 FROM sales AS S INNER JOIN members AS M
    ->                 ON S.customer_id = M.customer_id
    ->                 WHERE S.order_date < M.join_date
    ->             ) AS A
    ->         WHERE A.rnk = 1
    ->     ) AS R
    ->     INNER JOIN
    ->     menu AS M
    -> ON R.product_id = M.product_id
    -> ORDER BY R.customer_id;
+-------------+--------------+
| customer_id | product_name |
+-------------+--------------+
| A           | sushi        |
| A           | curry        |
| B           | sushi        |
+-------------+--------------+
3 rows in set (0.00 sec)

-- 8. What is the total items and amount spent for each member before they became a member?
mysql> SELECT R.customer_id AS customer, COUNT(R.product_id) AS item_count, SUM(M.price) AS amount_spent
    -> FROM
    ->     (
    ->         SELECT S.customer_id, S.product_id
    ->         FROM sales AS S INNER JOIN members AS M
    ->         ON S.customer_id = M.customer_id
    ->         WHERE S.order_date < M.join_date
    ->     ) AS R
    ->     INNER JOIN
    ->     menu AS M
    -> ON R.product_id = M.product_id
    -> GROUP BY R.customer_id
    -> ORDER BY customer;
+----------+------------+--------------+
| customer | item_count | amount_spent |
+----------+------------+--------------+
| A        |          2 |           25 |
| B        |          3 |           40 |
+----------+------------+--------------+
2 rows in set (0.01 sec)

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
mysql> WITH total_points_customers AS (
    ->     SELECT
    ->         S.customer_id, IF(M.product_name = 'sushi', M.price * 20, M.price * 10) AS points_earned
    ->     FROM sales AS S INNER JOIN menu AS M
    ->     ON S.product_id = M.product_id
    -> )
    -> SELECT customer_id, SUM(points_earned) AS total_points
    -> FROM total_points_customers
    -> GROUP BY customer_id
    -> ORDER BY customer_id;
+-------------+--------------+
| customer_id | total_points |
+-------------+--------------+
| A           |          860 |
| B           |          940 |
| C           |          360 |
+-------------+--------------+
3 rows in set (0.00 sec)

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi -
--     how many points do customer A and B have at the end of January?
mysql> WITH
    ->     subscribers AS (
    ->         SELECT S.customer_id, S.order_date, S.product_id, M.join_date
    ->         FROM sales AS S INNER JOIN members AS M
    ->         ON S.customer_id = M.customer_id
    ->     ),
    ->     subscriber_points AS (
    ->         SELECT
    ->             S.customer_id,
    ->             CASE
    ->                 WHEN S.order_date >= S.join_date AND S.order_date <= (S.join_date + 6) THEN M.price * 20
    ->                 WHEN M.product_name = 'sushi' THEN M.price * 20
    ->                 ELSE M.price * 10
    ->             END AS points
    ->         FROM subscribers AS S INNER JOIN menu AS M
    ->         ON S.product_id = M.product_id
    ->         WHERE S.order_date < '2021-02-01'
    ->     )
    -> SELECT customer_id, SUM(points) AS points_offers
    -> FROM subscriber_points
    -> GROUP BY customer_id
    -> ORDER BY customer_id;
+-------------+---------------+
| customer_id | points_offers |
+-------------+---------------+
| A           |          1370 |
| B           |           820 |
+-------------+---------------+
2 rows in set (0.00 sec)
```
