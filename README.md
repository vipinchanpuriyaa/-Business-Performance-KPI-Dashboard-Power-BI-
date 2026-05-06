# 📊 Revenue Optimization Dashboard

**End-to-End Data Analytics Project (SQL • Python • Power BI)**

---

## 🧠 Project Objective

To analyze sales, customers, products, and payments data to:

* Track revenue performance
* Identify growth trends (MoM, YoY)
* Rank top customers & products
* Support business decision-making using interactive dashboards

---

## 🏗️ Architecture Overview

```
Raw CSV Files
     ↓
Python (ETL + EDA)
     ↓
MySQL Database (Revenue_Optimization)
     ↓
Power BI (Star Schema + DAX)
     ↓
Executive Dashboard
```
# Revenue Optimization Dashboard




<img width="1374" height="737" alt="image" src="https://github.com/user-attachments/assets/3ca9d7e3-d6ec-4c29-94e8-29f11a24cab3" />

---

## 📁 Dataset Tables

| Table Name | Description                          |
| ---------- | ------------------------------------ |
| customers  | Customer profile, segment, region    |
| products   | Product details and categories       |
| sales      | Transaction-level sales data         |
| payments   | Payment mode & status                |
| date       | Calendar table for time intelligence |

---

## 🛠️ Step 1: Python – ETL & Data Cleaning

### ✔ Tasks Performed

* Loaded CSV files using Pandas
* Handled missing values
* Merged fact & dimension data
* Created revenue & profit columns
* Exported clean data to MySQL

### 🔑 Key Logic

```python
sales['revenue'] = (
    sales['quantity'] * sales['base_price']
) * (1 - sales['discount_pct'] / 100)
```

📂 **Folder**: `/python/etl_pipeline.ipynb`

---

## 🗄️ Step 2: SQL – Database & Analysis

### Database

```sql
CREATE DATABASE Revenue_Optimization;
```

### Example Analytical Queries

#### 🔹 Top 5 Customers by Revenue

```sql
SELECT customer_id, SUM(revenue) AS total_revenue
FROM sales
GROUP BY customer_id
ORDER BY total_revenue DESC
LIMIT 5;
```

#### 🔹 Revenue by Region

```sql
SELECT c.region, SUM(s.revenue) AS revenue
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
GROUP BY c.region;
```

🔥 1. Customer Lifetime Value (CLTV)
```sql
SELECT 
    customer_id,
    SUM(revenue) AS total_revenue,
    COUNT(DISTINCT transaction_id) AS total_orders,
    AVG(revenue) AS avg_order_value
FROM sales
GROUP BY customer_id
ORDER BY total_revenue DESC;

🔥 2. Monthly Revenue with Growth % (MoM)
```sql
SELECT 
    DATE_FORMAT(transaction_date, '%Y-%m') AS month,
    SUM(revenue) AS monthly_revenue,
    
    LAG(SUM(revenue)) OVER (ORDER BY DATE_FORMAT(transaction_date, '%Y-%m')) AS prev_month_revenue,
    
    ROUND(
        (SUM(revenue) - LAG(SUM(revenue)) OVER (ORDER BY DATE_FORMAT(transaction_date, '%Y-%m')))
        / LAG(SUM(revenue)) OVER (ORDER BY DATE_FORMAT(transaction_date, '%Y-%m')) * 100,
        2
    ) AS mom_growth_pct

FROM sales
GROUP BY month;

```sql
🔥 3. Top Customers by Segment (Window Function)

```sql

SELECT *
FROM (
    SELECT 
        c.segment,
        s.customer_id,
        SUM(s.revenue) AS total_revenue,
        RANK() OVER (
            PARTITION BY c.segment 
            ORDER BY SUM(s.revenue) DESC
        ) AS rank_in_segment
    FROM sales s
    JOIN customers c ON s.customer_id = c.customer_id
    GROUP BY c.segment, s.customer_id
) ranked
WHERE rank_in_segment <= 3;

```sql

🔥 4. Cohort Analysis (Customer Retention)
```sql
SELECT 
    customer_id,
    MIN(DATE_FORMAT(transaction_date, '%Y-%m')) AS cohort_month,
    DATE_FORMAT(transaction_date, '%Y-%m') AS activity_month
FROM sales
GROUP BY customer_id, activity_month;

🔥 5. Revenue Contribution % by Product

```sql

SELECT 
    product_id,
    SUM(revenue) AS product_revenue,
    
    ROUND(
        SUM(revenue) * 100.0 / SUM(SUM(revenue)) OVER (),
        2
    ) AS revenue_contribution_pct

FROM sales
GROUP BY product_id
ORDER BY product_revenue DESC;

```sql

🔥 6. Query Optimization Example (Important)

-- Create index for performance
```sql
CREATE INDEX idx_sales_customer 
ON sales(customer_id);
```sql
CREATE INDEX idx_sales_date 
ON sales(transaction_date);
```sql
---




## 📐 Step 3: Power BI – Data Modeling (Star Schema)

### ⭐ Fact Table

* **Sales**

### ⭐ Dimension Tables

* Customers
* Products
* Payments
* Date

### Relationships

```
Date[Date]        → Sales[transaction_date]
Customers[id]    → Sales[customer_id]
Products[id]     → Sales[product_id]
Sales[txn_id]    → Payments[transaction_id]
```

✔ One-to-Many
✔ Single Direction
✔ Date table marked as Date Table

---

## 🧮 Step 4: DAX Measures (Core Metrics)

### 🔹 Total Revenue

```DAX
Total Revenue = SUM(sales[revenue])
```

### 🔹 Total Orders

```DAX
Total Orders = DISTINCTCOUNT(sales[transaction_id])
```

### 🔹 AOV

```DAX
AOV = DIVIDE([Total Revenue], [Total Orders])
```

### 🔹 Revenue MoM %

```DAX
Revenue MoM % =
VAR PrevMonth =
    CALCULATE(
        [Total Revenue],
        DATEADD('Date'[Date], -1, MONTH)
    )
RETURN
DIVIDE([Total Revenue] - PrevMonth, PrevMonth)
```

### 🔹 Running Total Revenue

```DAX
Running Total Revenue =
CALCULATE(
    [Total Revenue],
    FILTER(
        ALL('Date'),
        'Date'[Date] <= MAX('Date'[Date])
    )
)
```

### 🔹 Customer Rank

```DAX
Customer Rank =
RANKX(
    ALL(customers[customer_name]),
    [Total Revenue],
    ,
    DESC
)
```

---

## 📊 Step 5: Dashboard Design (Power BI)

### 🎯 KPI Cards

* Total Revenue
* Total Orders
* AOV
* Revenue YoY %
* YoY Target Achievement

### 📈 Trend Analysis

* Monthly Revenue (Line Chart)
* Running Total Revenue (Area Chart)
* Revenue MoM % trend

### 🧍 Customer Analysis

* Top N Customers (Bar Chart)
* Revenue by Customer Segment
* Customer Rank Table

### 📦 Product Analysis

* Revenue by Category (Donut)
* Top Products by Revenue
* Product Rank Matrix

### 💳 Payment Analysis

* Revenue by Payment Category
* Payment Mode Share %

---

## 🎨 Dashboard UI Best Practices

* Background: **Light grey (#F5F6FA)**
* Accent color: **Amber / Gold (#FFB000)**
* KPI cards with shadow & rounded edges
* Consistent font (Segoe UI)

---

## 🧠 Key Business Insights

* South region leads total revenue
* Electronics category contributes highest revenue
* Retail segment outperforms enterprise
* Strong MoM recovery after February dip

---

## 📂 Repository Structure

```
Revenue-Optimization-Dashboard/
│
├── data/
│   ├── customers.csv
│   ├── products.csv
│   ├── sales.csv
│   └── payments.csv
│
├── python/
│   └── etl_pipeline.ipynb
│
├── sql/
│   └── analysis_queries.sql
│
├── powerbi/
│   └── Revenue_Optimization_Dashboard.pbix
│
├── images/
│   └── dashboard_preview.png
│
└── README.md
```

---

## 🚀 Tools & Technologies

* **Python** (Pandas, NumPy)
* **MySQL**
* **Power BI**
* **DAX**
* **GitHub**

---

