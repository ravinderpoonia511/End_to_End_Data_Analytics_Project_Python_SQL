**Sales Data Analysis with Python and SQL**
*Overview*
This project involves fetching sales data from the Kaggle API, cleaning it using Python, and analyzing it in SQL. The primary goal is to derive insights such as revenue generation, sales trends, and growth comparisons across different products and categories.

*Prerequisites*
Required Python packages:
kaggle
pandas
mysql-connector-python
SQLAlchemy

You can install the required packages using:
pip install kaggle pandas mysql-connector-python SQLAlchemy

Setup
Kaggle API Token:

Create a Kaggle API token and store it in the .kaggle folder on your C drive.

Download Dataset:
import kaggle
!kaggle datasets download ankitbansal06/retail-orders -f orders.csv


Extract and Load Data:
import zipfile
with zipfile.ZipFile('orders.csv.zip') as zip_ref:
    zip_ref.extractall()

    
Data Cleaning:
Load the CSV file and handle missing values.
Normalize column names and create new columns for discounts, sale prices, and profits.

Database Connection:
Use SQLAlchemy to connect to a MySQL database.
Insert the cleaned DataFrame into a SQL table.

SQL Analysis
Several SQL queries are used to derive insights from the data:

Top 10 Highest Revenue Generating Products:

``` sql
WITH cte AS (
    SELECT product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY product_id
)
SELECT product_id, sales
FROM (
    SELECT *, DENSE_RANK() OVER (ORDER BY sales DESC) rnk FROM cte
) a1
WHERE rnk <= 10;```


Top 5 Highest Selling Products by Region:
``` sql
WITH cte AS (
    SELECT region, product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY region, product_id
)
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY region ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn <= 5;```


Month-over-Month Growth Comparison (2022 vs 2023):

``` sql
WITH cte AS (
    SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month,
           SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
       SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
       SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;```




Highest Sales Month by Category:

```sql
WITH cte AS (
    SELECT category, FORMAT(order_date, 'yyyyMM') AS order_year_month,
           SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY category, FORMAT(order_date, 'yyyyMM')
)
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn
    FROM cte
) a
WHERE rn = 1;```


Sub-category with Highest Profit Growth (2022 vs 2023):
``` sql
WITH cte AS (
    SELECT sub_category, YEAR(order_date) AS order_year,
           SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY sub_category, YEAR(order_date)
),
cte2 AS (
    SELECT sub_category,
           SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
           SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
    FROM cte
    GROUP BY sub_category
),
cte3 AS (
    SELECT *,
           (sales_2023 - sales_2022) * 100 / sales_2022 AS profit_per,
           DENSE_RANK() OVER (ORDER BY (sales_2023 - sales_2022) * 100 / sales_2022 DESC) rnk
    FROM cte2
)
SELECT *
FROM cte3
WHERE rnk = 1;```


**Conclusion**
This project demonstrates how to integrate Python and SQL for effective sales data analysis, enabling data-driven decision-making.
