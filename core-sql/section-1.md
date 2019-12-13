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
> - _VARCHAR2_:  a string datatype with a maximum length of 255 characters. 
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

:book: [Read more](https://www.techonthenet.com/oracle/tables/create_table.php) about `CREATE TABLE`.

Spend a few minutes looking at the `CREATE TABLE` statements in the  **create_schema.sql** file found in the :file_folder: utils folder. While we have not covered many of the options yet, this should give you a good idea of the syntax for creating a table.

### DDL vs DML

The exercise above provides a good segue into DDL and DML. 

Data Definition Language (DDL) | Data Manipulation Language (DML)
--- | ---
Keywords to access and manipulate objects or structures  such as views, tables and procedures. | Keywords to access and manipulate data. These statements are transactional in nature and can be committed or rolled back.
Examples: CREATE, ALTER, DROP | Examples: SELECT, INSERT, UPDATE, DELETE

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

> For example,
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

**Precision** refers to the number of significant digits in a number. 
> For example, the numbers 12.345 and 0.000012345 both have a precision of 5.

**Scale**, on the other hand refers to the number of significant digits to the right of the decimal point, to and including the least significant digit. 
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

## Basic SQL Queries

In this part, we'll cover writing basic DDL and DML queries. 

### Create

You've already spent a little time with the `CREATE` query in [Exercise 1](section-1.md#Columns) so I'll spare you and move on to the next DDL keyword!

### Alter
We all make mistakes and data requirements are ever changing. Hence the keyword `ALTER` allows you to change a table even after you've created it. There are many things that can be done while altering a table, although the most common are adding or changing an existing column and adding or changing constraints.

For example, the query below adds a column `PHONE` with a datatype _VARCHAR2_ not exceeding _30_ characters in length to the table **API_CUSTOMER**.

```SQL
ALTER TABLE API_CUSTOMER 
ADD (PHONE VARCHAR2(30));
```

:book: [Read more](https://docs.oracle.com/cd/E17952_01/refman-5.1-en/alter-table.html) about `ALTER TABLE`.

### Drop
Dropping a table not only means deleting its data but also its underlying structure. In essence, it removes the table from the database including all rows, indexes and privileges.

```SQL
DROP TABLE API_CUSTOMER;
```

:book: [Read more](https://docs.oracle.com/cd/B19306_01/server.102/b14200/statements_9003.htm) about `DROP TABLE`.

:bulb: The series of steps to modify the data and structure of a database are collectively known as a **migration**.

**Exercise 5** :computer: 
Here's a thought experiment. Consider what it would take to add a `NOT NULL` column to a table that already has data. :hushed: 

<details><summary>Solution:</summary>

It is actually a multi-step process.
- Add the new column as `NULL`.
- Populate the column with data.
- Then alter the column to `NOT NULL`.

</details>

### Comments
Comments are part of the query which do not get executed and make it easier to read and maintain your code. They're a life saver when we want to try out various queries to see which one works without having to delete the others. They also serve as an efficient communication tool so that other people can try and make sense of the queries we write and vice versa. 

:book: [Read more](https://docs.oracle.com/cd/B12037_01/server.101/b10759/sql_elements006.htm) about comments.

Two ways to comment are:

**1. Code Blocks**
Begin with an asterisk and a slash. `/*` Add text comment. `*/` End with an asterisk and a slash. You do not need to separate the opening and closing characters with a space.

```SQL
/* This is a comment block. 
It can be used to talk through what the query is intended to do.
Comments like these come in handy during troubleshooting. */
```

**2. Inline Comments**
Begin the comment with -- (two hyphens). Keep comment text to only 1 line. End the comment with a line break.

```SQL
SELECT * FROM API_CUSTOMER; --you'll be learning about the SELECT keyword next!
```

### Select

Once data is stored in the database we need a way to retrieve said data. Retrieving the entire database every time and then going through it in the application is not a viable design. Therefore we need a way to find the exact set of data that we need. This may involve retrieving data from multiple tables at the same time, filtering out those rows we are not interested in, sorting the data and in some cases aggregating it. Queries to retrieve data are known as `SELECT` queries. 

The simplest of them are those that operate on a single table and perform no aggregation just like the one below which retrieves all rows from the table **API_CUSTOMER**.


```SQL
SELECT * FROM API_CUSTOMER;
```

Here, the `*` means all columns from the given table. But what if we wanted only specific columns from that table? In that case, replace `*` with the corresponding column names.

```SQL
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER;
```

**Exercise 6** :computer: 

Go ahead and try it! 
Write `SELECT` queries against tables from our core schema that you were introduced to in [Relations/Tables](section-1.md#Relations/Tables). 

### Aliases

Aliases give a database table, or a column in a table, a temporary name. As your queries grow in size, it gets more cumbersome to reference databases, tables and/or columns by their full names. Your query will also be more prone to typographical errors. All this hassle can be avoided by using short and sweet aliases. 

```SQL
SELECT FIRST_NAME AS f, LAST_NAME AS l FROM API_CUSTOMER;
```

If the above query were bigger and referenced the `FIRST_NAME` and `LAST_NAME` columns multiple times, we'd just be typing `f` and `l` instead. Convenient right? 

**Exercise 7** :computer: 

Give it a shot for a couple of minutes. And look closely at the output. Do the column names in the result change? 

### Rownum

So far, we've learnt how to filter on the specific columns we want to see from a given table but what about the rows? Our queries are still returning all the rows present. In cases where tables contain hundreds of thousands of rows, we're using up a lot of resources trying to retrieve all that data, when in fact we might've been looking only for a subset of it. Enter `ROWNUM`!

The `ROWNUM` keyword allows us to specify the maximum number of desired rows. Any rows above that are discarded from the result set.

For example, suppose we just wanted to see what a row in **API_CUSTOMER_ORDER** looked like but we did not care to retrieve all the records. We can just run the query: 

```SQL
SELECT * FROM API_CUSTOMER_ORDER 
WHERE ROWNUM < 3;
```

:bulb: The column `ROWNUM` is a pseudocolumn that Oracle has available for use in queries. It is calculated at runtime based on the result set in question so you will not find it on disk.

### Select + Where

`ROWNUM` is great but what if we were looking for specific records based on a certain criteria? After all, can you imagine life today without the search :mag_right: feature?! 

The `WHERE` clause allows us to enter a filter criteria and only records matching that criteria are retrieved.

For example, the query below returns all columns from **API_CUSTOMER** whose last name is Doe and first name is John. 

```SQL
SELECT * FROM API_CUSTOMER
WHERE LAST_NAME = 'Doe' AND FIRST_NAME = 'John';
```

**Exercise 8** :computer: 

How about you try implementing the above query (or a similar query) with everything you've learnt so far? Incorporate comments, aliases and column selection in addition to the `WHERE` clause. 

:books: Read more about the following:
- full `SELECT` syntax supported by Oracle [here](https://docs.oracle.com/cd/B28359_01/server.111/b28286/statements_10002.htm#SQLRF01702).
- operators that can be used in queries [here](https://docs.oracle.com/html/A95915_01/sqopr.htm). Please review the **comparison** and **logical** operators. 

### Insert

Pretty straightforward - we use `INSERT` to add new data into our tables. 

Two ways to insert are:

**1. Insert a single row**
Specify the exact value for each column. 

```SQL
INSERT INTO API_CUSTOMER (CUSTOMER_ID, FIRST_NAME, MIDDLE_INITIAL, LAST_NAME, TITLE, 
    ALIAS, EMAIL, CREATED_TIMESTAMP, MODIFIED_TIMESTAMP, MODIFIED_BY)
VALUES (1, 'John', 'D', 'Doe', 'Mr', 
    'Johnny', 'johnny220680@gmailfake.com', systimestamp, systimestamp, 'system');
```
**2. Insert multiple rows**
Pass the results of a `SELECT` into an `INSERT` statement. 

:book: [Read more](https://www.techonthenet.com/oracle/insert.php) about `INSERT`.

### Update

We use `UPDATE` to change existing data. You can update one or more rows using a single `UPDATE` statement. Oracle will determine the rows to be updated based on the `WHERE` clause and will then execute the appropriate update operations on every row that matches the clause condition.

For example, the query below updates John Doe's email in the **API_CUSTOMER** table.

```SQL
UPDATE API_CUSTOMER 
SET EMAIL = 'johnny220680@yahoofake.com' 
WHERE CUSTOMER_ID = 1;
``` 

:book: [Read more](https://www.techonthenet.com/oracle/update.php) about `UPDATE`.

### Delete
We use `DELETE` to delete data. You can delete one or more rows using a single `DELETE` statement. Similar to `UPDATE`, Oracle will determine the rows to be deleted based on the `WHERE` clause and will then execute the appropriate delete operation on every row that matches the clause condition.

For example, the query below deletes John Doe's record from the **API_CUSTOMER** table.

```SQL
DELETE FROM API_CUSTOMER 
WHERE CUSTOMER_ID = 1;
```

:book: [Read more](https://www.techonthenet.com/oracle/delete.php) about `DELETE`.

:bulb: Modifying data using `INSERT`/`UPDATE`/`DELETE` statements is a DML operation and can therefore be done as part of a transaction.
 
:bulb: Before we move on, here's a little cheatsheet for a database question that's asked in plenty of interviews - **What's the difference between `DELETE`, `TRUNCATE` and `DROP`?**

DELETE | TRUNCATE | DROP
--- | --- | ---
Used to remove some or all rows from a table. A WHERE clause can be used to only remove some rows. If no WHERE condition is specified, all rows will be removed. | Used to remove all rows from a table. | Used to remove a table from the database. All the tables' rows, indexes and privileges will also be removed.
After performing a DELETE operation, you need to COMMIT or ROLLBACK the transaction to make the change permanent or to undo it. | The operation cannot be rolled back. | The operation cannot be rolled back.
