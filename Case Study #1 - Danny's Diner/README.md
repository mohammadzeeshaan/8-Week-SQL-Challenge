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

<img width="1080" height="525" alt="image" src="https://github.com/user-attachments/assets/c9666d87-bbb0-4529-9bef-c6556af06459" />

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
