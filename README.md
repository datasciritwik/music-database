# Music Insights Analyzer

## Project Overview

Music Insights Analyzer is an SQL-based analytical project designed to provide deep insights into customer behaviour, sales trends, and musical preferences within a music store's database. The project leverages various SQL queries to extract and analyze data, offering valuable information to optimize marketing strategies, understand customer dynamics, and enhance business intelligence in the music industry.

## Features

- **Senior Employee Identification**: Identify the senior-most employee based on job title levels.
- **Top Invoicing Countries**: Determine the countries with the most invoices, focusing on the top three.
- **Top Invoice Values**: Retrieve and analyze the highest.
- **Best Customer Cities**: Identify the city generating the highest total invoice amounts for targeted promotional events.
Best Customers by Name: List the top customers by total spending, including their full names and billing cities.
Rock Music Listener Profiles: List the email, first name, and last name of all rock music listeners, ordered alphabetically by email.
- **Top Rock Artists**: Identify the top 10 artists by the number of rock tracks produced.
- **Longer than Average Tracks**: List tracks that exceed the average song length, ordered by duration.
- **Customer Spending on Top Artists**: Analyze customer spending on the top artists.
Popular Music Genres by Country: Identify the most popular music genres in each country by purchase amount.
- **Top Spending Customers by Country**: Find the top spending customer in each country, including cases where multiple customers share the top spending amount.

## Queries Used

1. **Senior Employee Identification**
   ```sql
   SELECT * FROM employee ORDER BY levels DESC LIMIT 1;
   ```

2. **Top Invoicing Countries**
   ```sql
   SELECT billing_country, COUNT(billing_country) AS count 
   FROM invoice 
   GROUP BY billing_country 
   ORDER BY count DESC LIMIT 3;
   ```

3. **Top Invoice Values**
   ```sql
   SELECT * FROM invoice ORDER BY total DESC LIMIT 3;
   ```

4. **Best Customer Cities**
   ```sql
   SELECT billing_city, SUM(total) AS total_sum 
   FROM invoice 
   GROUP BY billing_city 
   ORDER BY total_sum DESC;
   ```

5. **Best Customers by Name**
   ```sql
   SELECT c.first_name || ' ' || c.last_name AS Customer_Name, i.billing_city, SUM(i.total) AS total_sum 
   FROM invoice i 
   JOIN customer c ON i.customer_id = c.customer_id 
   GROUP BY Customer_Name, i.billing_city 
   ORDER BY total_sum DESC;
   ```

6. **Rock Music Listener Profiles**
   ```sql
   SELECT DISTINCT c.email, c.first_name || ' ' || c.last_name AS Customer_Name 
   FROM customer c 
   JOIN invoice ON c.customer_id = invoice.customer_id 
   JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id 
   WHERE track_id IN (
      SELECT track_id FROM track 
      JOIN genre ON track.genre_id = genre.genre_id 
      WHERE genre.name LIKE 'Rock'
   ) 
   ORDER BY email;
   ```

7. **Top Rock Artists**
   ```sql
   WITH rock_tracks AS (
      SELECT t.track_id, al.artist_id 
      FROM track t 
      JOIN album al ON t.album_id = al.album_id 
      JOIN genre g ON t.genre_id = g.genre_id 
      WHERE g.name = 'Rock'
   ) 
   SELECT a.artist_id, a.name, COUNT(rt.track_id) AS track_count 
   FROM artist a 
   JOIN rock_tracks rt ON a.artist_id = rt.artist_id 
   GROUP BY a.artist_id, a.name 
   ORDER BY track_count DESC LIMIT 10;
   ```

8. **Longer than Average Tracks**
   ```SQL
   SELECT name, milliseconds 
   FROM track 
   WHERE milliseconds > (SELECT AVG(milliseconds) FROM track) 
   ORDER BY milliseconds DESC;
   ```

9. **Customer Spending on Top Artists**
   ```sql
   WITH best_selling_artist AS (
      SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price*invoice_line.quantity) AS total_sales 
      FROM invoice_line 
      JOIN track ON track.track_id = invoice_line.track_id 
      JOIN album ON album.album_id = track.album_id 
      JOIN artist ON artist.artist_id = album.artist_id 
      GROUP BY 1 
      ORDER BY 3 DESC LIMIT 1
   ) 
   SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price*il.quantity) AS amount_spent 
   FROM invoice i 
   JOIN customer c ON c.customer_id = i.customer_id 
   JOIN invoice_line il ON il.invoice_id = i.invoice_id 
   JOIN track t ON t.track_id = il.track_id 
   JOIN album alb ON alb.album_id = t.album_id 
   JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id 
   GROUP BY 1, 2, 3, 4 
   ORDER BY 5 DESC;
   ```

10. **Popular Music Genres by Country**
    ```sql
    WITH popular_genre AS (
       SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
       ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
       FROM invoice_line 
       JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id 
       JOIN customer ON customer.customer_id = invoice.customer_id 
       JOIN track ON track.track_id = invoice_line.track_id 
       JOIN genre ON genre.genre_id = track.genre_id 
       GROUP BY 2, 3, 4 
       ORDER BY 2 ASC, 1 DESC
    ) 
    SELECT * FROM popular_genre WHERE RowNo <= 1;
    ```

11. **Top Spending Customers by Country**
    ```sql
    WITH Customter_with_country AS (
       SELECT customer.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spending, 
       ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
       FROM invoice 
       JOIN customer ON customer.customer_id = invoice.customer_id 
       GROUP BY 1, 2, 3, 4 
       ORDER BY 4 ASC, 5 DESC
    ) 
    SELECT * FROM Customter_with_country WHERE RowNo <= 1;
    ```

## Getting Started

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/yourusername/music-database.git
   cd music-database
   ```

2. **Set Up the Database**:
   - Ensure you have the required database with the relevant tables and data as indicated in the queries.

3. **Run the Queries**:
   - Use a SQL client or command line to execute the provided queries and analyze the results.


Thank for reading 🤗
