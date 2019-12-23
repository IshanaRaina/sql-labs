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
