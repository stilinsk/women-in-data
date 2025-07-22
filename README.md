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

