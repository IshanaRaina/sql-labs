# Section 6

## Triggers
A trigger is a piece of code that can be executed in response to any of the following events:
- A DML statement (insert, update, delete)
- A DDL statement (create, alter, drop)
- A database operation (servererror, logon, logoff, startup, shutdown)

A simple trigger is one that is fired at just one of the following timing points:
- Before the statement is executed
- After the statement is executed
- Before each row that the statement affects is modified
- After each row that the statement affects is modified

### Compound Trigger
A compound trigger is one that fires at more than one timing event. 

> General syntax for a compound trigger:

```SQl
CREATE OR REPLACE TRIGGER <trigger name>
FOR <insert|update|delete> <of column_name> ON <tablename>
COMPOUND TRIGGER
<declare section>
BEFORE
<before section>
BEFORE EACH ROW
<before each row section>
AFTER EACH ROW
<after each row section>
AFTER
<after section>
END;
```

### Firing Condition
The firing condition is specified when the trigger is created. 

When firing due to a DML change, a trigger has access to the values in the row before and after the change by using the `:NEW` and `:OLD` prefixes on a column name. 

> Create a trigger that fires whenever a row on **API_ITEM** is updated and writes a line to the output with the new and old prices.

```SQL
CREATE OR REPLACE TRIGGER log_item_changes
  BEFORE UPDATE ON api_item
  FOR EACH ROW
DECLARE
BEGIN
  IF :old.price <> :new.price THEN
    DBMS_OUTPUT.PUT_LINE('old price :' || :old.price || ' new price: ' || :new.price);
  END IF;
END;
```

:book: [Read more](https://docs.oracle.com/database/121/TDDDG/tdddg_triggers.htm#TDDDG50000) about triggers. 

## Exercises :computer: 

This exercise entails creating a trigger for the purpose of auditing salary changes. 

1. Create a table to store the audit data - we are interested in who made the change, when they made the change, old salary, new salary and employee ID of the person whose salary was changed. 
2. Next, create a trigger that fires whenever an employee's salary gets updated. 
3. Next, check to see if the trigger works to correctly populate the audit table.

<details><summary>Solution:</summary>

Step 1: 

```SQL
CREATE TABLE SPY_TBL (
  WHO VARCHAR2(50), WHEN_DONE TIMESTAMP, OLD_SAL NUMBER(8,2) , NEW_SAL NUMBER(8,2) ,
  EMP_ID NUMBER(6,0));
```

Step 2:

```SQL
CREATE OR REPLACE TRIGGER SPY_TRIGGER
BEFORE UPDATE OF SALARY ON EMPLOYEES
FOR EACH ROW
BEGIN
--Check to see if the salary has changed
  IF :OLD.salary <> :NEW.salary THEN
-- USER is current login, CURRENT_TIMESTAMP is current date/time
    INSERT INTO SPY_TBL (WHO, WHEN_DONE, OLD_SAL, NEW_SAL, EMP_ID) 
    VALUES (USER, CURRENT_TIMESTAMP, :OLD.SALARY, :NEW.SALARY, :NEW.EMPLOYEE_ID);
  END IF;
END;
```

Step 3:

```SQL
UPDATE EMPLOYEES SET SALARY=6004 WHERE EMPLOYEE_ID=104;
```

```SQL
SELECT * FROM SPY_TBL;
```

</details>
