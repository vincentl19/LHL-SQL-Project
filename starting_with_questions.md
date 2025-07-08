**Answer the following questions and provide the SQL queries used to find the answer.**

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries: 

<!-- Countries with highest transaction revenues -->
```sql
/* get total revenue by country */
WITH country_totaltransactionrevenue AS (
	SELECT 
        country
        , CAST(SUM(totaltransactionrevenue) AS money) totaltransactionrevenue
	FROM all_sessions
	GROUP BY country
)
/* remove rows with nulls */
SELECT
	country
	, totaltransactionrevenue
FROM country_totaltransactionrevenue
WHERE totaltransactionrevenue IS NOT NULL
ORDER BY totaltransactionrevenue DESC
;
```

<!-- Cities with highest transaction revenues -->
```sql
/* get total revenue by city */

WITH city_totaltransactionrevenue AS (
	SELECT 
		city
		, CAST(SUM(totaltransactionrevenue) AS money) totaltransactionrevenue
	FROM all_sessions
	GROUP BY city
)
/* remove rows with nulls */
SELECT
	city
	, totaltransactionrevenue
FROM city_totaltransactionrevenue
WHERE 1=1
	AND totaltransactionrevenue IS NOT NULL
	AND city NOT LIKE '%not %'
ORDER BY totaltransactionrevenue DESC
;
```
Answer:

totaltransactionrevenue was used to calculte the highest transaction revenue for countries and cities because:
(1) totaltransactionrevenue had less null values compared to transactionrevenue
(2) Unreliable data within the analytics table, where productquantity * productprice != productrevenue OR unit_price * units_sold != revenue

Countries and cities were considered separately, due to inconsistencies between the correct matching of all cities to the correct countries. As long as the value for country or city was not null, the revenue was accounted for.


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
<!-- Avg products ordered in each country -->
```sql
/* CTE to find total products ordered */
WITH units_sold AS (
	SELECT 
		a.fullvisitorid AS fullvisitorid
		, city
		, country
		, SUM(units_sold) as total_units_sold
	FROM analytics a
	JOIN all_sessions als USING(fullvisitorid)
	WHERE 1=1
		AND units_sold IS NOT NULL
		AND country NOT LIKE '%not %'
	GROUP BY 1,2,3
	ORDER BY a.fullvisitorid
)
/* Find avg total products ordered by country */

SELECT country, ROUND(AVG(total_units_sold)) AS avg_units_sold
FROM units_sold
GROUP BY 1
ORDER BY avg_units_sold DESC
```

<!-- Avg products ordered in each city -->
```sql

/* CTE to find total products ordered */
WITH units_sold AS (
	SELECT 
		a.fullvisitorid AS fullvisitorid
		, city
		, country
		, SUM(units_sold) as total_units_sold
	FROM analytics a
	JOIN all_sessions als USING(fullvisitorid)
	WHERE 1=1
		AND units_sold IS NOT NULL
		AND city NOT LIKE '%not %'
	GROUP BY 1,2,3
	ORDER BY a.fullvisitorid
)
/* Find avg total products ordered by city */
SELECT city, ROUND(AVG(total_units_sold)) AS avg_units_sold
FROM units_sold
GROUP BY 1
ORDER BY avg_units_sold DESC
```

Answer:

units_sold was used to represent products ordered by visitors, because this was the only variable that could be related back to the visitor's city or country. Other variables such as productsku, orderedquantity, and total_ordered were already aggreagted values that could not be linked back to indvidual orders. The average orders by city and country was derived by first gathering total orders by fullvisitorid, which had a reliable relationship to city and country. As with the first question, avg orders by city and country were calculated independantly.



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
/* CTE to find total products ordered */
WITH analytics_products_sold AS (
    SELECT visitid, SUM(units_sold) total_units_sold
    FROM analytics
    GROUP BY visitid
    HAVING SUM(units_sold) IS NOT NULL
)
/* JOIN CTE above to CTE below to obtain geography values */
, clean_analytics_cte AS (
SELECT 
	aps.visitid AS visit_id
	, aps.total_units_sold AS total_units_sold
	, city
	, country
	, v2productcategory
FROM analytics_products_sold aps
JOIN all_sessions als USING(visitid)
WHERE 1=1
 AND city NOT LIKE '%not %'
 AND country NOT LIKE '%not %'
ORDER BY total_units_sold
)
/* Organize by product category to find pattern */
SELECT 
	city
	, country
	, v2productcategory AS product_category
	, SUM(total_units_sold) as total_products_sold
FROM clean_analytics_cte
WHERE 1=1
	AND v2productcategory NOT LIKE '%not %'
GROUP BY 1,2,3
ORDER BY 2;
```


Answer:

Assumptions:
(1) analytics.units_sold = all_sessions.productquantity when joined USING(visitid)
(2) Only one type of product was ordered per visitid

Based on query above that shows total products ordered by city, country, and type, there is no pattern in the types of products ordered from city to city or country to country. There is not enough evidence or context regarding the deifinitions of all of the variables and others, such as date (seasonality) or sentimentscore, that may impact how the data is interpreated. Without sufficient definitions for each variable, the null hypothesis must be accepted.


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

<!-- Top-selling products from each country -->
```sql

/* CTE to get total products sold removing nulls */
WITH analytics_products_sold AS (
SELECT visitid, SUM(units_sold) total_units_sold
FROM analytics
GROUP BY visitid
HAVING SUM(units_sold) IS NOT NULL
)
/* CTE to obtain city, country, product name removing nulls */
, clean_analytics_cte AS (
SELECT 
	aps.visitid AS visit_id
	, aps.total_units_sold AS total_units_sold
	, city
	, country
	, v2productname
FROM analytics_products_sold aps
JOIN all_sessions als USING(visitid)
WHERE 1=1
 AND city NOT LIKE '%not %'
 AND country NOT LIKE '%not %'
ORDER BY total_units_sold
)

/* combine two above CTEs into one CTE */
, total_products_sold_cte AS (
SELECT 
	country
	, v2productname AS product_name
	, SUM(total_units_sold) as sum_total_products_sold
FROM clean_analytics_cte
WHERE 1=1
	AND v2productname NOT LIKE '%not %'
GROUP BY 1,2
ORDER BY 1
)

/* final query to get required fields */
SELECT
	country
	, product_name
	, sum_total_products_sold
	, product_rank
FROM (
    /* subquery to rank total products sold */
	SELECT 
		country
		, product_name
		, sum_total_products_sold
		, RANK()OVER(PARTITION BY country ORDER BY sum_total_products_sold DESC) as product_rank
	FROM total_products_sold_cte
	GROUP BY 1,2,3
	ORDER BY 1
) ranked
WHERE product_rank = 1
;
```


<!-- Top-selling products from each city -->
```sql
/* CTE to get total products sold removing nulls */
WITH analytics_products_sold AS (
SELECT visitid, SUM(units_sold) total_units_sold
FROM analytics
GROUP BY visitid
HAVING SUM(units_sold) IS NOT NULL
)
/* CTE to obtain city, country, product name removing nulls */
, clean_analytics_cte AS (
SELECT 
	aps.visitid AS visit_id
	, aps.total_units_sold AS total_units_sold
	, city
	, country
	, v2productname
FROM analytics_products_sold aps
JOIN all_sessions als USING(visitid)
WHERE 1=1
 AND city NOT LIKE '%not %'
 AND country NOT LIKE '%not %'
ORDER BY total_units_sold
)
/* combine two above CTEs into one CTE */
, total_products_sold_cte AS (
SELECT 
	city
	--, country
	, v2productname AS product_name
	, SUM(total_units_sold) as sum_total_products_sold
FROM clean_analytics_cte
WHERE 1=1
	AND v2productname NOT LIKE '%not %'
GROUP BY 1,2
ORDER BY 1
)
/* final query to get required fields */
SELECT
	city
	, product_name
	, sum_total_products_sold
	, product_rank
FROM (
        /* subquery to rank total products sold */
	SELECT 
		city
		--, country
		, product_name
		, sum_total_products_sold
		, RANK()OVER(PARTITION BY city ORDER BY sum_total_products_sold DESC) as product_rank
	FROM total_products_sold_cte
	GROUP BY 1,2,3
	ORDER BY 1
) ranked
WHERE product_rank = 1
;
```

Answer:
There is no pattern in the top selling product between cities. However, the top selling product for each country was different, however, there is not enough information to determine why the top selling product in one country is also not the top selling product in another country when comparing between neighbouring countries, and between the top ranked products within the same country.



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

```sql
SELECT 
  country, 
  city, 
  SUM(totaltransactionrevenue)::money AS total_revenue,
  ROUND(100.0 * SUM(totaltransactionrevenue) / SUM(SUM(totaltransactionrevenue)) OVER (), 2) AS revenue_percent
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL AND city NOT LIKE '%not %'
GROUP BY country, city
ORDER BY total_revenue DESC;
```

Answer:

The majority of revenue (86%) from this ecommerce site is generated from the United States, with the second highest country by revenue being Isreal at 7%. Of the revenue generated from the United States, a large porition is generated from cities in the state of California - 4 of the top 10 cities being located in there. The cities with the least revenue generated seem to be located on the east side of North America and as a result, the demand from these areas may not be as high due to factors such as lack of awareness, climate, etc.






