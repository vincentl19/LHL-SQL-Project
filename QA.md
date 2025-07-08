**What are your risk areas? Identify and describe them.**

1. Duplicate rows of data that have the same information
2. Multiple fields to calculate revenue, not not sure which one to rely on
3. Improperly formatted fields, no consistent format within some fields
4. Incorrect calculation of revenue from units_sold * unit_price
5. Missing rows of date across tables (e.g. productsku in products table vs sales_by_sku)
6. Not enough data nomarlization



**QA Process:**
**Describe your QA process and include the SQL queries used to execute it.**

<!-- Check for duplicate data in analytics table by creating a custom key using multiple fields 

165267 rows returned with COUNT > 1 signaling there are duplicates
-->
SELECT * 
FROM(
	SELECT 
		COUNT(*) AS count_custom_key
		, CONCAT(visitid, visitstarttime, fullvisitorid, pageviews, timeonsite, units_sold, bounces) AS custom_key
	FROM analytics
	GROUP BY 2
	)
WHERE count_custom_key > 1
;
<!-- Calculate revenue check 

results show that calculated_revenue <> revenue
-->
SELECT 
visitid, SUM(units_sold*unit_price) as calcuated_revenue, revenue
FROM analytics
WHERE units_sold IS NOT NULL AND unit_price IS NOT NULL AND revenue IS NOT NULL
GROUP BY visitid, revenue
ORDER BY visitid

<!-- Count prouductsku in product table vs sales_by_sku 

COUNTS do not match
-->
SELECT COUNT(DISTINCT productsku)
FROM sales_by_sku
UNION
SELECT COUNT(DISTINCT sku)
FROM products

<!-- Normalization -->
sales_report = products JOIN sales_by_sku, however with missing rows

v2productname exists in all_sessions but a simliar field name exists in the product table
