# Music-Store

-- Q1. Who is the Senior most employee based on the job title

select * from employee
order by  levels desc
limit 1

-- Q2. Which country have the most Invoices

select billing_country, count(billing_country) from invoice
group by billing_country
order by count(billing_country) desc
limit 1

-- Q3. What are top 3 values of total invoice

select total from invoice
order by total desc
limit 3


-- Q4. Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money.
-- Write a query that returns one city that has the highest sum of invoice totals Return both the city name & sum of all invoice totals

select billing_city, sum(total)from invoice
group by billing_city
order by sum(total) desc
limit 1


-- Q5. Who is the best customer? The customer who has spent the most money will be declared the best customer. 
-- Write a query that returns the person who has spent the most money



SELECT 
    i.customer_id, c.first_name AS Customer_Name, 
    SUM(i.total) AS Total_Invoice
FROM customer AS c 
JOIN invoice AS i
USING (customer_id)
GROUP BY  i.customer_id
ORDER BY  SUM(i.total) DESC
limit 1


-- Question Set 2 - Moderate
-- Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
-- Return your list ordered alphabetically by email starting with A

select c.email, c.first_name, c.last_name, g.Name
from customer c
join invoice i using (customer_id)
join invoice_line il using (invoice_id)
join track t using (track_id)
join genre g using (genre_id)
where g.Name = 'Rock'
group by c.customer_id,g.Name
order by c.email

-- Q2: Let's invite the artists who have written the most rock music in our dataset.
-- Write a query that returns the Artist name and total track count of the top 10 rock bands


select art.Name as Artist , count(*) Track_count, g.Name as Genre
from track t
join genre g using (genre_id)
join album a using (album_id)
join artist art using (artist_id)
where g.name = 'Rock'
group by Artist, Genre
order by Track_count Desc
limit 10


-- Q3: Return all the track names that have a song length longer than the average song length. 
-- Return the Name and Milliseconds for each track.
-- Order by the song length with the longest songs listed first.


select name , milliseconds 
from track 
where milliseconds >(select avg(milliseconds) from track)
order by milliseconds  desc


-- Question Set 3 - Advance 
-- Q1: Find how much amount spent by each customer on best artists? 
-- Write a query to return customer name, artist name and total spent

with best_selling_artist as(
select art.artist_id, art.Name as Artist,SUM(il.unit_price * il.quantity) AS total_spend
from customer c 
join invoice i using(customer_id)
join invoice_line il using(invoice_id)
join track t using(track_id)
join album a using(album_id)
join artist art using(artist_id)
group by art.artist_id,Artist
order by total_spend desc
limit 1
)

select c.first_name, c.last_name, bsa.Artist,SUM(il.unit_price * il.quantity) AS total_spend
from customer c 
join invoice i using(customer_id)
join invoice_line il using(invoice_id)
join track t using(track_id)
join album a using(album_id)
join best_selling_artist bsa using (artist_id)
group by c.first_name, c.last_name, bsa.Artist
order by total_spend desc


-- Q2: We want to find out the most popular music Genre for each country We determine the most popular genre as the genre 
-- with the highest amount of purchases.Write a query that returns each country along with the top Genre. 
-- For countries where the maximum number of purchases is shared return all Genres

-- M1
with new_table as (
select c.country, g.Name Genre,count(il.quantity) AS total_spend,
row_number() over (partition by c.country order by count(il.quantity) desc ) as rn
from customer c 
join invoice i using(customer_id)
join invoice_line il using(invoice_id)
join track t using(track_id)
join album a using(album_id)
join genre g using (genre_id)
group by c.country, g.Name
order by c.country
)

select country, Genre,total_spend, rn
from new_table 
where rn=1

-- M2

with recursive new_table as (
select c.country, g.Name Genre,count(il.quantity) AS perchase
from customer c 
join invoice i using(customer_id)
join invoice_line il using(invoice_id)
join track t using(track_id)
join album a using(album_id)
join genre g using (genre_id)
group by c.country, g.Name
order by c.country
),
max_purchase as ( select country, max(perchase) as max_perchase from new_table group by 1)

select * from new_table nt
join max_purchase mp using (country)
where mp.max_perchase =nt.perchase


-- Q3: Write a query that determines the customer that has spent the most on music for each country. 
-- Write a query that returns the country along with the top customer and how much they spent. 
-- For countries where the top amount spent is shared, provide all customers who spent this amount

-- M1

with new_table as (
select customer_id, first_name,last_name ,i.billing_country ,
sum(total) AS Amount,
row_number() over (partition by  i.billing_country order by sum(total) desc ) as rn
from customer c 
join invoice i using(customer_id)
join invoice_line il using(invoice_id)
group by 1,2,3,4
order by i.billing_country, Amount desc
)

select * from new_table 
where rn=1



-- M2

with recursive new_table as(
select customer_id, first_name,last_name ,i.billing_country ,
sum(i.total) AS Amount
from customer c 
join invoice i using(customer_id)
join invoice_line il using(invoice_id)
group by 1,2,3,4
order by i.billing_country
),
max_amt as (select max(nt.amount) as Amount, nt.billing_country from new_table as nt
		   group by nt.billing_country)
		 
select * from new_table nt
join max_amt  ma using (billing_country)
where nt.Amount =ma.Amount


















