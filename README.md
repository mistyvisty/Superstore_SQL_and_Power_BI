# 🛒 Superstore Sales Analysis — SQL + Power BI

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mistyvisty/Superstore_SQL_and_Power_BI/blob/main/Superstore_Sales_SQL_Analysis.ipynb)

![SQL](https://img.shields.io/badge/SQL-Basic_→_Window_Functions-4479A1?style=flat-square)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat-square&logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)

A retail analysis of **9,994 order lines (5,009 orders, 793 customers, 2014–2017)** from a US superstore. I wrote **14 SQL queries**, from basic aggregations up to **CTEs and window functions**, to answer real business questions, then built a **Power BI dashboard** for stakeholders.

**The headline finding:** every one of the **933 order lines discounted above 40% lost money**, and order lines discounted above 20% lost **\$135,376** in total.

---

## 🎯 Business Problem

The store runs discounts across every product category. Are they paying off? Which regions, products and customers actually make money, and which only *look* valuable because of high sales?

---

## 🔍 Key Findings

| # | Finding | Evidence |
|---|---|---|
| 1 | **Discounts above 20% destroy profit.** Every line above 40% discount lost money (933 of 933) | Q10 |
| 2 | **Furniture barely breaks even:** \$742K sales but only 2.5% margin; **Tables lost \$17,725** | Q3, Q6 |
| 3 | **5 of the top 10 states by sales lose money**; Texas, Ohio and Pennsylvania alone lost **\$58,261** | Q9 |
| 4 | **The #1 customer by revenue loses money:** Sean Miller spent \$25,043 at a **–\$1,981** profit | Q4, Q12 |
| 5 | **Central has the worst margin (7.9%)** despite \$501K in sales; West leads at 14.9% | Q2 |
| 6 | **Revenue grew 51% (2014–2017)** but dipped 2.8% in 2015; profit grew every year (+89%) | Q7, Q15 |
| 7 | **Customer loyalty is strong and rising:** 87% of 2014's customers were still buying in 2017 | Q13 |
| 8 | **Technology is the best category** (17.4% margin); Copiers earn a 37.2% margin | Q3, Q5 |

---

## 🧮 SQL Skills Covered

| Level | Concepts | Queries |
|---|---|---|
| 🟢 **Basic** | `SELECT`, `SUM`, `COUNT`, `AVG`, `ROUND`, `COUNT(DISTINCT)` | Q1 |
| 🟢 **Basic** | `GROUP BY`, `ORDER BY`, `LIMIT` | Q2, Q3, Q4, Q5, Q8, Q9, Q11 |
| 🟡 **Intermediate** | `HAVING` (filtering groups) | Q6 |
| 🟡 **Intermediate** | `CASE WHEN` (custom buckets) | Q10 |
| 🟡 **Intermediate** | Date functions: `strftime`, `julianday` | Q7, Q13, Q14 |
| 🔴 **Advanced** | CTEs (`WITH`), including multiple CTEs | Q12, Q13, Q15 |
| 🔴 **Advanced** | `JOIN` between CTEs (cohort analysis) | Q13 |
| 🔴 **Advanced** | Window functions: `RANK() OVER (PARTITION BY …)` | Q12 |
| 🔴 **Advanced** | Window functions: running total `SUM() OVER`, `LAG()` | Q15 |

---

## 🗄️ The Queries

Each query answers one business question. Click a query to see the SQL and its result.

### 🟢 Basic

<details>
<summary><b>Q1 — What is the overall business performance?</b> · <code>SUM</code> <code>COUNT DISTINCT</code> <code>AVG</code></summary>

```sql
SELECT
    ROUND(SUM(Sales), 2)                    AS Total_Revenue,
    ROUND(SUM(Profit), 2)                   AS Total_Profit,
    ROUND(SUM(Profit) / SUM(Sales) * 100, 1) AS Profit_Margin_Pct,
    COUNT(DISTINCT "Order ID")              AS Total_Orders,
    COUNT(DISTINCT "Customer ID")           AS Total_Customers,
    ROUND(SUM(Sales) / COUNT(DISTINCT "Order ID"), 2) AS Avg_Order_Value
FROM orders;
```

| Revenue | Profit | Margin | Orders | Customers | Avg order value |
|---|---|---|---|---|---|
| \$2,297,201 | \$286,397 | 12.5% | 5,009 | 793 | \$458.61 |

</details>

<details>
<summary><b>Q2 — Which region generates the most sales and profit?</b> · <code>GROUP BY</code> <code>ORDER BY</code></summary>

```sql
SELECT
    Region,
    ROUND(SUM(Sales), 2)                    AS Total_Sales,
    ROUND(SUM(Profit), 2)                   AS Total_Profit,
    ROUND(SUM(Profit) / SUM(Sales) * 100, 1) AS Profit_Margin_Pct,
    COUNT(DISTINCT "Order ID")              AS Total_Orders
FROM orders
GROUP BY Region
ORDER BY Total_Sales DESC;
```

| Region | Sales | Profit | Margin |
|---|---|---|---|
| West | \$725,458 | \$108,418 | **14.9%** |
| East | \$678,781 | \$91,523 | 13.5% |
| Central | \$501,240 | \$39,706 | **7.9%** ⚠️ |
| South | \$391,722 | \$46,749 | 11.9% |

</details>

<details>
<summary><b>Q3 — Which product categories are most and least profitable?</b> · multiple aggregations</summary>

```sql
SELECT
    Category,
    ROUND(SUM(Sales), 2)                    AS Total_Sales,
    ROUND(SUM(Profit), 2)                   AS Total_Profit,
    ROUND(SUM(Profit) / SUM(Sales) * 100, 1) AS Profit_Margin_Pct,
    ROUND(AVG(Discount) * 100, 1)           AS Avg_Discount_Pct,
    SUM(Quantity)                           AS Total_Units_Sold
FROM orders
GROUP BY Category
ORDER BY Total_Sales DESC;
```

| Category | Sales | Profit | Margin | Avg discount |
|---|---|---|---|---|
| Technology | \$836,154 | \$145,455 | **17.4%** | 13.2% |
| Furniture | \$742,000 | \$18,451 | **2.5%** ⚠️ | 17.4% |
| Office Supplies | \$719,047 | \$122,491 | 17.0% | 15.7% |

</details>

<details>
<summary><b>Q4 — Who are the top 10 customers, and are they profitable?</b> · <code>GROUP BY</code> multiple columns · <code>LIMIT</code></summary>

```sql
SELECT
    "Customer Name",
    Segment,
    ROUND(SUM(Sales), 2)        AS Total_Spent,
    ROUND(SUM(Profit), 2)       AS Profit_Generated,
    COUNT(DISTINCT "Order ID")  AS Total_Orders
FROM orders
GROUP BY "Customer ID", "Customer Name", Segment
ORDER BY Total_Spent DESC
LIMIT 10;
```

| Customer | Segment | Spent | Profit |
|---|---|---|---|
| **Sean Miller** | Home Office | \$25,043 | **–\$1,981** ❌ |
| Tamara Chand | Corporate | \$19,052 | \$8,981 |
| Raymond Buch | Consumer | \$15,117 | \$6,976 |
| … | | | |

**Revenue isn't profit:** the biggest spender is unprofitable.

</details>

<details>
<summary><b>Q5 — Which sub-categories make the most money?</b> · <code>GROUP BY</code> two columns</summary>

```sql
SELECT
    Category,
    "Sub-Category",
    ROUND(SUM(Sales), 2)                    AS Total_Sales,
    ROUND(SUM(Profit), 2)                   AS Total_Profit,
    ROUND(SUM(Profit) / SUM(Sales) * 100, 1) AS Profit_Margin_Pct
FROM orders
GROUP BY Category, "Sub-Category"
ORDER BY Total_Profit DESC
LIMIT 10;
```

Top 3: **Copiers** (\$55,618, 37.2% margin), **Phones** (\$44,516), **Accessories** (\$41,937). **Paper** has the best margin of all at 43.4%.

</details>

<details>
<summary><b>Q8 — Which customer segment is most valuable?</b> · <code>GROUP BY</code> · <code>COUNT DISTINCT</code></summary>

```sql
SELECT
    Segment,
    COUNT(DISTINCT "Customer ID")           AS Total_Customers,
    ROUND(SUM(Sales), 2)                    AS Total_Sales,
    ROUND(SUM(Profit), 2)                   AS Total_Profit,
    ROUND(SUM(Profit) / SUM(Sales) * 100, 1) AS Profit_Margin_Pct
FROM orders
GROUP BY Segment
ORDER BY Total_Sales DESC;
```

| Segment | Customers | Sales | Margin |
|---|---|---|---|
| Consumer | 409 | \$1,161,401 | 11.5% |
| Corporate | 236 | \$706,146 | 13.0% |
| Home Office | 148 | \$429,653 | **14.0%** |

</details>

<details>
<summary><b>Q9 — Which states perform well vs poorly?</b> · <code>GROUP BY</code> · <code>LIMIT</code></summary>

```sql
SELECT
    State,
    Region,
    ROUND(SUM(Sales), 2)                    AS Total_Sales,
    ROUND(SUM(Profit), 2)                   AS Total_Profit,
    ROUND(SUM(Profit) / SUM(Sales) * 100, 1) AS Profit_Margin_Pct
FROM orders
GROUP BY State, Region
ORDER BY Total_Sales DESC
LIMIT 10;
```

**5 of the top 10 states lose money:**

| State | Sales | Profit | Margin |
|---|---|---|---|
| Texas | \$170,188 | **–\$25,729** | –15.1% |
| Ohio | \$78,258 | **–\$16,971** | –21.7% |
| Pennsylvania | \$116,512 | **–\$15,560** | –13.4% |
| Illinois | \$80,166 | **–\$12,608** | –15.7% |
| Florida | \$89,474 | **–\$3,399** | –3.8% |

</details>

<details>
<summary><b>Q11 — Which shipping mode do customers prefer?</b> · <code>GROUP BY</code></summary>

```sql
SELECT
    "Ship Mode",
    COUNT(DISTINCT "Order ID")  AS Total_Orders,
    ROUND(SUM(Sales), 2)        AS Total_Sales,
    ROUND(SUM(Profit), 2)       AS Total_Profit
FROM orders
GROUP BY "Ship Mode"
ORDER BY Total_Orders DESC;
```

**Standard Class dominates** with 2,994 of 5,009 orders (60%). All four modes are profitable.

</details>

### 🟡 Intermediate

<details>
<summary><b>Q6 — Which sub-categories lose money?</b> · <code>HAVING</code></summary>

```sql
SELECT
    Category,
    "Sub-Category",
    ROUND(SUM(Sales), 2)          AS Total_Sales,
    ROUND(SUM(Profit), 2)         AS Total_Profit,
    ROUND(AVG(Discount) * 100, 1) AS Avg_Discount_Pct
FROM orders
GROUP BY Category, "Sub-Category"
HAVING SUM(Profit) < 0
ORDER BY Total_Profit ASC;
```

| Sub-category | Sales | Profit | Avg discount |
|---|---|---|---|
| **Tables** | \$206,966 | **–\$17,725** | 26.1% |
| **Bookcases** | \$114,880 | **–\$3,473** | 21.1% |
| Supplies | \$46,674 | –\$1,189 | 7.7% |

`HAVING` filters *after* grouping, which `WHERE` can't do.

</details>

<details>
<summary><b>Q7 — Is the business growing year over year?</b> · <code>strftime</code> date function</summary>

```sql
SELECT
    strftime('%Y', "Order Date")            AS Year,
    ROUND(SUM(Sales), 2)                    AS Total_Sales,
    ROUND(SUM(Profit), 2)                   AS Total_Profit,
    ROUND(SUM(Profit) / SUM(Sales) * 100, 1) AS Profit_Margin_Pct,
    COUNT(DISTINCT "Order ID")              AS Total_Orders
FROM orders
GROUP BY Year
ORDER BY Year;
```

| Year | Sales | Profit | Orders |
|---|---|---|---|
| 2014 | \$484,248 | \$49,544 | 969 |
| 2015 | \$470,533 | \$61,619 | 1,038 |
| 2016 | \$609,206 | \$81,795 | 1,315 |
| 2017 | \$733,215 | \$93,439 | 1,687 |

</details>

<details>
<summary><b>Q10 — At what discount level do we start losing money?</b> ⭐ · <code>CASE WHEN</code></summary>

```sql
SELECT
    CASE
        WHEN Discount = 0     THEN '1. 0% — No Discount'
        WHEN Discount <= 0.10 THEN '2. 1-10% — Low'
        WHEN Discount <= 0.20 THEN '3. 11-20% — Medium'
        WHEN Discount <= 0.30 THEN '4. 21-30% — High'
        ELSE                       '5. 31%+ — Very High'
    END                     AS Discount_Range,
    COUNT(*)                AS Order_Lines,
    ROUND(SUM(Sales), 2)    AS Total_Sales,
    ROUND(SUM(Profit), 2)   AS Total_Profit,
    ROUND(AVG(Profit), 2)   AS Avg_Profit_Per_Line
FROM orders
GROUP BY Discount_Range
ORDER BY Discount_Range;
```

| Discount | Order lines | Total profit | Avg profit / line |
|---|---|---|---|
| 0% | 4,798 | \$320,988 | **+\$66.90** |
| 1–10% | 94 | \$9,029 | **+\$96.06** |
| 11–20% | 3,709 | \$91,756 | +\$24.74 |
| 21–30% | 227 | –\$10,369 | **–\$45.68** ❌ |
| 31%+ | 1,166 | **–\$125,007** | **–\$107.21** ❌ |

**This is the most important query in the project.** Profit turns negative above 20% discount, and every single line above 40% loses money.

</details>

<details>
<summary><b>Q14 — Which region and segment ship slowest?</b> · <code>julianday</code> date arithmetic</summary>

```sql
SELECT
    Region,
    Segment,
    ROUND(AVG(julianday("Ship Date") - julianday("Order Date")), 1) AS Avg_Ship_Days,
    COUNT(DISTINCT "Order ID") AS Total_Orders
FROM orders
GROUP BY Region, Segment
ORDER BY Avg_Ship_Days DESC;
```

Average shipping time is **4.0 days**, and it barely varies by region or segment (slowest: Central Home Office, 4.1 days). **Ship mode is what drives delivery time:** Same Day 0 days, First Class 2.2, Second Class 3.2, Standard 5.0.

</details>

### 🔴 Advanced

<details>
<summary><b>Q12 — Who are the top 3 customers in each segment?</b> ⭐ · CTE · <code>RANK() OVER (PARTITION BY)</code></summary>

```sql
WITH customer_sales AS (
    -- Step 1: total sales and profit per customer
    SELECT
        "Customer Name",
        Segment,
        ROUND(SUM(Sales), 2)  AS Total_Sales,
        ROUND(SUM(Profit), 2) AS Total_Profit
    FROM orders
    GROUP BY "Customer ID", "Customer Name", Segment
),
ranked AS (
    -- Step 2: rank customers within each segment
    SELECT
        *,
        RANK() OVER (PARTITION BY Segment ORDER BY Total_Sales DESC) AS Rank_In_Segment
    FROM customer_sales
)
-- Step 3: keep the top 3 per segment
SELECT *
FROM ranked
WHERE Rank_In_Segment <= 3
ORDER BY Segment, Rank_In_Segment;
```

| Segment | #1 | #2 | #3 |
|---|---|---|---|
| Consumer | Raymond Buch (\$15,117) | Adrian Barton (\$14,474) | Ken Lonsdale (\$14,175) |
| Corporate | Tamara Chand (\$19,052) | Todd Sumrall (\$11,892) | Bill Shonely (\$10,502) |
| Home Office | Sean Miller (\$25,043, **–\$1,981 profit**) | Tom Ashbrook (\$14,596) | Maria Etezadi (\$10,664) |

`PARTITION BY` ranks customers **within each segment in one query**, instead of writing three separate queries.

</details>

<details>
<summary><b>Q13 — How loyal are customers? (2014 cohort retention)</b> · multiple CTEs · <code>JOIN</code></summary>

```sql
WITH first_order AS (
    SELECT "Customer ID", MIN(strftime('%Y', "Order Date")) AS first_year
    FROM orders
    GROUP BY "Customer ID"
),
yearly_orders AS (
    SELECT DISTINCT "Customer ID", strftime('%Y', "Order Date") AS order_year
    FROM orders
)
SELECT
    f.first_year  AS Cohort_Year,
    y.order_year  AS Active_In_Year,
    COUNT(DISTINCT y."Customer ID") AS Active_Customers
FROM first_order f
JOIN yearly_orders y ON f."Customer ID" = y."Customer ID"
WHERE f.first_year = '2014'
GROUP BY f.first_year, y.order_year
ORDER BY y.order_year;
```

| Year | Active 2014-cohort customers | Retention |
|---|---|---|
| 2014 | 595 | 100% |
| 2015 | 437 | 73.4% |
| 2016 | 485 | 81.5% |
| 2017 | 517 | **86.9%** |

**Retention dipped in 2015, then recovered every year:** 87% of first-year customers were still buying three years later.

</details>

<details>
<summary><b>Q15 — Running revenue total and year-over-year growth</b> ⭐ · CTE · <code>SUM() OVER</code> · <code>LAG()</code></summary>

```sql
WITH yearly AS (
    SELECT
        strftime('%Y', "Order Date") AS Year,
        ROUND(SUM(Sales), 2)         AS Total_Sales
    FROM orders
    GROUP BY Year
)
SELECT
    Year,
    Total_Sales,
    ROUND(SUM(Total_Sales) OVER (ORDER BY Year), 2) AS Running_Total,
    ROUND(
        (Total_Sales - LAG(Total_Sales) OVER (ORDER BY Year)) * 100.0
        / LAG(Total_Sales) OVER (ORDER BY Year), 1
    ) AS YoY_Growth_Pct
FROM yearly
ORDER BY Year;
```

| Year | Sales | Running total | YoY growth |
|---|---|---|---|
| 2014 | \$484,248 | \$484,248 | — |
| 2015 | \$470,533 | \$954,780 | **–2.8%** |
| 2016 | \$609,206 | \$1,563,986 | +29.5% |
| 2017 | \$733,215 | \$2,297,201 | +20.4% |

`LAG()` looks at the previous row, so growth is calculated in SQL instead of by hand. It also reveals the **2015 dip** that total growth figures hide.

</details>

---

## 📊 Power BI Dashboard

![Superstore Dashboard](Superstore_PowerBI_Dashboard.png)

- KPI cards: total revenue, total profit, total orders
- Year slicer (2014–2017)
- Sales by region, sales by category, profit by sub-category
- Yearly sales trend
- Sales by state (map)

---

## 🎯 Business Recommendations

1. **Cap discounts at 20%.** Order lines discounted above 20% lost \$135,376 over four years, and every line above 40% lost money.
2. **Fix Furniture pricing, starting with Tables and Bookcases.** Together they lost \$21,198, with average discounts of 26% and 21%.
3. **Audit discounting in Texas, Ohio and Pennsylvania.** These three states lost \$58,261 combined.
4. **Review unprofitable high-value accounts.** The top customer by revenue, Sean Miller, generated a \$1,981 loss.
5. **Invest in Technology**, the highest-margin category (17.4%), especially Copiers (37.2%).

---

## ⚠️ Limitations

- **Sample Superstore is a public teaching dataset**, so patterns may not reflect a real retailer
- "Losses" are **historical**: capping discounts may also reduce sales volume, so the full \$135K wouldn't be recovered
- Discount effects are **associations**; discounted items may differ from undiscounted ones in other ways

---

## 🛠️ Tech Stack

`SQL` `SQLite` `Python` `pandas` `matplotlib` `seaborn` `Power BI`

---

## 🚀 How to Run

1. Click **Open in Colab** above
2. When prompted, upload **`sales.zip`** from this repo
3. Run all cells in order. The notebook loads the CSV into an in-memory SQLite database and runs every query

**Dataset:** [Superstore Dataset — Vivek468 (Kaggle)](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)

---

## 📁 Files

| File | Description |
|---|---|
| `Superstore_Sales_SQL_Analysis.ipynb` | All SQL queries, results and charts |
| `Superstore_PowerBI_Dashboard.png` | Power BI dashboard screenshot |
| `sales.zip` | Dataset (`Sample - Superstore.csv`) |

---

## 👩‍💻 Author

**Preeti Bhardwaj** — Software Developer | GenAI & Agentic Systems | RAG & LLM Engineering

[Portfolio](https://mistyvisty.github.io/) · [GitHub](https://github.com/mistyvisty) · [Medium](https://medium.com/@bhardwajpreeti357)
