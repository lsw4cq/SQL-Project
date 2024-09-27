What are your risk areas? Identify and describe them.

There is no documentation provided about the data set. There are duplicate values across tables and no way to accurately tell what is the source of truth and how the data was manipulated across tables. 

Some of the big issues are no primary key in all_sessions. One row in all_sessions could be listed as four rows in analytics. The data between the two tables can vary and be inconsistent at best. 

There are SKUs that have characters which means uppercase/lowercase  


QA Process:
Describe your QA process and include the SQL queries used to execute it.
