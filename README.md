# -Walmart-Sales-Performance-Analysis-Advanced-MySQL-Project
To leverage advanced MySQL techniques to analyse Walmart’s transaction data and generate actionable insights on sales performance, customer behaviour, and operational efficiency.
Business Context

Walmart operates across multiple cities with diverse product lines and customer segments. Understanding sales trends, customer preferences, and product performance is essential to optimise marketing strategies, improve branch operations, and increase profitability.

Dataset

Walmart Sales Dataset – containing transaction-level data including:

Branch and City

Customer demographics

Product lines

Sales figures & gross income

Payment methods

Scope & Tasks
1. Identifying the Top Branch by Sales Growth Rate

Calculated monthly sales per branch.

Determined branch with highest growth over time.

CODE:
with Monthlysales AS (
SELECT
Branch,
DATE_FORMAT (STR_TO_DATE(Date,' %d-%m-%Y'),'%Y-%m') AS Month_Year,
SUM(Total) AS Monthly_Total_Sales
FROM walmart
GROUP BY Branch, Month_Year
),
GrowthData AS (
SELECT
Branch,Month_Year,Monthly_Total_Sales,
LAG(Monthly_Total_Sales) OVER (PARTITION BY Branch ORDER BY Month_Year) AS Prev_Month_Sales,
ROUND(((Monthly_Total_Sales - LAG(Monthly_Total_Sales) OVER (PARTITION BY Branch ORDER BY Month_Year))/ LAG(Monthly_Total_Sales) 
OVER (PARTITION BY Branch ORDER BY Month_Year)) * 100) AS GrowthPercent
FROM Monthlysales
)
SELECT Branch,
ROUND(AVG(GrowthPercent), 0) AS Avg_Growth_Percent
FROM GrowthData
GROUP BY Branch
ORDER BY Avg_Growth_Percent DESC;



2. Most Profitable Product Line per Branch

Computed profit margins = Gross Income − COGS.

Identified top-performing product lines by branch.

CODE:
SELECT Branch, 
 `Product line`, 
 total_profit
FROM (
 SELECT Branch, 
 `Product line`, 
 SUM(2 * `gross income` - Total) AS total_profit,
 RANK() OVER (PARTITION BY Branch ORDER BY SUM(2 * `gross income` - Total) DESC) AS profit_rank
 FROM walmart
 GROUP BY Branch, `Product line`
) AS ranked_data
WHERE profit_rank = 1;

3. Customer Segmentation by Spending

Classified customers into High, Medium, and Low spenders.

CODE:
 select `customer ID`,
  ROUND(sum(total),0)as TotalSpending,
  case 
     when  ROUND(sum(total),0) < 20000 then "Low"
     when  ROUND(sum(total),0) between 20000  and 22000 then "Medium"
     else "High"
     end as  SpendingTier
  from walmart
  group by `Customer ID`
  ORDER BY TotalSpending DESC;


4. Detecting Anomalies in Sales Transactions

Flagged unusually high or low sales compared to product line averages.

CODE:
SELECT
        w.`Product line`,
        AVG(w.Total) over(partition by w. `Product line`)AS Avg_Sales,
        STDDEV(w.Total) over(partition by w. `Product line`)AS StdDev_Sales,
        w.Total,   
case
    when w.Total > avg(w.Total)
    over(partition by w. `Product line`)+2 * STDDEV(w.Total)
	over(partition by w.`Product line`) THEN 'HIGH ANOMALY'
	when w.Total < avg(w.Total)
    over(partition by w.`Product line`)-2 * STDDEV(w.Total)
	over(partition by w.`Product line`) THEN 'LOW ANOMALY'
    else 'NORMAL'
    end as Anomaly_type
from walmart w
order by w.`Product line`,w.total desc;

5. Most Popular Payment Method by City

Determined top payment preference for targeted marketing.

CODE:
WITH payment_rank AS (
    SELECT  
        City,         
        Payment,         
        COUNT(Payment) AS PaymentCount,        
        RANK() OVER (PARTITION BY City ORDER BY COUNT(Payment) DESC) AS rnk
    FROM walmart
    GROUP BY City, Payment
)
SELECT *
FROM payment_rank
WHERE rnk = 1;


6. Monthly Sales Distribution by Gender

Analysed male vs. female sales trends over months.

CODE:
SELECT  
    DATE_FORMAT(STR_TO_DATE(`Date`, '%d-%m-%Y'), '%Y-%m') AS month_year,
    Gender,
    SUM(Total) AS total_sales
FROM walmart
GROUP BY month_year, Gender
ORDER BY month_year, Gender;

7. Best Product Line by Customer Type

Compared product preferences for Member vs. Normal customers.

CODE:
WITH RankedProductLines AS (
select 
    `customer type`,
     `Product line`,
     SUM(Total) AS TotalSales,
     RANK() OVER (PARTITION BY `Customer type` ORDER BY SUM(Total) DESC) AS `Rank`    
FROM walmart
GROUP BY `Customer type`, `Product line`
 )
SELECT     
`Customer type`, `Product line` AS PreferredProductLine, TotalSales
FROM RankedProductLines 
WHERE `Rank` = 1

8. Identifying Repeat Customers

Found customers making purchases within a 30-day window.

CODE:
SELECT DISTINCT a.`Customer ID`
FROM walmart a
JOIN walmart b 
    ON a.`Customer ID` = b.`Customer ID`
   AND a.`Invoice ID` <> b.`Invoice ID`
   AND ABS(DATEDIFF(
       STR_TO_DATE(a.`Date`, '%d-%m-%Y'),
       STR_TO_DATE(b.`Date`, '%d-%m-%Y')
   )) <= 30;

9. Top 5 Customers by Sales Volume

Ranked customers contributing the most revenue.

CODE:

select
    `customer ID`,
    sum(total) as total_sale
from walmart
group by  `customer ID`
order by total_sale desc
limit 5;


10. Sales Trends by Day of the Week

Determined high-performing sales days for operational planning.

CODE:
select 
    dayname(STR_TO_DATE(`Date`, '%d-%m-%Y'))as dayofweek,
    count(*) as nub_of_sale,
    round(sum(total),0) as total_sale
from walmart
group by dayofweek
order by total_sale desc
limit 1;

Tools & Technologies

MySQL (Advanced SQL queries: CTEs, Window Functions, Aggregations)

MS PowerPoint (For result presentation)

Excel / CSV (For intermediate analysis & visualisation)

Deliverables

SQL Script with all queries and outputs.

PowerPoint Presentation containing queries, tables, and visual insights.

Video Presentation (≤ 5 minutes) explaining methodology, findings, and recommendations.

Key Insights

Identified top-performing branch and profitable product lines.

Segmented customers for targeted marketing.

Detected sales anomalies for fraud detection and process improvement.

Highlighted peak days and payment preferences for better resource allocation.
