# Walmart Sales Analysis with SQL and Python

## Project Overview




This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. We utilize Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions. 

---

## Project Steps taken

### 1. Setting Up the Environment
   - **Tools Used**: Visual Studio Code (VS Code), Python, SQL (MySQL)
   - **Goal**: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.

### 2. Setting Up Kaggle API
   - **API Setup**: Obtain Kaggle API token from [Kaggle](https://www.kaggle.com/) by navigating to profile settings and downloading the JSON file.
   - **Configure Kaggle**: 
      - Place the downloaded `kaggle.json` file in my local `.kaggle` folder.
      - Used the command `kaggle datasets download -d <dataset-path>` to pull datasets directly into my project.

### 3. Download Walmart Sales Data
   - **Data Source**: Used the Kaggle API to download the Walmart sales datasets from Kaggle.
   - **Dataset Link**: [Walmart Sales Dataset](https://www.kaggle.com/najir0123/walmart-10k-sales-datasets)
   - **Storage**: Saved the data in the `data/` folder for easy reference and access.

### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas numpy sqlalchemy mysql-connector-python 
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

### 5. Explore the Data
   - **Goal**: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
   - **Analysis**: Used functions like `.info()`, `.describe()`, and `.head()` to get a quick overview of the data structure and statistics.

### 6. Data Cleaning
   - **Remove Duplicates**: Identified and removed duplicate entries to avoid skewed results.
   - **Handle Missing Values**: Dropped rows or columns with missing values if they are insignificant; fill values where essential.
   - **Fix Data Types**: Ensure all columns have consistent data types (e.g., dates as `datetime`, prices as `float`).
   - **Currency Formatting**: Used `.replace()` to handle and format currency values for analysis.
   - **Validation**: Checked for any remaining inconsistencies and verify the cleaned data.

### 7. Feature Engineering
   - **Create New Columns**: Calculate the `Total Amount` for each transaction by multiplying `unit_price` by `quantity` and adding this as a new column.
   - **Enhance Dataset**: Adding this calculated field will streamline further SQL analysis and aggregation tasks.

### 8. Load Data into MySQL 
   - **Set Up Connections**: Connect to MySQL using `sqlalchemy` and load the cleaned data into each database.
   - **Table Creation**: Set up tables in MySQL using Python SQLAlchemy to automate table creation and data insertion.
   - **Verification**: Run initial SQL queries to confirm that the data has been loaded accurately.

### 9. SQL Analysis: Complex Queries and Business Problem Solving
   - **Business Problem-Solving**: Write and execute complex SQL queries to answer critical business questions, such as:

   1) Analyze Payment Methods and Sales
          
   ```sql
    SELECT 
    payment_method,
    COUNT(*) AS no_payments,
    SUM(quantity) AS no_qty_sold
    FROM walmart
    GROUP BY payment_method;
   ```

   2) Identify the Highest-Rated Category in Each Branch
   
   ```sql
SELECT branch, category, avg_rating
FROM (
    SELECT 
        branch,
        category,
        AVG(rating) AS avg_rating,
        RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS ranked
    FROM walmart
    GROUP BY branch, category
) AS ranked
WHERE ranked = 1;
   ```
   3) Determine the Busiest Day for Each Branch
   ```sql
SELECT branch, day_name, no_transactions
FROM (
    SELECT 
        branch,
        DAYNAME(STR_TO_DATE(date, '%d/%m/%Y')) AS day_name,
        COUNT(*) AS no_transactions,
        RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS ranked
    FROM walmart
    GROUP BY branch, day_name
) AS ranked
WHERE ranked = 1;
   ```
   
   4) Calculate Total Quantity Sold by Payment Method
   ```sql
SELECT 
    payment_method,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
   ```   
   
   5) Analyze Category Ratings by City
   ```sql
SELECT 
    city,
    category,
    MIN(rating) AS min_rating,
    MAX(rating) AS max_rating,
    AVG(rating) AS avg_rating
FROM walmart
GROUP BY city, category;
   ``` 
   
   6) Calculate Total Profit by Category
   ```sql
SELECT 
    category,
    SUM(unit_price * quantity * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;
   ```  
   
   7) Determine the Most Common Payment Method per Branch
   ```sql
WITH cte AS (
    SELECT 
        branch,
        payment_method,
        COUNT(*) AS total_trans,
        RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS ranked
    FROM walmart
    GROUP BY branch, payment_method
)
SELECT branch, payment_method AS preferred_payment_method
FROM cte
WHERE ranked = 1;
   ```  
   
   8) Analyze Sales Shifts Throughout the Day
   ```sql
SELECT
    branch,
    CASE 
        WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
        WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS num_invoices
FROM walmart
GROUP BY branch, shift
ORDER BY branch, num_invoices DESC;
   ```
   
   9) Identify Branches with Highest Revenue Decline Year-Over-Year
   ```sql
WITH revenue_2022 AS (
    SELECT 
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2022
    GROUP BY branch
),
revenue_2023 AS (
    SELECT 
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2023
    GROUP BY branch
)
SELECT 
    r2022.branch,
    r2022.revenue AS last_year_revenue,
    r2023.revenue AS current_year_revenue,
    ROUND(((r2022.revenue - r2023.revenue) / r2022.revenue) * 100, 2) AS revenue_decrease_ratio
FROM revenue_2022 AS r2022
JOIN revenue_2023 AS r2023 ON r2022.branch = r2023.branch
WHERE r2022.revenue > r2023.revenue
ORDER BY revenue_decrease_ratio DESC
LIMIT 5;
   ```

### 10. Project Publishing and Documentation
   - **Documentation**: Maintained well-structured documentation of the entire process.
   - **Project Publishing**: Publish the completed project on GitHub.
---

## Results and Insights

**Sales Insights**

Key Categories: The dataset reveals that Health and Beauty, Electronic Accessories, and Food and Beverages are prominent categories, with numerous transactions recorded.

Branches with Highest Sales: Branches such as San Antonio (WALM003) and Irving (WALM013) frequently appear in high-value transactions, indicating strong sales performance in these locations.

Preferred Payment Methods: The most common payment methods include Ewallet, Cash, and Credit Card, with Ewallet being particularly popular among customers.

**Profitability**

Most Profitable Product Categories: Categories like Health and Beauty and Fashion Accessories show higher profit margins, suggesting they are more lucrative for the business.

Locations of High Profitability: Branches such as Bryan (WALM042) and San Antonio (WALM003) not only have high sales but also significant profit margins, indicating their effectiveness in generating revenue.

**Customer Behavior**

Trends in Ratings: Products generally receive favorable ratings, with many items scoring above 8. This suggests customer satisfaction is relatively high across various categories.

Payment Preferences: Customers tend to favor digital payment options like Ewallet, reflecting a trend towards cashless transactions.

Peak Shopping Hours: The data indicates peak shopping times are often around late morning to early afternoon (10 AM - 2 PM), which is typical for retail environments.
This analysis provides a comprehensive overview of sales dynamics, profitability, and customer behavior within the Walmart dataset.

---

---

## Acknowledgments

- **Data Source**: Kaggle’s Walmart Sales Dataset
