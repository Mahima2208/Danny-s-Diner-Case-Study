# Danny-s-Diner-Case-Study

![start slide](https://8weeksqlchallenge.com/images/case-study-designs/1.png)


        CREATE SCHEMA dannys_diner;

        SET search_path = dannys_diner;

        CREATE TABLE sales (

          "customer_id" VARCHAR(1),
  
          "order_date" DATE,
  
          "product_id" INTEGER
  
        );



      INSERT INTO sales

      ("customer_id", "order_date", "product_id")

      VALUES

       ('A', '2021-01-01', '1'),

       ('A', '2021-01-01', '2'),

       ('A', '2021-01-07', '2'),

      ('A', '2021-01-10', '3'),

      ('A', '2021-01-11', '3'),

      ('A', '2021-01-11', '3'),

      ('B', '2021-01-01', '2'),

        ('B', '2021-01-02', '2'),

        ('B', '2021-01-04', '1'),

        ('B', '2021-01-11', '1'),

        ('B', '2021-01-16', '3'),

        ('B', '2021-02-01', '3'),

        ('C', '2021-01-01', '3'),

        ('C', '2021-01-01', '3'),

         ('C', '2021-01-07', '3');




      CREATE TABLE menu (

        "product_id" INTEGER,

        "product_name" VARCHAR(5),

        "price" INTEGER

  );

  INSERT INTO menu

    ("product_id", "product_name", "price")

  VALUES

    ('1', 'sushi', '10'),

    ('2', 'curry', '15'),

    ('3', 'ramen', '12');



  CREATE TABLE members (

    "customer_id" VARCHAR(1),

    "join_date" DATE

  );

  INSERT INTO members

    ("customer_id", "join_date")

  VALUES

    ('A', '2021-01-07'),

    ('B', '2021-01-09');
  
  
  
  

  SELECT * FROM members

  SELECT * FROM dannys_diner.menu

  SELECT * FROM dannys_diner.sales



## What is the total amount each customer spent at the restaurant?

  
  
    SELECT customer_id, SUM(price)

    FROM sales JOIN menu on menu.product_id=sales.product_id

    GROUP BY customer_id


## How many days has each customer visited the restaurant?

    SELECT COUNT(DISTINCT order_date) as number_of_days,customer_id

    from sales

    GROUP BY customer_id
  
## What was the first item from the menu purchased by each customer?
 
      WITH ordered_sales AS

      (

       SELECT sales.customer_id,

      sales.product_id,

      menu.product_name,

      sales.order_date,

      DENSE_RANK() over(PARTITION BY sales.customer_id ORDER BY sales.order_date) AS First_item

      FROM sales JOIN menu 

      ON sales.product_id=menu.product_id
  
     )

      SELECT 

	      customer_id,
  
	      product_name
  
      FROM ordered_sales

      WHERE  First_item= 1;


## What is the most purchased item on the menu and how many times was it purchased by all customers?

    SELECT * FROM menu

    SELECT (COUNT(sales.product_id)) AS max_no,product_name

    from sales JOIN menu on sales.product_id=menu.product_id

    GROUP BY sales.product_id,product_name

    ORDER BY max_no DESC

    LIMIT 1


## Which item was the most popular for each customer?

    WITH fav_item AS

    (

    SELECT customer_id,

    product_name,

    COUNT(menu.product_id) AS order_count,

    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(menu.product_id) DESC) AS rank

    FROM menu JOIN sales

    on menu.product_id=sales.product_id

    GROUP BY sales.customer_id, menu.product_name
  
    )

    SELECT customer_id,product_name,order_count

    FROM fav_item

    where rank=1


## Which item was purchased first by the customer after they became a member?

    WITH ordered_sales AS

   (

    SELECT sales.customer_id,

    sales.product_id,

    sales.order_date,

    members.join_date,

    DENSE_RANK() over(PARTITION BY sales.customer_id ORDER BY sales.order_date ) AS First_item

    FROM sales JOIN members 

    ON sales.customer_id=members.customer_id

    where sales.order_date>=members.join_date
  
   )

    SELECT 

	    customer_id,
  
	    order_date,
  
	    join_date,
  
	    ordered_sales.product_id,
  
	    product_name
  
    FROM ordered_sales JOIN menu ON ordered_sales.product_id=menu.product_id

    WHERE  First_item= 1;


## Which item was purchased just before the customer became a member?

    WITH ordered_sales AS

    (

    SELECT sales.customer_id,

    sales.product_id,

    sales.order_date,

    members.join_date,

    DENSE_RANK() over(PARTITION BY sales.customer_id ORDER BY sales.order_date DESC ) AS First_item

    FROM sales JOIN members 

    ON sales.customer_id=members.customer_id

    where sales.order_date<members.join_date
  
   )

    SELECT 

    customer_id,

    order_date,

    join_date,

    ordered_sales.product_id,
  
	  product_name
  
    FROM ordered_sales JOIN menu ON ordered_sales.product_id=menu.product_id

    WHERE  First_item= 1;


## What is the total items and amount spent for each member before they became a member?

    SELECT s.customer_id, COUNT(DISTINCT s.product_id) AS unique_menu_item, SUM(mm.price) AS total_sales

    FROM sales AS s

    JOIN members AS m

     ON s.customer_id = m.customer_id
 
    JOIN menu AS mm

     ON s.product_id = mm.product_id
 
    WHERE s.order_date < m.join_date

    GROUP BY s.customer_id;


## If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    WITH price_points AS

     (
 
      SELECT *, 
 
         CASE

          WHEN product_id = 1 THEN price * 20

          ELSE price * 10

          END AS points

         FROM menu

    )SELECT s.customer_id, SUM(p.points) AS total_points
 
    FROM price_points AS p

    JOIN sales AS s

     ON p.product_id = s.product_id
 
    GROUP BY s.customer_id


## In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

    SELECT s.customer_id,

      SUM(CASE WHEN product_name = 'sushi' 

        OR order_date BETWEEN CAST(join_date as timestamp) 

        AND CAST(join_date as timestamp) + INTERVAL '6 DAY' THEN price*10*2

         ELSE price*10 END) AS points
       
    FROM dannys_diner.sales AS s

    JOIN dannys_diner.menu AS m

    ON s.product_id = m.product_id

    LEFT JOIN dannys_diner.members AS m1

    ON s.customer_id = m1.customer_id

    WHERE s.customer_id IN ('A', 'B')

    AND EXTRACT(month from order_date) = 1

    GROUP BY s.customer_id
