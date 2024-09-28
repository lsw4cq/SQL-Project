# Question 1: What products should be ordered now? 

## SQL Queries:

    WITH monthlyorders AS (SELECT productsku as sku, v2productname as name, 
	    extract(month from date) as monthordered, 
	COUNT(*) OVER(Partition BY extract(month from date)) as num_ordered
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP BY productsku, v2productname, EXTRACT(month FROM date)),

    leadtimeorders AS (SELECT a.productsku as sku, v2productname as name,
	    EXTRACT(month from a.date + p.restockingleadtime) AS leadmonth,
        COUNT(*) OVER(Partition BY extract(month from a.date + p.      restockingleadtime)) as num_ordered
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

## Answer: 
When we look at what was ordered last year in the same month as now and for the products with over a month of restocking lead time we find that two products need to be ordered. We could add to this query a percent growth expectation since we hope that more people will order each year but I did not do that since this business is trending down. 

Men's Short Sleeve Badge Tee Charcoal (9 in stock, 10 ordered historically in October)
Women's Short Sleeve Hero Tee Heather (1 in stock, 6 ordered historically in October)
# Question 2: What channel generated the most revenue? 
## SQL Queries:

    WITH channelrevenue AS (SELECT channelgrouping, SUM(totaltransactionrevenue) as total
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP BY channelgrouping
    ORDER BY total DESC)

    SELECT channelgrouping, total, total / SUM(total) OVER () * 100 as percentage
    FROM channelrevenue

### Looking for trends by month and channel grouping

    WITH channelgroupingmonthly AS (
    SELECT EXTRACT(YEAR FROM date) AS year, EXTRACT(MONTH FROM date) AS month, channelgrouping,
    	SUM(totaltransactionrevenue) AS monthlytotal
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP BY year, month, channelgrouping),
	
    monthlytotals AS (
    SELECT year, month,
        SUM(CASE WHEN channelgrouping = 'Referral' THEN monthlytotal ELSE 0 END) AS referraltotal,
        SUM(CASE WHEN channelgrouping = 'Direct' THEN monthlytotal ELSE 0 END) AS directtotal,
        SUM(CASE WHEN channelgrouping = 'Paid Ads' THEN monthlytotal ELSE 0 END) AS paidadstotal,
        SUM(CASE WHEN channelgrouping = 'Organic Search' THEN monthlytotal ELSE 0 END) AS organictotal,
	SUM(CASE WHEN channelgrouping = 'Display' THEN monthlytotal ELSE 0 END) AS displaytotal,
	SUM(CASE WHEN channelgrouping = 'Affiliates' THEN monthlytotal ELSE 0 END) AS affiliatestotal,
	SUM(CASE WHEN channelgrouping = 'Other' THEN monthlytotal ELSE 0 END) AS othertotal,
        SUM(monthlytotal) AS totalrevenueformonth
    FROM channelgroupingmonthly
    GROUP BY year, month)
	
    SELECT year, month, referraltotal,
        referraltotal / totalrevenueformonth * 100 AS referralpercent, directtotal,
        directtotal / totalrevenueformonth * 100 AS directpercent, paidadstotal,
        paidadstotal / totalrevenueformonth * 100 AS paidadspercent, organictotal,
        organictotal / totalrevenueformonth * 100 AS organicpercent,
        displaytotal / totalrevenueformonth * 100 AS displaypercent,
        affiliatestotal / totalrevenueformonth * 100 AS affiliatespercent,
        othertotal / totalrevenueformonth * 100 AS otherpercent
    FROM monthlytotals
    ORDER BY year, month;

## Answer:
They have a pretty nice split across referrals, directs, and organic searches. Paid Searches accounted for less than 2% of revenue while the other three accounted for the rest. It isn't clear if they are paying for ads or how much they are paying for them so there is no way to make a recommendation on paid searches. 

The trend by month is wild. There is no real pattern that we can see because of the state of the data. 


# Question 3: Is this business profitable or growing? 

## SQL Queries:

### Checking to see revenue from all_sessions table broken down by month/year

    CREATE TEMP TABLE revenueovertime AS (WITH basicinfo AS (SELECT EXTRACT(YEAR FROM date) AS year, 
        EXTRACT(MONTH FROM date) AS month, 
        COUNT(*) AS rowcount
    FROM all_sessions
    WHERE date IS NOT NULL
    GROUP BY year, month
    ORDER BY year, month)

    SELECT year, month, rowcount, 
        CASE WHEN Lag(rowcount) OVER (order by year, month) IS NULL THEN NULL
        ELSE (rowcount - LAG(rowcount) OVER (Order by year, month))* 100 / LAG(rowcount) OVER (order by year, month) 
        END AS diff
    FROM basicinfo)

### Getting average growth rate for the dataset
    SELECT AVG(diff)
    FROM revenueovertime
    WHERE diff > -90 
> WHERE statement is optional but recommended because August 2017 is an outlier. 
## Answer:

On average they see a 10% drop in sales each month. There is an outlier in August 2017 records since only 39 were recorded which drops the average down significantly. Without that outlier there is a 2.9% drop each month. 