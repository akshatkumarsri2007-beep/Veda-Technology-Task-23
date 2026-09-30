#  SQL Window Functions on AdventureWorks Sales Data
# Nmae -** Akshat Srivastava**

**ROW_NUMBER · RANK · DENSE_RANK · LAG · Running Totals · Moving Averages**

*Task 23 · Level 2 · Data Analytics Internship at **Veda Technology***

---

##  Overview

This project explores **SQL window functions** to answer real business questions on an AdventureWorks-style sales dataset: who are the top customers, which products lead revenue, how fast is the business growing, and how often do customers come back?

Window functions calculate across related rows **without collapsing them** like `GROUP BY` does, which makes rankings, trends and running totals much simpler than self-joins or subqueries.

##  Objectives

- Use `ROW_NUMBER`, `RANK` and `LAG` to solve business questions
- Understand the difference between `ROW_NUMBER`, `RANK` and `DENSE_RANK`
- Calculate month-over-month and year-over-year growth
- Build running totals and moving averages with window frames
- Write clean, readable analytical SQL

##  Dataset

A simplified, synthetic AdventureWorks-style dataset (8 tables joined into one flat table called `sales`).

| Item | Details |
|---|---|
| Order lines | 11,124 |
| Orders | 4,480 |
| Period | Jan 2024 to Sep 2026 |
| Customers | 500 |
| Products | 38 across 4 categories |
| Territories | 10 |
| Salespeople | 10 |

**Main columns:** `SalesOrderID`, `OrderDate`, `CustomerName`, `City`, `TerritoryName`, `SalesPersonName`, `SalesQuota`, `ProductName`, `Category`, `Subcategory`, `OrderQty`, `UnitPrice`, `LineTotal`, `TotalDue`

>  **Note:** The data is synthetic, so figures such as quota achievement are illustrative and not real business results.

##  Tech Stack

- **SQL** (SQLite 3.25+ for window function support)
- **Python** with **pandas** and **sqlite3**
- **Google Colab** as the working environment
  
##  Repository Structure

```
├── data/
│   └── AdventureWorks_All_In_One.csv
├── notebook/
│   └── Task23_SQL_Window_Functions.ipynb
├── report/
│   └── Task23_SQL_Window_Functions_Report.pdf
└── README.md
```

##  Functions Covered

| Function | What it does | Business use |
|---|---|---|
| `ROW_NUMBER()` | Unique number for every row | Top N per group, first order per customer |
| `RANK()` | Same rank for ties, next rank skipped (1, 1, 3) | Ranking products, customers, salespeople |
| `DENSE_RANK()` | Same rank for ties, no gaps (1, 1, 2) | Ranking without gaps |
| `LAG()` | Reads the previous row's value | MoM / YoY growth, gap between orders |
| `SUM() OVER()` | Running total | Cumulative revenue |
| `AVG() OVER()` | Moving average with a window frame | Smoothing monthly trends |
| `PARTITION BY` | Restarts calculation for each group | Per customer, category or territory |

##  The 12 Queries

| # | Business question | Function |
|---|---|---|
| 1 | Number each customer's orders in date order | `ROW_NUMBER` |
| 2 | Top 3 products in every category | `ROW_NUMBER` |
| 3 | Rank products by total revenue | `RANK` |
| 4 | Compare the three ranking functions on tied order counts | `ROW_NUMBER` / `RANK` / `DENSE_RANK` |
| 5 | Top 3 customers in each territory | `RANK` + `PARTITION BY` |
| 6 | Rank salespeople and show quota achievement | `RANK` |
| 7 | Month-over-month revenue growth | `LAG` |
| 8 | Days between a customer's consecutive orders | `LAG` + `PARTITION BY` |
| 9 | Year-over-year growth by category | `LAG` + `PARTITION BY` |
| 10 | Cumulative revenue over time | `SUM() OVER()` |
| 11 | 3-month moving average of revenue | `AVG() OVER()` |
| 12 | First order of every customer | `ROW_NUMBER = 1` |

### Example: Month-over-Month Growth

```sql
WITH monthly AS (
  SELECT SUBSTR(OrderDate,1,7) AS month, ROUND(SUM(LineTotal),2) AS revenue
  FROM sales GROUP BY 1
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
       ROUND((revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0
             / LAG(revenue) OVER (ORDER BY month), 2) AS mom_growth_pct
FROM monthly
ORDER BY month;
```

##  Key Findings

-  **Top product:** Road-750 led revenue with about **$2.35M**, closely followed by Mountain-300 (about $2.30M).
-  **Top salesperson:** Jose Carson ranked first with about **$2.12M** in sales, followed by Tete Ansman-Wolfe (about $2.03M).
-  **Volatile months:** Monthly revenue swings noticeably (for example, Feb 2024 dropped about 29.8% from Jan 2024). The 3-month moving average reveals the underlying trend.
-  **Ranking ties:** Several customers share 20 orders. `ROW_NUMBER` split them arbitrarily, while `RANK` and `DENSE_RANK` gave them the same rank.
-  **Category growth:** `LAG` made year-over-year comparison easy. For example, Accessories grew about 6.8% from 2024 to 2025.

##  Challenges and Notes

- The flat table repeats order-level values (`SubTotal`, `TotalDue`) on every line of the same order. I used `SELECT DISTINCT` for order-level analysis and `LineTotal` for revenue to avoid **double counting**.
- `LAG` returns `NULL` for the first row of each group because no earlier row exists.
- 2026 data covers January to September only, so yearly comparisons for 2026 are partial.

##  What I Learned

- When to use `ROW_NUMBER` vs `RANK` vs `DENSE_RANK`
- How `PARTITION BY` restarts a calculation for each group
- How to use `LAG` to compare a row with its previous row
- How window frames (`ROWS BETWEEN ... AND ...`) control moving averages
- How to avoid double counting when joining tables into a flat dataset

##  Full Report

A detailed PDF report with charts, query outputs and conclusions is available in the `report/` folder.
