# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
The ecommerce data set is a tricky dataset to work with. It has a number of unformatted incomplete data points. However, the goal of this project is to display how answers can still be derived from data, after setting up parameteres and cleaning the data - along with educated assumptions to fill in the blanks when necessary. Using SQL to creatively and effectively clean, extract, and generate accurate insights.

## Process
(1) Get familiar with the data - ERD diagram, SELECT * from each table to see what kind of data we are working with, execute simple joins to determine whether there are relationships we can build within the data set
(2) Create hypothesis to why the data is the way it is (does the data seem accurate? are we missing key data points? what can we do to QA the data? etc.)
(3) Tackle project questions and record assumptions throughout the process
(4) QA answers by checking derived results through another way of querying the data in separate steps and cross referencing results

## Results
Insights into the total revenue by visitor, city, and/or country were generated from this data set. Although there were potentially multiple ways to calculate this, the data completeness of the field, totalrevenuegenerated, unlocked the ability to answer majority of revenue related questions. This data set is far from being complete and being considered robust, however by cleaning up and removing redundant data points, a reasonable narrative with the right guidlines could be crafted questions at hand.

## Challenges 
(1) Ambiguous data formating and reliability across all tables led to a lot of QA to determine which fields should be used to ansewr the question best.
(2) Lots of null values, but it is not clear why certain values are null when other related columns are filled properly
(3) Inconsistent inclusion of unique ids across tables that resulted in additional testing and cleaning before starting the final query
(4) Relationships between seemingly related tables such as analytics and all_sessions or prodcuts and sales_by_sku did not match up on id or structured use case

## Future Goals
(1) Create clean tables for each of the given tables, by removing duplicate rows
(2) Normalize the data to get as close to 3NF
(3) Standardize and reorganize table column names so there are consistent naming conventions across every table and better grouping of columns
(4) Futher validate which fields are the most robust to calculate revenue