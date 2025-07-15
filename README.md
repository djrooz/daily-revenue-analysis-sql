# daily-revenue-analysis-sql

# 📊 Daily Revenue Analysis using SQL

This project demonstrates how to calculate and analyze daily revenue for a delivery service using SQL. It includes:

- Daily revenue computation
- Absolute revenue growth (vs. previous day)
- Percentage revenue growth (vs. previous day)

The analysis is performed with **PostgreSQL** using **CTEs** and **window functions**.

---

## 🧩 Data Structure

We used the following tables:

- `orders`: contains order IDs, product IDs (array), and creation time.
- `products`: contains product prices.
- `courier_actions`: used to filter only successfully delivered orders.

---

## 🛠️ Tools & Techniques

- SQL (PostgreSQL)
- Common Table Expressions (CTE)
- `UNNEST()` for arrays
- Window functions: `LAG()`, `ROUND()`, `COALESCE()`

---

## 🧮 SQL Query

```sql
WITH cte AS (
    SELECT 
        order_id,
        creation_time::date AS date,
        unnest(product_ids) AS product_id
    FROM orders
),
cte2 AS (
    SELECT 
        date,
        ROUND(SUM(price), 1) AS daily_revenue
    FROM cte c
    JOIN products p ON c.product_id = p.product_id
    WHERE c.order_id IN (
        SELECT order_id
        FROM courier_actions
        WHERE action = 'deliver_order'
    )
    GROUP BY 1
    ORDER BY 1
)
SELECT 
    date,
    daily_revenue,
    COALESCE(ROUND(daily_revenue - LAG(daily_revenue) OVER (ORDER BY date), 1), 0) AS revenue_growth_abs,
    COALESCE(
        ROUND((daily_revenue - LAG(daily_revenue) OVER (ORDER BY date)) * 100.0 
              / LAG(daily_revenue) OVER (ORDER BY date), 1),
        0
    ) AS revenue_growth_percentage
FROM cte2;
