I have analyzed DVD rental data using postgresql in PG Admin along with the tools of sql.

It seems to be describing a relational database schema for a DVD rental system. Here's an overview of how these tables can be structured and related to one another,
along with a brief explanation of their purpose:

**KEY TABLES AND THEIR RELATIONSHIP**

1. Actor Table
Purpose: Stores details about actors.
Fields:
actor_id (Primary Key)
first_name
last_name

2. Films Table
Purpose: Stores details about films available for rent.
Fields:
film_id (Primary Key)
title
description
release_year
language_id (Foreign Key to language table)
rental_duration
rental_rate
length
rating_id (Foreign Key to rating table)

3. Staff Table

Purpose: Stores details about the staff managing rentals.
Fields:
staff_id (Primary Key)
first_name
last_name
email
store_id (Foreign Key to store table)
active
username

4. Rentals Table

Purpose: Tracks rental transactions.
Fields:
rental_id (Primary Key)
rental_date
inventory_id (Foreign Key to inventory table)
customer_id (Foreign Key to customer table)
return_date
staff_id (Foreign Key to staff table)

5. Film Actor Table (Many-to-Many Relationship)

Purpose: Links films with actors.
Fields:
film_id (Foreign Key to film table)
actor_id (Foreign Key to actor table)

6. Language Table

Purpose: Stores languages for films.
Fields:
language_id (Primary Key)
name

7. Inventory Table

Purpose: Stores information about copies of films in stock.
Fields:
inventory_id (Primary Key)
film_id (Foreign Key to film table)
store_id (Foreign Key to store table)

8. Store Table

Purpose: Represents stores where rentals are managed.
Fields:
store_id (Primary Key)
manager_staff_id (Foreign Key to staff table)
address_id (Foreign Key to address table)

9. Customer Table

Purpose: Stores customer details.
Fields:
customer_id (Primary Key)
first_name
last_name
email
address_id (Foreign Key to address table)
active

10. Address Table

Purpose: Stores address details for customers and stores.
Fields:
address_id (Primary Key)
address
city
postal_code


**SCHEMA EXAMPLE**

Here’s how the relationships look:

Actor ↔ Film (Many-to-Many via film_actor)
Film ↔ Inventory ↔ Stor
Customer ↔ Rental ↔ Staff
Film ↔ Language


To include features like ORDER BY, GROUP BY, JOIN, and SQL clauses like ASC, DESC, OFFSET, and LIMIT in a DVD Rental ER Diagram, 
we focus on how data is structured in the database schema to support these queries.

Here’s a textual description of a conceptual ER Diagram for a DVD Rental system with query-supporting attributes:

**ENTITIES AND ATTRIBUTE**

1. Customer

CustomerID (Primary Key)
Name
Email
Phone

2. Rental

RentalID (Primary Key)
RentalDate
ReturnDate
CustomerID (Foreign Key from Customer)
DVDID (Foreign Key from DVD)

3. DVD

DVDID (Primary Key)
Title
Genre
ReleaseYear
Price

4. Staff

StaffID (Primary Key)
Name
Role

5. Payment

PaymentID (Primary Key)
RentalID (Foreign Key from Rental)
PaymentDate
Amount

**RELATIONSHIPS**

1. Customer rents DVDs through the Rental table.

2. Each Rental is linked to a Payment.

3. Staff manages the rental and return process.


**FEATURES OF SQL QUERIES**

1. ORDER BY

Support sortable fields like RentalDate in the Rental entity, Price in DVD.


2. GROUP BY

Aggregate queries based on Genre in DVD or CustomerID in Rental.


3. JOIN

Relationships ensure JOIN operations across Customer, Rental, DVD, and Payment.


4. ASC/DESC

Sort orders can be applied on any date or numeric fields.


5. OFFSET/LIMIT

Pagination on query results like rentals or payments.



-----finding table and find a data in table by colum-----

select * from actor;

select * from film;

select  title,description from film;

select * from staff;

select first_name,last_name,address_id from staff;

select * from film where rental_rate < 1 and length >100 and  release_year != 2006;

select title, rating from film where rental_rate < 1 and length >100 and title like 'S%o%';

select title, rating from film where rental_rate < 1 and length >100 and title like 'S%l';

select title, rating from film where rental_rate < 1 and length >100 and title like 'S_o%';

select title, rating from film where rental_rate < 1 and length >100 and title like 'A%a%a%';

select title, rating,rental_rate from film where rental_rate::text like '0%' limit 5;

select title, rating,rental_rate from film where rental_rate::text like '0%' limit 5 offset 3;

select * from film where rental_rate::text like '0%' order by title desc;

select * from film where rental_rate::text like '0%' order by title asc;

select count(*) from film ;


----- title rental_duration asc, rating desc -----

select * from film where rental_rate::text like '0%' order by rental_duration asc,rating desc;

----- join with two or more tables -----

select f.title,i.inventory_id from film f join inventory i on f.film_id = i.film_id;

select f.title,count(i.inventory_id) as inventory_count from film f join inventory i on f.film_id = i.film_id group by f.title order by inventory_count desc;

select * from film ;  

select * from film where language_id != 1;

select count(film_id),rating from film group by rating;

select l.language_id,count(f.film_id) from film f join language l on f.language_id = l.language_id group by l.language_id ;

select * from language;

select count(*) from rental;

select c.customer_id, count(distinct fc.category_id) as rental_category from rental r
join inventory i on r.inventory_id = i.inventory_id
join film_category fc on i.film_id = fc.film_id
join customer c on r.customer_id =c.customer_id group by c.customer_id;

select c.customer_id, count(distinct fc.film_id) as rental_category from rental r
join inventory i on r.inventory_id = i.inventory_id
join film_category fc on i.film_id = fc.film_id
join customer c on r.customer_id =c.customer_id group by c.customer_id order by rental_category desc;

select count(*) as total from category;
with category_count as (select c.customer_id, count(distinct fc.category_id) as rental_category from rental r
join inventory i on r.inventory_id = i.inventory_id
join film_category fc on i.film_id = fc.film_id
join customer c on r.customer_id =c.customer_id group by c.customer_id),
total_category as (select count(*) as total from category)

select c.customer_id , c.first_name from category_count cc join customer c on cc.customer_id = c.customer_id
join total_category tc on cc.rental_category = tc.total ;

----- using with condition,group by,order by,desc,asc -----

with category_count as (select c.customer_id, count(distinct fc.category_id) as rental_category from rental r
join inventory i on r.inventory_id = i.inventory_id
join film_category fc on i.film_id = fc.film_id
join customer c on r.customer_id =c.customer_id group by c.customer_id)

select * from category_count;

select a.actor_id, a.first_name ,count(distinct c.customer_id) as fancount from actor a
join film_actor fa on fa.actor_id = a.actor_id
join inventory i on fa.film_id = i.film_id
join rental r on i.inventory_id = r.inventory_id
join customer c on r.customer_id = c.customer_id group by a.actor_id
order by fancount desc;


REFERENCE LINK:https://neon.tech/postgresqltutorial/printable-postgresql-sample-database-diagram.pdf
