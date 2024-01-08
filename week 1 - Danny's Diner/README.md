# Week 1 - Danny’s Diner

Dataset Code:

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
```

Solutions:

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

```
