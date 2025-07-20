# 🍜 Case Study #1: Danny's Diner 
<img width="1080" height="1080" alt="image" src="https://github.com/user-attachments/assets/6b35c29d-59c8-4967-8a69-3d75cb8413ee" />

## 📚 Table of Contents
1.Summary of the Problem Statement (Danny’s Diner)<br>
2.Entity Relationship Diagram<br>
3.Question and Solution

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/).

***

## 🧠 Summary of the Problem Statement (Danny’s Diner)
Danny wants to analyze his restaurant’s customer data to gain insights into customer visit patterns, spending habits, and favorite menu items. These insights will help him improve customer experience and decide whether to expand his loyalty program. He also needs help preparing easy-to-use datasets for non-SQL users. Danny has shared sample data from three tables: sales, menu, and members for this analysis.

***

## Entity Relationship Diagram
<img width="1054" height="507" alt="image" src="https://github.com/user-attachments/assets/02504384-1843-4848-80d9-489560b8980b" />


***

## Question and Solution
**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT 
    s.customer_id, 
    SUM(m.price) AS total_sales
FROM 
    dannys_diner.sales s
JOIN 
    dannys_diner.menu m 
    ON s.product_id = m.product_id
GROUP BY 
    s.customer_id
ORDER BY 
    s.customer_id ASC;
````


  #### 🧩 Basic Steps
- Join sales and menu tables on product_id to include price details.
- Select customer_id and calculate total using SUM(m.price).
- Group records by customer_id to aggregate totals per customer.

  #### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

**2. How many days has each customer visited the restaurant?**
````sql
SELECT 
  customer_id, 
  COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id;
````
#### 🧩 Basic Steps
- Select customer_id and count the number of distinct visit dates using COUNT(DISTINCT order_date).
- This gives the number of unique days each customer visited.
- Use GROUP BY customer_id to calculate this count for each customer separately.

  #### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**3. What was the first item from the menu purchased by each customer?**
````sql
WITH first_item_purchased AS (
  SELECT 
    s.customer_id, 
    s.order_date, 
    m.product_name, 
    RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS first_item
  FROM dannys_diner.sales s
  JOIN dannys_diner.menu m 
    ON s.product_id = m.product_id
  GROUP BY s.customer_id, s.order_date, m.product_name
)
SELECT 
  customer_id, 
  product_name 
FROM first_item_purchased
WHERE first_item = 1;
````

#### 🧩 Basic Steps
- Create a CTE to identify the first item(s) each customer purchased.
- Join sales and menu tables to get the product name.
- Use RANK() to order purchases by date for each customer.
- Filter rows where the rank is 1 to get the first purchase(s).
- Select customer_id and product_name as final output.

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

  ***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
````sql
SELECT 
  COUNT(s.product_id) AS items_purchased, 
  m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.menu m 
  ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY items_purchased DESC
LIMIT 1;
````

#### 🧩 Basic Steps
- Join sales and menu tables using product_id to get product names for each sale.
- Count how many times each product was sold using COUNT(s.product_id).
- Group results by product_name to get item-wise sales count.
- Order the results in descending order of items_purchased.
- Use LIMIT 1 to get the most purchased menu item.

  #### Answer:
| items_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


- Most purchased item on the menu is ramen which is 8 times.

  ***

**5. Which item was the most popular for each customer?**
````sql
with most_popular_item as(
Select s.customer_id, m.product_name,  COUNT(m.product_id) AS order_count,
rank()over(partition by customer_id order by count(s.customer_id) desc) as  most_popular
from dannys_diner.sales s 
join dannys_diner.menu m on s.product_id = m.product_id
group by s.customer_id, m.product_name
order by customer_id,most_popular desc)
select customer_id, product_name, order_count
from most_popular_item
where most_popular = 1
````

#### 🧩 Basic Steps
Join sales and menu tables using product_id.
Group the data by both customer_id and product_name.
Count how many times each customer ordered each product (order_count).
Use RANK() to assign a popularity rank per customer, sorted by descending order count.
Filter only those rows where the rank is 1, indicating each customer’s most frequently ordered item.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu.

***

**6. Which item was purchased first by the customer after they became a member?**
````sql
WITH first_item_purchased AS (
  SELECT 
    s.customer_id, 
    s.order_date, 
    m.join_date, 
    mn.product_name,
    RANK() OVER (
      PARTITION BY s.customer_id 
      ORDER BY s.order_date ASC
    ) AS rank
  FROM dannys_diner.sales s
  JOIN dannys_diner.members m 
    ON s.customer_id = m.customer_id
  JOIN dannys_diner.menu mn 
    ON s.product_id = mn.product_id
  WHERE m.join_date < s.order_date
  GROUP BY s.customer_id, s.order_date, m.join_date, mn.product_name
)
SELECT 
  customer_id, 
  product_name
FROM first_item_purchased
WHERE rank = 1;
````
#### 🧩 Basic Steps
- Join sales, members, and menu tables using appropriate keys.
- Filter only those purchases that occurred after a customer became a member.
- Use RANK() to assign a rank to each product ordered after membership, sorted by earliest order_date for each customer.
- Select only the products where rank = 1, i.e., the first item purchased after becoming a member.
- Return the customer_id and the corresponding product_name.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi.

  ***

  **7. Which item was purchased just before the customer became a member?**
````sql
  WITH first_item_purchased AS (
  SELECT 
    s.customer_id, 
    s.order_date, 
    m.join_date, 
    mn.product_name,
    RANK() OVER (
      PARTITION BY s.customer_id 
      ORDER BY s.order_date DESC
    ) AS rank
  FROM dannys_diner.sales s
  JOIN dannys_diner.members m 
    ON s.customer_id = m.customer_id
  JOIN dannys_diner.menu mn 
    ON s.product_id = mn.product_id
  WHERE m.join_date > s.order_date
  GROUP BY 
    s.customer_id, 
    m.customer_id, 
    s.order_date, 
    m.join_date, 
    mn.product_name
)
SELECT 
  customer_id, 
  product_name
FROM first_item_purchased
WHERE rank = 1;
````

#### 🧩 Basic Steps
- Joins sales, members, and menu tables to associate customer purchases with membership details and product names.
- Filters for transactions that happened before a customer became a member (join_date > order_date).
- Uses RANK() to rank the purchases in reverse order (latest first) before membership for each customer.
- Retrieves the most recent item purchased before membership activation (rank = 1).
- Final output shows customer_id and product_name.

  #### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

- Both customers' last order before becoming members are sushi.However A has ordered "curry" on the same date.

***

**8. What is the total items and amount spent for each member before they became a member?**
````sql
WITH total_item_amount AS (
  SELECT 
    s.customer_id, 
    m.product_name,  
    SUM(m.price) AS price, 
    mn.join_date,  
    s.order_date
  FROM dannys_diner.sales s 
  JOIN dannys_diner.menu m 
    ON s.product_id = m.product_id
  JOIN dannys_diner.members mn 
    ON s.customer_id = mn.customer_id
  WHERE mn.join_date > s.order_date
  GROUP BY 
    s.customer_id, 
    m.product_name, 
    mn.join_date,  
    s.order_date
  ORDER BY s.customer_id ASC
)
SELECT 
  customer_id, 
  COUNT(product_name) AS total_items, 
  SUM(price) AS amount_spent
FROM total_item_amount
GROUP BY customer_id;
````
#### 🧩 Basic Steps
- CTE (total_item_amount): Joins sales, menu, and members tables.
- Filters records before the customer became a member (join_date > order_date).
- Counts how many distinct items the customer bought (COUNT(product_name)).
- Sums the total amount spent on those items (SUM(price)).
- Groups result by customer to get summary per user.

#### Answer:
| customer_id | total_items | amount_spent |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
SELECT 
  s.customer_id,
  SUM(
    CASE 
      WHEN product_name = 'sushi' THEN m.price * 20
      ELSE m.price * 10 
    END
  ) AS total_points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m 
  ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
#### 🧩 Basic Steps
- Goal: Calculate total loyalty points per customer.
- Points logic:
-     If the item is sushi, award 20 points per price unit.
-     For all other items, award 10 points per price unit.
- Uses a CASE statement inside SUM() to apply the logic per product.
- Joins sales with menu to access product_name and price.
- Groups by customer_id to get a per-customer summary.
- Orders the final output by customer_id in ascending order.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

```sql
WITH dates_cte AS (
  SELECT 
    customer_id, 
      join_date, 
      join_date + 6 AS valid_date, 
      DATE_TRUNC(
        'month', '2021-01-31'::DATE)
        + interval '1 month' 
        - interval '1 day' AS last_date
  FROM dannys_diner.members
)

SELECT 
  sales.customer_id, 
  SUM(CASE
    WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
    WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price
    ELSE 10 * menu.price END) AS points
FROM dannys_diner.sales
INNER JOIN dates_cte AS dates
  ON sales.customer_id = dates.customer_id
  AND dates.join_date <= sales.order_date
  AND sales.order_date <= dates.last_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id;
```

#### Steps:
- A CTE is used to calculate each member’s 7-day loyalty window by adding 6 days to their join date.
- The query joins the sales, menu, and the CTE (members) tables using customer_id and product_id.
- It filters orders to include only those placed between the customer’s join date and 2021-01-31.
- Points are assigned based on the purchase date and item:
    - If the purchase is within the 7-day program window: price × 10 × 2.
    - If the purchase is outside the program window and the item is ‘sushi’: price × 10 × 2.
    - If the purchase is outside the window and not sushi: price × 10.
- The total points are summed per customer.
- The final output lists each customer_id with their customer_points, sorted by customer_id.


#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1020 |
| B           | 320 |

- Total points for Customer A is 1,020.
- Total points for Customer B is 320.

### Thank you!






