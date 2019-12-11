# Section 1

## Introduction to SQL Concepts
SQL stands for **Structured** **Query** **Language** and is the query language used to interact with relational databases. 

In case you're wondering about what a relational database is, it is any database that stores data in the form of **relations** or tables wherein the table is made up of rows and columns. A row, also callde a **tuple**, contains an individual record of the data whereas a column, also called an **attribute**, is an identifying feature of that record.

With that primer, let's get straight to it! :smile: 

For this course we will be using a fictitious **online pet store called API**.

:bell: A few pointers:
- There are scripts to drop and re-create the database tables and other objects. This gives us the freedom to make any changes we want in the exercises and then be able to return to the standard setup with ease. 
    - The script **create_schema.sql** will drop and re-create all of the relevant objects and data.
- All tables start with the term `API`.
- Column names have been given meaningful names and their purpose should be fairly clear.
- Primary keys are typically the table name (minus API) with an `_id` added at the end.
- Foreign keys are named after the referenced primary key when possible. 
    - When that would result in confusion (E.g. multiple references to the same table) we give it a reasonable name and then add an `_id` at the end.
- All tables have three standard audit columns on them:
    - `CREATED_TIMESTAMP`
    - `MODIFIED_TIMESTAMP`
    - `MODIFIED_BY`

Confused?! :anguished: Don't you worry, all of these terms will be explained next.

### Relations/Tables

In a relational database, the object in which we store data is called a table. A table is identified by its name and is comprised of one or more columns that specify the structure of the data in that table.

> Listed below are a few of the tables that our database consists of. There are other tables and objects in the schema but they will be introduced in specific exercises. The tables below form the core schema.

Table | Description
--- | ---
API_CUSTOMER | This table represents a customer. Someone that created an account with API and that can order things.
API_ADDRESS | This table represents a customer. Someone that created an account with API and that can order things.
API_PACKAGE_TYPE | This table contains the different kinds of packaging :package: associated with the products that API offers such as `Each`, `Pallet`, `Box` and `Barrel`.  Note that a given product can be offered in many different package types and that some package types may be comprised of smaller package types. For example, you can purchase a single package (that would be an `Each` package type) or a `Box` package type (a `Box` package type contains 24 `Each` package type).
API_ITEM | This table represents items that API has (or has had) for sale.
API_CUSTOMER_ORDER | This table represents a customer order.
API_CUSTOMER_ORDER_ITEM | This table details the items that made up a customer order.
API_EMPLOYEE | This table contains the list of API employees and information on each employee.

### Rows
A row represents an instance of the data element that the table stores. A table can have any number of rows in it.

> For example, in the **API_CUSTOMER** table each row represents a different customer. 

### Columns
A column represents a piece of information in a table. It has both a name to identify the column and a datatype to specify the nature of data in that column.

> For example, in the **API_CUSTOMER** table the `EMAIL` column is specified as:
> - type of _VARCHAR2_:  (a string) with a maximum length of 255 characters. 
> - _NOT NULL_ so whenever we have data in that table it must have a value in the `EMAIL` column.

:bulb: A relational database usually consists of multiple tables. Together these tables comprise the **database model.** Note that there are many ways to represent the same set of data (using tables and columns). A database model is just one particular choice in terms of how to represent the data.

**Exercise 1** :computer:

Let's look at a `CREATE` statement. As the name suggests, it is used to create tables in the database. When creating a table you must specify the table name, the columns that make up the table and their data types. Apart from these 3, there are many more options available, some of which we will go over later in the course. 

```SQL
CREATE TABLE API_PACKAGE_TYPE (
         PACKAGE_TYPE_ID         NUMBER(22) PRIMARY KEY,
         CODE                    VARCHAR2(10) NOT NULL,
         NAME                    VARCHAR2(50) NOT NULL,
         DESCRIPTION             VARCHAR2(255) NOT NULL,
         CREATED_TIMESTAMP       TIMESTAMP(6) NOT NULL,
         MODIFIED_TIMESTAMP      TIMESTAMP(6) NOT NULL,
         MODIFIED_BY             VARCHAR2(50));
```

:book: [Read more](https://www.techonthenet.com/oracle/tables/create_table.php) about **`CREATE TABLE`**

Spend 5 minutes :alarm_clock: to look at the `CREATE TABLE` statements in the  **create_schema.sql** file found in the :file_folder: utils folder.

### DDL vs DML

The exercise above provides a good segue into DDL and DML. 

Data Definition Language (DDL) | Data Manipulation Language (DML)
--- | ---
Keywords to access and manipulate objects or structures  such as views, tables and procedures. | Keywords to access and manipulate data. These statements are transactional in nature and can be committed or rolled back.
Examples: CREATE, ALTER, DROP | Examples: SELECT, DELETE, UPDATE

:book: [Read more](http://www.orafaq.com/wiki/SQL_FAQ#What_are_the_difference_between_DDL.2C_DML_and_DCL_commands.3F) for a discussion on the differences.

**Exercise 2** :computer:

The distinction between DDL and DML has been purposely created and the reason for that is access privileges. Imagine being the database administrator for this online pet store. 
- Now consider the case of Alice - an analyst in your company. Would you be okay granting Alice the permissions to alter the database schema or accidentally delete a table or two? Think about her job responsibilities. She needs permissions to read the data in the tables so that she can create reports and perform her analysis. And it might also be beneficial to grant her permission to insert, update and delete the data. This is because analysts uncover a lot of errors with data during the data cleaning process such as duplicates, inconsistencies etc. Mind you, the important term here being **access to data** and not the database structure itself.
- Next, consider Bob - an end user who is sitting in his mom's garage one day, and decides he wants to buy a cat. :cat: Obviously, he's going to need food to feed the cat so he settles on using API's API :joy: (what I mean is, the online pet store's Application Programming Interface) to place an order for Cat Food. What permissions should end users like Bob be granted?

 ### Primary Keys and References

We often need to query our inserted data to find specific information, update it when needed, and in some cases, delete it. But how do we find **exactly** the record we're looking for? Bob might tell you that manually searching the database is the answer. But I'm here to tell you, there's a better solution. And it's called a key! :key:

A **primary key** :key: is a combination of one or more columns on a table which together provide a unique identifier for a row in that table. When a primary key is defined, every row has a value and the value is different for every row.  Primary keys that use multiple columns are known as composite primary keys. Each table can only have one primary key defined. 

In many cases the data in various tables is related. The relation is maintained via **references**. A column(s) in one table will contain a value that references a specific row in another table via its primary key. 

The table that contains a copy of the other table's primary key is called a **child table**.
The table whose primary key is referenced is known as the **parent table**. 
The column in the child table that references the primary key in the parent table is called a **foreign key**.

> For example:
> - The data in the table API_CUSTOMER_ORDER is related to the data in API_CUSTOMER (the orders belonging to a particular customer). 
> - The column `CUSTOMER_ID` is the primary key :key: of API_CUSTOMER. The table API_CUSTOMER_ORDER keeps a reference to API_CUSTOMER in the column `CUSTOMER_ID`. 
> - This makes API_CUSTOMER the parent table and API_CUSTOMER_ORDER the child table.

**Exercise 3** :computer: 

Open the **create_schema.sql** file. Can you find the line of code that confirms the above example? 

### Datatypes

Listed below are some of the most commonly used datatypes you'll come across in the **create_schema.sql** file. This is by no means a comprehensive list, not even close actually! 

:book: [Read more](https://docs.oracle.com/cd/A58617_01/server.804/a58241/ch5.htm) about datatypes.

Datatype | Description
--- | ---
varchar2(size) | Maximum size of 4000 bytes.
number(p,s) | Precision can range from 1 to 38. Scale can range from -84 to 127.
date | A date between Jan 1, 4712 BC and Dec 31, 9999 AD.
timestamp | Fractional seconds precision.

:bulb: Note that:

Precision refers to the number of significant digits in a number. 
> For example, the numbers 12.345 and 0.000012345 both have a precision of 5.

Scale, on the other hand refers to the number of significant digits to the right of the decimal point, to and including the least significant digit. 
> For example, 12.345 has a scale of 3 decimal places. 

Similarly, negative scale refers to the number of significant digits to the left of the decimal point, to but not including the least significant digit.

**Exercise 4** :computer: 

Take one last look (for now :wink:) at our favorite file **create_schema.sql** to go over the various datatypes that have been used in the `CREATE TABLE` statements. Note the nuances between datatypes that are the same yet allocate memory very differently based on the size that is passed as an argument to the datatype. 

> For example,

Column | Datatype
--- | ---
FIRST_NAME | VARCHAR2(50)
MIDDLE_INITIAL | VARCHAR2(1)
EMAIL | VARCHAR2(255)
