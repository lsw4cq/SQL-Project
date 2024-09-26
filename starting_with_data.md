Question 1: What products should be ordered now? 

SQL Queries:

WITH monthlyorders AS (SELECT productsku as sku, v2productname as name, 
	extract(month from date) as monthordered, 
	COUNT(*) OVER(Partition BY extract(month from date)) as num_ordered
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL
GROUP BY productsku, v2productname, EXTRACT(month FROM date)),

leadtimeorders AS (SELECT a.productsku as sku, v2productname as name,
	EXTRACT(month from a.date + p.restockingleadtime) AS leadmonth,
	COUNT(*) OVER(Partition BY extract(month from a.date + p.restockingleadtime)) as num_ordered
FROM all_sessions a
JOIN products p
ON a.productsku=p.sku
WHERE totaltransactionrevenue IS NOT NULL AND a.date IS NOT NULL
GROUP BY a.productsku, v2productname, EXTRACT(month FROM a.date + p.restockingleadtime)),

combinedorders AS (SELECT * FROM monthlyorders UNION SELECT * FROM leadtimeorders),

combinedtotals AS (SELECT sku, name, SUM(num_ordered) as totalordered, monthordered
FROM combinedorders
GROUP BY monthordered, sku, name)

SELECT p.name, p.stocklevel, m.totalordered
FROM products p
JOIN combinedtotals m
ON p.sku=m.sku
WHERE 
	 (p.stocklevel <= m.totalordered
	AND
	extract(month from CURRENT_DATE) + CASE 
		WHEN extract(month from CURRENT_DATE) = 12 
		THEN -11 
		ELSE 1 END = m.monthordered)


Answer: 
When we look at what was ordered last year in the same month as now and for the products with over a month of restocking lead time we find that two products need to be ordered

Men's Short Sleeve Badge Tee Charcoal (9 in stock, 10 ordered historically in October)
Women's Short Sleeve Hero Tee Heather (1 in stock, 6 ordered historically in October)


Question 2: What products are in stock and have never been ordered? 

SQL Queries:

Answer:



Question 3: What channel generated the most revenue? 

SQL Queries:

Answer:



Question 4: What areas (regions) should they focus on to expand growth?  

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
