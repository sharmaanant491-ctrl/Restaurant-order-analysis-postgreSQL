<div align="center">

<img width="1307" height="768" alt="real thumbnail project order analysis " src="https://github.com/user-attachments/assets/3608bfa9-2868-47a7-a0c0-392a62e14fb0" />



# 🍽️ Restaurant Order Analysis: PostgreSQL Data Insights

### An End-to-End SQL Project Exploring Customer Behavior, Menu Performance & Revenue Patterns

</div>

---

## 📌 Project Overview

The **Taste of the World Café** introduced a new international menu and wanted to understand how customers were responding to the updated offerings and which menu items were contributing most to restaurant performance.

This project analyzes **12,266 restaurant transaction records** from Q1 (January–March 2023) to answer important business questions:

- Which cuisines and menu items are the most popular?
- Which items generate the highest revenue?
- What are the least and most expensive menu items?
- What are the ordering patterns across customers?
- How many large or bulk orders were placed?
- What was the highest-value order?
- Which menu categories have the strongest revenue potential?
- What opportunities exist for improving menu performance?

The analysis was performed using **PostgreSQL** with SQL techniques including **CTEs, Window Functions, Subqueries, JOINs, Aggregations, GROUP BY, HAVING, and Revenue Analysis**.

The objective is to transform raw restaurant transaction data into meaningful business insights that can support **menu optimization, revenue growth, customer understanding, and strategic decision-making**.

---

## 📊 Dataset Overview

- **Database:** PostgreSQL
- **Schema:** `restaurant_db`
- **Timeframe:** January – March 2023
- **Records:** 12,266
- **Tables:** `menu_items`, `order_details`

### `menu_items`

| Column | Description |
|---|---|
| `menu_item_id` | Unique dish identifier |
| `item_name` | Name of the dish |
| `category` | Cuisine/category of the dish |
| `price` | Price of the dish |

### `order_details`

| Column | Description |
|---|---|
| `order_id` | Unique order identifier |
| `order_date` | Date of transaction |
| `order_time` | Time of transaction |
| `item_id` | ID linking the order to a menu item |

---

# 🎯 Objective 1: Understanding the Menu Structure

Before analyzing customer behavior, it is important to understand the pricing structure and composition of the restaurant's menu.

---
<br>
 🔎 Explore Menu Structure
The first step is to inspect the available menu items.

```sql
SELECT *
FROM menu_items;
```
<br>
<br>
📌 How Many Items Are on the Menu?
Understanding the total number of menu items helps provide context for menu variety and demand distribution.

```sql
SELECT COUNT(*) AS total_menu_items
FROM menu_items;
```
<br>
<br>
💰 What Are the Least & Most Expensive Items?
Pricing extremes help identify premium menu offerings and lower-priced items.
Using `DENSE_RANK()` ensures that items with the same price are handled fairly.

```sql
WITH price_rank AS (
    SELECT
        item_name,
        category,
        price,
        DENSE_RANK() OVER (ORDER BY price DESC) AS expensive_rank,
        DENSE_RANK() OVER (ORDER BY price ASC) AS cheapest_rank
    FROM menu_items
)

SELECT *
FROM price_rank
WHERE expensive_rank = 1
   OR cheapest_rank = 1;
```
<br>
<br>
🍝 Italian Cuisine Pricing Analysis
The Italian category can be analyzed separately to understand its pricing range and average positioning.
<br>
<br>

```sql
SELECT
    COUNT(*) AS total_italian_dishes,
    MIN(price) AS cheapest_italian_item,
    MAX(price) AS most_expensive_italian_item,
    ROUND(AVG(price), 2) AS avg_italian_price
FROM menu_items
WHERE category = 'Italian';
```
<br>
<br>
📊 Category Distribution & Pricing Strategy
Comparing the number of dishes and average price across categories helps identify premium, mid-range, and lower-priced categories.

```sql
SELECT
    category,
    COUNT(*) AS total_items,
    ROUND(AVG(price), 2) AS avg_dish_price
FROM menu_items
GROUP BY category
ORDER BY avg_dish_price DESC;
```
<br>
<br>
# 🎯 Objective 2: Understanding Order Patterns
After understanding the menu and pricing structure, the next step is to analyze customer ordering behavior and restaurant transaction volume.
<br>
<br>
📅 What Is the Date Range?
This validates the time coverage of the dataset

```sql
SELECT
    MIN(order_date) AS first_order,
    MAX(order_date) AS last_order
FROM order_details;
```
<br>
<br>
📦 How Many Orders & Items Were Sold?
This measures the overall operational volume of the restaurant.

```sql
SELECT
    COUNT(DISTINCT order_id) AS total_orders,
    COUNT(*) AS total_items_sold
FROM order_details;
```
<br>
<br>
🏆 Which Orders Had the Most Items?
Large orders may indicate group dining, celebrations, catering opportunities, or high-value customers.

```sql
SELECT
    order_id,
    COUNT(*) AS num_items
FROM order_details
GROUP BY order_id
ORDER BY num_items DESC;
```
<br>
<br>

📈 How Many Orders Had More Than 12 Items?
Identifying large orders helps evaluate potential bulk-ordering and catering opportunities.

```sql
SELECT COUNT(*) AS large_orders
FROM (
    SELECT
        order_id
    FROM order_details
    GROUP BY order_id
    HAVING COUNT(*) > 12
) AS bulk_orders;
```
<br>

# 🎯 Objective 3: Customer Behavior & Revenue Analysis
This stage combines menu information with order data to understand customer demand, item popularity, and revenue contribution.
<br>

<br>
<br>
🔗 Combine Menu & Order Data
A `JOIN` is used to connect customer orders with the corresponding menu items.

```sql
SELECT
    o.order_id,
    o.order_date,
    m.item_name,
    m.category,
    m.price
FROM order_details AS o
LEFT JOIN menu_items AS m
    ON o.item_id = m.menu_item_id;
```
<br>
<br>
📊 Most & Least Ordered Items
Order volume shows which dishes customers purchase most frequently, while revenue shows the financial contribution of those items.

```sql
SELECT
    m.item_name,
    m.category,
    COUNT(*) AS times_ordered,
    SUM(m.price) AS total_revenue
FROM order_details AS o
LEFT JOIN menu_items AS m
    ON o.item_id = m.menu_item_id
GROUP BY
    m.item_name,
    m.category
ORDER BY times_ordered DESC;
```
<br>
<br>

#### This analysis helps identify:

#### ⭐ Highly popular dishes
#### 💰 High-revenue dishes
#### 📉 Less frequently ordered dishes
#### 🍽️ Revenue concentration across menu items
#### Popularity and revenue do not always tell the same story. A lower-priced item may be ordered frequently while a premium item may generate more revenue with fewer orders.
<br>
<br>

###  💵 Top 5 Highest-Spending Orders
Identifying the highest-value orders helps understand revenue concentration and customer spending behavior.

```sql
SELECT
    o.order_id,
    SUM(m.price) AS total_spend
FROM order_details AS o
LEFT JOIN menu_items AS m
    ON o.item_id = m.menu_item_id
GROUP BY o.order_id
ORDER BY total_spend DESC
LIMIT 5;
```
<br>
<br>

## 🥇 Inspect the Highest-Spending Order

After identifying the highest-value order, we can examine the individual items that contributed to it.

```sql
WITH highest_order AS (
    SELECT
        o.order_id,
        SUM(m.price) AS total_spend
    FROM order_details AS o
    LEFT JOIN menu_items AS m
        ON o.item_id = m.menu_item_id
    GROUP BY o.order_id
    ORDER BY total_spend DESC
    LIMIT 1
)

SELECT
    o.order_id,
    m.item_name,
    m.category,
    m.price
FROM order_details AS o
LEFT JOIN menu_items AS m
    ON o.item_id = m.menu_item_id
WHERE o.order_id = (
    SELECT order_id
    FROM highest_order
);
```
<br>

# 🎯 Objective 4: Revenue Analysis by Category

<br>

Analyzing revenue at the category level provides a broader view of restaurant performance.

```sql
SELECT
    m.category,
    COUNT(*) AS items_sold,
    SUM(m.price) AS total_revenue,
    ROUND(AVG(m.price), 2) AS average_item_price
FROM order_details AS o
LEFT JOIN menu_items AS m
    ON o.item_id = m.menu_item_id
GROUP BY m.category
ORDER BY total_revenue DESC;
```
This helps identify which cuisine categories contribute most to overall revenue.
<br>
<br>
📊 Revenue Contribution by Menu Item
To understand which individual dishes are driving restaurant revenue:

```sql
SELECT
    m.item_name,
    m.category,
    COUNT(*) AS quantity_sold,
    SUM(m.price) AS revenue_generated
FROM order_details AS o
LEFT JOIN menu_items AS m
    ON o.item_id = m.menu_item_id
GROUP BY
    m.item_name,
    m.category
ORDER BY revenue_generated DESC;
```
<br>
<br>
🔥 Top Performing Menu Items

The highest-performing menu items can be ranked using a window function.
```sql
WITH item_performance AS (
    SELECT
        m.item_name,
        m.category,
        COUNT(*) AS quantity_sold,
        SUM(m.price) AS revenue_generated
    FROM order_details AS o
    LEFT JOIN menu_items AS m
        ON o.item_id = m.menu_item_id
    GROUP BY
        m.item_name,
        m.category
)

SELECT
    *,
    DENSE_RANK() OVER (
        ORDER BY revenue_generated DESC
    ) AS revenue_rank
FROM item_performance
ORDER BY revenue_rank;
```
<br>
<br>

📅 Order Activity by Date

Analyzing order activity by date helps identify changes in transaction volume.
```sql
SELECT
    order_date,
    COUNT(DISTINCT order_id) AS total_orders,
    COUNT(*) AS items_sold
FROM order_details
GROUP BY order_date
ORDER BY order_date;
```
<br>
<br>

⏰ Order Activity by Hour
Order time can be used to understand when customer demand is highest.

```sql
SELECT
    EXTRACT(HOUR FROM order_time) AS order_hour,
    COUNT(*) AS items_ordered
FROM order_details
GROUP BY EXTRACT(HOUR FROM order_time)
ORDER BY order_hour;
```
This can help identify peak operating hours and support staffing or promotional decisions.
<br>
<br>

# 🔗 SQL  Techniques Used

SQL |Concept	Application
|---|---|
SELECT |	Data exploration
WHERE |	Filtering records
GROUP BY |	Category and order analysis
HAVING |	Filtering grouped results
ORDER BY |	Ranking results
JOIN |	Combining menu and order data
COUNT() |	Measuring orders and item frequency
SUM() |	Revenue calculations
AVG()	| Average pricing
MIN() / MAX() |	Price and date ranges
CTE	 | Breaking complex analysis into logical steps
DENSE_RANK() |	Ranking menu items and prices
Subqueries |	Nested analytical calculations
EXTRACT() |	Time-based analysis
Aggregation | Revenue and popularity analysis
<br>
<br>

# Final Validation
 What Was the Most Expensive Order?
The following query calculates the maximum order value across all transactions.

```sql
SELECT
    MAX(order_total) AS highest_order_value
FROM (
    SELECT
        o.order_id,
        SUM(m.price) AS order_total
    FROM order_details AS o
    LEFT JOIN menu_items AS m
        ON o.item_id = m.menu_item_id
    GROUP BY o.order_id
) AS order_totals;
```
<br>
<br>

# 📋 Analytical Bias Audit

| Stage       | Risk                                                | Mitigation                                              |
| ----------- | --------------------------------------------------- | ------------------------------------------------------- |
| Data Scope  | Analysis covers only Q1                             | Findings are interpreted within the available timeframe |
| Ranking     | Multiple items may have identical prices            | `DENSE_RANK()` is used                                  |
| Popularity  | High order volume does not always mean high revenue | Volume and revenue are compared                         |
| Pricing     | Premium items may have lower order frequency        | Price and demand are analyzed together                  |
| Bulk Orders | Large orders may represent unusual events           | Bulk orders are analyzed separately                     |

<br>
<br>

💡 Strategic Recommendations
## 1️⃣ **Menu Engineering Optimization**

Menu items can be categorized into four performance groups:

⭐ High Volume / High Revenue — Stars
📈 High Volume / Low Revenue — Traffic Drivers
💎 Low Volume / High Revenue — Premium Niche
📉 Low Volume / Low Revenue — Candidates for Review

This framework can help management decide which dishes should be promoted, repositioned, repriced, or reviewed.
<br>
<br>

## 2️⃣ **Premium Bundle Strategy**
High-revenue categories can be used to create curated meal combinations.
Potential strategies include:
Premium dinner combinations
Appetizer + main course bundles
Family meal packages
Dessert add-ons
Premium upselling options
These strategies can help increase Average Order Value (AOV).
<br>
<br>

## 3️⃣ **Corporate & Group Ordering Strategy**
Orders containing more than 12 items can indicate potential group-ordering behavior.
The restaurant could introduce:
Corporate lunch packages
Event catering bundles
Group meal packages
Pre-set menus
Large-order promotions
This could create an additional and more predictable revenue stream.
<br>
<br>

## 4️⃣ **Revenue Concentration Monitoring**
If a small number of menu items contribute a significant portion of revenue, management should monitor those items carefully.
Possible strategies include:
Keeping high-performing dishes prominently positioned
Creating complementary add-ons
Offering personalized promotions
Using upselling recommendations
Monitoring inventory availability
<br>
<br>

## 5️⃣ **Balance Popularity With Revenue**
A menu item should not be evaluated using order frequency alone.
A stronger performance framework considers:
Order Volume + Price + Revenue Contribution
This provides a more complete understanding of menu performance.
<br>
<br>

## 📈 Key Takeaways
This project demonstrates how PostgreSQL and SQL can transform raw restaurant transactions into actionable business insights.

The analysis covers:
🍽️ Menu composition
💰 Menu pricing
📊 Order volume
⭐ Item popularity
💵 Revenue contribution
🏆 Highest-value orders
📦 Bulk ordering behavior
⏰ Peak ordering periods
🧠 Menu engineering
📈 Business recommendations

<br>

The project demonstrates that SQL is not only useful for querying databases but can also be used to answer practical business questions and support data-driven decision-making.
<br>

## 🛠️ Tools & Technologies
PostgreSQL
SQL
GitHub
Maven Analytics Dataset
<br>
<br>

## 📚 Dataset Credit
The dataset used in this project is based on the Restaurant Orders SQL Case Study by Maven Analytics.
The dataset contains restaurant menu and order information used to explore menu performance, customer ordering behavior, and revenue patterns.
🔗 https://mavenanalytics.io/
<br>
<br>

### 📅 Project Information
Detail	Information
Project	Restaurant Order Analysis
Database	PostgreSQL
Domain	SQL / Data Analytics
Dataset	Maven Analytics Restaurant Orders
Completed	September 6, 2026
<br>
<br>

👨‍💻 Author
Anant Sharma

B.Tech — Electronics & Communication Engineering
Jaypee University of Engineering and Technology
<br>

Skills
<ul>
  <li>SQL & Data Analytics</li>
  <li>PostgreSQL/li>
  <li>Power Bi </li>
    <li>Data Cleaning </li>
    <li>MS Excel </li>
</ul>






















