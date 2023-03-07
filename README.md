# Advance_SQL_Assignment
Github repository for Advance SQL Assignment


### Q1. Write a query that gives an overview of how many films have replacements costs in the following cost ranges

low: 9.99 - 19.99
medium: 20.00 - 24.99
high: 25.00 - 29.99

### Solution:
### Query:

SELECT  
SUM(CASE WHEN replacement_cost BETWEEN 9.99 AND 19.99 THEN 1 ELSE 0 END) AS low,  
SUM(CASE WHEN replacement_cost BETWEEN 20.00 AND 24.99 THEN 1 ELSE 0 END) AS medium,  
SUM(CASE WHEN replacement_cost BETWEEN 25.00 AND 29.99 THEN 1 ELSE 0 END) AS high  
FROM film;  

### Output:
<img width="1225" alt="Screenshot 2023-03-07 at 4 48 56 PM" src="https://user-images.githubusercontent.com/122514456/223416292-9445db44-8d6c-4a98-b83b-5f97688152fd.png">


### Q2. Write a query to create a list of the film titles including their film title, film length and film category name ordered descendingly by the film length. Filter the results to only the movies in the category 'Drama' or 'Sports'.  
Eg.	"STAR OPERATION"		"Sports"		181
	  "JACKET FRISCO"		  "Drama"		  181  

### Solution:
### Query:

SELECT f.title, f.length, c.name  
FROM film AS f INNER JOIN film_category AS fc  
ON f.film_id = fc.film_id  
INNER JOIN category AS c  
ON fc.category_id = c.category_id  
WHERE c.name IN ('Sports', 'Drama')  
ORDER BY f.length DESC;  

### Output:
<img width="1225" alt="Screenshot 2023-03-07 at 4 56 22 PM" src="https://user-images.githubusercontent.com/122514456/223416343-872f824d-0bae-44c6-b4f7-b6488358db98.png">


### Q3. Write a query to create a list of the addresses that are not associated to any customer.

### Solution:
### Query:

SELECT a.address_id, a.address, a.district, a.city_id  
FROM address AS a LEFT JOIN customer AS c  
ON a.address_id = c.address_id  
WHERE c.customer_id IS NULL;  

### Output:
<img width="1225" alt="Screenshot 2023-03-07 at 4 57 33 PM" src="https://user-images.githubusercontent.com/122514456/223416384-63079ea4-4305-4328-9fcd-4891b0b8790c.png">


### Q4. Write a query to create a list of the revenue (sum of amount) grouped by a column in the format "country, city" ordered in decreasing amount of revenue.  
eg. 	"Poland, Bydgoszcz"		52.88

### Solution:
### Query:

SELECT CONCAT(co.country, ', ', ci.city) AS Country_City, round(sum(p.amount), 2) AS revenue  
FROM payment AS p INNER JOIN customer AS cu  
ON p.customer_id = cu.customer_id  
INNER JOIN address AS a  
ON cu.address_id = a.address_id  
INNER JOIN city AS ci  
ON ci.city_id = a.city_id  
INNER JOIN country AS co   
ON co.country_id = ci.country_id  
GROUP BY co.country, ci.city;  

### Output:
<img width="1225" alt="Screenshot 2023-03-07 at 4 58 56 PM" src="https://user-images.githubusercontent.com/122514456/223416468-c5fed281-36e2-4b4d-858f-3c8fb56eb642.png">



### Q5. Write a query to create a list with the average of the sales amount each staff_id has per customer.
result: 	2	56.64
		      1	55.91  

### Solution:
### Query:

SELECT t1.staff_id, round(AVG(t1.total_sum), 2)  
FROM  
(SELECT p.staff_id, p.customer_id, SUM(p.amount) AS total_sum  
FROM payment AS p  
GROUP BY p.staff_id, p.customer_id) AS t1  
GROUP BY t1.staff_id;   

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 03 38 PM" src="https://user-images.githubusercontent.com/122514456/223416542-33483922-ebc2-4b8f-b423-1a1afc167fc1.png">


### Q6. Write a query that shows average daily revenue of all Sundays.

### Solution:
### Query:

SELECT ROUND(AVG(t1.sum_by_each_sunday), 2)  
FROM  
(SELECT DATE(payment_date), SUM(amount) AS sum_by_each_sunday  
FROM payment  
WHERE DAYNAME(payment_date)='Sunday'  
GROUP BY DATE(payment_date)) AS t1;  

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 05 22 PM" src="https://user-images.githubusercontent.com/122514456/223416572-6afdf152-383f-4795-a298-3d604fd59fdb.png">


### Q7. Write a query to create a list that shows how much the average customer spent in total (customer life-time value) grouped by the different districts.

### Solution:
### Query:

SELECT t1.district, AVG(t1.avg_amt_by_district)  
FROM  
(SELECT a.district, c.customer_id, SUM(p.amount) AS avg_amt_by_district  
FROM payment AS p  
INNER JOIN customer AS c  
ON p.customer_id = c.customer_id  
INNER JOIN address AS a  
ON a.address_id = c.address_id  
GROUP BY a.district, c.customer_id) AS t1  
GROUP BY t1.district;  

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 06 52 PM" src="https://user-images.githubusercontent.com/122514456/223416616-530e2e0b-675b-477e-a8ee-9e5bd744c4d5.png">


### Q8. Write a query to list down the highest overall revenue collected (sum of amount per title) by a film in each category. Result should display the film title, category name and total revenue.
eg. 	"FOOL MOCKINGBIRD"		  "Action"		  175.77
	    "DOGMA FAMILY"			    "Animation"	  178.7
	    "BACKLASH UNDEFEATED"	  "Children"	  158.81  

### Solution:
### Query:

SELECT t2.title, t2.name, t2.max_revenue_by_category  
FROM  
	(SELECT distinct t1.title, t1.name, t1.total_revenue_by_film,  
MAX(t1.total_revenue_by_film) OVER(PARTITION BY t1.name) AS max_revenue_by_category  
FROM  
		(SELECT f.title, c.name,   
		SUM(p.amount) AS total_revenue_by_film  
		FROM film AS f  
		INNER JOIN film_category AS fc  
		ON f.film_id = fc.film_id  
		INNER JOIN category AS c  
		ON fc.category_id = c.category_id  
		INNER JOIN inventory AS i  
		ON i.film_id = f.film_id  
		INNER JOIN rental AS r  
		ON r.inventory_id = i.inventory_id  
		INNER JOIN payment AS p  
		ON p.rental_id = r.rental_id  
		GROUP BY f.title, c.name) AS t1  
	) AS t2  
WHERE max_revenue_by_category=t2.total_revenue_by_film;  

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 08 25 PM" src="https://user-images.githubusercontent.com/122514456/223416671-d72590f4-8e9c-41a7-90c6-60b2bc62eb91.png">


### Q9. Modify the table "rental" to be partitioned using PARTITION command based on ‘rental_date’ in below intervals:
	<2005  
  between 2005–2010  
	between 2011–2015  
	between 2016–2020  
	>2020 - Partitions are created yearly  

### Solution:
### Query:

ALTER TABLE rental  
PARTITION BY RANGE (YEAR(rental_date))  
(  
PARTITION p1_less_than_2005 VALUES LESS THAN (2005),  
PARTITION p2_between_2005_2010 VALUES LESS THAN (2011),  
PARTITION p3_between_2011_2015 VALUES LESS THAN (2016),  
PARTITION p4_between_2016_2020 VALUES LESS THAN (2021),  
PARTITION p5_greater_than_2020 VALUES LESS THAN (MAXVALUE)  
);  

### Q10. Modify the table "film" to be partitioned using PARTITION command based on ‘rating’ from below list. Further apply hash sub-partitioning based on ‘film_id’ into 4 sub-partitions.

partition_1 - "R"  
partition_2 - "PG-13", "PG"  
partition_3 - "G", "NC-17"  

### Solution:
### Query:

ALTER TABLE film  
PARTITION BY LIST (rating)  
SUBPARTITION BY HASH (film_id) SUBPARTITIONS 4 (  
PARTITION partition_1 VALUES ('R'),  
PARTITION partition_2 VALUES ('PG-13', 'PG'),  
PARTITION partition_3 VALUES ('G', 'NC-17'),  
);  

### Q11. Write a query to count the total number of addresses from the “address” table where the ‘postal_code’ is of the below formats. Use regular expression.

9*1**, 9*2**, 9*3**, 9*4**, 9*5**  

eg. postal codes - 91522, 80100, 92712, 60423, 91111, 9211  
result - 2  

### Solution:
### Query:

SELECT COUNT(address_id)  
FROM address  
WHERE postal_code REGEXP '9.[1-5]..';  

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 10 22 PM" src="https://user-images.githubusercontent.com/122514456/223416818-5f9c25ec-eca1-4068-9924-06154fde4dcd.png">


### Q12. Write a query to create a materialized view from the “payment” table where ‘amount’ is between(inclusive) $5 to $8. The view should manually refresh on demand. Also write a query to manually refresh the created materialized view.

### Solution:
### Query:

CREATE VIEW payment_between_5_8 AS  
SELECT *  
FROM payment  
WHERE amount BETWEEN 5 AND 8;  

delimiter $$;  
CREATE EVENT refresh_payment_between_5_8  
ON SCHEDULE EVERY 1 DAY  
DO  
BEGIN  
  CREATE OR REPLACE VIEW payment_between_5_8 AS  
  SELECT *  
  FROM payment  
  WHERE amount BETWEEN 5 AND 8;  
END$$;  
delimiter ;  

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 13 36 PM" src="https://user-images.githubusercontent.com/122514456/223416872-3c3a4308-009c-4695-a413-e7e8ca57a8d4.png">


### Q13. Write a query to list down the total sales of each staff with each customer from the ‘payment’ table. In the same result, list down the total sales of each staff i.e. sum of sales from all customers for a particular staff. Use the ROLLUP command. Also use GROUPING command to indicate null values.

### Solution:
### Query:

SELECT staff_id, customer_id, GROUPING(staff_id) AS flag1,  
GROUPING(customer_id) AS flag2, SUM(amount) AS grouped_amt  
FROM payment  
GROUP BY staff_id, customer_id  
WITH ROLLUP;  

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 17 10 PM" src="https://user-images.githubusercontent.com/122514456/223416921-4546d72f-0936-422c-97a3-7e7b5464c4bc.png">


### Q.14 Write a single query to display the customer_id, staff_id, payment_id, amount, amount on immediately previous payment_id, amount on immediately next payment_id ny_sales for the payments from customer_id ‘269’ to  staff_id ‘1’.

### Solution:
### Query:

SELECT customer_id, staff_id, payment_id, amount,  
LAG(amount, 1) OVER(ORDER BY payment_id) AS previous_payment_id_amt,  
LEAD(amount, 1) OVER(ORDER BY payment_id) AS next_payment_id_amt,  
LAG(amount, 1) OVER(PARTITION BY customer_id, staff_id ORDER BY payment_id) AS py_sales,  
LEAD(amount, 1) OVER(PARTITION BY customer_id, staff_id ORDER BY payment_id) AS ny_sales  
FROM payment  
WHERE customer_id=269 and staff_id=1;  

### Output:
<img width="1226" alt="Screenshot 2023-03-07 at 5 20 49 PM" src="https://user-images.githubusercontent.com/122514456/223416973-282bd952-729b-4efc-841a-a8f3aaf4da3a.png">

