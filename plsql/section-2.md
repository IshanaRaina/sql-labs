# Section 2

## Basic Functions and Stored Procedures

Functions | Stored Procedures
--- | ---
A PL/SQL block that has been given a name, a series of parameters that should be provided and that declares a return value. | A PL/SQL block that has been given a name, a series of parameters that should be provided and that does not have a return value. 
Functions can be used in SQL statements such as `SELECT`, `INSERT`, `UPDATE`, `DELETE` and `MERGE`. | Procedures cannot be used in SQL statements such as `SELECT`, `INSERT`, `UPDATE`, `DELETE` and `MERGE`.
Functions can perform queries and calculations using the provided parameters but should not alter the data in the database. | Procedures have no restriction in what they are allowed to do.

A function is just a basic PL/SQL block with a `CREATE OR REPLACE` statement, a name, a declaration of the expected variables, and a declaration of the output type.

> some stuff

```SQL
CREATE OR REPLACE FUNCTION largeOrderWeight(nPrice number, nWeight number) 
RETURN number
IS
BEGIN

   IF nPrice > 1000 AND nWeight > 1000 THEN
     RETURN nWeight;
   ELSE
     RETURN null;
   END IF;

END largeOrderWeight;
```
A procedure is similar to the function, but with a different keyword and the return clause.

> Compare the above function with the stored procedure below.

```SQL
CREATE OR REPLACE PROCEDURE logPriceAndWeight(nPrice number, nWeight number)
IS
BEGIN

  DBMS_OUTPUT.put_line('Price:' || nPrice);
  DBMS_OUTPUT.put_line('Weight:' || nWeight);

END logPriceAndWeight;
```

### In and Out Parameters
The parameters passed into functions and procedures can be `IN`, `OUT`, or `IN OUT`. 

- IN parameters allow information to be passed from the calling code into the function or procedure. They can be read but not written to.
- OUT parameters allow information to be passed from the function or procedure back to the calling code. They can be written to, but will always be null going into the function or procedure.
- IN OUT parameters can be read and written to.

:warning: There are restrictions when using `OUT` or `IN OUT` parameters in functions, the most notable of which is that such functions cannot be called from SQL. In general, when multiple outputs are desired, the recommendation is to use a procedure or to create a custom type (this will be covered in a subsequent module).


> A prcedure with in and out parameters

```SQL
CREATE OR REPLACE PROCEDURE innerProc(inParam in varchar2, inOutParam in out varchar2, 
    outParam out varchar2)
IS
BEGIN

  DBMS_OUTPUT.put_line('Starting innerProc inParam: ' || inParam);
  DBMS_OUTPUT.put_line('Starting innerProc inOutParam: ' || inOutParam);
  DBMS_OUTPUT.put_line('Starting innerProc outParam: ' || outParam);
  
  --Setting an in parameter causes a compilation time error
  --inParam := 'value from innerProc';
  inOutParam := 'value from innerProc';
  outParam := 'value from innerProc';

  DBMS_OUTPUT.put_line('Ending innerProc inParam: ' || inParam);
  DBMS_OUTPUT.put_line('Ending innerProc inOutParam: ' || inOutParam);
  DBMS_OUTPUT.put_line('Ending innerProc outParam: ' || outParam);

END innerProc;
```

> Second procedure calls the first.

```SQL
CREATE OR REPLACE PROCEDURE outerProc
IS

  inParam varchar2(22);
  inOutParam varchar2(22);
  outParam varchar2(22);

BEGIN

  inParam := 'value from outerProc';
  inOutParam := 'value from outerProc';
  outParam := 'value from outerProc';

  DBMS_OUTPUT.put_line('Before innerProc inParam: ' || inParam);
  DBMS_OUTPUT.put_line('Before innerProc inOutParam: ' || inOutParam);
  DBMS_OUTPUT.put_line('Before innerProc outParam: ' || outParam);
  
  --Call the inner procedure
  innerProc(inParam, inOutParam, outParam);

  DBMS_OUTPUT.put_line('After innerProc inParam: ' || inParam);
  DBMS_OUTPUT.put_line('After innerProc inOutParam: ' || inOutParam);
  DBMS_OUTPUT.put_line('After innerProc outParam: ' || outParam);
```

### Passing Data Around
There is a need to pass data from PL/SQL into SQL statements and back. There are several ways to do that, but in this module we will cover the most basic: 
- bind variables
- INTO keyword


#### Bind Variables
Bind variables in PL/SQL are just like bind variables in regular SQL. In addition to the usual benefits of using bind variables they allow PL/SQL to pass parameters to SQL.

> Delete the customer order item row whose ID is set via a bind variable. 

```SQL
CREATE OR REPLACE PROCEDURE deleteCustomerOrderItems
IS

  toDeleteId number(22);

BEGIN

  toDeleteId := 4;
  DELETE FROM api_customer_order_item WHERE customer_order_item_id = toDeleteId;
  COMMIT;

END deleteCustomerOrderItems;
```
:bulb: Note that the bind variable was implicit. There was no use of a colon before the bind variable and the PL/SQL engine was able to determine the correct match based on the name between the variable in  SQL and the variable in the calling PL/SQL procedure.

#### Into Keyword
Allows us to read one or more values from a statement into variables defined in a PL/SQL block.

> Query and print the number of items with the same names. 

```SQL
CREATE OR REPLACE PROCEDURE printItemInfo
IS
  sItemName varchar2(30);
  iItemCount number(22);
BEGIN
  SELECT name, count(*) INTO sItemName, iItemCount FROM api_item 
    WHERE name = 'API Canned Cat Food' GROUP BY name;  
  DBMS_OUTPUT.put_line('sItemName:' || sItemName || ' iItemCount:' || iItemCount);
END printItemInfo;
```
