# Digital Music Store Data Analysis using SQL

## Overview
This project aims to conduct data analysis on a digital music store dataset using SQL. The dataset contains information about customers, purchases, and music tracks. By leveraging SQL queries, we will explore various aspects of the data to derive insights that can inform business decisions and strategies.

## Dataset
The dataset used in this project consists of the following tables:
- `customers`: Contains information about customers, including customer ID, first-name, last-name, email, and country.
- `invoices`: Provides details about each invoice, such as invoice ID, customer ID, invoice date, and total amount.
- `invoice_Line`: Contains information such as, track ID, unit-price and quantity.
- `tracks`: Includes details about each music track, such as track ID, name, album ID, genre ID, composer, and duration in Milliseconds.
- `genres`: Provides information about music genres, including genre ID and name.
- `albums`: Contains details about music albums, including album ID, title, and artist ID.
- `artists`: Includes information about artists, such as artist ID and name.

## Objectives
1. Who is the best customer?.
2. Determine most popular music Genre for each country.
3. Artists who have written the most rock music.
4. Which countries have the most Invoices?.
5. Amount spent by each customer on artists.

## SQL Queries
To achieve the objectives outlined above, the following SQL queries will be utilized:
1. Best Customer:
```sql
SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;
```

2. Popular Music Genres by country:
```sql
WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1
```

3. Artists with most rock music:
```sql
SELECT artist.artist_id, artist.name,COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```

4. Most invoices by country:
```sql
SELECT COUNT(*) AS c, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY c DESC
```

5. Amount spent by customers:
```sql
WITH best_selling_artist AS (
	SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price*invoice_line.quantity) AS total_sales
	FROM invoice_line
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN album ON album.album_id = track.album_id
	JOIN artist ON artist.artist_id = album.artist_id
	GROUP BY 1
	ORDER BY 3 DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price*il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;
```

## Conclusion
By executing these SQL queries on the digital music store dataset, we can gain valuable insights into best customers , popular music genres by country, top-selling tracks and albums, most invoices by country, and customer purchasing behavior. These insights can help the digital music store make informed decisions to optimize its offerings, marketing strategies, and overall business performance.
