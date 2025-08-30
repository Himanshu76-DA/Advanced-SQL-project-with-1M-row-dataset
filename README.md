# Advanced-SQL-project-with-1M-row-dataset
This project demonstrates my strong foundation in SQL along with advanced-level queries. Built on a dataset of 1M+ rows, it highlights both my core SQL skills and ability to work with complex queries for real-world analysis


## 📂 Files  

- **Dataset:** [Download Here](./CAR_SALES_1M.zip)  
- **SQL Project File:** [View Here](./SQL_ADVANCE_PROJECT_QUERY_FILE.sql)


## SQL Project
Here are my 23 questions in which I used advanced SQL topics and core logic to solve.

## **Task1 : Find the top 3 brands in each country by total sales (SUM(price)), ordered by rank (highest sales first)**
```sql
WITH MY_CTE AS (SELECT COUNTRY,BRAND,
                SUM(sale)  AS TOTAL_SALE from car_sales
                group by country,BRAND),
				
MY_CTE2 AS (SELECT *,RANK() OVER(PARTITION BY COUNTRY 
            ORDER BY TOTAL_SALE DESC)
			 AS RNK FROM MY_CTE)
			 
SELECT * FROM MY_CTE2
WHERE RNK IN(1,2,3
)
```
## Task2 : For each brand, find the latest year’s top 2 cars by sale in each country, but only if that country sold more than 100 cars total.
```sql
WITH CTE AS(
			SELECT *,RANK() OVER(PARTITION BY BRAND,COUNTRY
			ORDER BY TOTAL_SALE DESC) AS RNK
			FROM(
				SELECT BRAND,COUNTRY,MODEL,SUM(SALE) AS TOTAL_SALE
				FROM CAR_SALES
				WHERE YEAR = (SELECT MAX(YEAR) FROM CAR_sALES)
				AND
      			    COUNTRY IN (SELECT COUNTRY FROM CAR_SALES
	  			  				GROUP BY COUNTRY
	              				HAVING COUNT(COUNTRY) > 100)
			GROUP BY BRAND,COUNTRY,MODEL))
SELECT * FROM CTE
WHERE RNK IN (1,2)
```


##Task3 : For each brand, find the average sale price per year and show the difference compared to the previous year’s average
```sql
WITH CTE AS(
	SELECT BRAND,YEAR,ROUND(AVG(SALE),2)AS AVG_SALE
	FROM CAR_SALES
	GROUP BY BRAND,YEAR)

SELECT *, LAG(AVG_SALE,1) OVER(PARTITION BY BRAND) AS PREV_YEAR_AVG,
           AVG_SALE -  LAG(AVG_SALE,1) OVER(PARTITION BY BRAND)AS "CURRENT-PREVIOUS"
		   FROM CTE
```


## Task 4: For each country, find the brand with the highest average sale price.
```sql
WITH MY_CTE AS(
		SELECT COUNTRY,BRAND,ROUND(AVG(SALE),2) AS AVG_SALE,ROW_NUMBER() 
		OVER(PARTITION BY COUNTRY ORDER BY AVG(SALE) DESC) AS RNK 
		FROM CAR_SALES
		GROUP BY COUNTRY,BRAND)

SELECT * FROM MY_CTE
		WHERE RNK = 1
```
## Task 5 : For each brand, calculate the total sales per country and then find the  percentage contribution of each country to that brand’s overall sales.
```sql
WITH MY_CTE AS (
             SELECT BRAND,COUNTRY,SUM(SALE) AS TOTAL_SALE FROM CAR_SALES
			 GROUP BY BRAND,COUNTRY
),
TOTAL_SALES AS(
             SELECT BRAND,SUM(sALE) AS TOTAL_SALE_BY_BRAND
			 FROM CAR_SALES 
			 GROUP BY BRAND
)
SELECT MY_CTE.BRAND,
       MY_CTE.COUNTRY,
	   MY_CTE.TOTAL_SALE,
	   ROUND((MY_CTE.TOTAL_SALE/TOTAL_SALES.TOTAL_SALE_BY_BRAND)* 100,2) AS PERCENT_HOLD
	   FROM MY_CTE
	   JOIN TOTAL_SALES
	   ON MY_CTE.BRAND = TOTAL_SALES.BRAND
```
## TASK 6 :  For each year, find the top 2 brands by total sales in each country.
```sql
WITH MY_CTE AS (
			SELECT YEAR,COUNTRY,BRAND,SUM(SALE) AS TOTAL_SALE,
			RANK() OVER(PARTITION BY YEAR,COUNTRY 
			ORDER BY SUM(SALE) DESC) AS RNK 
			FROM CAR_SALES
			GROUP BY YEAR,COUNTRY,BRAND
			ORDER BY YEAR)

SELECT * FROM MY_CTE
WHERE RNK IN (1,2)
ORDER BY YEAR ASC
```
## TASK 7: For each brand, find the year with the maximum total sales and show the sales amount in that year
```sql
WITH MY_CTE AS(
		SELECT BRAND,YEAR,SUM(SALE) AS TOTAL_SALE,RANK() 
        OVER(PARTITION BY BRAND ORDER BY SUM(SALE) DESC) AS RNK
		FROM CAR_SALES
		GROUP BY BRAND,YEAR)
SELECT BRAND,YEAR,TOTAL_SALE FROM MY_CTE
WHERE RNK  = 1
```

## Task 8 : For each brand in each country, find the top 2 most expensive cars sold (based on sale).
```sql
WITH MY_CTE AS(
         SELECT BRAND,COUNTRY,MODEL,SUM(SALE) AS TOTAL_SALE,
  		 ROW_NUMBER() OVER(PARTITION BY BRAND,COUNTRY ORDER BY SUM(SALE) DESC) AS RNK FROM CAR_SALES
		 GROUP BY BRAND,COUNTRY,MODEL)
SELECT  BRAND,COUNTRY,MODEL,TOTAL_SALE FROM MY_CTE
WHERE RNK IN (1,2)
```

## TASK 9: Find all brands where the total sales in 2024 are higher than  the average total sales of that brand across all years.
```sql
WITH "2024_DATA" AS(
        SELECT BRAND,SUM(SALE) AS "TOTAL_SALE(2024)"
        FROM CAR_SALES
        WHERE YEAR = 2024
        GROUP BY BRAND),
		
YEARLY_TOTAL AS(
        SELECT BRAND,YEAR,SUM(SALE) AS YEARLY_TOTAL_SALE
        FROM CAR_SALES
        GROUP BY BRAND,YEAR),

B_AVG AS(
        SELECT BRAND,ROUND(AVG(YEARLY_TOTAL_SALE),2) AS AVG_YEARLY_SALE
        FROM YEARLY_TOTAL
        GROUP BY BRAND)

SELECT D24.BRAND,D24."TOTAL_SALE(2024)",
	   B_AVG.AVG_YEARLY_SALE
	   FROM "2024_DATA" AS D24
	   JOIN B_AVG
       ON D24.BRAND = B_AVG.BRAND
	   WHERE D24."TOTAL_SALE(2024)" > B_AVG.AVG_YEARLY_SALE;
```

## Task10 : Find all customers (use first_name, last_name, address) who bought cars  of more than 1 brand in the same year. Show first_name, last_name, address, year, and brand_count."""
```sql
SELECT FIRST_NAME,LAST_NAME,ADDRESS,YEAR,
	   COUNT(DISTINCT BRAND) AS CNT
	   FROM CAR_SALES
	   GROUP BY FIRST_NAME,LAST_NAME,ADDRESS,YEAR
	   HAVING COUNT(DISTINCT BRAND)>1
```

## Task 11: For each brand, calculate the cumulative sales over the years (running total). Show brand, year, total_sale_year, and cumulative_sale"""
```sql
SELECT BRAND,YEAR,SUM(SALE) AS TOTAL_SALE,
	   SUM(SUM(SALE)) OVER(PARTITION BY BRAND ORDER BY YEAR) AS CUMU_SUM
	   FROM CAR_SALES
	   GROUP BY BRAND,YEAR
	   ORDER BY BRAND,YEAR
```

## Task 12 : Find all cars where the customer’s first name starts with ‘A’ or last name ends with ‘n’, and the car color contains ‘red’ (case-insensitive)"""
```sql
select * from car_sales
WHERE (UPPER(FIRST_NAME) LIKE 'A%' OR
      LOWER(LAST_NAME) LIKE'%n') AND COLOR ILIKE '%Red%'

```

## TASK 13: For each brand, calculate the percentage change in total sales  compared to the previous year. Show brand, year, total_sale, prev_year_sale, and pct_change."""

```sql
WITH BRAND_YOY
AS(
	SELECT  BRAND,YEAR,TOTAL_SALE,
		LAG(TOTAL_SALE) OVER(PARTITION BY BRAND) AS PRE_YEAR_SALE
		FROM(
			 SELECT BRAND,YEAR,SUM(SALE) AS TOTAL_SALE FROM CAR_SALES
			 GROUP BY BRAND,YEAR
			 ORDER BY BRAND,YEAR))
SELECT *,ROUND(((TOTAL_SALE - PRE_YEAR_SALE) / PRE_YEAR_SALE) * 100, 2) AS "YOY(%)" FROM BRAND_YOY
```

## TASK 14: For each brand, find the average sale price per year and  show the difference compared to the previous year’s average"""
```sql
SELECT *,
LAG(AVG_SALE) OVER(PARTITION BY BRAND ORDER BY YEAR) AS PRE_YEAR_AVG_SALE,
AVG_SALE - LAG(AVG_SALE) OVER(PARTITION BY BRAND ORDER BY YEAR) AS DIFFERENCE
FROM(
	 SELECT BRAND,YEAR, ROUND(AVG(SALE),2) AS AVG_SALE
	   		FROM CAR_SALES
	   		GROUP BY BRAND,YEAR)

```
## TASK 15: Customer Segmentation by Spend
## Classify customers into High Spender / Medium Spender / Low Spender categories based on their total purchase value.
## High Spender = total sale ≥ 1,000,000
## Medium Spender = total sale between 500,000 and 999,999
## Low Spender = total sale < 500,000"""
```sql
select FIRST_NAME,LAST_NAME,SUM(SALE),(CASE WHEN SUM(SALE) >= 1000000 THEN 'HIGH SPENDER'
								   WHEN SUM(SALE) >= 500000 THEN 'MEDIUM SPENDER'
								   ELSE 'LOW SPENDER'
							  END)
								   AS STATUS
								   from car_sales
								   GROUP BY FIRST_NAME,LAST_NAME,ADDRESS

```

## TASK 16 : Generate a sequence of years from 2001 to 2024 using a recursive CTE. Show column: year"
```sql
WITH RECURSIVE CTE AS(
		SELECT 2001 AS N
		UNION ALL
		SELECT N+1 FROM CTE
		WHERE N <2025
)
SELECT * FROM CTE
```

## TASK 17: List all pairs of customers who live in the same city (use address field)."
```sql
SELECT DISTINCT C.FIRST_NAME, C.LAST_NAME, C.COUNTRY
FROM CAR_SALES AS C
JOIN CAR_SALES AS C1
  ON C.COUNTRY = C1.COUNTRY
  AND C.FIRST_NAME <> C1.FIRST_NAME
  AND C.LAST_NAME <> C1.LAST_NAME;

```

## TASK 18: Write a query to get total sales per brand.
Then explain how you’d optimize it (indexes, avoiding unnecessary sorting, etc.)."
```sql
"FIRST WE CREATE INDEX ON BRAND TO SAVE TIME"
CREATE INDEX IDX_BRAND_CAR_SALES 
ON CAR_SALES(BRAND) "AS IN QUESTION ASKED ONLY FOR BRAND"
"AFTER CREATING INDEX NOW LETS MOVE TO THE QUESTION QUERY"
SELECT BRAND,SUM(SALE)AS TOTAL_SALE
FROM CAR_SALES
GROUP BY BRAND
```


## TASK 19: For each brand and year, calculate the total yearly sales.
Then compute a 3-year moving average of sales (current year + previous 2 years)."
```sql
SELECT *,
	ROUND(AVG(TOTAL_SALE) 
	OVER(PARTITION BY BRAND ORDER BY YEAR ASC
	ROWS BETWEEN 2 PRECEDING AND CURRENT ROW),2) AS MOVING_AVG_3YEAR
FROM(
	 SELECT BRAND,YEAR,SUM(SALE)AS TOTAL_SALE
	 FROM CAR_SALES
	 GROUP BY BRAND,YEAR)
```

## TASK 20: Write a query or procedure that takes a column name as input (brand or country)
and returns the total sales grouped by that column dynamically."
```sql
CREATE OR REPLACE FUNCTION F1(X TEXT)
RETURNS TABLE(group_column TEXT, TOTAL_SALE NUMERIC)
LANGUAGE PLPGSQL
AS $$
BEGIN
	  RETURN QUERY EXECUTE FORMAT(
	  'SELECT %I,SUM(SALE) FROM CAR_SALES
	  GROUP BY %I',X,X
	  );
END;
$$

"CALLING THE FUNCTION"

SELECT * FROM F1('brand')

```
## TASK 21: CREATE A FUNCTION THAT accepts a brand + year and returns
total sales, avg sale, min sale, max sale"
```sql
CREATE OR REPLACE FUNCTION F4(COL1 TEXT,COL2 TEXT)
RETURNS TABLE(CO1 TEXT,CO2 text,TOTAL_SALES NUMERIC,AVG_SALE NUMERIC,MIN_SALE NUMERIC,MAX_SALE NUMERIC)
LANGUAGE PLPGSQL 
AS
$$
BEGIN RETURN QUERY EXECUTE FORMAT
			('SELECT %I,%I,
					 SUM(SALE) as TOTAL_SALE,
					 AVG(SALE) AS AVG_SALE,
					 MIN(SALE) AS MIN_SALE,
					 MAX(SALE) AS MAX_SALE
					 FROM CAR_SALES
					 GROUP BY %I,%I',
					 COL1,COL2,COL1,COL2);
END;
$$;
SELECT * FROM F4('brand','year')
```

## TASK 22: Create a materialized view that stores the total sales per brand per year from the car_sales table.
After inserting new sales data into car_sales, demonstrate how to refresh the materialized view so it reflects the latest data.
Query the materialized view to show the top 3 brands per year based on total sales"
```sql
--- 1. CREATING MATERIALIZED VIEW:
CREATE MATERIALIZED VIEW MVIEW AS 
SELECT BRAND,YEAR,SUM(SALE) AS TOTAL_SALE
FROM CAR_SALES
GROUP BY BRAND,YEAR
ORDER BY BRAND,YEAR

--- 2. UPDATING TABLE WITH A HIGH NOTICABLE VALUE IN TRANSACTION TO AVOID DATA DISTURBANCE

BEGIN TRANSACTION;
INSERT INTO CAR_SALES 
VALUES('Audi','XYZ',2000,2000000000,788,'RED','USED','XYZ','XYY','XYZ','ZYX')


--- 3. REFRESHING THE MATERIALIZED VIEW (MVIEW)
REFRESH MATERIALIZED VIEW MVIEW

--- 4. NOW LETS SEE THE CHANGES
SELECT * FROM MVIEW

--- 5. LETS ROLLBACK THE TRANSACTION
ROLLBACK;

--- 6.TOP 3 BRANDS PER YEAR BY SALES
SELECT * FROM (
			SELECT *,RANK() OVER(PARTITION BY YEAR 
			ORDER BY TOTAL_SALE DESC)
			AS RNK
			FROM MVIEW)
WHERE RNK IN (1,2,3)

```
## TASK 23: For each brand, find the first year in which the brand’s total sales exceeded 1,000,000. 
Show the brand, year, and total sales for that year."
```sql
WITH MY_CTE AS(
				SELECT *,ROW_NUMBER() 
				OVER(PARTITION BY BRAND  
				ORDER BY YEAR ASC) AS FRNK
				FROM (SELECT BRAND,YEAR,
						SUM(SALE)AS TOTAL_SALE
						FROM CAR_SALES
						GROUP BY BRAND,YEAR
						ORDER BY BRAND,YEAR)
				WHERE TOTAL_SALE > 1000000
				ORDER BY BRAND,YEAR)
SELECT * FROM MY_CTE
WHERE FRNK = 1
```
