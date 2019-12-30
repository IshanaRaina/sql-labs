# Section 5

## Dynamic SQL
Dynamic SQL allows for the generation and execution of SQL statements at runtime. 

The following are some of the things that can be determined dynamically:

- The SQL statements to be executed
- The tables involved
- The columns involved
- The parameters involved
- An anonymous PL/SQL block
- A call to a different function or procedure
- Execute DDL
- Execute DML

Oracle provides two methods to perform dynamic SQL: 
- Native Dynamic SQL 
- DBMS_SQL package

### 1. Native Dynamic SQL
Native Dynamic SQL uses a keyword known as `EXECUTE IMMEDIATE`. The exact choice of syntax depends on the number of expected output rows as a result of executing the statement. In general, output results can be written to variables using the `INTO` keyword and parameters can be passed in using the `USING` keyword. 

> Create a function which given a table name returns a count for the number
of rows in the table.

```SQL
CREATE OR REPLACE FUNCTION getCountForTable(sTableName varchar2) 
RETURN number
IS
  sSQL varchar2(4000);
  iCount number;
BEGIN
   sSQL := 'select count(*) as cnt from ' || sTableName;
   EXECUTE IMMEDIATE sSQL INTO iCount;
   RETURN iCount;
END getCountForTable;
```

> To test the function, execute the query:
```SQL
SELECT getCountForTable('API_CUSTOMER') FROM dual;
```

> Create a function that counts the number of tables owned by a user. Note the use of `USING` to pass in a bind variable.

```SQL
CREATE OR REPLACE FUNCTION getTableCountForOwner(sOwnerName varchar2) 
RETURN number
IS
  sSQL varchar2(4000);
  iCount number;
BEGIN
   sSQL := 'select count(*) as cnt from all_tables where owner = upper(:1)';
   EXECUTE IMMEDIATE sSQL into iCount using sOwnerName;
   RETURN iCount;
END getTableCountForOwner;
```

:bulb: In general, if you can think of a string to do it, you can run it with `EXECUTE IMMEDIATE`.

> Use execute immediate to execute an anonymous PL/SQL block. Note the use of double single quotes due to the need to escape single quotes with single quotes.

```SQL
CREATE OR REPLACE PROCEDURE runBlock 
IS
  sSQL varchar2(4000);
BEGIN
   sSQL := 'begin dbms_output.put_line(''Hello!'');end;';
   EXECUTE IMMEDIATE sSQL;
END runBlock;
```

### 2. DBMS_SQL Package
DBMS_SQL differs from most Oracle packages in several ways:
- DBMS_SQL uses bind by value.
- You must call COLUMN_VALUE after fetching rows for the values to become available.
- You must call VARIABLE_VALUE to retrieve the value of an OUT parameter for an anonymous block.
- Enhanced security protections to prevent various attacks.

To use DBMS_SQL, you make a series of calls to the various procedures in the package. The methods called, the order and the data provided will determine the outcome.

> Delete the specified number of rows from the specified table using DBMS_SQL.

```SQL
CREATE OR REPLACE PROCEDURE deleteRows(sTableName varchar2, iDeleteNumberOfRows number) 
    as cursor_name integer;
    rows_processed integer;
    sSQL varchar2(4000);
BEGIN
    sSQL := 'delete from ' || sTableName || ' where rownum <= :x';
    cursor_name := dbms_sql.open_cursor;
    dbms_sql.parse(cursor_name, sSQL, dbms_sql.NATIVE);
    dbms_sql.bind_variable(cursor_name, ':x', iDeleteNumberOfRows);
    rows_processed := dbms_sql.execute(cursor_name);
    dbms_sql.close_cursor(cursor_name);
EXCEPTION
  WHEN OTHERS THEN
    dbms_sql.close_cursor(cursor_name);
END deleteRows;
```

:book: [Read more](https://docs.oracle.com/cd/B10501_01/appdev.920/a96590/adg09dyn.htm) about Dynamic SQL.

## Transactions
Whenever DML changes are made in PL/SQL they are done as part of a transaction. The changes will not be visible outside of the transaction until they are committed. If PL/SQL is called from code that is already part of a transaction then, by default, all changes made by that PL/SQL code will be part of the same transaction. When you commit or rollback in PL/SQL you are committing or rolling back all of the changes made as part of that transaction.

There are two options that provide finer control over transaction behavior: 
- autonomous transactions
- savepoints

### 1. Autonomous Transactions
An autonomous transaction is an independent transaction that can be performed within another transaction. 

To mark a code block as having an autonomous transaction use the `PRAGMA AUTONOMOUS_TRANSACTION` directive.

> Make a delete procedure an autonomous transaction.

```SQL
CREATE OR REPLACE PROCEDURE deleteFromTable(sTableName varchar2) 
IS 
PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
   EXECUTE IMMEDIATE 'delete from ' || sTableName;
   COMMIT;
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
END deleteFromTable;
```

### 2. Savepoints
Oracle provides a mechanism to avoid having to rollback all of the changes if there is an error. Declare a savepoint, give it a name, and Oracle will record the changes made up to that point. If there is an error you can choose to rollback all of the changes or only those changes made after the savepoint. Oracle allows multiple savepoints within a transaction. 

> Create multiple savepoints and rollback to a specific one.

```SQL
BEGIN
  UPDATE api_customer SET first_name = 'Dolly';
  
  SAVEPOINT first_name_updated;

  UPDATE api_customer SET last_name = 'The Clone';
  
  SAVEPOINT last_name_updated;

  UPDATE api_customer SET email = 'DollyTC@gmailfake.com';
  
  ROLLBACK TO first_name_updated;
  COMMIT;
END;
```
:bulb: Had we just executed rollback, it would have rolled back everything.

:book: [Read more](https://docs.oracle.com/database/121/CNCPT/transact.htm#CNCPT016) about transactions.

## Custom Datatypes
Oracle provides a mechanism to create user defined types to extend the SQL language. A type is composed of three parts:

- A name to identify the type.
- Attributes (data elements). Note that these can be other types.
- Methods - These methods are functions and procedures and can be used to provide access to the attributes or be used to perform other functionality.

> Create a type that holds a pair of data elements. One attribute will be called row_id and the other row_type. This type could be used when a function needs to return references that point to rows in more than one table. In that case we need to know the primary key value and the table it is for.

```SQL
CREATE TYPE api_row_reference AS OBJECT
    ( row_id        NUMBER(22)
    , row_type      VARCHAR2(30)
    ) ;
```

> Add two functions to the type from the prior example. Note that the functions can interact with the attributes just by referencing the name.

```SQL
CREATE TYPE api_row_reference AS OBJECT 
   ( row_id        NUMBER(22),
     row_type      VARCHAR2(30), 
     MEMBER FUNCTION isOfType(sType VARCHAR2) RETURN NUMBER,
     MEMBER FUNCTION toString RETURN VARCHAR2 
   ); 
```

:book: [Read more](https://docs.oracle.com/cd/A91202_01/901_doc/server.901/a88856/c14ordb.htm) about custom data types.
