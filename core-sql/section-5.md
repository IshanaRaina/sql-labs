# Section 5

## Analytic Functions

Analytic functions are related to aggregate functions. They were introduced in Oracle 8i to allow things that previously could not be done in a practical way in SQL. They are typically used in reporting and in warehouse applications to calculate thing such as rankings, moving window calculations, lag and lead analysis etc.

The main differences between aggregate and analytic functions:

Aggregate functions | Analytic functions
--- | ---
Perform operations on the set of rows associated to the same `GROUP BY` value. They cannot access rows associated to other `GROUP BY` values. | Are not restricted to those rows associated to a `GROUP BY`. They do not even require a group by but can be restricted to a subset of the rows. 
Furthermore, using a `GROUP BY` will alter the number of returned rows (it will almost certainly reduce it). | Using an analytic function will not alter the number of returned rows.

**Exercise** :computer: 

:bell: To run these examples you must first run the script **module_schema.sql**. The examples use the `RANK` and `AVG` analytic functions as these are some of the most intuitive of the functions. 

1. Display the list of employees, and for each employee display how their compensation ranks. Order the results by rank. Note the use of `ORDER BY` to enable the rank calculation.

```SQL
SELECT STATE_CODE, LAST_NAME, FIRST_NAME, TITLE, SALARY, 
RANK() OVER (ORDER BY SALARY DESC, BONUS_PERCENT DESC) AS RANK 
FROM API_EMPLOYEE ORDER BY RANK ASC, LAST_NAME DESC; 
```

2.  Display the list of employees, and for each employee display how their compensation ranks relative to other employees in their state. Order the results by state and rank. Note the use of `PARTITION BY` to make the rank relative to the state.

```SQL
SELECT STATE_CODE, LAST_NAME, FIRST_NAME, TITLE, SALARY,
RANK() OVER (
    PARTITION BY STATE_CODE ORDER BY SALARY DESC, BONUS_PERCENT DESC) AS RANK
FROM API_EMPLOYEE
ORDER BY STATE_CODE ASC, RANK ASC, LAST_NAME DESC; 
```

3. Keep the state rank, but add other analytic functions. Show the state average salary, the company average salary and the average salary by title.

```SQL
SELECT STATE_CODE, LAST_NAME, FIRST_NAME, TITLE, SALARY,
    RANK() OVER (PARTITION BY STATE_CODE ORDER BY SALARY DESC, BONUS_PERCENT DESC) 
        AS RANK,
    AVG(SALARY) OVER (PARTITION BY STATE_CODE) STATE_AVG,
    AVG(SALARY) OVER () COMPANY_AVG,
    AVG(SALARY) OVER (PARTITION BY TITLE) TITLE_AVERAGE
FROM API_EMPLOYEE
ORDER BY STATE_CODE ASC, RANK ASC, LAST_NAME DESC; 
```

4. Display the list of employees, and for each employee display how their salary ranks relative to their title. Show the title average salary. For each employee show the average salary for people hired within 30 days of that employee (before or after). Order the results by title and rank. Note the use of partition by to make the rank relative to the title. Note the use of a windowing clause to support the within 30 days sliding average.

```SQL
SELECT STATE_CODE, LAST_NAME, FIRST_NAME, TITLE, SALARY,
    RANK() OVER (PARTITION BY TITLE ORDER BY SALARY DESC, BONUS_PERCENT DESC) RANK,
    AVG(SALARY) OVER (PARTITION BY TITLE) TITLE_AVG,
    AVG(SALARY) OVER (PARTITION BY TITLE ORDER BY HIRE_DATE RANGE 
        BETWEEN 30 PRECEDING AND 30 FOLLOWING) TITLE_AVG_PLUS_MINUS_30
FROM API_EMPLOYEE
ORDER BY TITLE ASC, RANK ASC, LAST_NAME DESC; 
```

5. Given a proposed salary and bonus percentage see where it ranks in the absolute list of salaries and bonuses in the company.

```SQL
SELECT RANK(85000, 12) WITHIN GROUP
   (ORDER BY SALARY DESC, BONUS_PERCENT DESC) "RANK"
   FROM API_EMPLOYEE;
```

:bulb: **Processing Order** - Analytic functions are evaluated after the `WHERE`, `GROUP BY` and `HAVING` clauses but before the `ORDER BY` clause. Therefore they can only appear in the `SELECT` and the `ORDER BY` clauses. Note that this is not as restrictive as it seems since you can place an analytic function in a subquery and then use it as needed.

**Partitions** divide the rows into disjoint subsets. Every row falls into one and only one of these subsets. These subsets are then made available to the analytic function. This is one of the mechanisms used to restrict the rows that go into an analytic function. Note that partitions in the context of an analytic query have nothing to do with table and index partitions. The mathematical notion of partition applies however. 

> For example, the examples partitioned the data by state or title depending on the analysis being performed. If the goal is to do a comparison with other rows in the same state, and to ignore all other rows, then we partition by state. 

**Current Row** – The position of the current row determines the position and contents of the sliding window. As the row is changed so is the window.

**Window** – A sliding window of data within the partition. The window further restricts the range of rows that are made available to the analytic function. Defining a window requires that an order be specified for the rows in the partition. Once that order is determined, the position of the current row in that order is known, and the criteria that defines the window can be evaluated to determine the rows that will be provided to the analytic function. In general, the starting position of the window can be defined relative to the current row as a lag/lead by a fixed amount or the entire preceding set of rows. Similarly, the ending position of the window can be defined relative to the current row as a lag/lead by a fixed amount or the entire following set of rows. 

It is possible to have a varying window size per row based on the output of a function. 

:bulb: In general, 
- If you need to work with analytic functions we would recommend spending some time to understand the different analytic functions and the full syntax. If you can't figure out how to do something, spend some time doing research online. Odds are many others have tried to do something similar and you can find what the solution or the suggested alternative is.
- It is possible to create custom aggregate and analytic functions. However, odds are that the required functionality can be implemented using existing functions and features. Please check with your DBAs before going down that route.

:book: [Read more](http://docs.oracle.com/cd/E11882_01/server.112/e41084/functions004.htm#SQLRF06174) about analytic functions. 

**Exercise** :computer: 

Implementing analytic functions

## Oracle Metadata

Oracle contains vast amounts of metadata. In fact, if you can think of it, Oracle is probably tracking it. The number of views and tables is very large, and only a very seasoned DBA would be expected to know what most of them even do. These tables and views are most often used for advanced troubleshooting and scripting. Even administration tools such as OEM use these tables and views behind the covers (OEM will actually show you the SQL it is using to pull the data).

:warning: A warning is in order: never directly update an Oracle owned table without an extremely good reason. Odds are that Oracle and the DBAs have it locked down, but just in case, do not try it. 

Oracle provides a large number of packages to assist with all types of database administration. These packages allow users to gather statistics, partition tables, refresh materialized views etc. 

One type of data that Oracle has is metadata on all schema objects in the database (tables, indexes, constraints, triggers, views, partitions, synonyms, sequences, packages...). This metadata can be queried and Oracle provides some user friendly views for this purpose. This is known as the **data dictionary.**

> To find information on the table **API_ITEM** execute the query below.

```SQL
SELECT * FROM ALL_TABLES WHERE TABLE_NAME = 'API_ITEM';
```

:bulb: Note the use of `all_` If multiple tables with that name existed (different owners) you would see them in the result as multiple rows. The prefix `all_` spans the entire database, but only returns those things the current user has permissions to see. 
Similarly, 
- the prefix `user_` will only consider those objects owned by the current user. 
- the prefix `dba_` will show everything across all schemas.

> To find information on the columns in the table **API_ITEM**
execute the query below.

```SQL
SELECT * FROM ALL_TAB_COLS WHERE TABLE_NAME = 'API_ITEM';
```

> To find the partitions on the table **API_EMPLOYEE_HASH_PART** run the query below.

```SQL
SELECT * FROM ALL_TAB_PARTITIONS WHERE TABLE_NAME = 'API_EMPLOYEE_HASH_PART';
```

> Suppose we wanted to drop all tables that start with **API_**. We can create a SQL query that will generate the drop statements for us. It will have all of the tables. 

```SQL
SELECT 'DROP TABLE ' || TABLE_NAME || ';' 
FROM ALL_TABLES 
WHERE TABLE_NAME LIKE 'API_%'
```

Note that the query does not look into the views with referential integrity information, so the order is random. As a result, if it is run, some of the drops will fail. The query could be enhanced, or you can just run it a couple of times.

:bulb: In general: 
- When you are looking to find some metadata, there is a way to get it, the only question is how. You can search online (if you use the right keywords you will find the correct answers without too much difficulty) or you can read through the documentation. It is worthwhile to skim through the documentation once to get a feel for what is available.
- The information in the data dictionary is useful for research, but it can also be used to speed up support tasks and even write automated scripts.
- Database maintenance scripts often make use of the data dictionary to ensure that they are generic (don't have hard-coded table names) and keep running even when the schema is altered.

:books: Read more about the following:
- A short list of data dictionary views [here](http://docs.oracle.com/cd/B28359_01/server.111/b28310/tables014.htm).
- or check out the full list [here](http://docs.oracle.com/cd/B28359_01/nav/catalog_views.htm).
