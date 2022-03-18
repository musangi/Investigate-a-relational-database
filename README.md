# Investigate-a-relational-database
The sakila movie database is a SQL database of online DVD rentals. You will query the database to answer the questions about business decisions
/* Udacity 1st project: Investigate a Relational Database */


/* Slide 1: How many movies were rented out in each family friendly category? */

/*Question 2:
Now we need to know how the length of rental duration of these family-friendly movies compares to the duration that all movies
are rented for. Can you provide a table with the movie titles and divide them into 4 levels (first_quarter, second_quarter,
third_quarter, and final_quarter) based on the quartiles (25%, 50%, 75%) of the rental duration for movies across all categories?
Make sure to also indicate the category that these family-friendly movies fall into.*/


SELECT film_title, category_name, rental_duration, NTILE(4) OVER (ORDER BY rental_duration) AS standard_quartile
FROM
(
  SELECT f.title AS film_title,
         c.name AS category_name,
         f.rental_duration

  FROM film AS f
        JOIN film_category AS fc
          ON f.film_id = fc.film_id


        JOIN category AS c
          ON c.category_id = fc.category_id
          AND c.name IN ('Animation', 'Children', 'Classics','Comedy', 'Family', 'Music')
) AS t1


/* Slide 2: For each quartile, count how many times a film category was rented out? */

/* Question 3
Finally, provide a table with the family-friendly film category, each of the quartiles, and the corresponding count of
movies within each combination of film category for each corresponding rental duration category. The resulting table should have three columns:

Category
Rental length category
Count
*/

WITH t1 AS (
	       SELECT f.title AS film_title,
                c.name AS category_name,
	              f.rental_duration,
	              NTILE(4) OVER(ORDER BY rental_duration) AS standard_quartile
               FROM film AS f
               JOIN film_category AS fc
                ON f.film_id = fc.film_id

               JOIN category AS c
                 ON c.category_id = fc.category_id
                 AND c.name IN ('Animation','Children','Classics','Comedy','Family','Music')
             )

SELECT category_name,
       standard_quartile,
       COUNT(film_title)
FROM t1
GROUP BY 1,2
ORDER BY 1,2






/* Slide 3:Who were our top paying customers and how much were their monthly payments? */
/*Question 2
We would like to know who were our top 10 paying customers, how many payments they made on a monthly basis
during 2007, and what was the amount of the monthly payments.Can you write a query to capture the
customer name, month and year of payment, and total payment amount for each month by these top 10 paying customers?
*/
WITH t1 AS
(
  SELECT  c.first_name ||' '|| c.last_name AS fullname,
          c.customer_id,
          SUM(p.amount) As total
  FROM customer AS c
  JOIN Payment AS p
    ON c.customer_id = p.customer_id

  GROUP BY 1,2
  ORDER BY 3 DESC
  LIMIT 10
)
SELECT DATE_TRUNC('month', payment_date) AS pay_mon,
       fullname,
       COUNT(p.*) AS pay_countpermon,
       SUM(p.amount) AS pay_amount
FROM t1
      JOIN payment AS p
        ON p.customer_id = t1.customer_id
GROUP BY 1,2
ORDER BY 2


/* Slide 4: For the top paying customers, what is the difference across their monthly payments? */

/*Question 3
Finally, for each of these top 10 paying customers, I would like to find out the difference across their monthly payments during 2007.
Please go ahead and write a query to compare the payment amounts in each successive month. Repeat this for each
of these 10 paying customers. Also, it will be tremendously helpful if you can identify the customer name who paid the most
difference in terms of payments.*/

WITH t1 AS
(
  SELECT  c.first_name ||' '|| c.last_name AS fullname,
          c.customer_id,
          SUM(p.amount) AS total
  FROM customer AS c
  JOIN Payment AS p
    ON c.customer_id = p.customer_id

  GROUP BY 1,2
  ORDER BY 3 DESC
  LIMIT 10
),
t2 AS
(
  SELECT DATE_TRUNC('month', payment_date) AS pay_mon,
         fullname,
         COUNT(p.*) AS pay_countpermon,
         SUM(p.amount) AS pay_amount
  FROM t1
        JOIN payment AS p
          ON p.customer_id = t1.customer_id
  GROUP BY 1,2
  ORDER BY 2
)
SELECT *,
       COALESCE(LEAD(pay_amount) OVER (PARTITION BY fullname ORDER BY pay_mon) - pay_amount,null) AS difference
FROM t2
