# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
This project was to take five csv files with no documentation and try to parse out the meaning behind the data. 

My goals were to answer the questions and come up with three to five additional questions. I also wanted to clean the data and ensure my techniques were accurate.

## Process
### Reviewing the raw data
My first step was to open each excel sheet and just look at the data. I then determined which data type each column should be typed as. 
### Importing the data
After I reviewed the data, I then imported the data into a new SQL database. Once all was loaded, I created an ERD to reference while I continued to look at the data across the tables. 
### Filled in the other sheets
Then I started with the questions and playing around with the data. I queryed things that popped into my mind and answered questions while thinking about what else I could find out from the data. 

## Results
This data shows that the business needs a better database engineer. It also showed that while I could pull data and find trends, I couldn't use the whole dataset and had to clean out a significant portion of the data while querying which means trends will be skewed. 

## Challenges 
The biggest challenge with this data was to figure out how each table was connected and if they should be connected. The all_sessions and analytics table were the hardest to decipher. When I ran queries to figure out how they should connect, I discovered that the FullVisitID was not a unique value in the analytics table. Additionally, when I looked closer, visits that were one row in all_sessions were multiple rows in analytics. 

## Future Goals
If I had more time, I would attempt to recreate the analytics table and really dig into how it was constructed. I would also try to find out where the data was sourced and how it was collected. 