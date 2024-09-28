# Answer the following questions and provide the SQL queries used to find the answer.

> For each question there is the option to remove the City values that are not available in demo dataset. Since most questions asked for city and country I left them in so that we could get accurate country numbers. 
    
## Question 1: Which cities and countries have the highest level of transaction revenues on the site?


### SQL Queries:
    SELECT country, city, SUM(totaltransactionrevenue) as revenue, RANK() OVER (ORDER BY SUM(totaltransactionrevenue) DESC) as Rank
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP BY city, country
    ORDER BY revenue DESC
### Answer:

There are data points where the product quantity or product price is present but no value is in total transaction revenue. There is not a clear way to tell what sessions ended in a purchase and what sessions ended with products in shopping cart.

TOP 5: 
1. Not available in demo dataset, United States
2. San Francisco, United States
3. Sunnyvale, United States
4. Atlanta, United States
5. Palo Alto, United States


## Question 2: What is the average number of products ordered from visitors in each city and country?


### SQL Queries:
> For this question I am assuming that any time a product quantity is listed, it was ordered. 

    SELECT country, city, ROUND(AVG(productquantity),2) as avgproducts
    FROM all_sessions
    WHERE productquantity IS NOT NULL --optional AND city != 'not available in demo set'
    GROUP BY city, country
    ORDER BY revenue DESC;

    SELECT country, ROUND(AVG(productquantity),2) as  avgproducts
    FROM all_sessions
    WHERE productquantity IS NOT NULL --optional AND city != 'not available in demo set'
    GROUP BY country
    ORDER BY avgproducts DESC; 

### Answer:

#### City Answer

|country|city|avg|
|----|:----:|---:|
| United States |   not available in demo dataset   | 10.58 |
| Spain   | Madrid   | 10.00 |
|United States    |Salem |    8.00|
|United States    |Atlanta|     4.00|
|United States    |Houston |    2.00|
|United States    |New York |    1.17|
|United States    |Dallas   | 1.00|
|United States    |Detroit   | 1.00|
|Ireland    |Dublin |    1.00|
|United States    |Columbus  |  1.00|
|United States   | Los Angeles   | 1.00|
|United States    |Chicago   | 1.00|
|United States    |Mountain View  |  1.00|
|United States    |(not set)  |  1.00|
|United States    |Palo Alto   | 1.00|
|India    |Bengaluru   | 1.00|
|United States    |San Francisco  |  1.00|
|United States    |San Jose  |  1.00|
|United States    |Seattle  |  1.00|
|United States    |Sunnyvale  |  1.00|
|Argentina     |Not available in demo dataset |   1.00|
|Canada     |Not available in demo dataset   | 1.00|
|Colombia     |Not available in demo dataset  |  1.00|
|Finland    |Not available in demo dataset   | 1.00|
|France    |Not available in demo dataset   | 1.00|
|Mexico    |Not available in demo dataset   | 1.00|
|United States    |Ann Arbor   | 1.00|

#### Country Answer
|number|country|average|
|---|----|----|
|1  | Spain	|    10.00|
|2	|United States	       |4.02|
|3	|Colombia	|1.00|
|4	|Finland	|1.00|
|5	|France	|1.00|
|6	|Argentina	|1.00|
|7	|Ireland	|1.00|
|8	|Mexico	|1.00|
|9	|India	|1.00|
|10|	Canada	|1.00|


## Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

> For the purpose of this question I am assuming that each row item that has a category is a ordered product.
### SQL Queries:

#### Checking the category name against city name and getting which city mentions that category the most
    WITH categorycounts AS (SELECT DISTINCT v2productcategory, city, COUNT(*) as categorycount, RANK() OVER (PARTITION BY v2productcategory ORDER BY COUNT(*) DESC) as rank
    FROM all_sessions
    GROUP BY v2productcategory, city)

    SELECT *
    FROM categorycounts
    WHERE rank = 1
    ORDER BY categorycount DESC

#### Looking at each city's most ordered category, calculating the percent of all sales of that category to that city

    WITH categorycounts AS (
    SELECT DISTINCT city, v2productcategory, COUNT(*) as categorycount, RANK() OVER (PARTITION BY city ORDER BY COUNT(*) DESC) as rank
    FROM all_sessions
    GROUP BY city, v2productcategory), 

    categorytotals AS (
    SELECT  v2productcategory, COUNT(*) as total
    FROM all_sessions
    GROUP BY v2productcategory
    )

    SELECT c.city, c.v2productcategory, c.categorycount, t.total, CAST (c.categorycount AS real)/t.total*100 AS percentage
    FROM categorycounts c
    JOIN categorytotals t
    ON c.v2productcategory=t.v2productcategory
    WHERE rank = 1
    ORDER BY percentage desc

#### Switching the analysis to country
    WITH categorycounts AS (
    SELECT DISTINCT country, v2productcategory, COUNT(*) as categorycount, RANK() OVER (PARTITION BY country ORDER BY COUNT(*) DESC) as rank
    FROM all_sessions
    GROUP BY country, v2productcategory), 

    categorytotals AS (
    SELECT  v2productcategory, COUNT(*) as total
    FROM all_sessions
    GROUP BY v2productcategory
    )

    SELECT c.country, c.v2productcategory, c.categorycount, t.total, CAST (c.categorycount AS real)/t.total*100 AS percentage
    FROM categorycounts c
    JOIN categorytotals t
    ON c.v2productcategory=t.v2productcategory
    WHERE rank = 1
    ORDER BY percentage desc



### Answer:

When we zoom out to the country level we see that the following: 
- United States accounts for 47.37% of all Men's T-shirts'  
- The UK, India, Germany, and Canada accounts for about 30% of all home/shop by brand/YouTube/  

## Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


### SQL Queries:

#### Top selling products in each city
    WITH skubycity AS (SELECT productsku, city, COUNT (*), RANK() OVER(PARTition BY city ORDER BY COUNT(*) DESC)
    FROM all_sessions
    GROUP BY productsku, city)

    SELECT p.name, s.city, s.count
    FROM skubycity s
    JOIN products p
    ON s.productsku=p.sku
    WHERE rank = 1
    ORDER BY count DESC

#### Top selling products in each country
    WITH skubycountry AS (SELECT productsku, country, COUNT (*), RANK() OVER(PARTition BY country ORDER BY COUNT(*) DESC)
    FROM all_sessions
    GROUP BY productsku, country)

    SELECT p.name, s.country, s.count
    FROM skubycountry s
    JOIN products p
    ON s.productsku=p.sku
    WHERE rank = 1
    ORDER BY count DESC

### Answer:

When we look at the city data it looks like cameras, bottle infusers, and t-shirts are the most popular items. 
The country results show similar results but adds in the twill cap and bottle infuser. Both answers show that the hero tee should be the primary product if margins on it are the same as the bottle infuser. 

## Question 5: Can we summarize the impact of revenue generated from each city/country?

### SQL Queries:

#### City and Country breakdowns of revenue generated and percent of all revenue it accounts for

    SELECT city, SUM(totaltransactionrevenue) as sum, 
        ROUND((SUM(totaltransactionrevenue)/(SELECT SUM(totaltransactionrevenue) FROM all_sessions) * 100),2) as percentofsales
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP BY (city)
    ORDER BY sum desc

    SELECT country, SUM(totaltransactionrevenue) as sum, 
        ROUND((SUM(totaltransactionrevenue)/(SELECT SUM(totaltransactionrevenue) FROM all_sessions) * 100),2) as percentofsales
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP BY (country)
    ORDER BY sum desc

## Answer:

The country results show that a significant amount of revenue comes from the United States. The remaining countries account for less than 10% of revenue. The United States is the revenue. 

The city results show that 3 of the top 5 are in California, accounting for 20% of total revenue. This compares to the 42% from not available in demo dataset. 
