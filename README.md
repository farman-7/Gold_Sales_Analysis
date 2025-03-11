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

## 📁 Project Structure
```
📂 Data-Analysis-Project
├── 📁 data            # Raw & Processed Data
├── 📁 scripts         # Python scripts for data processing
├── 📁 reports         # Power BI reports & dashboards
├── 📁 sql_queries     # SQL scripts for data transformation
├── 📁 images          # Project screenshots
├── README.md         # Project documentation
```

## 🚀 Getting Started
1. **Clone the Repository**:
   ```sh
   git clone https://github.com/yourusername/Data-Analysis-Project.git
   cd Data-Analysis-Project
   ```
2. **Set up Virtual Environment (Python)**:
   ```sh
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```
3. **Install Dependencies**:
   ```sh
   pip install -r requirements.txt
   ```
4. **Run SQL Scripts** to create and populate tables.
5. **Load Data into Power BI** and visualize insights.

## 📌 Power BI Dashboard
Screenshots of the Power BI dashboard with key insights can be found in the `reports` folder.

## 📊 SQL Query: Sales Performance Over Time
```sql
-- Analyze Sales Performance Over Time
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
![Sales Performance Over Time](images/Sales_performance_over_time.png)

## 📞 Contact
For queries, feel free to connect via [LinkedIn](https://www.linkedin.com/in/yourprofile/) or email at your.email@example.com.

---

📝 *This project is open for contributions! Feel free to fork, star, or raise an issue.*
