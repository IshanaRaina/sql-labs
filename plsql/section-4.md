# Section 4

## Cursors
A cursor is a pointer :arrow_right: to information on the processing of a `SELECT` or DML operation. 

### Explicit Cursors
An explicit cursor is one that is defined in the declaration section of a PL/SQL block. It is created on a `SELECT` statement that will produce the rows for the cursor.

The **first step** to an explicit cursor is the declaration. This is where the cursor name, the associated `SELECT` statement and any variables passed in as arguments are defined.

> A cursor declaration for a cursor with no parameters:

```SQL
CURSOR allCustomerOrders IS
  SELECT customer_order_id, total_price FROM api_customer_order;
```

### Declared Parameters
A cursor can also declare parameters to be provided when opening it. This allows the PL/SQL code to execute arbitrary code and determine the appropriate values before opening the cursor.

> A cursor with one declared parameter:

```SQL
CURSOR allCustomerOrders(iCustomerId number) IS
  SELECT customer_order_id, total_price 
  FROM api_customer_order WHERE customer_id = iCustomerId;
```

### Opening a Cursor
The **second step** is to open the cursor. If the cursor definition listed any parameters, they must be provided at this time. When the cursor is opened, the first row returned by the SQL statement becomes the current row.

> Open the cursor seen in the first example (it has no parameters):

```SQL
OPEN  allCustomerOrders;
```
> Open the cursor seen in the second example (it has one parameter):

```SQL
OPEN  allCustomerOrders(1);
```
:bulb: The above example hard-codes a value, but a variable can also be passed in.

### Fetching Data
The **third step** is to access the data. This is done with the `FETCH` statement. The `FETCH` statement lists the cursor to fetch from and the list of variables to fetch the data into. The number of variables, their order and their type should match the columns returned from the `SELECT` statement. Every time the fetch is called, Oracle will return the data for that row and move the pointer to the next row.

> Fetch the data from the cursor into two variables:

```SQL
FETCH allCustomerOrders INTO iCurrentCustomerOrderId, iCurrentTotalAmount;
```

### Fetching Rowtype Variable
The above example is typically run in a loop, but that is not required. Oracle also provides a shorthand to avoid having to declare, track and pass around a ton of variables. You can declare a `ROWTYPE` object and set it to match the cursor. When fetching you fetch into the `ROWTYPE` variable.

> Using `ROWTYPE`:

```SQL
--In the declare section
customerOrderRow  allCustomerOrders%ROWTYPE;

--In the block body, when fetching
FETCH allCustomerOrders INTO customerOrderRow;
```

### Closing a Cursor
The **last step** is to close the cursor to allow Oracle to release all of the associated resources. 

:warning: Also, if you want to use a cursor twice in a PL/SQL block you need to close it before trying to open it again or Oracle will raise an error.

> Close the cursor:

```SQL
CLOSE allCustomerOrders;
```

### Status of Cursors
Oracle provides various attributes that allow you to check the status of a given cursor:

Attribute | Description
--- | ---
%FOUND | This will return true if the cursor contains at least one more row, false otherwise.
%NOTFOUND | The opposite of %FOUND.
%ROWCOUNT | The number of rows fetched by the select statement thus far. This is not a count of the total number of rows that will be returned.
%ISOPEN | True if the cursor is already open, false otherwise.

## Loops
Oracle provides three types of looping structures for use with cursors: 
- simple loops
- while loops 
- for loops

### 1. Simple Loops
Simple loops do not use the for or when keywords. They specify fetch and exit conditions at the start of the loop.

> An anonymous block using a simple loop to print the various customer order IDs and their totals:

```SQL
SET SERVEROUTPUT ON;
DECLARE
CURSOR allCustomerOrders IS
    SELECT customer_order_id, total_price FROM api_customer_order;
    
  iCurrentCustomerOrderId api_customer_order.customer_order_id%TYPE;
  iCurrentTotalPrice api_customer_order.total_price%TYPE;

BEGIN
  OPEN allCustomerOrders;
  LOOP
    FETCH allCustomerOrders into iCurrentCustomerOrderId, iCurrentTotalPrice;
    EXIT WHEN allCustomerOrders%notfound;
    
    DBMS_OUTPUT.PUT_LINE('iCurrentCustomerOrderId:' || iCurrentCustomerOrderId || 
      ' iCurrentTotalPrice:' || iCurrentTotalPrice);
    
  END LOOP;
  CLOSE allCustomerOrders;
END;
```
:bulb: Note the use of `%TYPE` to set the variable types. This is better than looking up the types and explicitly setting them, since if the underlying table changes odds are that the code will remain valid and a recompile is all that is needed.

### 2. While Loops
While loops use a `WHILE` keyword. It repeats a statement or group of statements while a given condition is true and tests the condition before executing the loop body.

> A general outline of a while loop:

```SQL
DECLARE @intFlag INT
SET @intFlag = 1
WHILE (@intFlag <=5)
BEGIN
    PRINT @intFlag
    SET @intFlag = @intFlag + 1
END
```

> An anonymous block using a while loop to print the various customer order IDs and their totals:

```SQL
SET SERVEROUTPUT ON;
DECLARE
  CURSOR allCustomerOrders IS
    SELECT customer_order_id, total_price FROM api_customer_order;
    
  iCurrentCustomerOrderId api_customer_order.customer_order_id%TYPE;
  iCurrentTotalPrice api_customer_order.total_price%TYPE;

BEGIN
  OPEN allCustomerOrders;
  FETCH allCustomerOrders INTO iCurrentCustomerOrderId, iCurrentTotalPrice;
  WHILE allCustomerOrders%found
  LOOP
    
    DBMS_OUTPUT.PUT_LINE('iCurrentCustomerOrderId:' || iCurrentCustomerOrderId || 
      ' iCurrentTotalPrice:' || iCurrentTotalPrice);
      
    FETCH allCustomerOrders INTO iCurrentCustomerOrderId, iCurrentTotalPrice;
  END LOOP;
  CLOSE allCustomerOrders;
END;
```

### 3. For Loops
For loops use a `FOR` keyword and will automatically take care of various loop aspects. You do not need to open, fetch or close the cursor.

> A simple example of a for loop:

```SQL
BEGIN
  DBMS_OUTPUT.PUT_LINE ('lower_bound < upper_bound');
 
  FOR i IN 1..3 LOOP
    DBMS_OUTPUT.PUT_LINE (i);
  END LOOP;
 
  DBMS_OUTPUT.PUT_LINE ('lower_bound = upper_bound');
 
  FOR i IN 2..2 LOOP
    DBMS_OUTPUT.PUT_LINE (i);
  END LOOP;
 
  DBMS_OUTPUT.PUT_LINE ('lower_bound > upper_bound');
 
  FOR i IN 3..1 LOOP
    DBMS_OUTPUT.PUT_LINE (i);
  END LOOP;
END;
```

> An anonymous block using a for loop to print the various customer order IDs and their totals. This example also uses a rowtype variable to reduce the number of variables:

```SQL
SET SERVEROUTPUT ON;
DECLARE
  CURSOR allCustomerOrders IS
    SELECT customer_order_id, total_price FROM api_customer_order;  
  currentCustomerOrder allCustomerOrders%ROWTYPE;

BEGIN
  FOR currentCustomerOrder IN allCustomerOrders
  LOOP    
    DBMS_OUTPUT.PUT_LINE('iCurrentCustomerOrderId:' || 
       currentCustomerOrder.customer_order_id || 
       'iCurrentTotalPrice:' || currentCustomerOrder.total_price);   
  END LOOP;  
END;
```

### Implicit Cursors
Implicit cursors remove the need to declare the cursor and the row variable. Oracle will take care of all this for you. 

> General syntax for an implicit cursor:

```SQL
SELECT my_column INTO my_variable FROM my_table WHERE my_condition;
```

> An anonymous block using a for loop with an implicit cursor to print the various
customer order IDs and their totals:

```SQL
SET SERVEROUTPUT ON;

DECLARE
BEGIN
  FOR currentCustomerOrder IN (
    SELECT customer_order_id, total_price FROM api_customer_order) LOOP
    
    DBMS_OUTPUT.PUT_LINE('iCurrentCustomerOrderId:' || 
      currentCustomerOrder.customer_order_id || 
      'iCurrentTotalPrice:' || currentCustomerOrder.total_price);
    
  END LOOP;
END;
```

### Updating with Cursors
It is possible to update or delete the last row that was fetched by the cursor. To do this the cursor must use the term `FOR UPDATE` in the declaration and the expression `WHERE CURRENT OF` to perform the update.

> Update the customer order total price using the cursor:

```SQL
DECLARE
  CURSOR allCustomerOrders IS
    SELECT customer_order_id, total_price FROM api_customer_order FOR UPDATE;
  currentCustomerOrder allCustomerOrders%rowtype;
BEGIN
  FOR currentCustomerOrder IN allCustomerOrders
  LOOP   
    UPDATE api_customer_order SET total_price = 10 WHERE CURRENT OF allCustomerOrders;   
  END LOOP;
  COMMIT;  
END;
```

Did the value update?!

:bulb: Note that the change to the value is not reflected in _currentCustomerOrder_. It will continue to hold the pre-update value, however the change will be made to the database record.

:skull: In general, updates such as the above are not recommended, nor is looping over a result set and issuing one update per row. 

:white_check_mark: The recommended approach is to:
- do the updates in a single SQL statement (updating multiple rows at once) or
- use the bulk collect operator (this is covered in a later module). 

These later approaches are much faster and are usually easier to read. The same comment applies to inserts and deletes.

:books: Read more about:
- [cursors](https://www.techonthenet.com/oracle/cursors/index.php)
- [loops](https://docs.oracle.com/cd/B28359_01/appdev.111/b28370/loop_statement.htm#LNPLS01328)
