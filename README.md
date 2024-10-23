# Pizza-Sales-Analysis---MySQL-Project
**Overview**

This project focuses on the analysis of pizza sales at a fictional pizza restaurant, PizzaHut, using MySQL. The project manages and analyzes sales data, order details, and pizza information. The data is stored in four key tables that track pizzas, orders, order details, and pizza types.

The project covers several important business insights, such as:

1) Tracking the total number of orders
2) Calculating revenue
3) Identifying the most popular pizzas and their categories
4) Analyzing ordering patterns
5) Extracting valuable business insights using SQL queries

   
**Project Structure**

* **Database Name:** pizzahut

* **Tables Used:**

  * **pizzas:** Contains details of the pizzas sold (type, size, price).

  * **orders:** Contains order data (order date, time).

  * **order_details:** Contains detailed information of orders, including the quantity and pizza ordered.

  * **pizza_types:** Contains information about pizza types (name, category, ingredients).

            
**Tables**

**1. pizzas**

This table contains details about all available pizzas at PizzaHut:

* **pizza_id:** Unique identifier for each pizza

* **pizza_type_id:** Refers to the pizza type in the pizza_types table

* **size:**  The size of the pizza (small, medium, large)

* **price:** The price of the pizza

**2. orders**

This table stores data about the orders placed by customers:

* **order_id:** Unique identifier for each order
  
* **date:** The date the order was placed
  
* **time:** The time the order was placed

**3. order_details**

This table contains specific details about each order, such as the pizza ordered and quantity:

* **order_details_id:** Unique identifier for each order item
  
* **order_id:** Links to the corresponding order in the orders table
  
* **pizza_id:** Refers to the corresponding pizza in the pizzas table
  
* **quantity:** Quantity of pizzas ordered

**4. pizza_types**

This table contains information about the different types of pizzas:

* **pizza_type_id:** Unique identifier for each pizza type
  
* **name:** The name of the pizza type (e.g., Margherita, Pepperoni)
  
* **category:** The category of the pizza (e.g., Vegetarian, Meat Lovers)
  
* **ingredients:** Key ingredients used in the pizza type
  

**SQL Queries and Business Insights**

**1. Total Orders Placed**

Retrieves the total number of orders placed:

     SELECT count(order_id) AS total_orders
     FROM orders;

**2. Total Revenue from Pizza Sales**

Calculates the total revenue generated from pizza sales:

     SELECT 
     round(SUM(order_details.quantity*pizzas.price),2) AS total_sales
     FROM order_details
     JOIN pizzas
          ON order_details.pizza_id = pizzas.pizza_id;
          
**3. Highest Priced Pizza**

Identifies the highest-priced pizza in the database:

     SELECT pizza_types.name, pizzas.price
     FROM pizza_types
     JOIN pizzas
          ON pizza_types.pizza_type_id = pizzas.pizza_type_id
     ORDER BY pizzas.price DESC 
     LIMIT 1;
     
**4. Most Common Pizza Size Ordered**

Identifies the most frequently ordered pizza size:

     SELECT pizzas.size, COUNT(order_details.order_details_id) AS order_count
     FROM pizzas
     JOIN order_details
      	 ON pizzas.pizza_id = order_details.pizza_id 
     GROUP BY pizzas.size ORDER BY order_count DESC;
     
**5. Top 5 Most Ordered Pizza Types**

Lists the top 5 most popular pizza types and their quantities:

     SELECT pizza_types.name, SUM(order_details.quantity) AS Quantity
     FROM pizza_types
     JOIN pizzas
         ON pizza_types.pizza_type_id = pizzas.pizza_type_id
     JOIN order_details
         ON order_details.pizza_id = pizzas.pizza_id
     GROUP BY pizza_types.name
     ORDER BY Quantity DESC
     LIMIT 5;
     
**6. Quantity of Each Pizza Category Ordered**

Displays the total quantity of pizzas ordered, grouped by pizza category:

     SELECT pizza_types.category, SUM(order_details.quantity) AS total_quantity
     FROM pizza_types
     JOIN pizzas
          ON pizza_types.pizza_type_id = pizzas.pizza_type_id
     JOIN order_details
          ON order_details.pizza_id = pizzas.pizza_id
     GROUP BY pizza_types.category
     ORDER BY total_quantity;
     
**7. Order Distribution by Hour of the Day**

Analyzes how orders are distributed across different hours of the day:

     SELECT hour(order_time) AS Hour, COUNT(order_id) order_count
     FROM orders
     GROUP BY hour(order_time);
     
**8. Category-wise Distribution of Pizzas**

Finds the distribution of pizzas ordered, grouped by category:

     SELECT category, COUNT(name) 
     FROM pizza_types
     GROUP BY category;
     
**9. Average Number of Pizzas Ordered Per Day**

Calculates the average number of pizzas ordered per day:

     SELECT ROUND(AVG(quantity),0) 
     FROM
     (SELECT orders.order_date, SUM(order_details.quantity) AS quantity
     FROM orders
     JOIN order_details
          ON orders.order_id = order_details.order_id
     GROUP BY order_date) AS order_quantity;
     
**10. Top 3 Most Ordered Pizza Types Based on Revenue**

Determines the top 3 pizza types with the highest revenue:

     SELECT pizza_types.name, round(SUM(order_details.quantity*pizzas.price),2) AS revenue
     FROM pizza_types
     JOIN pizzas
          ON pizza_types.pizza_type_id = pizzas.pizza_type_id
     JOIN order_details 
          ON order_details.pizza_id = pizzas.pizza_id
     GROUP BY pizza_types.name
     ORDER BY revenue DESC 
     LIMIT 3;

**11. Percentage Contribution of Each Pizza Type to Total Revenue**

Calculates how much each pizza category contributes to the total revenue:

     SELECT pizza_types.category,
     ROUND(SUM(order_details.quantity*pizzas.price) / 
     (SELECT 
     ROUND(SUM(order_details.quantity*pizzas.price),2) AS total_sales
     FROM order_details
     JOIN pizzas
          ON order_details.pizza_id = pizzas.pizza_id) * 100, 2) as revenue
     FROM pizza_types
     JOIN pizzas
          ON pizza_types.pizza_type_id = pizzas.pizza_type_id
     JOIN order_details
          ON order_details.pizza_id = pizzas.pizza_id
     GROUP BY pizza_types.category
     ORDER BY revenue DESC;

**12. Cumulative Revenue Over Time**

Analyzes cumulative revenue generated over time:

     SELECT order_date,
     SUM(revenue) OVER(ORDER BY order_date) AS cum_revenue
     FROM 
     (SELECT orders.order_date,
     SUM(order_details.quantity * pizzas.price) AS revenue
     FROM order_details
     JOIN pizzas
          ON order_details.pizza_id = pizzas.pizza_id
     JOIN orders
          ON orders.order_id = order_details.order_id
     GROUP BY orders.order_date) AS sales;
     
**13. Top 3 Most Ordered Pizza Types Based on Revenue per Category**

Identifies the top 3 most ordered pizza types by revenue for each pizza category:

     SELECT name, revenue
     FROM
     (SELECT category, name, revenue,
     RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
     FROM 
     (SELECT pizza_types.category, pizza_types.name,
     SUM((order_details.quantity) * pizzas.price) AS revenue
     FROM pizza_types
     JOIN pizzas
          ON pizza_types.pizza_type_id = pizzas.pizza_type_id
     JOIN order_details
          ON order_details.pizza_id = pizzas.pizza_id
     GROUP BY pizza_types.category, pizza_types.name) AS a) AS b
     WHERE rn <= 3;


**Conclusion**

This project demonstrates how to manage pizza sales data using SQL, with the pizzas, orders, order_details, and pizza_types tables working together to provide insights on pizza orders, revenue, and customer preferences. The SQL queries showcase how to generate business insights such as total sales, pizza popularity, and category analysis.
