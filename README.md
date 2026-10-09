# Music_Store_data_analysis_with_SQL
Music Store Data Analysis using SQL, joins, CTEs, aggregations, and window functions.
🎵 Music Store Data Analysis — SQL Project
📌 Project Overview
This project performs Music Store Data Analysis using SQL to extract meaningful business insights from a relational music store database.
The analysis focuses on customer spending, sales performance, invoice trends, music preferences, artist performance, and country-level purchasing behavior.
The project demonstrates practical SQL and Data Analyst skills, including:
- Data exploration
- Relational database analysis
- Multi-table joins
- Aggregation
- Subqueries
- CTEs
- Window functions
- Ranking
- Customer analysis
- Revenue analysis
- Business-oriented insights
The project contains 11 business questions across Easy, Moderate, and Advanced difficulty levels.     Music Store Analysis-Questions     Music Store Analysis-Questions     Music Store Analysis-Questions
🎯 Business Objective
The primary objective is to use SQL to answer business questions that can help a music store understand:
- Customer purchasing behavior
- Revenue performance
- High-value customers
- Geographic sales patterns
- Music genre preferences
- Artist popularity
- Customer spending by artist
- Country-level music preferences
The project progresses from basic SQL analysis to advanced analytical queries using window functions and CTEs.
🗄️ Database Schema
The database contains the following major tables:
artist
album
track
genre
media_type
playlist
playlist_track
employee
customer
invoice
invoice_line

The EER diagram establishes relationships between artists, albums, tracks, genres, customers, invoices, invoice lines, employees, playlists, and media types.
For example:
Artist
   ↓
Album
   ↓
Track
   ├── Genre
   └── Media Type

Customer
   ↓
Invoice
   ↓
Invoice Line
   ↓
Track

Playlist
   ↓
Playlist Track
   ↓
Track

Employee
   ↓
Customer

The uploaded schema shows these entities and their key relationships in the EER diagram.     Music_Database_schema_EER_Diagr…     Music_Database_schema_EER_Diagr…
🛠️ Tools & Technologies
Database
- MySQL
SQL Concepts
- SELECT
- WHERE
- ORDER BY
- GROUP BY
- DISTINCT
- JOIN
- COUNT()
- SUM()
- AVG()
- Subqueries
- CTEs
- ROW_NUMBER()
- RANK()
- Aggregate Functions
- Window Functions
📊 Project Questions & Analysis
🟢 Question Set 1 — Easy
1. Senior-Most Employee
Identify the senior-most employee based on job level/title.
SELECT *
FROM employee
ORDER BY levels DESC
LIMIT 1;

2. Countries With the Most Invoices
Analyze invoice distribution across billing countries.
SELECT
    COUNT(*) AS invoice_count,
    billing_country
FROM invoice
GROUP BY billing_country
ORDER BY invoice_count DESC;

3. Top 3 Invoice Values
Identify the three highest-value invoices.
SELECT
    total,
    billing_country
FROM invoice
ORDER BY total DESC
LIMIT 3;

4. City With the Highest Revenue
Identify the city where the store generated the highest total invoice revenue.
SELECT
    billing_city,
    SUM(total) AS invoice_total
FROM invoice
GROUP BY billing_city
ORDER BY invoice_total DESC
LIMIT 1;

The original project specifically asks for the city with the highest sum of invoice totals.     Music Store Analysis-Questions
5. Best Customer
Identify the customer who has spent the most money.
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    SUM(i.total) AS total_amount
FROM customer c
JOIN invoice i
    ON c.customer_id = i.customer_id
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name
ORDER BY total_amount DESC
LIMIT 1;

This addresses the project's definition of the "best customer" as the customer with the highest spending.     Music Store Analysis-Questions
🟡 Question Set 2 — Moderate
6. Rock Music Listeners
Identify customers who have purchased Rock music.
SELECT DISTINCT
    c.email,
    c.first_name,
    c.last_name,
    g.name AS genre
FROM customer c
JOIN invoice i
    ON c.customer_id = i.customer_id
JOIN invoice_line il
    ON i.invoice_id = il.invoice_id
JOIN track t
    ON il.track_id = t.track_id
JOIN genre g
    ON t.genre_id = g.genre_id
WHERE g.name = 'Rock'
ORDER BY c.email ASC;

The query uses multiple joins across customer, invoice, invoice_line, track, and genre, with DISTINCT to avoid duplicate listeners. The project requires the result to be ordered alphabetically by email.     Music Store Analysis-Questions
7. Top 10 Rock Artists
Identify the artists with the highest number of Rock tracks.
SELECT
    a.artist_id,
    a.name AS artist_name,
    COUNT(a.artist_id) AS number_of_songs
FROM track t
JOIN album al
    ON al.album_id = t.album_id
JOIN artist a
    ON a.artist_id = al.artist_id
JOIN genre g
    ON g.genre_id = t.genre_id
WHERE g.name = 'Rock'
GROUP BY
    a.artist_id,
    a.name
ORDER BY number_of_songs DESC
LIMIT 10;

This analysis identifies the top 10 Rock artists based on track count.     Music Store Analysis-Questions
8. Tracks Longer Than Average
Identify tracks whose duration is greater than the average track duration.
SELECT
    name,
    milliseconds
FROM track
WHERE milliseconds > (
    SELECT AVG(milliseconds)
    FROM track
)
ORDER BY milliseconds DESC;

This uses a subquery with AVG() to calculate the benchmark before filtering the tracks.     Music Store Analysis-Questions
🔴 Question Set 3 — Advanced
9. Customer Spending by Artist
Calculate how much each customer spent on each artist.
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    a.name AS artist_name,
    SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c
    ON c.customer_id = i.customer_id
JOIN invoice_line il
    ON il.invoice_id = i.invoice_id
JOIN track t
    ON t.track_id = il.track_id
JOIN album al
    ON al.album_id = t.album_id
JOIN artist a
    ON a.artist_id = al.artist_id
GROUP BY
    c.customer_id,
    c.first_name,
    c.last_name,
    a.artist_id,
    a.name
ORDER BY amount_spent DESC;

This analysis connects customer purchases to individual artists and calculates spending using:
Unit Price × Quantity

The project specifically asks for customer name, artist name, and total amount spent.     Music Store Analysis-Questions
10. Most Popular Genre by Country
Identify the most purchased genre in each country.
A RANK() window function is useful because it allows all genres tied for first place to be returned.
WITH popular_genre_sales AS (
    SELECT
        c.country,
        g.name AS genre,
        g.genre_id,
        COUNT(il.quantity) AS purchases,
        RANK() OVER (
            PARTITION BY c.country
            ORDER BY COUNT(il.quantity) DESC
        ) AS genre_rank
    FROM invoice_line il
    JOIN invoice i
        ON i.invoice_id = il.invoice_id
    JOIN customer c
        ON c.customer_id = i.customer_id
    JOIN track t
        ON t.track_id = il.track_id
    JOIN genre g
        ON g.genre_id = t.genre_id
    GROUP BY
        c.country,
        g.name,
        g.genre_id
)

SELECT
    country,
    genre,
    purchases
FROM popular_genre_sales
WHERE genre_rank = 1
ORDER BY country;

The requirement explicitly states that all genres should be returned when the maximum number of purchases is shared.     Music Store Analysis-Questions
11. Top Customer by Country
Identify the highest-spending customer in every country.
WITH customer_spending AS (
    SELECT
        c.country,
        c.customer_id,
        c.first_name,
        c.last_name,
        SUM(i.total) AS total_spending
    FROM invoice i
    JOIN customer c
        ON c.customer_id = i.customer_id
    GROUP BY
        c.country,
        c.customer_id,
        c.first_name,
        c.last_name
),

ranked_customers AS (
    SELECT
        *,
        RANK() OVER (
            PARTITION BY country
            ORDER BY total_spending DESC
        ) AS customer_rank
    FROM customer_spending
)

SELECT
    country,
    customer_id,
    first_name,
    last_name,
    total_spending
FROM ranked_customers
WHERE customer_rank = 1
ORDER BY country;

Using RANK() ensures that all customers sharing the highest spending amount within a country are returned, matching the project requirement.     Music Store Analysis-Questions
🔍 Key SQL Techniques Demonstrated
Basic SQL
SELECT
WHERE
ORDER BY
LIMIT
DISTINCT

Aggregation
COUNT()
SUM()
AVG()
GROUP BY

Relational Analysis
INNER JOIN
Multi-table JOIN

Advanced SQL
Subqueries
CTEs
ROW_NUMBER()
RANK()
Window Functions

💡 Business Insights
This project demonstrates how SQL can be used to answer practical business questions such as:
👥 Customer Analytics
- Who are the highest-value customers?
- Which customers purchase Rock music?
- How much does each customer spend on different artists?
- Who is the top customer in each country?
💰 Revenue Analytics
- Which cities generate the most revenue?
- What are the highest-value invoices?
- Which countries have the most invoices?
🎵 Music Analytics
- Which artists have the most Rock tracks?
- Which genres are most popular in different countries?
- Which tracks have above-average duration?
🌍 Geographic Analysis
- Compare purchasing behavior across countries.
- Identify country-specific genre preferences.
- Identify the highest-value customer in each country.
📁 Project Structure
Recommended GitHub repository structure:
Music-Store-SQL-Data-Analysis/
│
├── README.md
│
├── SQL/
│   └── MUSIC_STORE_DATABASE_query.sql
│
├── Database/
│   └── Music_Store_database_MySQL.sql
│
├── EER_Diagram/
│   ├── Music_Database_EER_Diagram.png
│   └── Music_Database_schema_EER_Diagram.pdf
│
└── Documentation/
    └── Music_Store_Analysis-Questions.pdf

🧠 Skills Demonstrated
SQL
MySQL
Data Analysis
Data Exploration
Data Cleaning
Relational Databases
Database Relationships
EER Modeling
Joins
Aggregation
Subqueries
CTEs
Window Functions
RANK()
ROW_NUMBER()
Customer Analysis
Revenue Analysis
Business Insights

📌 Project Highlights
- Analyzed a relational Music Store database.
- Solved 11 business-oriented SQL problems across three difficulty levels.
- Used multi-table joins to connect customers, invoices, tracks, albums, artists, and genres.
- Used aggregate functions for revenue and purchase analysis.
- Used subqueries for average track-length analysis.
- Used CTEs and window functions for advanced country-level analysis.
- Used RANK() to handle tied top genres and customers.
- Designed and implemented an EER database schema with primary and foreign-key relationships.
