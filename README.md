#  CRM Analytics Project

This project aims to perform in-depth customer analysis to support marketing and customer relationship management (CRM) strategies. It covers CLV calculation, customer segmentation using RFM, churn risk analysis, and new customer acquisition trends.

---

##  Tools Used

- **Excel**: For data cleaning and preprocessing
- **MySQL Workbench**: For building analytical tables and calculations
- **Tableau**: For creating dashboards and visualizations https://public.tableau.com/views/RFMandCLV/Country?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

---

##  Step 1: Data Cleaning (Excel)

- Removed cancelled invoices (e.g., Invoice IDs starting with 'C')
- Filtered out null or invalid `customer_id`
- Created `total_spending = quantity * price_usd` for later use in RFM and CLV calculations

---

##  Step 2: Customer Lifetime Value (CLV) – Next 180 Days

Calculate the estimated revenue a customer would generate over the next **180 days**, assuming past purchase behavior continues.

```sql
Create table crm.clv_180_days AS
WITH cte AS  -- to built avg_purchase purchase_frequency_per_day and customer_lifespan 
(
SELECT 
	customer_id,
    round(AVG(price_usd),2) AS avg_purchase,  
	COUNT(DISTINCT invoice) AS total_orders,
    datediff(MAX(invoicedate), Min(invoicedate)) AS customer_lifespan,    
	COUNT(DISTINCT invoice) * 1.0 / (
		CASE
			WHEN DATEDIFF(MAX(invoicedate), Min(invoicedate)) = 0 THEN 1
			ELSE DATEDIFF(MAX(invoicedate), Min(invoicedate))
		END
	) AS purchase_frequency_per_day
FROM crm.online_retail_cleaned 
WHERE customer_id IS NOT NULL AND customer_id <> 0
Group by customer_id
),
clv_table AS -- to calculate CLV 
(
SELECT 
	customer_id,
    ROUND(avg_purchase * purchase_frequency_per_day * 180, 2) AS CLV
FROM cte
)
select 
	customer_id,
    CLV AS next_180_days
FROM clv_table
ORDER BY CLV DESC;
```
---
## Step 3: RFM Segmentation

- Segment customers using RFM analysis based on:
- Recency: How recently a customer purchased
- Frequency: How often they purchased
- Monetary: How much they spent 

```sql
CREATE TABLE crm.rfm AS
WITH rfm AS (
    SELECT
        customer_id,
        DATEDIFF('2010-12-09', MAX(invoicedate)) AS recency_days,
        COUNT(DISTINCT invoice) AS frequency,
        SUM(total_spending) AS monetary_value
    FROM crm.online_retail_cleaned
    WHERE customer_id <> 0
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT
        customer_id,
        recency_days,
        frequency,
        monetary_value,
        CASE
            WHEN recency_days <= 30 THEN 1
            WHEN recency_days <= 90 THEN 2
            WHEN recency_days <= 180 THEN 3
            ELSE 4
        END AS recency_score,
        NTILE(4) OVER (ORDER BY frequency DESC) AS frequency_score,
        NTILE(4) OVER (ORDER BY monetary_value DESC) AS monetary_score
    FROM rfm
)
SELECT
    customer_id,
    recency_days,
    frequency,
    monetary_value,
    CONCAT(recency_score, frequency_score, monetary_score) AS rfm_score,
    CASE 
        WHEN CONCAT(recency_score, frequency_score, monetary_score) = '111' THEN 'Best Customers'
        WHEN CONCAT(recency_score, frequency_score, monetary_score) = '311' THEN 'Almost Lost'
        WHEN CONCAT(recency_score, frequency_score, monetary_score) = '411' THEN 'Lost Customers'
        WHEN LEFT(CONCAT(recency_score, frequency_score, monetary_score),1) = '1' THEN 'Recent Customers'
        WHEN SUBSTRING(CONCAT(recency_score, frequency_score, monetary_score),2,1) = '1' THEN 'Loyal Customers'
        WHEN RIGHT(CONCAT(recency_score, frequency_score, monetary_score),1) = '1' THEN 'Big Spenders'
        ELSE 'Others'
    END AS rfm_segment
FROM rfm_scores;
```

## Step 4:Churn Risk Analysis
Classify customers by their churn risk level based on inactivity.

```sql
CREATE TABLE crm.churn_risk_level AS
SELECT
    customer_id,
    DATEDIFF('2010-12-09', MAX(invoicedate)) AS churn_day,
    CASE
        WHEN DATEDIFF('2010-12-09', MAX(invoicedate)) < 30 THEN 'Low Risk'
        WHEN DATEDIFF('2010-12-09', MAX(invoicedate)) < 60 THEN 'Medium Risk'
        WHEN DATEDIFF('2010-12-09', MAX(invoicedate)) < 90 THEN 'High Risk'
        ELSE 'Churned'
    END AS churn_risk_level
FROM crm.online_retail_cleaned
GROUP BY customer_id;
```
## Step 5:Customer Acquisition Analysis
Identify new customer trends by month and country.

```sql
WITH first_purchase AS (
    SELECT
        customer_id,
        country,
        MIN(invoicedate) AS first_purchase_date
    FROM crm.online_retail_cleaned
    WHERE customer_id IS NOT NULL AND customer_id <> 0
    GROUP BY customer_id, country
)
SELECT
    country,
    DATE_FORMAT(first_purchase_date, '%Y-%m') AS month,
    COUNT(DISTINCT customer_id) AS new_customers
FROM first_purchase
GROUP BY country, month
ORDER BY month, country;
```
## Step 6: Visualization (Tableau)
![Capture](https://github.com/user-attachments/assets/7abdd425-adf9-4927-b796-466caf2e9c46)
![Capture2](https://github.com/user-attachments/assets/1bc64244-2cec-418b-ac49-6507555f6fe9)




