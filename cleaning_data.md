># Note: 
>## For this project I am assuming that I am the database engineer as well as the analyst and am creating tables and then altering them. 
# What issues will you address by cleaning the data?
## SKUs
* There are some SKUs listed in all_sessions that don't exist in products and skus that exist in sales_by_sku but not products 
* There are five skus that are in the sales_by_sku but aren't in all_sessions or products
* SKUs are varying length, some with characters and some numeric
## Column Types
* Price columns are bigint instead of numeric (this is because I used the import feature in pgadmin4)
## Products Table 
* Should there be negative values in the sentiment score? 
* Why are there products that have never been ordered before that have sentiment scores? And one product has NULL instead
# Queries:
Below, provide the SQL queries you used to clean your data.
### Discovering the schema
    SELECT *
    FROM information_schema.tables
    WHERE table_schema = 'public'
### Checking for column names and types across all tables
    WITH table_names AS (SELECT table_name
    FROM information_schema.tables
    WHERE table_schema = 'public')
    SELECT c.column_name, c.data_type, c.is_nullable, t.table_name
    FROM information_schema.columns c
    JOIN table_names t
    ON c.table_name=t.table_name
    WHERE c.table_name IN (t.table_name)
    ORDER BY t.table_name
### Check for uniformity across tables with same data 
#### SKUs
##### First finding tables with columns that have sku in the name
    WITH table_names AS (SELECT table_name
    FROM information_schema.tables
    WHERE table_schema = 'public')
    SELECT c.column_name, c.data_type, c.is_nullable, t.table_name
    FROM information_schema.columns c
    JOIN table_names t
    ON c.table_name=t.table_name
    WHERE c.table_name IN (t.table_name) AND UPPER(c.column_name) LIKE '%SKU%'
    ORDER BY c.data_type
##### Checking to make sure format of each column in each table are the same
    SELECT productsku, 'all_sessions' as table_name
    FROM all_sessions
    UNION 
    SELECT sku , 'products'
    FROM products
    UNION 
    SELECT sku , 'sales_by_sku'
    FROM sales_by_sku
    UNION 
    SELECT productsku , 'sales_report'
    FROM sales_report
    ORDER BY productsku
##### Checking to make sure no products that don't exist in product table were ordered
    SELECT DISTINCT(v2productname), itemquantity
    FROM all_sessions
    WHERE productsku IN (SELECT DISTINCT productsku
    FROM all_sessions
    WHERE productsku NOT IN (SELECT DISTINCT sku FROM products)
    ORDER BY productsku)
##### Adding SKUs listed in all_sessions to product table
    INSERT INTO products (sku)
    SELECT DISTINCT productsku
    FROM all_sessions
    WHERE productsku NOT IN (SELECT DISTINCT sku FROM products)
##### Creating relationship
    ALTER TABLE all_sessions
    ADD CONSTRAINT fk_products
    FOREIGN KEY (productsku) REFERENCES products(sku)
##### Adding SKUs listed in sales_by_sku to product table
    INSERT INTO products (sku)
    SELECT DISTINCT sku
    FROM sales_by_sku
    WHERE sku NOT IN (SELECT DISTINCT sku FROM products)
##### Creating relationship
    ALTER TABLE sales_by_sku
    ADD CONSTRAINT fk_products
    FOREIGN KEY (sku) REFERENCES products(sku)
##### Creating relationship
    ALTER TABLE sales_report
    ADD CONSTRAINT fk_products
    FOREIGN KEY (productsku) REFERENCES products(sku)
### Channel Grouping
##### Checking to see distinct values from both tables
    SELECT DISTINCT channelgrouping, 'analytics' as tablename
    FROM analytics
    UNION ALL 
    SELECT DISTINCT channelgrouping, 'all_sessions' as tablename
    FROM all_sessions
    ORDER BY channelgrouping
##### Getting sessions that changed channelgrouping
    SELECT *
    FROM all_sessions
    WHERE fullvisitorid IN (SELECT DISTINCT fullvisitorid
    FROM analytics
    WHERE channelgrouping = 'Social')
## ALL SESSIONS TABLE
### CREATING TEMP TABLE
    CREATE TEMP TABLE all_cleaning AS 
    SELECT * FROM all_sessions
#### Setting productprice column type to numeric
    ALTER TABLE all_cleaning
    ALTER COLUMN productprice TYPE numeric
#### Updating the productprice columns per instructions
    UPDATE all_cleaning
    SET productprice = ROUND(productprice/1000000.0,2)
 #### Verifying update by confirming number of zero values didn't change
    SELECT productprice
    FROM all_cleaning
    WHERE productprice = 0 
    RESULT 376 SAME AS all sessions table
 #### Setting totaltransactionrevenue column type to numeric
    ALTER TABLE all_cleaning
    ALTER COLUMN totaltransactionrevenue TYPE numeric
 #### Updating the totaltransactionrevenue columns per instructions
    UPDATE all_cleaning
    SET totaltransactionrevenue = ROUND(productprice/1000000.0,2)
 #### Verifying update by confirming number of zero values didn't change
    SELECT totaltransactionrevenue
    FROM all_cleaning
    WHERE totaltransactionrevenue = 0 
#### Setting productrevenue column type to numeric
    ALTER TABLE all_cleaning
    ALTER COLUMN productrevenue TYPE numeric
#### Updating the productrevenue columns per instructions
    UPDATE all_cleaning
    SET productrevenue = ROUND(productrevenue/1000000.0,2)
#### Verifying update by confirming number of zero values didn't change
    SELECT productrevenue
    FROM all_cleaning
    WHERE productrevenue = 0 
    RESULT 376 SAME AS all sessions table
#### Combining duplicate/erronous categories
    UPDATE all_cleaning
    SET v2productcategory = 'Home/Bags/'
    WHERE v2productcategory = 'Bags'
    UPDATE all_cleaning
    SET v2productcategory = 'Home/Nest/Nest-USA/'
    WHERE v2productcategory = 'Nest-USA'
    UPDATE all_cleaning
    SET v2productcategory = 'Unknown Cat Title'
    WHERE v2productcategory = '${escCatTitle}'
    UPDATE all_cleaning
    SET v2productcategory = 'Home/Apparel/'
    WHERE v2productcategory = 'Apparel'
### Committing changes to All Sessions table
    DROP TABLE all_sessions;
    CREATE TABLE all_sessions AS
    SELECT * FROM all_cleaning
## analytics Table
### Creating Temp Table
    CREATE TEMP TABLE analytics_cleaning AS (
    SELECT * FROM analytics)
#### Setting revenue column type to numeric
    ALTER TABLE analytics_cleaning
    ALTER COLUMN revenue TYPE numeric
#### Updating the revenue column per instructions
    UPDATE analytics_cleaning
    SET revenue = ROUND(revenue/1000000.0,2)
#### Verifying update by confirming number of zero values didn't change
    SELECT COUNT(revenue)
    FROM analytics_cleaning
    WHERE revenue = 0 
    RESULT 0 SAME AS analytics table
#### Setting unit_price column type to numeric
    ALTER TABLE analytics_cleaning
    ALTER COLUMN unit_price TYPE numeric;
#### Updating the unit_price column per instructions
    UPDATE analytics_cleaning
    SET unit_price = ROUND(revenue/1000000.0,2)
#### Verifying update by confirming number of zero values didn't change
    SELECT COUNT(unit_price)
    FROM analytics_cleaning
    WHERE unit_price = 0 
    RESULT 0 SAME AS analytics table
### Committing changes to analytics table
    DROP TABLE analytics;
    CREATE TABLE analytics AS
    SELECT * FROM analytics_cleaning
## Products Table
### Creating a temp table
    CREATE TEMP TABLE product_cleaning AS 
    SELECT * FROM products
#### Removing blank leading space from name
    UPDATE product_cleaning
    SET name = LTRIM(name)
#### Verifying leading spaces are removed
    SELECT name
    FROM product_cleaning
    WHERE name LIKE ' %'
#### Checking null value for sentiment scores
    SELECT *
    FROM product_cleaning
    WHERE sentimentscore IS NULL OR sentimentmagnitude IS NULL
#### Checking for duplicates
    SELECT sku, COUNT(sku)
    FROM product_cleaning
    GROUP BY sku
    HAVING COUNT(sku) > 1
### Commiting changes to products table
    DROP TABLE products;  
    CREATE TABLE products AS (SELECT * FROM product_cleaning)
## Sales by Sku Table
#### Confirmed no null values
    SELECT *
    FROM sales_by_sku 
    WHERE sku IS NULL OR total_ordered IS NULL
> SKUs are different formats and lengths but I saw no pattern for simliar ones that mean I should add zeros 
## Sales Report Table
### Create Temp Table
    CREATE TEMP TABLE report_cleaning AS (SELECT * FROM sales_report)
#### Remove leading spaces from name
    UPDATE report_cleaning
    SET name = LTRIM(name)
### Commiting changes to Sales Report table
    DROP TABLE sales_report;
    CREATE TABLE sales_report AS (SELECT * FROM report_cleaning)
