# Pizza_order_analysis-P4  
## Project Overview  
**Project title** : Pizza_order_analysis_P4  
**Author** : LABHALLA Halima  
**Level** : Intermediate   
**database** : Pizza_db    
**DBMS USED** : PostgreSQL  
**Number of tables** : 10  
   * `address`,  * `customers`,  * `ingredient`, * `inventory`, * `item`, * `orders`, * `recipe`, * `rota`,  * `shift`,  * `staff`

       ![Pizza](https://images.ctfassets.net/j8tkpy1gjhi5/5OvVmigx6VIUsyoKz1EHUs/b8173b7dcfbd6da341ce11bcebfa86ea/Salami-pizza-hero.jpg?w=768&fm=webp&q=80)  

### 🍕 **Situation**

📌 The client, **Ben**, is opening a new **pizzeria**.

🚫 It will **not** offer dine-in service — it will focus solely on **takeout** and **delivery**, similar to *Domino’s*.

📄 Ben has provided us with a **project brief**.

💻 Our **first task** is to design and build a **tailor-made relational database** that will capture all important **operational data** for the business.

📊 This database will later enable Ben to **monitor business performance** through **dashboards**, which we will develop in the next stages.

🛠️ Our focus is **exclusively on backend development** — Ben will hire a separate team to build the **frontend ordering system**.

---   

- table structure for our tables :

```sql
--orders_table 
DROP TABLE IF EXISTS orders; 
CREATE TABLE orders(
row_id INT PRIMARY KEY, 
order_id VARCHAR(10), 
created_at TIMESTAMP, 
item_id VARCHAR(10), 
quantity INT, 
cust_id INT,
delivery BOOLEAN, 
add_id INT
); 

ALTER TABLE orders
ALTER COLUMN created_at TYPE TIMESTAMP
USING created_at::TIMESTAMP;
ALTER TABLE orders
ALTER COLUMN created_at SET DATA TYPE TIMESTAMP;

--customers_table 
DROP TABLE IF EXISTS customers; 
CREATE TABLE customers(
cust_id INT PRIMARY KEY, 
cust_firstname VARCHAR(50), 
cust_lastname VARCHAR(50) 
);

--address table 
DROP TABLE IF EXISTS address; 
CREATE TABLE address(
add_id INT PRIMARY KEY, 
delivery_address1 VARCHAR(200),  
delivery_address2 VARCHAR(200) NULL, 
delivery_city VARCHAR(50), 
delivery_zipcode VARCHAR(20)
) 

--item_table 
DROP TABLE IF EXISTS item; 
CREATE TABLE item(
item_id VARCHAR(10) PRIMARY KEY, 
sku VARCHAR(20), 
item_name VARCHAR(100),
item_cat VARCHAR(100), 
item_size VARCHAR(10), 
item_price NUMERIC(10,2)
); 

--ingredients_table  
DROP TABLE IF EXISTS ingredients; 
CREATE TABLE ingredients(
ing_id varchar(10) PRIMARY KEY,
ing_name VARCHAR(200), 
ing_weight INT, 
ing_meas VARCHAR(20), 
ing_price NUMERIC(5,2)
); 

--recipe 
DROP TABLE IF EXISTS recipe; 
CREATE TABLE recipe(
row_id INT PRIMARY KEY, 
recipe_id VARCHAR(20), 
ing_id VARCHAR(10), 
quantity INT
); 

--inventory_table 
DROP TABLE IF EXISTS inventory; 
CREATE TABLE inventory(
inv_id INT PRIMARY KEY, 
item_id VARCHAR(10), 
quantity INT
); 

--staff_table 
DROP TABLE IF EXISTS staff; 
CREATE TABLE staff(
staff_id VARCHAR(20) PRIMARY KEY, 
first_name VARCHAR(50), 
last_name  VARCHAR(50), 
position VARCHAR(100), 
hourly_rate NUMERIC(5,2)
); 

--shift_table
DROP TABLE IF EXISTS shift; 
CREATE TABLE shift(
shift_id VARCHAR(20) PRIMARY KEY, 
days_of_week VARCHAR(20), 
start_time TIME, 
end_time TIME
); 

--rota_table
DROP TABLE IF EXISTS rota; 
CREATE TABLE rota(
row_id INT PRIMARY KEY, 
rota_id VARCHAR(20), 
date DATE, 
shift_id VARCHAR(20), 
staff_id VARCHAR(20)
); 
```
   
### 🔍 **Main focus areas** from the project brief:

- 📦 `Customer orders`  
- 🧾 `Stock levels`  
- 👨‍🍳 `Staff`

1. Customer orders 
 
`first_dashboard`   

📋 **Looking at the client brief for the _Orders Dashboard_**, we can see that there are **🔟 key data visualizations** Ben would like to include:

- 🧾 *Total Orders*  
- 💰 *Total Sales*  
- 🛒 *Total Items Sold*  
- 📊 *Average Order Value*  
- 📂 *Sales by Category*  
- 🏆 *Top Selling Items*  
- ⏰ *Orders by Hour*  
- 💵 *Sales by Hour*  
- 🗺️ *Orders by Address (Map View)*  


`✅ SQL Logic Behind the Orders Dashboard`

```sql
SELECT 
o.order_id, 
i.item_price, 
o.quantity, 
i.item_cat,
i.item_name, 
o.created_at, 
a.delivery_address1, 
a.delivery_address2, 
a.delivery_city, 
a.delivery_zipcode, 
o.delivery 
FROM orders  o
LEFT JOIN item i ON o.item_id = i.item_id
LEFT JOIN address a ON o.add_id = a.add_id; 
```

![first_dashboard](https://github.com/Hala531/Pizza_order_analysis-P4/raw/main/Capture%20d%E2%80%99%C3%A9cran%202025-04-29%20222602.png)


2. Stock control requirements

`second dashboard`    

📦 **For the inventory section of the dashboard**, we are asked to include **4 insightful visualizations**:

- 🧮 *Total Quantity by Ingredients*  
- 💲 *Total Cost of Ingredients*  
- 🍕 *Calculated Cost of Pizza*  
- 📉 *Percentage Stock Remaining by Ingredients*  

`✅ SQL Logic Behind the Inventory Dashboard`

```sql
SELECT
S1.item_name, 
S1. ing_id, 
S1.ing_name, 
S1.ing_weight, 
S1.ing_price, 
S1.order_quantity, 
S1.recipe_quantity, 
S1.order_quantity*S1.recipe_quantity AS ordered_weight, 
S1.ing_price/S1.ing_weight AS unit_cost,
(S1.order_quantity*S1.recipe_quantity)*(S1.ing_price/S1.ing_weight) AS ingredient_cost
FROM (SELECT 
o.item_id, 
i.sku, 
i.item_name, 
r.ing_id, 
ing.ing_name,
r.quantity AS recipe_quantity,
ing.ing_weight, 
ing.ing_price,
SUM(o.quantity) AS order_quantity 
FROM orders o 
LEFT JOIN item i ON o.item_id = i.item_id 
LEFT JOIN recipe r ON r.recipe_id = i.sku 
LEFT JOIN ingredients ing ON r.ing_id = ing.ing_id
GROUP BY 
o.item_id, i.sku, i.item_name, r.ing_id,ing.ing_name, r.quantity, ing.ing_weight, ing.ing_price) S1 
--creating a view 
CREATE VIEW  Stock AS
SELECT
    S1.item_name, 
    S1.ing_id, 
    S1.ing_name, 
    S1.ing_weight, 
    S1.ing_price, 
    S1.order_quantity, 
    S1.recipe_quantity, 
    S1.order_quantity * S1.recipe_quantity AS ordered_weight, 
    S1.ing_price / S1.ing_weight AS unit_cost,
    (S1.order_quantity * S1.recipe_quantity) * (S1.ing_price / S1.ing_weight) AS ingredient_cost
FROM (
    SELECT 
        o.item_id, 
        i.sku, 
        i.item_name, 
        r.ing_id, 
        ing.ing_name,
        r.quantity AS recipe_quantity,
        ing.ing_weight, 
        ing.ing_price,
        SUM(o.quantity) AS order_quantity 
    FROM orders o 
    LEFT JOIN item i ON o.item_id = i.item_id 
    LEFT JOIN recipe r ON r.recipe_id = i.sku 
    LEFT JOIN ingredients ing ON r.ing_id = ing.ing_id
    GROUP BY 
        o.item_id, i.sku, i.item_name, r.ing_id, ing.ing_name, r.quantity, ing.ing_weight, ing.ing_price
) S1;

```

```sql
--Total-weight-ordered
SELECT
S2.ing_name, 
s2.ordered_weight, 
ing.ing_weight, 
inv.quantity, 
ing.ing_weight*inv.quantity AS total_inv_weight,
(ing.ing_weight*inv.quantity) - S2.ordered_weight AS remaining_weight
FROM (SELECT
ing_id, 
ing_name, 
sum(ordered_weight) as ordered_weight
FROM 
stock 
GROUP BY ing_name, ing_id) S2 
LEFT JOIN inventory inv ON inv.item_id = S2.ing_id
LEFT JOIN ingredients ing  ON ing.ing_id = S2.ing_id  
```


![Second Dashboard](https://github.com/Hala531/Pizza_order_analysis-P4/raw/main/Capture%20d%E2%80%99%C3%A9cran%202025-04-29%20222758...png)


3. Staff Data Requirements

`Third dashboard`  
👨‍💼 **For this third and final dashboard (Staff Dashboard)**, if we look at the client brief, we can see we have just four key points:

- 💵 *Cost per Staff Member*  
- 💰 *Total Staff Cost*  
- ⏰ *Total Hours Worked*  
- 🕒 *Hours Worked by Staff Member*  

`✅ SQL Logic Behind the Staff Dashboard`  

```sql
SELECT 
    r.date, 
    s.first_name, 
    s.last_name, 
    s.hourly_rate, 
    sh.start_time, 
    sh.end_time, 
    EXTRACT(EPOCH FROM (sh.end_time - sh.start_time)) / 3600 AS hours_in_shift,
    (EXTRACT(EPOCH FROM (sh.end_time - sh.start_time)) / 3600) * s.hourly_rate AS staff_cost
FROM rota r
LEFT JOIN staff s ON r.staff_id = s.staff_id 
LEFT JOIN shift sh ON r.shift_id = sh.shift_id;
```

  





 

   



   
