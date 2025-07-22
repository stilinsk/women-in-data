# women-in-data

DBT otherwise known as data build tool is a tool we use in the modern data stack otherwise known formally as ELT for transformation of data .In this repo we will just be looking at all the transformation that we can do to any data and we will learn best industry practices for using dbt and advantages of using this specific tool in our data pipelines for data engineers and analytics engineers.

<img width="1206" height="1132" alt="Untitled Diagram drawio" src="https://github.com/user-attachments/assets/2d0279f7-7e1c-4d7c-af1e-eb64ad801436" />


 This will be the semantic model that will be exposed to the Gold layer for the data analysts .


 pre -requisites for the project
Snowflake account

vscode

python =<3.11

powerbi

prior knowledge of data modelling techniques and knowledge preferrrably upto 3rd NF( not a must tho)

We will start by creating a snowflake account ,snowflake provides a 30 daya free trial that you can just use for learning for this course and also they provide 400 dollar credit for the entire duration of th free trial ,when creating a free trial let the preference be th aws and let the region be any(use use east ohio) ,but if you use the AWS cloud services it would be standard practice to set the same region for the aws and for the snowflake.This will definitely ease any future pipelines you may want to create in future and you might want to use both aws cloud and snowflake as your data warehouse.

 We will start by creating our logins and then we creat tables for us to load data in them ( the csv tables will be provided )

 ```
SALES.RAW.ORDERS-- Use admin role
USE ROLE ACCOUNTADMIN;

-- Create the `data` role
CREATE ROLE IF NOT EXISTS data;
GRANT ROLE data TO ROLE ACCOUNTADMIN;

-- Create or use the default warehouse
CREATE WAREHOUSE IF NOT EXISTS COMPUTE;
GRANT OPERATE ON WAREHOUSE COMPUTE TO ROLE data;

-- Create the `deta` user and assign to role
CREATE USER IF NOT EXISTS deta
  PASSWORD='1234'
  LOGIN_NAME='deta'
  MUST_CHANGE_PASSWORD=FALSE
  DEFAULT_WAREHOUSE='COMPUTE'
  DEFAULT_ROLE='data'
  DEFAULT_NAMESPACE='SALES.RAW'
  COMMENT='DBT user used for data transformation';

GRANT ROLE data TO USER deta;

-- Create database and schema
CREATE DATABASE IF NOT EXISTS SALES;
CREATE SCHEMA IF NOT EXISTS SALES.RAW;

-- Set permissions for `data` role
GRANT ALL ON WAREHOUSE COMPUTE TO ROLE data;
GRANT ALL ON DATABASE SALES TO ROLE data;
GRANT ALL ON ALL SCHEMAS IN DATABASE SALES TO ROLE data;
GRANT ALL ON FUTURE SCHEMAS IN DATABASE SALES TO ROLE data;
GRANT ALL ON ALL TABLES IN SCHEMA SALES.RAW TO ROLE data;
GRANT ALL ON FUTURE TABLES IN SCHEMA SALES.RAW TO ROLE data;


-- Use the appropriate database and schema
USE DATABASE SALES;
USE SCHEMA RAW;

USE DATABASE SALES;
USE SCHEMA RAW;

-- Drop existing tables
DROP TABLE IF EXISTS SALES.RAW.Orders;
DROP TABLE IF EXISTS SALES.RAW.Products;
DROP TABLE IF EXISTS SALES.RAW.Customers;
DROP TABLE IF EXISTS SALES.RAW.Reviews;
DROP TABLE IF EXISTS SALES.RAW.Social_Media;
DROP TABLE IF EXISTS SALES.RAW.Web_Logs;

-- Orders table
CREATE OR REPLACE TABLE SALES.RAW.Orders (
    OrderID STRING,
    OrderDate STRING,
    CustomerID STRING,
    ProductID STRING,
    Quantity INTEGER,
    TotalAmount FLOAT,
    PaymentMethod STRING
);

-- Products table
CREATE OR REPLACE TABLE SALES.RAW.Products (
    ProductID STRING,
    ProductName STRING,
    Category STRING,
    Stock INTEGER,
    UnitPrice FLOAT
);

-- Customers table
CREATE OR REPLACE TABLE SALES.RAW.Customers (
    CustomerID STRING,
    CustomerName STRING,
    Email STRING,
    Location STRING,
    SignupDate STRING
);

-- Reviews table
CREATE OR REPLACE TABLE SALES.RAW.Reviews (
    timestamp STRING,
    customer_id STRING,
    product_id STRING,
    rating INTEGER,
    review_text STRING
);

-- Social_Media table
CREATE OR REPLACE TABLE SALES.RAW.Social_Media (
    timestamp STRING,
    platform STRING,
    content STRING,
    sentiment STRING
);

-- Web_Logs table
CREATE OR REPLACE TABLE SALES.RAW.Web_Logs (
    timestamp STRING,
    user_id STRING,
    page STRING,
    action STRING
);
```
We will be using vs code for this project and we will need to install python version greter or eqaul to 3.11 &  3.12 some of the packages that we will be needing for this project are still ot compatible.

dbt can be used in all the operating systems and the code is almost the same. we will be creating a dbt project folder in the desktop then we will need to create and activate the environment.

```
cd Desktop
mkdir course
cd course

python3 -m venv ( name of environment)  this case we assume we name our environment venv
venv\Scripts\activate
```
```
cd ~/Desktop
mkdir course
cd course

# Create virtual environment named "venv"
python3 -m venv venv

# Activate the virtual environment on Linux/macOS
source venv/bin/activate
```

We will need to install the dbt snowflake package

```
pip install --upgrade dbt-core dbt-snowflake
```
Then you create the .dbt configuration folder 

windows
```
mkdir %USERPROFILE%\.dbt

```

linux
```
mkdir -p ~/.dbt
```

we will then create a dbt prject using the following code

```dbt init wid``` after this we will navigate to the dbtlearn folder using the command cd and then use dbt debug to create the connection .Remember the roles,the warehouse the schema ,password that we used when creating the datawarehouse in snowflake and the inforamtion should be similar after providing the correct details the connection passes but note that if you don't have Git installed on your machine or if you are using Windows, this connection may fail initially. However, in the end, the connection should still be established and it should ideally work. Then, you will simply use the command code . to open the code editor installed on your machine, you should now be in vs code and the connection must be set to the data warehouse .

Hope we have all reaches at this stage and if you have then clap for yourself. and if you have not repeat the steps that we have carried out so far in this project and you should be good to go.


There are some of the folders that probably by now if you have open the text editor are visible to you. we will look at them and understand what they each entail.

1. **SNAPSHOT**: This is a table-like construct created in DBT to track changes in data. We use Slowly Changing Dimension 2 when we want to preserve old data. It tracks changes, including when data was altered or deleted. This feature can be useful for detecting fraud or data manipulation. Additionally, it ensures the schema of the data remains consistent for new entries, simplifying data cleaning. We run snapshots using the command `dbt snapshot`.

2. **SOURCES**: This is an abstraction layer on top of input tables in the data warehouse. Essentially, it defines where the table data will be accessed from in the data warehouse, guiding the model on where to find the data it needs.

3. **SEEDS**: These are CSV files uploaded from DBT to Snowflake via VS Code. Once uploaded, they become seeds. We run seeds using the command `dbt seed`.

4. **TESTS**: There are generic and singular tests. Generic tests include unique values, not null, accepted values, and relationships. Singular tests can be created and linked to macros. An example is a test that iterates over all columns in a table to ensure there are no null values.

5. **SCHEMA.YML**: This file is crucial for the project. It contains additional information such as column descriptions (visible in documentation) and test placements.

6. **MACROS**: These are Jinja templates used in SQL and created in macro files. Macros can be used as singular tests by linking to another file in the tests folder where the macro test is defined.

7. **PACKAGES.YML**: This is where we install packages used in the project, such as dbt utils and Great Expectations.

8. **ASSETS**: This file is not visible in the project's path but needs to be added, along with the connection to the `dbt_project.yml` path. It may contain an overview of the project after documentation in the localhost, such as a model's photo, accurately representing the project.

9. **DBT_PROJECTS.YML**: This file defines the entire project, including configurations, setting paths, and defining materializations (e.g., table, view, or ephemeral). Materializations depend on visualization needs; for dimensional tables, set materializations as tables, while for original tables not used in visualizations, set materializations as ephemeral. Incremental materializations are suitable for tables likely to have additional data.

10. **HOOKS AND EXPOSURES**: These are connections that link the BI tool to a webpage or visualization tool. This integration allows visualization to be part of the project and deployed through tools and exposures.


11. **MATERIALIZATIONS**: Materializing tables involves determining how we will view the models in the data warehouse, and materializations include three main types:
   - **Table**: This type is used for creating the final dimensional tables and fact tables that we want to view as tables in our data warehouse.
   - **View**: The other type of materialization is the view, where these are tables that we want to retain in our data warehouse but are not as critical as we will not be actively using them, given that we already have the dimension and fact tables. For this project, as we will see in the `src` folder models, we will be materializing them as views.
   - **Ephemeral**: This type involves reducing the code to just a Common Table Expression (CTE), and we can delete this data from our data warehouse. This may be used in cases where we need to save on space and cost, such as with the original data from tables. However, one should be extremely cautious before implementing this, as it becomes permanent.

#### sources 
These is where we define and direct dbt to access our files and folders from in the datawarehouse
```
version: 2

sources:
  - name: sales 
    schema: Raw 
    tables:
      - name: orders
        identifier: orders
      - name: customers
        identifier: customers
      - name: products
        identifier: products  
      - name: web_logs
        identifier: web_logs
      - name: social_media
        identifier: social_media
      - name: reviews
        identifier: reviews
    ```
```

We will move to build the models in the bronze, silver and gold

##### Bronze
Custoners.sql
```
With raw_customers as (
    select
        *
    from {{ source('sales', 'customers') }}
)
select
    customerId,
    customerName,
    email,
    location,
    signupdate

from raw_customers
```
 Orders.sql
```
With raw_orders as (
    select
        *
    from {{ source('sales', 'orders') }}
)
select
    orderId,
    orderDate,
    customerId,
    productId,
    quantity,
    TotalAmount,
    Paymentmethod
from raw_orders
```
Products.sql

```
with raw_products as (
    select
        *
    from {{ source('sales', 'products') }}
)
select 
    productId,
    productName,
    Category,
    Stock,
    UnitPrice
from raw_products
```
Reviews.sql
```
with raw_reviews as (
    select
        *
    from {{ source('sales', 'reviews') }}
)
select
    timestamp,
    customer_ID,
    product_ID,
    rating,
    review_text,
from raw_reviews
```
Social_media.sql
```
with raw_social_media as (
    select
        *
    from {{ source('sales', 'social_media') }}
)

select 
timestamp,
    platform,
    content,
    sentiment
from raw_social_media
```
Weblogs.sql
```
with raw_weblogs as (
    select
        *
    from {{ source('sales', 'web_logs') }}
)

select 
    timestamp,
    user_id,
    page,
    action
from raw_weblogs
```
#### Silver

dim_customers
```
{{ config(materialized='table') }}

with dim_customers as (
    select
        cast(replace(customerId, 'CUST', '') as integer) as customer_id,
        split_part(customerName, ' ', 1) as first_name,
        split_part(customerName, ' ', 2) as last_name,
        email as customer_email,
        location,
        cast(signupdate as date) as customer_signupdate
    from {{ ref('customers') }}
    where lower(customerId) not in ('customerid')
      and customerId is not null
)

select * from dim_customers
```
dim_products
```
{{ config(materialized='table') }}

with dim_products as (
    select
        cast(replace(productId, 'PROD', '') as integer) as product_id,
        productName as product_name,
        Category as product_category,
        cast(Stock as integer) as product_stock,
        cast(UnitPrice as float) as product_price
    from {{ ref('products') }}
    where productId is not null
)

select * from dim_products
```
dim_orders
```
{{ config(materialized='table') }}
with dim_orders as (
    select
        cast(replace(orderId, 'ORD', '') as integer) as order_id,
        cast(replace(customerId, 'CUST', '') as integer) as customer_id,
        cast(replace(productId, 'PROD', '') as integer) as product_id,
        cast(orderDate as date) as order_date,
        cast(date_trunc('hour', cast(orderDate as timestamp)) as time) as order_hour,
        quantity,
        cast(totalAmount as float) as total_amount,
        paymentMethod as payment_method
    from {{ ref('orders') }}
    where orderId is not null
)
select * from dim_orders
```
dim_dates
```
with dim_tables as (
    select
        {{ dbt_utils.generate_surrogate_key(['orderID', 'orderDate']) }} as date_id,
        cast(orderDate as date) as order_date,
        
        extract(year from cast(orderDate as date)) as order_year,
        extract(month from cast(orderDate as date)) as order_month,
        extract(day from cast(orderDate as date)) as order_day,
        dayname(cast(orderDate as date)) as day_of_week,
        lpad(cast(extract(hour from cast(orderDate as timestamp)) as string), 2, '0') || ':00 hrs' as order_hour
    from {{ ref('orders') }}
)

select * from dim_tables

```
dim_reviews
```
{{ config(materialized='table') }}

with dim_reviews as (
    select
        {{ dbt_utils.generate_surrogate_key(['timestamp', 'customer_ID', 'product_ID']) }} as review_id,
        cast(replace(customer_ID, 'CUST', '') as integer) as customer_id,
        cast(replace(product_ID, 'PROD', '') as integer) as product_id,
        cast(timestamp as date) as review_date,
        cast(date_trunc('hour', cast(timestamp as timestamp)) as time) as review_hour,
        cast(rating as integer) as rating,
        cast(review_text as text) as review_text,
        cast(current_timestamp() as timestamp) as updated_at
    from {{ ref('reviews') }}
    where customer_ID is not null
)

select * from dim_reviews
```
dim_socialmedia
```
{{ config(materialized='table') }}

with dim_socialmedia as (
    select  
        {{ dbt_utils.generate_surrogate_key(['timestamp', 'platform', 'content']) }} as post_id,
        
         cast(timestamp as date) as post_date, 
        cast(date_trunc('hour', cast(timestamp as timestamp)) as time) as post_hour,
        cast(platform as text) as platform,
        cast(content as text) as content,
        cast(sentiment as text) as sentiment,
        
    from {{ ref('social_media') }}
    where timestamp is not null
      and lower(platform) not in ('platform')  -- remove accidental header row
)

select * from dim_socialmedia
```
dim_weblogs
```
{{ config(materialized='table') }}

with dim_weblogs as (
    select
        {{ dbt_utils.generate_surrogate_key(['timestamp', 'user_id', 'page', 'action']) }} as activity_id,
        cast(replace(user_id, 'CUST', '') as integer) as customer_id,
        cast(timestamp as date) as activity_date,
        cast(date_trunc('hour', cast(timestamp as timestamp)) as time) as activity_hour,
        cast(page as text) as page,
        cast(action as text) as action_taken
    from {{ ref('web_logs') }}
    where timestamp is not null
      and lower(user_id) not in ('user_id')
)

select * from dim_weblogs
```
dim_sales
```
{{ config(materialized='table') }}

with orders as (
    select
        order_id,
        customer_id,
        product_id,
        order_date,
        order_hour,
        quantity,
        total_amount,
        payment_method
    from {{ ref('dim_orders') }}
),

customers as (
    select
        customer_id,
        customer_signupdate
    from {{ ref('dim_customers') }}
),

products as (
    select
        product_id,
        product_price
    from {{ ref('dim_products') }}
),

reviews as (
    select
        review_id,
        customer_id,
        product_id,
        review_date
    from {{ ref('dim_reviews') }}
),

weblogs_agg as (
    select
        customer_id,
        activity_id,
        activity_date
    from {{ ref('dim_weblogs') }}
    qualify row_number() over (partition by customer_id order by activity_date) = 1 -- First record per customer
),

dim_dates as (
    select
        date_id,
        order_date
    from {{ ref('dim_date') }}
)

select
    -- Unique sales ID
    {{ dbt_utils.generate_surrogate_key(['o.order_id', 'o.product_id', 'o.customer_id']) }} as sales_id,

    -- Foreign keys
    o.order_id,
    o.customer_id,
    o.product_id,
    r.review_id,
    d.date_id,
    w.activity_id,

    -- Measures
    o.quantity,
    o.total_amount,
    p.product_price,

    -- Dates
    o.order_date,
    o.order_hour,
    r.review_date,
    w.activity_date as first_web_activity_date,

    -- Payment
    o.payment_method,

    -- Behavioral logic
    case
        when c.customer_signupdate <= o.order_date then true
        else false
    end as signup_before_order,

    -- Time gaps
    datediff('day', o.order_date, r.review_date) as order_to_review_gap_days,
    datediff('day', w.activity_date, o.order_date) as activity_to_order_gap_days -- Added simple calculation

from orders o
left join customers c on o.customer_id = c.customer_id
left join products p on o.product_id = p.product_id
left join reviews r
    on o.customer_id = r.customer_id
    and o.product_id = r.product_id
left join weblogs_agg w
    on o.customer_id = w.customer_id
left join dim_dates d on o.order_date = d.order_date
where o.order_id is not null
```

#### Gold
 customers_table
```
{{ config(materialized='table') }}

with customers_table as (
    select  
         customer_id,
        first_name,
        last_name,
        customer_email,
        location,
        customer_signupdate
        
    from {{ ref('dim_customers') }}
     
)

select *
from customers_table
where customer_id is not null
```
products_table
```
{{ config(materialized='table') }}

with product_table as (
    select * from {{ ref('dim_products') }}
)

select  
     product_id,
    product_name,
     product_category,
    product_stock
 
    
from product_table
where product_Id is not null
```
Orders_table
```
{{ config(materialized='table') }}

with orders_table as (
    select  
        order_id,
        payment_method
        from {{ ref('dim_orders') }}
)

select * from orders_table
where order_id is not null
```
Reviews_table
```
{{ config(materialized='table') }}

with reviews_table as (
    select
         review_id,
        review_date,
        rating,
        review_text
    
    from {{ ref('dim_reviews') }}
    where review_id is not null
)

select * from reviews_table
```

