<div align="center">

<img src="assets/restaurant-order-analysis-thumbnail.jpg" alt="Restaurant Order Analysis - PostgreSQL Data Insights" width="100%">

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

## 🔎 Explore Menu Structure

The first step is to inspect the available menu items.

```sql
SELECT *
FROM menu_items;
