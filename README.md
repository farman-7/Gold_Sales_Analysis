# Data Analysis Project

## 📌 Overview
This project focuses on data analysis using **Power BI, SQL, and Python**. It involves data cleaning, transformation, visualization, and insights generation to aid business decision-making.

## 📊 Features
- **Data Extraction & Cleaning**: Using SQL and Python (pandas, NumPy)
- **Data Transformation**: ETL pipeline with SQL queries
- **Data Visualization**: Power BI dashboards
- **Key Metrics & Insights**: Business trends, KPIs, forecasting

## 🔧 Technologies Used
- **Power BI**: Dashboarding & Reporting
- **SQL**: Data Extraction, Transformation & Querying
- **Python**: Data Cleaning & Analysis (pandas, NumPy, matplotlib, seaborn)
- **Excel**: Data Preprocessing

## 📌 Power BI Dashboard
Screenshots of the Power BI dashboard with key insights can be found in the `reports` folder.

## 📊 SQL Query: Sales Performance Over Time
```sql
SELECT 
DATETRUNC(MONTH, order_date) AS Orders_Date,
SUM(sales_amount) as Total_Sales,
COUNT(DISTINCT customer_key) as Total_customer,
SUM(quantity) as Total_quantity
FROM [gold.fact_sales]
WHERE order_date IS NOT NULL
GROUP BY DATETRUNC(MONTH, order_date)
ORDER BY DATETRUNC(MONTH, order_date);
```

### 📈 Output:
![Sales proformance over time](https://github.com/user-attachments/assets/7f4522e6-a401-4034-ad2e-6f6d0364cebf)

### 📊 Running Total and Moving Average of Sales
```sql
SELECT 
order_date,
total_sales,
SUM(total_sales) OVER (ORDER BY order_date) AS Running_total_sales,
AVG(avg_price) OVER (ORDER BY order_date) AS Moving_avg_price
FROM
(
SELECT 
DATETRUNC(year, order_date) AS order_date,
SUM(sales_amount) AS total_sales,
AVG(price) AS avg_price
FROM [gold.fact_sales]
WHERE order_date IS NOT NULL
GROUP BY DATETRUNC(year, order_date)
) t;
```
### 📈 Output:
![Total_sales](https://github.com/user-attachments/assets/1a8ffcce-951e-4f75-9870-0122cbada61b)

## 📊 Performance Analysis

### 🔍 Overview
This analysis evaluates the **yearly performance of products** by:
- Comparing each product’s sales to its **average performance**.
- Assessing the **year-over-year (YoY) growth**.

### 📌 SQL Query

```sql
WITH yearly_product_sales AS (
SELECT
YEAR(f.order_date) AS order_year,
p.product_name,
SUM(f.sales_amount) AS current_sales
FROM [gold.fact_sales] f
LEFT JOIN [gold.dim_products] p
ON f.product_key = p.product_key
WHERE f.order_date IS NOT NULL
GROUP BY YEAR(f.order_date), p.product_name
)

SELECT
order_year,
product_name,
current_sales,
AVG(current_sales) OVER (PARTITION BY product_name) avg_sales,
current_sales - AVG(current_sales) OVER (PARTITION BY product_name) AS diff_avg,
CASE WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) > 0 THEN 'Above Avg'
     WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) < 0 THEN 'Below Avg'
     ELSE 'Avg'
END avg_change,
LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) AS py_sales,
current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) AS diff_py,
CASE WHEN current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) > 0 THEN 'Increase'
     WHEN current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) < 0 THEN 'Decrease'
     ELSE 'No Change'
END py_change
FROM yearly_product_sales
ORDER BY product_name, order_year;
```
📌 Insights:
Products with Above Avg sales perform better than their historical average.
Year-over-Year changes (Increase/Decrease) help track product demand trends.

### 📈 Output:
![Performance analysis](https://github.com/user-attachments/assets/61bc2fc5-351e-4cbc-90b3-d18b962b98a4)

## 📊 Part-To-Whole Analysis

### 🔍 Overview
This analysis identifies **which product categories contribute the most to overall sales**.  
By calculating the **percentage share of each category**, businesses can focus on the most profitable ones.

### 📌 SQL Query
```sql
WITH category_sales AS (
SELECT 
category,
SUM(sales_amount) AS Total_sales
FROM [gold.fact_sales] f
LEFT JOIN [gold.dim_products] p
ON p.product_key = f.product_key
GROUP BY category)

SELECT 
category,
Total_sales,
SUM(Total_sales) OVER () overall_sales,
CONCAT(ROUND((CAST(Total_sales AS FLOAT) / SUM(Total_sales) OVER ()) * 100, 2),'%') AS percentage_of_total
FROM category_sales
ORDER BY Total_sales DESC;
```
📌 Insights:
Electronics contributes the highest (33.33%) to overall sales.
Furniture & Clothing together account for nearly 50% of total sales.
Lower-performing categories (e.g., Accessories, Others) may require marketing boosts. ** Need to fix it **

### 📈 Output:
![part-to-whole](https://github.com/user-attachments/assets/132ffb3a-3236-45ba-95c0-a6f0a51068b8)

## Data Segmentation
## Segment products into cost range and count how many product fall into each segment 

```sql
WITH Product_segments AS (
SELECT 
product_key,
product_name,
cost,
CASE WHEN cost< 100 THEN 'Below 100'
	 WHEN cost BETWEEN 100 AND 500 THEN '100-500'
	 WHEN cost BETWEEN 500 AND 1000 THEN '500-1000'
	 ELSE 'Above 1000'
END cost_range
FROM [gold.dim_products])

SELECT 
cost_range,
COUNT(product_key) AS total_products
FROM Product_segments
GROUP BY cost_range
ORDER BY total_products DESC
```

/*Group customers into three segments based on their spending behavior:
	-VIP: Customers with at least 12 months of history and spending more than €5,000. Regular: Customers with at least 12 months of history but spending €5,000 or less.
	-New: Customers with a lifespan less than 12 months.
And find the total number of customers by each group
*/
```sql
WITH customer_spending AS (
SELECT
c.customer_key,
SUM(f.sales_amount) AS total_spending,
MIN(order_date) AS first_order,
MAX(order_date) AS last_order,
DATEDIFF(MONTH,MIN(order_date),MAX(order_date)) AS lifespan
FROM [gold.fact_sales] f
LEFT JOIN [gold.dim_customers] c
ON f.customer_key = c.customer_key
GROUP BY c.customer_key
)

SELECT 
customer_segment,
COUNT(customer_key) AS total_customers
FROM (
	SELECT
	customer_key,
	CASE WHEN lifespan >= 12 AND total_spending >5000 THEN 'Vip'
		 WHEN lifespan >= 12 AND total_spending <=5000 THEN 'Regular'
		 ELSE 'New'
	END customer_segment
	FROM customer_spending )t
GROUP BY customer_segment
ORDER BY total_customers DESC
```sql

/*
===============================================================================================
Customer Report
===============================================================================================
Purpose:

 - This report consolidates key customer metrics and behaviors

Highlights:
	1. Gathers essential fields such as names, ages, and transaction details.
	2. Segments customers into categories (VIP, Regular, New) and age groups.
	3. Aggregates customer-level metrics:
		- total orders
		- total sales
		- total quantity purchased
		- total products
		- lifespan (in months)
4. Calculates valuable KPIS:

	- recency (months since last order)
	- average order value
	- average monthly spend
=============================================================================================
*/
```sql
With base_query AS (
/*--------------------------------------------------------------------------------------------
1) Base Query Retrieves core columns from table
-----------------------------------------------------------------------------------------------*/
SELECT
f.order_number,
f.product_key,
f.order_date,
f.sales_amount,
f.quantity,
c.customer_key,
c.customer_number,
-- c.birthdate,
CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
DATEDIFF(YEAR,c.birthdate, GETDATE()) age
FROM [gold.fact_sales] f
LEFT JOIN [gold.dim_customers] c
ON c.customer_key= f.customer_key
WHERE order_date IS NOT NULL
)

, customer_aggregation AS(
/*--------------------------------------------------------------------------------------------
	2. Segments customers into categories (VIP, Regular, New) and age groups.
-----------------------------------------------------------------------------------------------*/
SELECT 
	customer_key,
	customer_number,
	customer_name,
	age,
	COUNT(DISTINCT order_number) AS total_orders,
	SUM(sales_amount) AS total_sales,
	SUM(quantity) AS total_quantity,
	COUNT(DISTINCT product_key) AS total_products,
	MAX(order_date) AS last_order_date,
	DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan
FROM base_query
GROUP BY
	customer_key,
	customer_number,
	customer_name,
	age
	)

SELECT 
customer_key,
customer_number,
customer_name,
age,
CASE 
	WHEN age <20 THEN 'Under 20'
	WHEN age between 20 and 29 THEN '20-29'
	WHEN age between 30 and 39 THEN '30-39'
	WHEN age between 40 and 49 THEN '40-49'
	ELSE '50 and above'
END AS age_group,
CASE 
	WHEN lifespan >= 12 AND total_sales >5000 THEN 'Vip'
	WHEN lifespan >= 12 AND total_sales <=5000 THEN 'Regular'
	ELSE 'New'
END customer_segment,
last_order_date,
DATEDIFF(MONTH, last_order_date, GETDATE()) AS recency,
total_orders,
total_sales,
total_quantity,
total_products,
lifespan,
-- Compute average order value(AVO)
total_sales / total_orders AS avg_order_value,
-- Compute Avg Monthly spend
CASE 
	WHEN lifespan = 0 THEN total_sales
	ELSE total_sales / lifespan
END AS avg_monthly_spend
FROM customer_aggregation
```




/*
=====================================================================================================================================================
Product Report
=====================================================================================================================================================
Purpose:

	-This report consolidates key product metrics and behaviors.

Highlights:

	1. Gathers essential fields such as product name, category, subcategory, and cost.
	2. Segments products by revenue to identify High-Performers, Mid-Range, or Low-Performers
	3. Aggregates product-level metrics:
		- total orders
		- total sales
		- total quantity sold
		- total customers (unique)
		- lifespan (in months)
	4. Calculates valuable KPIS:
		- recency (months since last sale)
		- average order revenue (AOR)
		- average monthly revenue
=====================================================================================================================================================
*/



```sql
WITH base_query AS (
/*
=====================================================================================================================================================
1) Base Query: Retrieves core columns from fact_sales and dim_products
===================================================================================================================================================== */
SELECT
f.order_number,
f.order_date,
f.customer_key,
f.sales_amount,
f.quantity,
p.product_key,
p.product_name,
p.category,
p.subcategory,
p.cost
FROM [gold.fact_sales] f
LEFT JOIN [gold.dim_products] p
ON f.product_key = p.product_key
WHERE order_date IS NOT NULL --only consider valid sales dates
),

product_aggregations AS (
/*
=====================================================================================================================================================
2) Product Aggregations: Summarizes key metrics at the product level.
=====================================================================================================================================================*/
SELECT
	product_key,
	product_name,
	category,
	subcategory,
	cost,
	DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
	MAX(order_date) AS last_sale_date,
	COUNT (DISTINCT order_number) AS total_orders,
	COUNT (DISTINCT customer_key) AS total_customers,
	SUM(sales_amount) AS total_sales,
	SUM(quantity) AS total_quantity,
	ROUND (AVG(CAST(sales_amount AS FLOAT) / NULLIF (quantity, 0)),1) AS avg_selling_price
FROM base_query
GROUP BY
	product_key,
	product_name,
	category,
	subcategory,
	cost)
/*
=====================================================================================================================================================
3) Final Query: Combines all product results into one output
=====================================================================================================================================================*/
SELECT
	product_key,
	product_name,
	category,
	subcategory,
	cost,
	last_sale_date,
	DATEDIFF(MONTH, last_sale_date, GETDATE()) AS recency_in_months,
	CASE
		WHEN total_sales > 50000 THEN 'High-Performer'
		WHEN total_sales >= 10000 THEN 'Mid-Range'
		ELSE 'Low-Performer'
	END AS product_segment,
	lifespan,
	total_orders,
	total_sales,
	total_quantity,
	total_customers,
	avg_selling_price,
	--Average Order Revenue (AOR)
	CASE
		WHEN total_orders = 0 THEN 0
		ELSE total_sales / total_orders
	END AS avg_order_revenue,
	-- Average Monthly Revenue
	CASE
		WHEN lifespan = 0 THEN total_sales
		ELSE total_sales / lifespan
	END AS avg_monthly_revenue
FROM product_aggregations
```sql

## 📞 Contact
For queries, feel free to connect via [LinkedIn](https://www.linkedin.com/in/yourprofile/) or email at your.email@example.com.

---

📝 *This project is open for contributions! Feel free to fork, star, or raise an issue.*
