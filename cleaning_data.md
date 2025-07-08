**What issues will you address by cleaning the data?**
1. Duplicate rows
2. Removing null values
3. Get distinct ids




**Queries:**
**Below, provide the SQL queries you used to clean your data.**
  
    WHERE revenue IS NOT NULL AND units_sold IS NOT NULL

    WHERE 1=1
	AND v2productcategory NOT LIKE '%not %'

    SELECT DISTINCT fullvisitorid

    HAVING SUM(units_sold) IS NOT NULL


