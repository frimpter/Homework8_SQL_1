-- HOMEWORK 8: SQL

use sakila;

-- 1a. Display the first and last names of all actors from the table actor.

select first_name, last_name
from actor;


-- 1b. Display the first and last name of each actor in a single column in upper case letters. Name the column Actor Name.

select first_name, last_name, concat(first_name, ' ', last_name) as 'Actor_Name'
from actor;


-- 2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?

select actor_id, first_name, last_name
from actor
having first_name = "Joe";


-- 2b. Find all actors whose last name contain the letters GEN:

select actor_id, first_name, last_name
from actor
where last_name like "%GEN%";


-- 2c. Find all actors whose last names contain the letters LI. This time, order the rows by last name and first name, in that order:

select actor_id, first_name, last_name
from actor
where last_name like "%LI%"
order by last_name, first_name ASC;


-- 2d. Using IN, display the country_id and country columns of the following countries: Afghanistan, Bangladesh, and China:

select country_id, country
from country
where country in ("Afghanistan", "Bangladesh", "China");


-- 3a. Add a middle_name column to the table actor. Position it between first_name and last_name. Hint: you will need to specify the data type.

alter table actor
add middle_name varchar(50) after first_name;
select * from actor;


-- 3b. You realize that some of these actors have tremendously long last names. Change the data type of the middle_name column to blobs.

alter table actor
modify column middle_name blob;
select * from actor;

-- 3c. Now delete the middle_name column.

alter table actor
drop column middle_name;
select * from actor;


-- 4a. List the last names of actors, as well as how many actors have that last name.

select last_name, count(last_name) as 'frequency'
from actor
group by last_name;


-- 4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors

select last_name, count(last_name) as 'frequency'
from actor
group by last_name
having frequency > 1;


-- 4c. Oh, no! The actor HARPO WILLIAMS was accidentally entered in the actor table as GROUCHO WILLIAMS, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.

update actor
set first_name = "HARPO"
where first_name = "GROUCHO" and last_name = "WILLIAMS";
select * from actor order by first_name ASC;


-- 4d. Perhaps we were too hasty in changing GROUCHO to HARPO. It turns out that GROUCHO was the correct name after all! In a single query, if the first name of the actor is currently HARPO, change it to GROUCHO. Otherwise, change the first name to MUCHO GROUCHO, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO MUCHO GROUCHO, HOWEVER! (Hint: update the record using a unique identifier.)
-- This task is unclear. Change "HARPO WILLIAMS" back to "GROUCHO WILLIAMS." OK, but there is no scenario to match the "Otherwise..." part of the task.

update actor
set first_name = "GROUCHO"
where first_name = "HARPO" and last_name = "WILLIAMS";
select * from actor order by first_name ASC;


-- 5a. You cannot locate the schema of the address table. Which query would you use to re-create it?
-- Hint: https://dev.mysql.com/doc/refman/5.7/en/show-create-table.html

show create table address;


-- 6a. Use JOIN to display the first and last names, as well as the address, of each staff member. Use the tables staff and address:

select staff.first_name, staff.last_name, address.address
from staff
join address on (staff.address_id = address.address_id);


-- 6b. Use JOIN to display the total amount rung up by each staff member in August of 2005. Use tables staff and payment.

select concat(first_name, ' ', last_name) as 'staff_member', sum(amount) as 'total_amount'
from staff
join payment on (staff.staff_id = payment.staff_id)
where payment_date like "2005-08%"
group by staff_member;


-- 6c. List each film and the number of actors who are listed for that film. Use tables film_actor and film. Use inner join.

select title, count(actor_id) as 'total_actors'
from film
inner join film_actor on (film.film_id = film_actor.film_id)
group by title
order by total_actors desc;


-- 6d. How many copies of the film Hunchback Impossible exist in the inventory system?

select title, count(inventory_id) as 'total_inventory'
from film
join inventory on (film.film_id = inventory.film_id)
where title like "Hunchback Impossible"
group by title;


-- 6e. Using the tables payment and customer and the JOIN command, list the total paid by each customer. List the customers alphabetically by last name:
-- 	![Total amount paid](Images/total_payment.png)

select first_name, last_name, sum(amount) as 'total_paid'
from customer
join payment on (customer.customer_id = payment.customer_id)
group by first_name, last_name
order by last_name asc;


-- 7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters K and Q have also soared in popularity. Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.

select title
from film
where language_id in
(select language_id from language where language_id = 1)
AND title like 'K%'
OR title like 'Q%'
order by title asc;


-- 7b. Use subqueries to display all actors who appear in the film Alone Trip.

select concat(first_name, ' ', last_name) as 'actors_in_alone_trip'
from actor
where actor_id in
(select actor_id from film_actor where film_id in
(select film_id from film where title = "Alone Trip")
);


-- 7c. You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.

select concat(first_name, ' ', last_name) as 'customer', email, country
from customer c
join address a on (c.address_id = a.address_id)
join city on (a.city_id = city.city_id)
join country on (city.country_id = country.country_id)
where country = "Canada"
order by customer;


-- 7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as famiy films.

select title as 'family_films' from film
where film_id in 
(select film_id from film_category where category_id in 
(select category_id from category where name = "Family")
)
order by title asc;

-- 7e. Display the most frequently rented movies in descending order.

select title, count(rental_id) as 'total_rentals'
from film f
inner join inventory i on (f.film_id = i.film_id)
inner join rental r on (i.inventory_id = r.inventory_id)
group by title
order by total_rentals desc;


-- 7f. Write a query to display how much business, in dollars, each store brought in.

select s.store_id, sum(p.amount) as 'total_sales'
from store s
join customer c on (s.store_id = c.store_id)
join payment p on (c.customer_id = p.customer_id)
group by s.store_id
order by total_sales desc;


-- 7g. Write a query to display for each store its store ID, city, and country.

select s.store_id, city.city, country.country
from store s
join address a on (s.address_id = a.address_id)
join city on (a.city_id = city.city_id)
join country on (city.country_id = country.country_id);


-- 7h. List the top five genres in gross revenue in descending order. (Hint: you may need to use the following tables: category, film_category, inventory, payment, and rental.)

select c.name, sum(p.amount) as 'total_revenue'
from category c
join film_category fc on (c.category_id = fc.category_id)
join inventory i on (fc.film_id = i.film_id)
join rental r on (i.inventory_id = r.inventory_id)
join payment p on (r.customer_id = p.customer_id)
group by c.name
order by total_revenue desc
limit 5;


-- 8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.

create view top_5_by_genre as 
select c.name, sum(p.amount) as 'total_revenue'
from category c
join film_category fc on (c.category_id = fc.category_id)
join inventory i on (fc.film_id = i.film_id)
join rental r on (i.inventory_id = r.inventory_id)
join payment p on (r.customer_id = p.customer_id)
group by c.name
order by total_revenue desc
limit 5;


-- 8b. How would you display the view that you created in 8a?

select * from top_5_by_genre;


-- 8c. You find that you no longer need the view top_five_genres. Write a query to delete it.

drop view top_5_by_genre;
