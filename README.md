# MusicStore-DataCleaning
/* Senior most employee*/
select first_name, last_name, levels from music_store.employee order by levels desc limit 1;


/*Which countries have the most invoices*/
select billing_country, count(*) from music_store.invoice group by billing_country order by count(*) desc limit 1;


/*What are the top 3 of total invoices*/
select billing_country, sum(total) as total from music_store.invoice group by billing_country order by total desc limit 3;


/*Which city has the best customers. We would like to throw a promotional music festival in the city we made the most money. Write a query that 
returns one city that has the highest sum of invoice totals. Return both the city name and sum of all the invoice totals*/
select billing_city, sum(total) as total from music_store.invoice group by billing_city  order by total desc limit 1;


/* Who is the best customer. The customer who has spent the most money will be declared the best customer. Write a query that returns
the person who has spent the most money.*/
select  music_store.customer.customer_id,music_store.customer.first_name, music_store.customer.last_name, sum(music_store.invoice.total) as total from music_store.customer 
Join music_store.invoice 
On music_store.customer.customer_id=music_store.invoice.customer_id
group by music_store.invoice.customer_id
order by total  desc 
limit 1;


/* Write a query to return email, first name , last name and genre of all rock music listeners. return your list oredered alphabatically by email 
starting with a*/
select distinct music_store.customer.first_name,music_store.customer.last_name,music_store.customer.email from music_store.customer
join music_store.invoice 
on music_store.customer.customer_id=music_store.invoice.customer_id
join music_store.invoice_line 
on music_store.invoice.invoice_id=music_store.invoice_line.invoice_id
where music_store.invoice_line.track_id in (
select music_store.track.track_id from music_store.track
join music_store.genre
on music_store.track.genre_id=music_store.genre.genre_id
where music_store.genre.name like 'Rock')
order by music_store.customer.email;


/* Lets invite the artists who have written the most rock music in our dataset. Write a query name that returns the Artist name and total track
count of the top 10 rock band*/
select music_store.artist.artist_id,music_store.artist.name, count(music_store.artist.artist_id)  as number_of_songs from music_store.track 
join music_store.album 
on music_store.album.album_id=music_store.track.album_id
join music_store.artist 
on music_store.artist.artist_id=music_store.album.artist_id
join music_store.genre
on music_store.genre.genre_id=music_store.track.genre_id
where music_store.genre.name like 'Rock'
group by music_store.artist.artist_id
order by number_of_songs desc;


/*Return all the track names that have a song length longer than the average song length. Return the name and
milliseconds for each track. order by the song length with the longest songs listed first*/
select music_store.track.name ,music_store.track.milliseconds from music_store.track
where music_store.track.milliseconds > (
select  avg(music_store.track.milliseconds) as avg_track_length from music_store.track) 
order by music_store.track.milliseconds desc ;


/*Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent.*/
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
 
 
 /* We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where 
the maximum number of purchases is shared return all Genres. */

WITH popular_genre AS(
SELECT COUNT( music_store.invoice_line.quantity) AS purchases,
music_store.customer.country, music_store.genre.name, music_store.genre.genre_id,
ROW_NUMBER() OVER( PARTITION BY music_store.customer.country ORDER BY COUNT(music_store.invoice_line.quantity) DESC) AS RowNo
FROM music_store.invoice_line
JOIN music_store.invoice ON music_store.invoice.invoice_id= music_store.invoice_line.invoice_id
JOIN music_store.customer ON music_store.customer.customer_id= music_store.invoice.customer_id
JOIN music_store.track ON music_track.trak_id = music_store.invoice_line.track_id
JOIN music_store.genre ON music_store.genre.genre_id= music_store.track.genre_id
GROUP BY 2,3,4
ORDER BY 2 ASC, 1 DESC)
SELECT * FROM popular_genre WHERE RowNo <= 1;
