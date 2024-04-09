# Music Database
### Running SQL queries on a database
### Here we have a database of music albums and the the purchases of the same by N number of customers from different location.
### We will be performing some operations of this database to find out the details according to the requirements.
### Please download the database in order to run the operations.

# Database and Tools
### Postgre SQL
### PgAdmin4

# Schema of the database
![DatabaseSchema](https://github.com/mallickrishiraj/musicdatabase/assets/84040127/90028a09-ff10-4309-b81a-fb76adedbb37)


## 1. Lets find the senior most employee based on the job title. We can do so using the following the simple query: 

select * from employee
order by levels desc
limit 1


## 2. Which countries have most Invoices?

select count(*) as count,
billing_country from invoice
group by billing_country
order by count desc


## 3. What are the top 5 values of invoice total?

select total from invoice
order by total desc
limit 5

## 4. Which city has best customers? In order to arrange a party we would like to know about the city with highest sales and profit. We can find out this by sum of invoice grouped by city.

select billing_city, sum(total) as invoicestotal
from invoice
group by billing_city
order by invoicestotal desc


## 5. We need to list down the best customer who had bought the most number of albums. And give him the greatest fan of all time award. For this we need to join two tables from the schema that we have. One would be customer and other one would be the invoices table.

select customer.customer_id, customer.first_name, customer.last_name, sum(invoice.total) as purchases from customer
join invoice on customer.customer_id = invoice.customer_id
group by customer.customer_id
order by purchases desc
limit 1

## 6. Lets find out the details about the music listeners using the Genre. Here we need to perform a join again according to the schema. And we will list the customer details in alphabetical order using email.

select distinct email, first_name, last_name
from customer
join invoice on customer.customer_id = invoice.customer_id
join invoice_line on invoice.invoice_id = invoice_line.invoice_id
where track_id IN 
(select track_id from track 
join genre on track.genre_id = genre.genre_id
where genre.name like 'Rock')
order by email 

## 7. Lets return the top 5 artist who has contributed the most number of songs. By filtering with a specific genre.

select artist.artist_id, artist.name , COUNT(artist.artist_id) as numberOfSongs
from track
join album on album.album_id = track.album_id
join artist on artist.artist_id = album.artist_id
join genre on genre.genre_id = track.genre_id
where genre.name like 'Rock' 
group by artist.artist_id
order by numberOfSongs desc
limit 5


## 8. Find the name on the songs name having the length longer than the average song length.

select name, milliseconds as lengthInMilliseconds from track
where milliseconds > (select avg(milliseconds) as avgTrackLength from track)
order by milliseconds desc



## 9. Find out how much amount is spent by each customer on different artists. We have to display the customer details, the artist name and the total amount spent. First lets create a CTE to make a temporary table.

WITH best_selling_artist AS (
	select artist.artist_id AS artist_id, artist.name AS artist_name, 
	sum(invoice_line.unit_price*invoice_line.quantity) AS total_sales
	from invoice_line
	join track on track.track_id = invoice_line.track_id
	join album on album.album_id = track.album_id
	join artist on artist.artist_id = album.artist_id
	group by 1
	order by 3 desc
	limit 1
)
select c.customer_id, 
c.first_name, 
c.last_name, 
bsa.artist_name, 
sum(il.unit_price*il.quantity) as amount_spent
from invoice i
join customer c on c.customer_id = i.customer_id
join invoice_line il on il.invoice_id = i.invoice_id
join track t on t.track_id = il.track_id
join album alb on alb.album_id = t.album_id
join best_selling_artist bsa on bsa.artist_id = alb.artist_id
group by 1,2,3,4
order by 5 desc


## 10. Find out the most popular music Genre for each country. The most popular genre can be found by knowing the total sales of each genre and sorting them. Here we are also using the row number function to find and use the highest value of each country

WITH popular_genre AS 
(
    select count(invoice_line.quantity) as purchases, customer.country, genre.name, genre.genre_id, 
	row_number() over(partition by customer.country order by count(invoice_line.quantity) desc) as RowNo 
    from invoice_line 
	join invoice on invoice.invoice_id = invoice_line.invoice_id
	join customer on customer.customer_id = invoice.customer_id
	join track on track.track_id = invoice_line.track_id
	join genre on genre.genre_id = track.genre_id
	group by 2,3,4
	order by 2 asc, 1 Desc
)
select * from popular_genre where RowNo <= 1


## 11. Determine the customer details who has spent the most on music for each country. 


WITH Customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1
