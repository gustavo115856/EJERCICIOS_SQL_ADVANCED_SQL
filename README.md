# EJERCICIOS_SQL_ADVANCED_SQL
EJERCICIOS SQL
ADVANCED SQL
1-SQL Tutorial Lesson: Top-Selling Artists
As the lead data analyst for a prominent music event management company, you have been entrusted with a dataset containing concert revenue and detailed information about various artists.
Your mission is to unlock valuable insights by analyzing the concert revenue data and identifying the top revenue-generating artists within each music genre.
Write a query to rank the artists within each genre based on their revenue per member and extract the top revenue-generating artist from each genre. Display the output of the artist name, genre, concert revenue, number of members, and revenue per band member, sorted by the highest revenue per member within each genre.
SELECT
  artist_name,
  concert_revenue,
  genre,
  number_of_members,
  revenue_per_member
FROM (
  SELECT
    artist_name,
    concert_revenue,
    genre,
    number_of_members,
    concert_revenue / number_of_members AS revenue_per_member,
    RANK() OVER (
      PARTITION BY genre
      ORDER BY concert_revenue / number_of_members DESC) AS ranked_concerts
  FROM concerts) AS subquery
WHERE ranked_concerts = 1
ORDER BY revenue_per_member DESC;

 
2-Card Launch Success
Your team at JPMorgan Chase is soon launching a new credit card. You are asked to estimate how many cards you'll issue in the first month.
Before you can answer this question, you want to first get some perspective on how well new credit card launches typically do in their first month.
Write a query that outputs the name of the credit card, and how many cards were issued in its launch month. The launch month is the earliest record in the monthly_cards_issued table for a given card. Order the results starting from the biggest issued amount.
WITH card_launch AS (
  SELECT 
    card_name,
    issued_amount,
    MAKE_DATE(issue_year, issue_month, 1) AS issue_date,
    MIN(MAKE_DATE(issue_year, issue_month, 1)) OVER (
      PARTITION BY card_name) AS launch_date
  FROM monthly_cards_issued
)
SELECT card_name,issued_amount FROM card_launch WHERE issue_date = launch_date ORDER BY issued_amount DESC;


3-Top 5 Artists
Assume there are three Spotify tables: artists, songs, and global_song_rank, which contain information about the artists, songs, and music charts, respectively.
Write a query to find the top 5 artists whose songs appear most frequently in the Top 10 of the global_song_rank table. Display the top 5 artist names in ascending order, along with their song appearance ranking.
If two or more artists have the same number of song appearances, they should be assigned the same ranking, and the rank numbers should be continuous (i.e. 1, 2, 2, 3, 4, 5). If you've never seen a rank order like this before, do the rank window function tutorial.
WITH top_10_cte AS (
  SELECT 
    artists.artist_name,
    DENSE_RANK() OVER ( ORDER BY COUNT(songs.song_id) DESC) AS artist_rank FROM artists INNER JOIN songs
    ON artists.artist_id = songs.artist_id INNER JOIN global_song_rank AS ranking ON songs.song_id = ranking.song_id
  WHERE ranking.rank <= 10 GROUP BY artists.artist_name
)
SELECT artist_name, artist_rank FROM top_10_cte WHERE artist_rank <= 5;


4-SQL Tutorial Lesson: Stock Performance
The Bloomberg terminal is the go-to resource for financial professionals, offering convenient access to a wide array of financial datasets. In this SQL interview query for Data Analyst at Bloomberg, you're given the historical data on Google's stock performance.
Your task is to:
-Calculate the difference in closing prices between consecutive months.
-Calculate the difference between the closing price of the current month and the closing price from 3 months prior.
This question serves as a platform for you to explore into the dataset and execute your queries. Please refrain from submitting your solution for this question.
SELECT 
    date,
    ticker,
    close,
    LAG(close) OVER (PARTITION BY ticker ORDER BY date) AS previous_close,
    close - LAG(close) OVER (PARTITION BY ticker ORDER BY date) AS monthly_difference
FROM 
    stock_prices
ORDER BY 
    date;
SELECT 
    date,
    ticker,
    close,
    LAG(close, 3) OVER (PARTITION BY ticker ORDER BY date) AS close_3_months_ago,
    close - LAG(close, 3) OVER (PARTITION BY ticker ORDER BY date) AS difference_3_months
FROM 
    stock_prices
ORDER BY 
    date;


5-Maximize Prime Item Inventory
Effective April 3rd 2024, we have updated the problem statement to provide additional clarity.
Amazon wants to maximize the storage capacity of its 500,000 square-foot warehouse by prioritizing a specific batch of prime items. The specific prime product batch detailed in the inventory table must be maintained.
So, if the prime product batch specified in the item_category column included 1 laptop and 1 side table, that would be the base batch. We could not add another laptop without also adding a side table; they come all together as a batch set.
After prioritizing the maximum number of prime batches, any remaining square footage will be utilized to stock non-prime batches, which also come in batch sets and cannot be separated into individual items.
Write a query to find the maximum number of prime and non-prime batches that can be stored in the 500,000 square feet warehouse based on the following criteria:
Prioritize stocking prime batches
After accommodating prime items, allocate any remaining space to non-prime batches
Output the item_type with prime_eligible first followed by not_prime, along with the maximum number of batches that can be stocked.
Asumptions:
Again, products must be stocked in batches, so we want to find the largest available quantity of prime batches, and then the largest available quantity of non-prime batches
Non-prime items must always be available in stock to meet customer demand, so the non-prime item count should never be zero.
Item count should be whole numbers (integers).
WITH summary AS (
  SELECT 
    SUM(square_footage) FILTER (WHERE item_type = 'prime_eligible') AS prime_sq_ft,
    COUNT(item_id) FILTER (WHERE item_type = 'prime_eligible') AS prime_item_count,
    SUM(square_footage) FILTER (WHERE item_type = 'not_prime') AS not_prime_sq_ft,
    COUNT(item_id) FILTER (WHERE item_type = 'not_prime') AS not_prime_item_count
  FROM inventory
),
prime_occupied_area AS ( SELECT FLOOR(500000/prime_sq_ft)*prime_sq_ft AS max_prime_area FROM summary
)
SELECT 'prime_eligible' AS item_type, FLOOR(500000/prime_sq_ft)*prime_item_count AS item_coun FROM summary
UNION ALL SELECT 'not_prime' AS item_type,FLOOR((500000-(SELECT max_prime_area FROM prime_occupied_area)) 
    / not_prime_sq_ft) * not_prime_item_count AS item_count FROM summary;


6-SQL LOWER Practice Exercise
Assume you're given the customer table containing all customer details.
The branch manager is looking for a male customer whose name ends with "son" and he's 20 years old.
Write a SQL query which uses LOWER and LIKE to find this customer's details.
SELECT *
FROM customers
WHERE LOWER(customer_name) LIKE '%son'
  AND gender = 'Male'
  AND age = 20;


7-Instacart Exploration  Case Study Checkpoint #1
  Explore the ic_products data. Here are some questions to investigate:
How many items are there?
How many different products are in the dataset?
Table Schema
Here are the schemas for all 5 tables in the Instacart market data.
ic_order_products_curr, ic_order_products_prior
These files specify which products were purchased in each Instacart order. ic_order_products_prior contains previous order contents for all customers, and ic_order_products_curr contains current orders; the table fields are the same.
The 'reordered' field indicates that the customer has a previous order that contains the product. Other fields should be self-explanatory.
SELECT COUNT(*) AS total_items
FROM (
    SELECT * FROM ic_order_products_curr
    UNION ALL
    SELECT * FROM ic_order_products_prior
) AS all_orders;
SELECT COUNT(DISTINCT product_id) AS unique_products
FROM ic_products;
