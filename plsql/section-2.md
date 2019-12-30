# Section 2

## Basic Functions and Stored Procedures

Functions | Stored Procedures
--- | ---
A PL/SQL block that has been given a name, a series of parameters that should be provided and that declares a return value. | A PL/SQL block that has been given a name, a series of parameters that should be provided and that does not have a return value. 
Functions can be used in SQL statements such as `SELECT`, `INSERT`, `UPDATE`, `DELETE` and `MERGE`. | Procedures cannot be used in SQL statements such as `SELECT`, `INSERT`, `UPDATE`, `DELETE` and `MERGE`.
Functions can perform queries and calculations using the provided parameters but should not alter the data in the database. | Procedures have no restriction in what they are allowed to do.

A function is just a basic PL/SQL block with a `CREATE OR REPLACE` statement, a name, a declaration of the expected variables, and a declaration of the output type.

> Create a function that identifies large order weights. If weight and price of the order are both greater than 100, return the weight. If not, return null.

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
A procedure is similar to the function above, but with a different keyword and no return clause.

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

:book: [Read more](https://docs.oracle.com/cd/B28359_01/appdev.111/b28843/tdddg_procedures.htm) about stored procedures.

## Exercises :computer: 

One last thing before we jump into exercises - **Oracle Dual Table**.

In Oracle, `SELECT` statements have to be accompanied by a `FROM` clause. However, sometimes we write queries that don't particularly require any table. 

> For example, what if we wanted to calculate the length of string 'Hello World!'? My query would look something like this:

```SQL
SELECT length('Hello World!') AS str_length
FROM ???
;
```

We don't need any table to find the length of a given string but I have to use a `FROM` clause. This is where the DUAL table comes into play. It is a special table that belongs to the schema of the user SYS but is accessible to all users. The **DUAL table** has one column named `DUMMY` whose data type is _VARCHAR2()_ and contains one row with a value X.

> Using dual table, the query can be written as follows:

```SQL
SELECT length('Hello World!') AS str_length
FROM dual
;
```

:bulb: The reason this is important for the following exercises is: Oracle treats the use of DUAL the same as calling a function which simply evaluates the expression used in the `SELECT` statement.

:alarm_clock: Access the **HR** database for the following exercises. 

1. Remember the dating ability exercise you'd solved based on employees' salary? Let's build on that using a function. Use employee ID as your parameter and return the dating advice. Check your function against `<function_name>(101)`, `<function_name>(103)` and `<function_name>(106)`. Remeber to use the dual table. 
2. Create a function that takes salary as the parameter and returns the dating advice. Check your function against `<function_name>(15000)`, `<function_name>(7000)` and `<function_name>(5)`.
3. Create a test that uses the function created in step 2 and shows all employees with the advice.
    - Discuss what the biggest advantage of using salary instead of employee ID as the parameter is?  What is the disadvantage? 
5. Create a function that takes employee ID as the parameter and returns the region name. Check your function against `<function_name>(101)`.
6. Create a function that takes department ID as the parameter and returns the corresponding department name. Check your function against `<function_name>(90)`. Also, run the function against all department IDs.
7. Repeat exercise step 6 but with a procedure. Check your procedure against `<procedure_name>(90)`.
8. Food for thought - let's use the function to make the above procedure simpler.
9. Create a procedure that takes employee ID and NewSalary as parameters. The procedure should update the salary to the new number. :bell: NewSalary should always be higher than the current salary.
10. Create a procedure called `ADD_REGION` that adds a region to the **REGIONS** table. Also use error handling on duplicate values. 
11. Create a procedure called `NUKE_REGION` that removes a region based on its ID. Check to see if the region doesn't exist and inform the user that nothing was deleted. 

<details><summary>Solution 1:</summary>

```SQL
CREATE OR REPLACE FUNCTION GoldDigger (sasquatch number)
RETURN varchar2 
IS advice varchar2(5);
BEGIN
  select 
    case when salary >= 10000 then 'Marry' 
       when salary >= 6000 then 'Date' 
       else 'Loser' 
    end into advice
  from employees where employee_id = sasquatch;  
  RETURN advice;
END;
```

Checking the function:
```SQL
SELECT GoldDigger(101), GoldDigger(103), GoldDigger(106) 
FROM dual;
```

</details>


<details><summary>Alternative Solution 1:</summary>

```SQL
CREATE OR REPLACE FUNCTION GoldDigger2 (id number)
RETURN varchar2
IS
sal number;
BEGIN
  select salary into sal from employees where employee_id = id;
  if sal >= 10000 then return 'Marry';
     elsif sal >= 6000 then return 'Date';
     else return 'Loser';
  end if;
END GoldDigger2;
```

Checking the function:
```SQL
SELECT GoldDigger2(101), GoldDigger2(103), GoldDigger2(106) 
FROM dual;
```

</details>

<details><summary>Solution 2:</summary>

```SQL
CREATE FUNCTION GoldDigger3 (sal number)
RETURN varchar2 AS
BEGIN
  if sal >= 10000 then return 'Marry';
     elsif sal >= 6000 then return 'Date';
     else return 'Loser';
  end if;
END GoldDigger3;
```

Checking the function:
```SQL
SELECT GoldDigger3(15000), GoldDigger3(7000), GoldDigger3(5) 
FROM dual;
```

</details>

<details><summary>Alternative Solution 2:</summary>

```SQL
CREATE FUNCTION GoldDigger4 (sal number)
RETURN varchar2 AS 
BEGIN
  return (case when sal >= 10000 then 'Marry'
               when sal >= 6000 then 'Date'
               else 'Loser' end);
END GoldDigger4;
```

Checking the function:
```SQL
SELECT GoldDigger4(15000), GoldDigger4(7000), GoldDigger4(5) 
FROM dual;
```

</details>

<details><summary>Solution 3:</summary>

```SQL
SELECT First_Name, Last_Name, GoldDigger4(Salary) as Advice 
FROM employees;
```

</details>

<details><summary>Solution 4:</summary>

```SQL
CREATE OR REPLACE FUNCTION Emp_Region (id number) 
RETURN varchar2 AS 
region varchar2(50);
BEGIN
  select Region_Name into region from emp_by_region where employee_id = id;
  return region;
END;
```
Checking the function:
```SQL
SELECT Emp_Region(101) 
FROM dual;
```

</details>

<details><summary>Solution 5:</summary>

```SQL
CREATE OR REPLACE FUNCTION get_dept ( id number ) 
  RETURN VARCHAR2
IS
  dept VARCHAR2(50) := 'Not Assigned';
BEGIN  
  select department_name INTO dept
  from departments
  where department_id = id;
  RETURN dept;
END;
```
Checking the function against 1 ID:
```SQL
SELECT get_dept(90) 
FROM dual;
```

Checking the function against all IDs:
```SQL
SELECT first_name, last_name, get_dept(department_id) 
FROM employees;
```

</details>

<details><summary>Solution 6:</summary>

```SQL
CREATE OR REPLACE PROCEDURE get_dept_proc ( id number ) 
IS
  dept VARCHAR2(50);
BEGIN  
  select department_name INTO dept
  from departments
  where department_id = id;

  DBMS_OUTPUT.PUT_LINE( dept );
END;
```

Checking the procedure:
```SQL
SET SERVEROUTPUT ON;
EXECUTE get_dept_proc(90);
```

</details>

<details><summary>Solution 7:</summary>

```SQL
CREATE OR REPLACE PROCEDURE get_dept_proc2 ( id number ) 
IS
  dept VARCHAR2(50);
BEGIN  
  DBMS_OUTPUT.PUT_LINE( get_dept(id) );  
END;
```

Checking the procedure:
```SQL
SET SERVEROUTPUT ON;
EXECUTE get_dept_proc2(90);
```

</details>

<details><summary>Solution 8:</summary>

```SQL
CREATE PROCEDURE set_salary (id number, NewSalary number)
AS 
BEGIN
  update employees set salary = NewSalary
  where employee_id = id
    and NewSalary > salary;
END;
```

Executing the procedure:
```SQL
-- current salary is 24000
EXECUTE set_salary(100,25000); 
```

Checking the procedure:
```SQL
SELECT * 
FROM employees 
WHERE employee_id = 100;
```

</details>

<details><summary>Solution 9:</summary>

```SQL
CREATE OR REPLACE  PROCEDURE ADD_REGION(id NUMBER, name VARCHAR2) AS 
BEGIN
  INSERT INTO regions (region_id, region_name) VALUES (id, name);
  COMMIT;
  DBMS_OUTPUT.PUT_LINE(name || ' added');
EXCEPTION
  WHEN DUP_VAL_ON_INDEX THEN     
    DBMS_OUTPUT.PUT_LINE('Region ID already exists');
  WHEN OTHERS THEN  
    DBMS_OUTPUT.PUT_LINE('Something bad happened');
    ROLLBACK;
END;
```

Executing the procedure:
```SQL
EXECUTE ADD_REGION(666,'EVIL PLACE');
```

Checking the procedure:
```SQL
SELECT * FROM regions;
```

</details>

<details><summary>Solution 10:</summary>

```SQL
CREATE OR REPLACE  PROCEDURE NUKE_REGION(id NUMBER) AS 
  howMany number;
BEGIN
  --see if any exist
  select count(*) into howMany from regions where region_id = id;
  --if not, let them know
  IF howMany = 0 THEN DBMS_OUTPUT.PUT_LINE('Nothing to delete');
  ELSE DELETE FROM regions WHERE region_id=id;
     --commit'
     DBMS_OUTPUT.PUT_LINE(id || ' deleted');
  END IF;
END;
```

Executing the procedure:
```SQL
EXECUTE NUKE_REGION(666);
```

Checking the procedure:
```SQL
SELECT * FROM regions;
```

</details>
