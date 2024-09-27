# What are your risk areas? Identify and describe them.

## Lack of Documentation
There is no documentation provided about the data set. There are duplicate values across tables and no way to accurately tell what is the source of truth and how the data was manipulated to create said tables. The analytics table clearing came from all_sessions but there isn't a one-to-one. For example, one visit can render four rows in analytics with data missing from each row. 
## Structure of Database
There are no documented relationships making it difficult to see how some of the tables were created and why they were created. 
Some of the big issues are no primary key in all_sessions. One row in all_sessions could be listed as four rows in analytics. The data between the two tables can vary and be inconsistent at best. 

There are SKUs that have characters which means uppercase/lowercase  

## NULL Values
* There are many null values that bring into question the accuracy of the dataset.
## Channel Grouping 
* Analytics table has 'social', all_sessions does not have 'social'
# QA Process:
Describe your QA process and include the SQL queries used to execute it.
