## Analysis of Scale Model Cars:
### Introduction:

I am conducting a sales analysis for SMC, a company specializing in scale model products. As their data analyst, I work with the sales and marketing teams to identify trends and patterns that can help improve sales.

### About SMC Company:

SMC is a wholesaler of scale model products, with sales offices in the USA, France, Japan, Australia, and the UK. They have partnerships with over 13 product vendors, allowing them to offer competitive prices to their customers (retailers) worldwide.

The company groups its sales team/customers into four sales territories:

+ Europe, Middle East, and Africa (EMEA)
+ North America (NA)
+ Asia Pacific Region (APAC)
+ Japan

Their product line offerings are as follows:

|ProductLine| Total_Products_offered
|---|---|
|Classic Cars|	38
|Motorcycles|	13
|Planes|	12
|Ships|	9
|Trains|	3
|Trucks and Buses|	11
|Vintage Cars|	24

### Goals: 
+ Determine which product lines yield high profits and analyze their inventory levels.
+ Understand customer buying patterns to enhance loyalty programs and campaigns.
+ Assess the budget needed for acquiring new customers.

### Findings:
+ Their most profitable product lines are Classic Cars and Vintage Cars, and they have sufficient inventory to meet customer demand.
+ After September 2004, no new customers were acquired, and revenue generation was solely from existing customers.
+ Their profits are highly concentrated among two VIP customers: Euro+ Shopping Channel from Spain and Mini Gifts Distributors Ltd from the USA.

### Recommendations: 
+ Spread the inventory cost across high-performance products and implement effective inventory management planning.
+ Allocate the budget for acquiring new customers
+ Run campaigns for less engaged customers, and implement loyalty programs for VIP customers.

### Database Schema:
<img width="501" alt="image" src="https://github.com/user-attachments/assets/6714f129-157f-4d95-a273-03f226d6c684">

### Individual Table Descriptions:
+ `Customers`      : contains the customer data and links to the table `Orders` and `Payments`
+ `Employees`      : contains all the employee information and links to the `Customers` table via `salesRepEmployeeNumber`field and to the `Offices` table
+ `Offices`        : contains the sales office information,operating country & territory and links to the `Customers` and `Employees` table
+ `Orders`         : contains the customers' sales orders which links to the `OrderDetails` and `Customers` table
+ `OrderDetails`   : contains the sales order line for each sales order containing the order number, the respective quantity ordered and their unit price. it links to the  `Orders` and `Products`  table
+ `Payments`       : contains customers' payment records for each customer number and links to the `Customers` table
+ `Products`       : contains a list of scale model cars, their name, product line, quantity in stock and buy price. it links to the `OrderDetails` and  `ProductLines`  table
+ `ProductLines`   : contains a list of product line categories linked to the `Products` table

#### Basic CTE query to obtain the attributes information from all the tables using the Union operator
```
Attributes (Table_Name , number_of_attributes,number_of_rows) AS ( 
SELECT CAST( 'customers' AS TEXT) AS Table_Name,
      COUNT(*)  AS number_of_attributes,
      (SELECT COUNT(*) FROM customers) AS number_of_rows
    FROM pragma_table_info('customers')
```

|Table_Name|Number_of_attributes|Number_of_rows
|---|---|---|
|customers|13|122
|employees|	8	|23
|offices|	9	|7
|orderdetails|	5	|2996
|orders|	7	|326
|payments	|4	|273
|productlines|	4|	7
|products	|9	|110

$~$

## Data Analysis:

### Profit Distribution:

SMC offers seven product lines. As a wholesaler, they purchase products from vendors at a certain price and sell them to retailers at a price that covers their operating expenses.

We use the below tables and their respective fields to obtain the profits:

$~$
<img width="400" alt="image" src="https://github.com/user-attachments/assets/b3b163c3-1f11-4c29-991e-38f1eb53fbc7">



#### Metric used for profit calculation: 

profit = ( `priceEach ` -  `buyPrice `) *  `quantityOrdered `

#### The CTE query to obtain profit information based on product lines is as follows:
```
WITH
profit_info(productCode,productName,productLine,customerNumber,buyPrice,priceEach,quantityOrdered,profits) AS (
	SELECT p.productCode, p.productName,p.productLine,
		o.customerNumber,
		p.buyPrice, 
		od.priceEach,
		od.quantityOrdered,
		(od.priceEach - p.buyPrice) *od.quantityOrdered AS profits,
	FROM products p
	JOIN orderdetails  od ON p.productCode = od.productCode
	JOIN  orders o ON  o.orderNumber = od.orderNumber
	JOIN customers c ON c.customerNumber = o.customerNumber
)
```

### Profit distribution across the Product Lines:

Their most profitable product lines are Classic Cars and Vintage Cars. The 1992 Ferrari 360 Spider, under the Classic Cars product line, and the 1928 Mercedes-Benz SSK, under the Vintage Cars product line, both generate the highest profits.

|productLine	|Total_profits
|---|---|
|Classic Cars	|81783338.0
|Vintage Cars	|45487994.0
|Trucks and Buses	|27123304.0
|Motorcycles	|20565280.0
|Planes	|16170003.0
|Ships|	15771029.0
|Trains	|7871143.0

$~$

|productLine|productName	|Total_profits|
|---|---|---|
|Classic Cars	|1992 Ferrari 360 Spider red	|4538276.0
|Vintage Cars	|1928 Mercedes-Benz SSK	|2766158.0

$~$

<img width="501" alt="image" src="https://github.com/user-attachments/assets/a56c7b39-345c-4f27-8f5f-99bbea1a4a92">

### Inventory Analysis:
The tables below are used to measure out-of-stock products and their performance. 
Since high-performing products generate profits, it is important to stock up on these products in a timely manner to ensure timely delivery.
To identify products that are running low on stock, I can use the `status` field in the orders table.

![image](https://github.com/user-attachments/assets/de17be7d-3f5f-4c90-a7c3-e5e59a8231a4)

SMC categorizes orders under one of six statuses.

|order_status|
|---|
|Shipped|
|Resolved|
|Cancelled|
|On Hold|
|Disputed|
|In Process|

To obtain the current 'Quantity Ordered' information, 'Shipped,' 'Resolved,' and 'Cancelled' orders should be omitted, as these orders have already been filled or cancelled.

The respective tables need to be filtered by order numbers with an order status of 'In Process,' 'On Hold,' or 'Disputed.'

'On Hold' and 'Disputed' orders occur due to customer credit limitations during the payment process or incorrect orders sent to the customer, which must be filled or refilled by the company.

#### Metric used for Inventory and product performance calculation :

Low stock           = quantityOrdered / `quantityInStock`

Product Performance = quantityOrdered × `priceEach`

#### Basic CTE query to obtain the Quantity Ordered:
```
WITH
stock_vs_order  AS  (
SELECT  P.productCode,P.productName, p.productLine,P.quantityInStock,OO.Ordered_Quantity
 FROM products P
 JOIN ( SELECT OD. productCode, SUM(OD.quantityOrdered) AS ordered_quantity
 FROM orderdetails OD
 JOIN orders  O ON OD.orderNumber = O.orderNumber
 WHERE O.status NOT IN ('Shipped','Resolved','Cancelled')
 GROUP BY  OD. productCode) AS OO ON P.productCode = OO.productCode
	)
```

#### Top 10 High performing product running low on stock:

After identifying the top 10 high-performing products using the Product Performance metric and mapping them with the low stock metric, we obtained the table below. 
Their inventories are sufficiently stocked to meet demand, with only 0.01 of their existing stock used.

productName	|productLine	|Product_perform	|Ordered_Quantity|quantityInStock|Low_stock|
|----|---|---|---|---|---|
1992 Ferrari 360 Spider red|	Classic Cars|	276839.98	|48	|8347|0.01|
1952 Alpine Renault 1300|	Classic Cars	|190017.96	|50	|7305|0.01|
2003 Harley-Davidson Eagle Drag Bike	|Motorcycles	|170686.0	|56	|5582|0.01|
1998 Chrysler Plymouth Prowler|	Classic Cars|	142530.63	|28	|4724|0.01|
1917 Grand Touring Sedan|	Vintage Cars	|140535.6	|113	|2724|0.04|
2002 Suzuki XREO	|Motorcycles|	135767.03|		21	|9997|0.0|


### Customer Analysis:
To tailor marketing and communication strategies according to customer behavior, it is essential to understand customer behaviors. 
The company could run loyalty programs for customers who generate more profits and campaigns for those who are less engaged. 
To identify the top and bottom performers, I used the same CTE query mentioned in the profit analysis.

#### Metric used for profit calculation: 

profit = ( `priceEach ` -  `buyPrice `) *  `quantityOrdered `

#### The CTE query to obtain the profit information based on the customer:
```
WITH
profit_info(productCode,productName,productLine,customerNumber,buyPrice,priceEach,quantityOrdered,profits) AS (
	SELECT p.productCode, p.productName,p.productLine,
		o.customerNumber,
		p.buyPrice, 
		od.priceEach,
		od.quantityOrdered,
		(od.priceEach - p.buyPrice) *od.quantityOrdered AS profits,
	FROM products p
	JOIN orderdetails  od ON p.productCode = od.productCode
	JOIN  orders o ON  o.orderNumber = od.orderNumber
	JOIN customers c ON c.customerNumber = o.customerNumber
)
```

$~$

The top 3 VIP customers for whom SMC should implement loyalty programs are as follows:

Customer_Name	|City|	Country	|Profit|
|---|---|---|---|
Euro+ Shopping Channel|	Madrid	|Spain|	307322.49|
Mini Gifts Distributors Ltd.	|San Rafael	|USA	|236769.39|
Muscle Machine Inc	|NYC	|USA	|72370.09|

The bottom 10 least engaged customers for whom the company should run campaigns to encourage more spending are as follows:

Customer_Name	|City	|Country|	Profit|
|---|---|---|---|
Boards & Toys Co.|	Glendale	|USA	|2610.87|
Auto-Moto Classics Inc.	|Brickhaven	|USA	|6586.02|
Frau da Collezione|	Milan	|Italy	|9532.93|
Atelier graphique	|Nantes	|France	|10063.8|
Double Decker Gift Stores, Ltd	|London|	UK	|10868.04|
Royale Belge	|Charleroi|	Belgium	|11693.99|
Bavarian Collectables Imports, Co.	|Munich	|Germany	|13033.35|
Microscale Inc.	|NYC|	USA|	13066.02|
Cambridge Collectables Co.	|Cambridge|	USA	|13734.7|
Men 'R' US Retailers, Ltd.	|Los Angeles|	USA	|14928.37|

### Market Spending Analysis and findings:
To help the marketing team decide how much to spend on acquiring new customers, I should calculate the new customer props and their potential value each month.
After September 2004, there were no new customers and the value generation came solely from existing customers. 
Therefore, it is worthwhile to invest in marketing and communication to acquire new customers and improve the SMC's profitability.

#### Metric used for the customer props calculation:
new_customers_props = (No.of new customers/Total_Customers) *100
$~$

new_customers_total_props = new customer amount / Total amount * 100

#### The table below shows the customer props and their potential value for each month of the year:
After September 2004, no new customers were acquired, and their value generation is nil.

<img width="501" alt="image" src="https://github.com/user-attachments/assets/de4b737f-3961-4d93-9481-7cc7304b4cb9">

$~$

It was observed that after September 2004, value generation came solely from existing customers.
Please find the table below for more information:
|year_month	|No_of_new_customers|	Total_customers	|New_Customer_Total| customer_total|
|---|---|---|---|---|
200408|	2	|11|	99077.05|	378094.3|
200409|	5	|15	|234910.15	|476445.53
200410	|0	|7|	0|	185103.43
200411	|0	|23|	0|	857187.3
200412	|0	|23	|0	|819285.62
200501|	0|	5|	0|	137468.06
200502	|0|	8|	0|	252321.22
200503	|0	|8	|0|	385268.09
200504|	0	|5|	0|	183897.72
200505	|0	|9|	0	|272248.93
200506	|0	|2	|0|	59089.26

### Spending Analysis:
The amount to be allocated for customer acquisition depends on the budget available for the marketing and sales team. 
To determine how much to spend, I need to identify the average revenue generated per customer.

#### Metric used: 
Customer Lifetime Value (LTV), is the average amount of revenue one single customer generates for the duration of their business with you.

Used the CTE query under the profit analysis section, the average amount of revenue generated per customer is as follows:

|Average_revenue_per_customer|
|---|
|38076.65|

Since Customer Lifetime Value (CLV) takes into account the duration of the customer’s relationship with the company, it is beneficial to include data on the customer’s relationship with SMC in years.

After segmenting the data based on their duration with SMC, the average revenue can be categorized into three duration buckets.

+ 0.0 - Customer with the duration between 0-1 year

+ 1.0 - Customer with the duration between 1-2 years

+ 2.0 - Customer with duration of 2 years and above.

|Duration_relation_years	| Average_profit_per_customer|
|---|---|
0.0	|23263.72|
1.0	|34500.85|
2.0	|55072.8|

#### Findings: 
Depending on the budget of the sales and marketing team, they could spend around or above the average revenue generated by a customer. 
Long-term customers typically generate more revenue compared to new customers. 
The company should consider the long-term value generated by customers when planning their spending.
