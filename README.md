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

📂 **Folder**: `/sql/analysis_queries.sql`

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

