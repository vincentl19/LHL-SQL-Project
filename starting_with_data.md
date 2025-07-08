**Question 1: Find all duplicate records**

SQL Queries:

<!-- For analytics table-->
```sql
SELECT * 
FROM(
	SELECT 
		COUNT(*) AS count_custom_key
		, CONCAT(visitid, visitstarttime, fullvisitorid, pageviews, timeonsite, bounce) AS custom_key
	FROM analytics
	GROUP BY 2
	)
WHERE count_custom_key != 1
```

<!-- For products table-->
```sql
SELECT DISTINCT sku
FROM products
/* no duplicate records */
```

<!-- For sales_by_sku table-->
```sql
SELECT DISTINCT productsku
FROM sales_by_sku
/* no duplicate records */
```

Answer: 
Duplicate records were found in the analytics table by creating a custom primary key through concatenating fields that should be unique for each visitid


**Question 2: Find the total number of unique visitors (fullVisitorID)**

SQL Queries:
```sql
SELECT COUNT(*)
FROM (
	SELECT DISTINCT fullvisitorid
	FROM analytics
	--120018 ids
	UNION
	SELECT DISTINCT fullvisitorid
	FROM all_sessions
	--14223 ids
)
```

Answer:
There are 130,345 unique fullVisitorIds that are present in the analytics table and all_sessions table. Assuming all fullvisitorids are unique and correct, a UNION was used to find the total number of unique visitors.



**Question 3: Find the total number of unique visitors by referring sites**

SQL Queries:
```sql
SELECT COUNT(DISTINCT fullvisitorid), pagepathlevel1
FROM all_sessions
GROUP BY pagepathlevel1
ORDER BY 1 DESC
```

Answer:
Used COUNT DISTINCT to only consider unique visitors



**Question 4: Find each unique product viewed by each visitor**


SQL Queries:
```sql
SELECT 
  fullvisitorid
  , ARRAY_AGG(DISTINCT productsku) AS product_list
FROM all_sessions
GROUP BY fullvisitorid
ORDER BY fullvisitorid
```

Answer:
DISTINCT productsku to get unique productsskus viewed. ARRAY_AGG to get a list of unique productskus per fullvisitorid


**Question 5: Compute the percentage of visitors to the site that actually makes a purchase**

SQL Queries:
```sql
/* get visitors with orders */
WITH visitors_with_orders AS (
  SELECT COUNT(DISTINCT fullvisitorid) AS visitors_with_orders
  FROM analytics
  WHERE revenue IS NOT NULL AND units_sold IS NOT NULL
),
/* get total number of visitors */

all_visitors AS (
  SELECT COUNT(DISTINCT fullvisitorid) AS all_visitors
  FROM analytics
)
/* divide CTE 1 with CTE 2 to find percentage */
SELECT 
  ROUND(
    (vwo.visitors_with_orders * 100.0) / av.all_visitors, 
    2
  ) AS percent_visitors_with_orders
FROM visitors_with_orders vwo, all_visitors av;
```

Answer:
4.83% of all visitors actually made a purchase.
