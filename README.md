# SQL-Music-Store-Database-Queries
## Dataset: https://drive.google.com/file/d/1eOw8GZ7-T25-8OWQf5DNroI8m6kzyq3G/view
## Easy 
1. Who is the most senior employee based on job title?  

select concat(first_name,' ',last_name) from employee  
order by levels desc  
limit 1;  

2. Which country have the most invoices?  

select billing_country, count(invoice_id)  
from invoice  
group by invoice.billing_country  
order by count(invoice_id) desc  
limit 1;  

3. What are top 3 values of total invoice?  

select total from invoice  
order by total desc  
limit 3;  

4. Which city has the best customers? We would like to throw a promotional Music 
Festival in the city we made the most money. Write a query that returns one city that 
has the highest sum of invoice totals. Return both the city name & sum of all invoice 
totals  

select billing_city, sum(total) as t from invoice  
group by billing_city  
order by t desc  
limit 1;  

5. Who is the best customer? The customer who has spent the most money will be 
declared the best customer. Write a query that returns the person who has spent the 
most money.

select customer.customer_id, customer.first_name, customer.last_name,  
sum(invoice.total) as t  
from customer  
inner join invoice on customer.customer_id = invoice.customer_id  
group by customer.customer_id  
order by t desc  
limit 1;  

## Moderate  

1. Write query to return the email, first name, last name, & Genre of all Rock Music 
listeners. Return your list ordered alphabetically by email starting with A 
select email, first_name, last_name, genre.name as genre  

from customer  
join invoice on customer.customer_id =invoice.customer_id  
join invoice_line on invoice.invoice_id = invoice_line.invoice_id  
join track on invoice_line.track_id= track.track_id  
join genre on track.genre_id = genre.genre_id  
where genre.name like 'Rock'  
group by email, first_name, last_name, genre.name  
order by email;  

2. Let's invite the artists who have written the most rock music in our dataset. Write a 
query that returns the Artist name and total track count of the top 10 rock bands.

select artist.name, count(track.track_id) as tracks  
from artist  
join album on artist.artist_id = album.artist_id  
join track on album.album_id = track.album_id  
join genre on track.genre_id= genre.genre_id  
where genre.name like 'Rock'  
group by artist.name  
order by tracks desc  
limit 10;  

3. Return all the track names that have a song length longer than the average song 
length. Return the Name and Milliseconds for each track. Order by the song length 
with the longest songs listed first.  

select track.name, milliseconds  
from track  
where milliseconds > (Select avg(milliseconds) from track)  
order by milliseconds desc  

## Advanced  

1. Find how much amount spent by each customer on artists? Write a query to return 
customer name, artist name and total spent  

SELECT CONCAT_WS(' ', C.first_name, C.last_name)customer_name, 
(A.name)artist_name, 
SUM(IL.unit_price * IL.quantity)total_spent  
FROM customer C  
JOIN invoice I  
ON C.customer_id = I.customer_id  
JOIN invoice_line IL  
ON I.invoice_id = IL.invoice_id  
JOIN track T  
ON IL.track_id = T.track_id  
JOIN album al 
ON T.album_id = al.album_id  
JOIN artist A  
ON al.artist_id = A.artist_id  
GROUP BY C.first_name, C.last_name, A.name  
ORDER BY total_spent DESC  

2. We want to find out the most popular music Genre for each country. We determine 
the most popular genre as the genre with the highest number of purchases. Write a 
query that returns each country along with the top Genre. For countries where the 
maximum number of purchases is shared return all Genres. 

WITH popular_genre AS  
( 
SELECT COUNT(invoice_line.quantity) AS purchases,  
customer.country,  
genre.name,  
RANK() OVER (PARTITION BY customer.country ORDER BY 
COUNT(invoice_line.quantity) DESC) AS RankNo  
FROM invoice_line  
JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id 
JOIN customer ON customer.customer_id = invoice.customer_id 
JOIN track ON track.track_id = invoice_line.track_id 
JOIN genre ON genre.genre_id = track.genre_id 
GROUP BY customer.country, genre.name  
) 
SELECT *  
FROM popular_genre  
WHERE RankNo = 1;  

3. Write a query that determines the customer that has spent the most on music for each 
country. Write a query that returns the country along with the top customer and how 
much they spent. For countries where the top amount spent is shared, provide all 
customers who spent this amount.  

With spent as( 
SELECT  customer.customer_id,CONCAT_WS(' ', customer.first_name, 
customer.last_name)customer_name,  
SUM(invoice_line.unit_price * invoice_line.quantity)total_spent, 
customer.country,  
RANK() OVER (PARTITION BY customer.country ORDER BY 
SUM(invoice_line.unit_price * invoice_line.quantity) DESC) AS RankNo  
FROM invoice_line 
JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id  
JOIN customer ON customer.customer_id = invoice.customer_id  
JOIN track ON track.track_id = invoice_line.track_id  
JOIN genre ON genre.genre_id = track.genre_id  
group by customer.customer_id,customer_name,customer.country)  
select *  
from spent  
where RankNo=1; 
