# Section 3 

## Packages and Debugging

### Packages
A package is a mechanism to help organize functions and procedures. They consist of two parts:
- a specification (think interface)
- a body (think implementation)


> A package containing one procedure that prints the provided message.

```SQL
CREATE OR REPLACE PACKAGE hello_world_pkg AS
   PROCEDURE printMessage(sMessage varchar2);   
END hello_world_pkg;
/

CREATE OR REPLACE PACKAGE BODY hello_world_pkg AS
   PROCEDURE printMessage(sMessage varchar2) IS
   BEGIN
     dbms_output.put_line(sMessage);
   END printMessage;   
END hello_world_pkg;
/
```

> Call the procedure in the package to print hello world

```SQL
SET serveroutput ON;
CALL hello_world_pkg.printMessage('Hello World!');
```

:bulb: Note that the package name is added before the procedure name. Without this Oracle will not be able to identify the package.

### Compilation Dependencies
The package can be compiled directly from the GUI (right-click and compile) or via the command line. 
- In the SQLDeveloper GUI, an invalid object shows up as :x: (a red cross)
- In the dictionary query results, it will show up as having a status of INVALID. 

> To find invalid objects in the current user's schema that are functions, procedures or packages, execute the query below.

```SQL
SELECT * 
FROM USER_OBJECTS WHERE OBJECT_TYPE 
IN ('FUNCTION','PROCEDURE','PACKAGE') AND STATUS <> 'VALID';
```

### Debugging :question: 
