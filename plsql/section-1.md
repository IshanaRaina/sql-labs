# Section 1

## Introduction to PL/SQL

### Blocks
Let's keep the tradition alive! 

> Start with a minimal anonymous PL/SQL block to print 'Hello World!'

```SQL
BEGIN
  DBMS_OUTPUT.put_line ('Hello World!');
END;
```

:bulb: The above block could be executed via SQLDeveloper but is not re-usable because it lacks a name. Note that you need to run `SET SERVEROUTPUT ON;` first to be able to see the output.

### Declare
> Next, create a variable and assign the 'Hello World!' message to the variable before printing it. 

```SQL
DECLARE
  sMessage varchar2(255); 
BEGIN
  sMessage := 'Hello World!';
  DBMS_OUTPUT.put_line(sMessage);
END;
```

### Exception
A block can also contain an 'EXCEPTION' clause where error handling for the block is declared. In the exception block, choose what exceptions to handle, and what code to execute for each exception.

> Building on the previous block, add a catch-all exception clause that prints the error message.

```SQL
DECLARE
  sMessage varchar2(255); 
BEGIN
  sMessage := 'Hello World!';
  DBMS_OUTPUT.put_line(sMessage);
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.put_line(SQLERRM);
END;
```

### Integration with SQL
Moving on from 'Hello World!', let's turn our attention back to our online pet store API. 

One of the most powerful features of PL/SQL is its integration with SQL. To invoke an SQL statement just include it inside the PL/SQL block.

> Let's delete some records from **API_CUSTOMER_ORDER_ITEM** and print a message.

```SQL
DECLARE
  sMessage varchar2(255); 
BEGIN
  sMessage := 'Deleted some items!';
  DELETE FROM API_CUSTOMER_ORDER_ITEM WHERE ROWNUM < 5;
  COMMIT;
  DBMS_OUTPUT.put_line(sMessage);
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    DBMS_OUTPUT.put_line(SQLERRM);
END;
```

### Nesting blocks within blocks
 > Redo the previous example nesting the `DELETE` statement and its `COMMIT/ROLLBACK` in a sub-block.

```SQL
DECLARE
  sMessage varchar2(255); 
BEGIN
  sMessage := 'Deleted some items!';
  BEGIN
    DELETE FROM API_CUSTOMER_ORDER_ITEM WHERE ROWNUM < 5;
    COMMIT;
  EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
    DBMS_OUTPUT.put_line(SQLERRM);
  END;
  DBMS_OUTPUT.put_line(sMessage);
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.put_line(SQLERRM);
END;
```

### Error catching
In the previous examples, you've come across `WHEN OTHERS THEN` within the `EXCEPTION` block. 

> Let's work on yet another example that throws an error and lets `WHEN OTHERS THEN` catch it. 

```SQL
DECLARE
  sMessage varchar2(255); 
BEGIN
  sMessage := 'Ran after the error!';
  raise_application_error(-20001, 'Custom error message');
  DBMS_OUTPUT.put_line(sMessage);
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.put_line(SQLERRM);
END;
```

### Custom Errors
When an error is caught the code can inspect both the error name and the code. This allows programmatic decisions to be based on it. 

> Define, raise and catch a custom error.

```SQL
DECLARE
  custom_error EXCEPTION;
  sMessage varchar2(255); 
BEGIN
  sMessage := 'Ran after the error!';
  RAISE custom_error;
  DBMS_OUTPUT.put_line(sMessage);
EXCEPTION
  WHEN custom_error THEN
    DBMS_OUTPUT.put_line('Caught the custom error');
    DBMS_OUTPUT.put_line(SQLERRM);
  WHEN OTHERS THEN
    DBMS_OUTPUT.put_line(SQLERRM);
END;
```

> Catch and re-raise a custom error.

```SQL
DECLARE
  custom_error EXCEPTION;
  sMessage varchar2(255); 
BEGIN
  sMessage := 'Ran after the error!';
  RAISE custom_error;
  DBMS_OUTPUT.put_line(sMessage);
EXCEPTION
  WHEN custom_error THEN
    DBMS_OUTPUT.put_line('Caught the custom error');
    DBMS_OUTPUT.put_line(SQLERRM);
    RAISE;
  WHEN OTHERS THEN
    DBMS_OUTPUT.put_line(SQLERRM);
END;
```

## Views
A view is a named query that can be used as a virtual table in SQL (including in other views). The view can then be used in SQL queries and also in other views as if it were a table.

:bulb: The first line contains the view definition and name but the rest of the query is just a regular `SELECT` query. 

> Create a view that joins customers to the items they've purchased and includes the item and package type details.

```SQL
CREATE OR REPLACE VIEW V_API_CUSTOMER_ITEMS AS
SELECT C.CUSTOMER_ID, C.FIRST_NAME, C.LAST_NAME, C.EMAIL, I.ITEM_ID, I.SKU, 
    I.NAME AS ITEM_NAME, I.DESCRIPTION AS ITEM_DESCRIPTION, I.WEIGHT, I.PRICE, 
    I.DIMENSIONS, PT.CODE, PT.NAME AS PACKAGE_TYPE_NAME, 
    PT.DESCRIPTION AS PACKAGE_TYPE_DESCRIPTION
FROM API_CUSTOMER C 
JOIN API_CUSTOMER_ORDER CO ON C.CUSTOMER_ID = C.CUSTOMER_ID
JOIN API_CUSTOMER_ORDER_ITEM COI ON COI.CUSTOMER_ORDER_ID = CO.CUSTOMER_ORDER_ID
JOIN API_ITEM I ON I.ITEM_ID = COI.ITEM_ID
JOIN API_PACKAGE_TYPE PT ON PT.PACKAGE_TYPE_ID = I.PACKAGE_TYPE_ID;
```

> Use the view in a query. Retrieve all items purchased by people with a last name of 'Doe'.

```SQL
SELECT * FROM V_API_CUSTOMER_ITEMS WHERE LAST_NAME = 'Doe';
```

:bulb: It looks like a regular query and works like one. If we did not know that  **V_API_CUSTOMER_ITEMS** was a view, we could be forgiven for thinking it was a table. That motivates the naming convention of starting the name with the term **V_**. Using a view within in view requires no special syntax or changes. It just means that the view query itself makes use of a view.

### Materialized View
A materialized view is a view where the contents of the query have been executed and stored on disk as a table. In the Oracle data dictionary it will be listed both as a view and as a table. A a table, it will have statistics and indexes can be created on it.

> Create a materialized view using the same logic as the view **V_API_CUSTOMER_ITEMS**.

```SQL
CREATE MATERIALIZED VIEW MV_API_CUSTOMER_ITEMS
  NOLOGGING
  CACHE
  BUILD IMMEDIATE
  AS
  SELECT C.CUSTOMER_ID, C.FIRST_NAME, C.LAST_NAME, C.EMAIL, I.ITEM_ID, I.SKU, 
    I.NAME AS ITEM_NAME, I.DESCRIPTION AS ITEM_DESCRIPTION, I.WEIGHT, I.PRICE, 
    I.DIMENSIONS, PT.CODE, PT.NAME AS PACKAGE_TYPE_NAME, 
    PT.DESCRIPTION AS PACKAGE_TYPE_DESCRIPTION
  FROM API_CUSTOMER C
    JOIN API_CUSTOMER_ORDER CO ON C.CUSTOMER_ID = C.CUSTOMER_ID
    JOIN API_CUSTOMER_ORDER_ITEM COI ON COI.CUSTOMER_ORDER_ID = CO.CUSTOMER_ORDER_ID
    JOIN API_ITEM I ON I.ITEM_ID = COI.ITEM_ID
    JOIN API_PACKAGE_TYPE PT ON PT.PACKAGE_TYPE_ID = I.PACKAGE_TYPE_ID;
```

:bulb: The syntax is very similar to that of the view, but has a few extra options to specify creation and refresh conditions.

> Created some indexes on the materialized view to speed up lookups.

```SQL
CREATE INDEX MV_CUST_ITEM_NAME_LOOKUP_IDX 
ON MV_API_CUSTOMER_ITEMS(LAST_NAME, FIRST_NAME);
```

```SQL
CREATE INDEX MV_CUST_ITEM_EMAIL_LOOKUP_IDX ON MV_API_CUSTOMER_ITEMS(EMAIL);
```
:bulb: There is no difference as far as the indexes are concerned about a materialized view and a regular table.

> Use the materialized view in a query. Retrieve all items purchased by people with a last name of 'Doe'.

```SQL
SELECT * FROM MV_API_CUSTOMER_ITEMS WHERE LAST_NAME = 'Doe';
```
:warning: When you run the above query it returns nothing. That is because the materialized view was created before the rows were inserted into the API tables. The materialized view has a stale view of the data.

> Refresh the materialized view and run the query again.

```SQL
CALL DBMS_MVIEW.refresh('MV_API_CUSTOMER_ITEMS');
```

Do you :eyes: the results now?
