# Section 2

## Joins

So far, we've queried one table at a time but what if we want to retrieve data that lies in more than one table? Joins allow us to access data that resides in more than one table **inside of a single statement**.

> For example, to retrieve all items ordered by John Doe in one query we can execute the following:

```SQL
SELECT C.LAST_NAME, COI.ITEM_COUNT, I.ITEM_ID, I.SKU, I.NAME 
FROM API_CUSTOMER C JOIN API_CUSTOMER_ORDER CO 
ON CO.CUSTOMER_ID = C.CUSTOMER_ID
JOIN API_CUSTOMER_ORDER_ITEM COI 
ON COI.CUSTOMER_ORDER_ID = CO.CUSTOMER_ORDER_ID
JOIN API_ITEM I 
ON I.ITEM_ID = COI.ITEM_ID
WHERE C.LAST_NAME = 'Doe' AND C.FIRST_NAME = 'John';
```

The above query joins four tables, retrieves some columns from each table and filters the results to only include the ones for John Doe.

:bulb: Note that:
- The column names in the query are slightly different from the way you're used to seeing them. This is because data is being retrieved from multiple tables and that's why SQL needs to be told which table the column belongs to so it can retrieve the correct column. 
    > For example, `C.LAST_NAME, COI.ITEM_COUNT` is telling SQL to retrieve the `LAST_NAME` column from the table **C** (which we know to be **API_CUSTOMER** thanks to its alias) and the `ITEM_COUNT` column from the table **COI** (which we know to be **API_CUSTOMER_ORDER_ITEM** thanks to its alias).
- As stated above the query, the syntax for `JOIN` is pretty straightforward. We first specify the 2 tables that we'd like joined separated by the keyword `JOIN`. 
    > For example, `FROM API_CUSTOMER C JOIN API_CUSTOMER_ORDER CO`
- Next, we tell SQL which columns it should perform the join operation on. 
    > For example, `ON CO.CUSTOMER_ID = C.CUSTOMER_ID`
- When we use the keyword `JOIN` in our queries, it defaults to an **inner join**.

### Alternative Join

A common alternative used in inner joins involves skipping the join keyword altogether and just listing all of the tables in the `FROM` clause, with the join conditions listed as additional `WHERE` clause conditions.

> For example, the query from the previous example can be re-written as:

```SQL
SELECT C.LAST_NAME, COI.ITEM_COUNT, I.ITEM_ID, I.SKU, I.NAME
FROM API_CUSTOMER C, API_CUSTOMER_ORDER CO, API_CUSTOMER_ORDER_ITEM COI, API_ITEM I
WHERE CO.CUSTOMER_ID = C.CUSTOMER_ID AND
COI.CUSTOMER_ORDER_ID = CO.CUSTOMER_ORDER_ID AND
I.ITEM_ID = COI.ITEM_ID AND
C.LAST_NAME = 'Doe' AND C.FIRST_NAME = 'John';
```

The above is generally regarded as the less desirable approach - it is difficult to tell at a glance what the join conditions are and they are interspersed with other conditions to filter data.


### Inner Join

An inner join is one in which values from the two columns must exist and satisfy the join condition for either to be returned. If either side is missing the value, the other side will be not be included in the result set. 

> For example, on joining **API_CUSTOMER** to **API_CUSTOMER_ORDER** with an inner join on `CUSTOMER_ID`, if a customer does not have any orders, that customer will not appear in the query results.
> 
:bulb: Before diving into the exercise, note that the join conditions used in the next examples are based on the equality operator, but other operators are also allowed. We can use `<=` or `<>` (to list a few) instead of `=` in the join condition of one of the next example queries. This is fairly uncommon, but it is needed on occasion.

**Exercise 1** :computer: 

Access the **Human Resources** database :arrow_forward: 

1. List the employee first and last names along with their job titles.
2. List the employee first and last names along with their job titles for all employees whose last name is 'King'.
3. List employee names, jobs, salary and max_salary only if those employees earn the max_salary or more.
4. List department name, first and last name and job titles using the alternative join method that you saw above.
5. Access the **Pet Store** database :arrow_forward: List the orders from largest to smallest and also show the first and last name of the customer who placed that order. 

<details><summary>Solution 1:</summary>

```SQL
SELECT FIRST_NAME, LAST_NAME, JOB_TITLE
FROM EMPLOYEES INNER JOIN JOBS 
ON EMPLOYEES.JOB_ID = JOBS.JOB_ID;
```
</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT FIRST_NAME AS F, LAST_NAME, JOB_TITLE
FROM EMPLOYEES E JOIN JOBS J
ON E.Job_ID = J.Job_ID
WHERE LAST_NAME = 'King';
```
</details>

<details><summary>Solution 3:</summary>

```SQL
SELECT FIRST_NAME, LAST_NAME, JOB_TITLE, SALARY, MAX_SALARY
FROM EMPLOYEES E JOIN JOBS J
ON E.JOB_ID = J.JOB_ID 
WHERE E.SALARY >= J.MAX_SALARY;
```
</details>

<details><summary>Solution 4:</summary>

```SQL
SELECT DEPARTMENT_NAME, FIRST_NAME, LAST_NAME, JOB_TITLE
FROM DEPARTMENTS D, EMPLOYEES E, JOBS J
WHERE D.DEPARTMENT_ID = E.DEPARTMENT_ID
AND E.JOB_ID = J.JOB_ID;
```
</details>

<details><summary>Solution 5:</summary>

```SQL
SELECT C.LAST_NAME, C.FIRST_NAME, CO.* 
FROM API_CUSTOMER C JOIN API_CUSTOMER_ORDER CO 
ON C.CUSTOMER_ID = CO.CUSTOMER_ID 
ORDER BY TOTAL_PRICE DESC, LAST_NAME ASC, FIRST_NAME ASC;
```
</details>

### Outer Join

An outer join is one in which rows from one table will be returned even if no match exists for the join condition on the other table. 

There are three types of outer joins:
- Left
- Right
- Full

#### 1. Left Outer Join

Left outer joins will join tables A and B and return all rows in A whether they have a match in B or not.
- If a match in B is present, then it will return the corresponding column values from B. 
- If there is no match in B will will return null for all B columns.

> For example, to left outer join **API_ITEM** to **API_PACKAGE_TYPE**:

```SQL
SELECT I.*, PT.* 
FROM API_ITEM I LEFT OUTER JOIN API_PACKAGE_TYPE PT 
ON I.PACKAGE_TYPE_ID = PT.PACKAGE_TYPE_ID;
```

:bulb: `LEFT OUTER JOIN` can also be written as `LEFT JOIN`. SQL understands that an outer join needs to be performed.

**Exercise 2** :computer: 

Access the **Human Resources** database :arrow_forward: 
1. List departments which have no employees. 
2. List employees that don't belong to any department.
3. List employees that don't belong to any department without using a join. :wink: 
4. List the department name, first and last name and job titles of those employees whose first name is `NULL`. Replace the `NULL` first names with 'Harvey' in the result.
5. List departments along with their manager's full names. 
6. Access the **Pet Store** database :arrow_forward: List all items that have never been ordered. 

<details><summary>Solution 1:</summary>

```SQL
SELECT DEPARTMENT_NAME
FROM DEPARTMENTS d LEFT JOIN EMPLOYEES E
ON D.DEPARTMENT_ID = E.DEPARTMENT_ID
WHERE E.EMPLOYEE_ID IS NULL;
```
</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT E.FIRST_NAME, E.LAST_NAME
FROM EMPLOYEES E LEFT JOIN DEPARTMENTS D
ON D.DEPARTMENT_ID = E.DEPARTMENT_ID
WHERE E.DEPARTMENT_ID IS NULL;
```
</details>

<details><summary>Solution 3:</summary>

```SQL
SELECT FIRST_NAME, LAST_NAME 
FROM EMPLOYEES 
WHERE DEPARTMENT_ID IS NULL;
```
</details>

<details><summary>Solution 4:</summary>

```SQL
SELECT DEPARTMENT_NAME, NVL(FIRST_NAME,'Harvey'), LAST_NAME, JOB_TITLE
FROM DEPARTMENTS D LEFT JOIN EMPLOYEES E 
ON D.DEPARTMENT_ID = E.DEPARTMENT_ID
LEFT JOIN JOBS J
ON E.JOB_ID = J.JOB_ID
WHERE FIRST_NAME IS NULL;
```
</details>

<details><summary>Solution 5:</summary>

```SQL
SELECT DEPARTMENT_NAME, FIRST_NAME || ' ' || LAST_NAME  AS FULLNAME
FROM DEPARTMENTS D LEFT JOIN EMPLOYEES E
ON D.MANAGER_ID = E.EMPLOYEE_ID
WHERE FIRST_NAME IS NOT NULL;
```
</details>

<details><summary>Solution 6:</summary>

```SQL
SELECT A.* FROM API_ITEM A 
LEFT JOIN API_CUSTOMER_ORDER_ITEM B
ON A.ITEM_ID = B.ITEM_ID
WHERE B.ITEM_ID IS NULL;
```
</details>

#### 2. Right Outer Join

Right outer joins will join tables A and B and return all rows in B whether they have a match in A or not. 
- If a match in A is present, then it will return the corresponding column values from A. 
- If there is no match in A will will return null for all A columns.

> For example, to right outer join **API_ITEM** to **API_PACKAGE_TYPE**:

```SQL
SELECT I.*, PT.* 
FROM API_ITEM I RIGHT OUTER JOIN API_PACKAGE_TYPE PT 
ON I.PACKAGE_TYPE_ID = PT.PACKAGE_TYPE_ID;
```

**Exercise 3** :computer: 

Access the **Pet Store** database :arrow_forward: Who let the dogs :dog2: out?! Oops, I meant, who bought the dog food'?! List the item name (ie 'API Dog Food') and customer name.

<details><summary>Solution:</summary>

```SQL
SELECT DISTINCT I.NAME AS "ITEM NAME", C.FIRST_NAME, C.LAST_NAME 
FROM API_ITEM I RIGHT JOIN API_CUSTOMER_ORDER_ITEM OI
ON I.ITEM_ID = OI.ITEM_ID
RIGHT JOIN API_CUSTOMER_ORDER O
ON OI.CUSTOMER_ORDER_ID = O.CUSTOMER_ORDER_ID
RIGHT JOIN API_CUSTOMER C
ON O.CUSTOMER_ID = C.CUSTOMER_ID
WHERE I.NAME = 'API Dog Food';
```
</details>

#### 3. Full Outer Join

Full outer joins will join tables A and B and return all rows in A whether they have a match in B or not and all rows in B whether they have a match in A or not. 
- When there is a match, the corresponding column values from A and B will appear. 
- When one of the tables is missing, the columns associated to that table will be `NULL`.

> For example, to full outer join **API_ITEM** to **API_PACKAGE_TYPE**:

```SQL
SELECT I.*, PT.* 
FROM API_ITEM I FULL OUTER JOIN API_PACKAGE_TYPE PT 
ON I.PACKAGE_TYPE_ID = PT.PACKAGE_TYPE_ID;
```

**Exercise 4** :computer: 

Access the **Human Resources** database :arrow_forward: 
1. List all employee first and last names along with all their corresponding department names and order the results in descending order of department.
2. List all employees first and last names along with all their corresponding job titles where job title is `NULL`.

<details><summary>Solution 1:</summary>

```SQL
SELECT  FIRST_NAME, LAST_NAME, DEPARTMENT_NAME
FROM DEPARTMENTS D FULL JOIN EMPLOYEES E
ON D.DEPARTMENT_ID = E.DEPARTMENT_ID
ORDER BY DEPARTMENT_NAME DESC;
```
</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT FIRST_NAME, LAST_NAME, JOB_TITLE
FROM EMPLOYEES FULL JOIN JOBS
ON EMPLOYEES.JOB_ID = JOBS.JOB_ID
WHERE JOBS.JOB_ID IS NULL;
```
</details>

### Self Join

As the name suggests, a self join is a join in which a given table is joined with itself. 

**Exercise 5** :computer: 

Access the **Human Resources** database :arrow_forward: 
1. List names of all managers and their employees, ordered by manager.
2. Let's take it one step further - list employee's full name, the department they belong to followed by their manager's full name.
3. Let's take it another step further! List employee's full name, employee's manager name, department name and the department manager's name.

<details><summary>Solution 1:</summary>

```SQL
SELECT MGR.FIRST_NAME MGR, EMP.FIRST_NAME EMP
FROM EMPLOYEES EMP JOIN EMPLOYEES MGR
ON EMP.MANAGER_ID = MGR.EMPLOYEE_ID
ORDER BY MGR;
```
</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT EMP.FIRST_NAME || ' ' || EMP.LAST_NAME AS EMPLOYEE, 
    DEPARTMENT_NAME, 
    MGR.FIRST_NAME || ' ' || MGR.LAST_NAME AS MANAGER
FROM EMPLOYEES EMP JOIN DEPARTMENTS DEPT
ON EMP.DEPARTMENT_ID = DEPT.DEPARTMENT_ID
JOIN EMPLOYEES MGR
ON DEPT.MANAGER_ID = MGR.EMPLOYEE_ID;
```
</details>

<details><summary>Solution 3:</summary>

```SQL
SELECT E.FIRST_NAME || ' ' || E.LAST_NAME AS EMP, 
    M.FIRST_NAME || ' ' || M.LAST_NAME AS EMPMGR, DEPARTMENT_NAME, 
    DM.FIRST_NAME || ' ' || DM.LAST_NAME AS DEPTMGR, E.SALARY
FROM EMPLOYEES E LEFT JOIN EMPLOYEES M ON E.MANAGER_ID = M.EMPLOYEE_ID
LEFT JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
LEFT JOIN EMPLOYEES DM ON DM.EMPLOYEE_ID = D.MANAGER_ID;
```
</details>


## Set Operations
There are operators that allow us to perform set operations on the results of queries.

### Union
The set of distinct (no repeats) rows from the union of the two result sets.

> For example, the following query contains no repeats due to the operator being `UNION`.

```SQL
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER 
UNION
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER 
WHERE LAST_NAME = 'Doe';
```

**Exercise 6** :computer: 

Access the **Human Resources** database :arrow_forward: List all salaries (ie employee salary, min salary and max salary) in a single column.

<details><summary>Solution:</summary>

```SQL
SELECT SALARY FROM EMPLOYEES
UNION
SELECT MIN_SALARY FROM JOBS
UNION
SELECT MAX_SALARY FROM JOBS;
```

</details>

### Union All
The set of rows from the union of the two result sets, allowing repeats.

> For example, the following query contains repeats due to the operator being `UNION ALL`.

```SQL
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER
UNION ALL
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER 
WHERE LAST_NAME = 'Doe';
```

**Exercise 7** :computer: 

1. Access the **Human Resources** database :arrow_forward: List the names of all employees followed by the names of all countries in a single column.
2. Access the **Pet Store** database :arrow_forward: List first names of all customers whose last name is 'Johnson' and the last names of all customers whose last name is not 'Johnson', in a single column.

<details><summary>Solution 1:</summary>

```SQL
SELECT FIRST_NAME FROM EMPLOYEES
UNION ALL
SELECT COUNTRY_NAME FROM COUNTRIES;
```
</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT FIRST_NAME FROM API_CUSTOMER
WHERE LAST_NAME = 'Johnson'
UNION ALL
SELECT LAST_NAME FROM API_CUSTOMER
WHERE LAST_NAME <> 'Johnson';
```
</details>

### Intersect
The intersection between the two result sets.

> For example, the following query contains the intersection of the two result sets.

```SQL
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER
INTERSECT
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER 
WHERE LAST_NAME = 'Doe';
```

### Minus
The first result set after removing all rows found in the second result set.

> For example, the following query contains the results of the first query that are not in the second one.

```SQL
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER
MINUS
SELECT FIRST_NAME, LAST_NAME FROM API_CUSTOMER WHERE LAST_NAME = 'Doe';
```

:bulb: The set operators can be used more than once in the query. Oracle will apply precedence rules, however, it is strongly recommended that you use parentheses when the query could be confusing.

### Exists (Semi Join)

Sometimes we don't really care about the data in one of the tables, we just want to retrieve all the rows in A that have a matching row in B. In those cases it is better to use an `EXISTS` statement than one of the other join variations. This gives the Oracle optimizer more information to work with and also ensures that the cardinality of the result set matches the number of returned rows from A.

> For example, in order to retrieve all items that appear in at least one order, we run a semi-join from **API_ITEM** to **API_CUSTOMER_ORDER_ITEM**:

```SQl
SELECT I.* FROM API_ITEM I
WHERE EXISTS (
SELECT 1 FROM API_CUSTOMER_ORDER_ITEM COI WHERE COI.ITEM_ID = I.ITEM_ID
);
```

:book: [Read more](http://www.techonthenet.com/oracle/exists.php) about `EXISTS`.

### In (Semi Join)

Sometimes we want to return those rows where a column's value belongs to a set of values (with more than one value). This can often be done with a series of `OR` clauses, but that can get very messy and if the list needs to be determined at runtime, it will not work at all. 

A better solution is to use the `IN` condition. 

- An `IN` condition allows you to specify a set of hard-coded values that the column must have to meet.
- You can also avoid hard-coding and use the results of a query. This is risky, because if the number of results exceeds a limit (configurable, but defaulted by Oracle to 1000) an error will be thrown during query execution. When using an IN condition in this way, consider replacing it with an `EXISTS` query (or changing the query in some other way).

> For example, in order to retrieve all items named 'API Dog Food' or 'API Cat Food', we run a semi-join on **API_ITEM** using an `IN` condition:

```SQL
SELECT * FROM API_ITEM WHERE NAME IN ('API Dog Food', 'API Cat Food');
```

> Another example, in order to retrieve all items that appear in at least one order, we run a semi-join from **API_ITEM** to **API_CUSTOMER_ORDER_ITEM** using an `IN` condition: 

```SQL
SELECT I.* FROM API_ITEM I
WHERE I.ITEM_ID IN (SELECT ITEM_ID FROM API_CUSTOMER_ORDER_ITEM COI);
```

:skull: Note that this is an example of how `IN` should **not** be used. The example in the `EXISTS` semi-join is the preferred alternative.

:books: Read more about `IN`:
- [Synatx and examples](http://www.techonthenet.com/oracle/in.php)
- [Flow chart](http://docs.oracle.com/cd/B19306_01/server.102/b14200/conditions013.htm)

### Not Exists (Anti Join)

Sometimes we need to find all the rows in A for which there is not a matching row in B. 
We can do this by adding a `NOT` operator in front of an `EXISTS` query.

> For example, in order to retrieve all items that have never been ordered, we run an anti-join from **API_ITEM** to **API_CUSTOMER_ORDER_ITEM**:

```SQL
SELECT I.* FROM API_ITEM I
WHERE NOT EXISTS (
SELECT 1 FROM API_CUSTOMER_ORDER_ITEM COI WHERE COI.ITEM_ID = I.ITEM_ID
);
```

### Not In (Anti Join)

Sometimes we want to return those rows where a column's value is not in a specified set of values. This can be done by adding the `NOT` operator in front of an `IN` condition. The same warnings that applied to the `IN` condition apply to `NOT IN` (the alternative is `NOT EXISTS`).

> For example, in order to retrieve all items other than 'API Dog Food' or 'API Cat Food', we run an anti-join on **API_ITEM** using a `NOT IN` condition:

```SQL
SELECT * FROM API_ITEM WHERE NAME NOT IN ('API Dog Food', 'API Cat Food');
```

:book: [Read more](http://docs.oracle.com/cd/B28359_01/server.111/b28286/queries006.htm) about `JOINS` in all their glory! :bomb:

## Unique values

### Select Distinct (Unique Values)

In a table, a column may contain many duplicate values and sometimes you only want to list the different (distinct) values. The `DISTINCT` keyword is used to return only distinct (different) values.

> For example, the syntax would look like:

`SELECT DISTINCT column_name,column_name FROM table_name;`

### Select DistinctRow (Unique Records)

DISTINCTROW, on the other hand, checks all fields in the table that are being queried, and eliminates duplicates based on the entire record (not just the selected fields). Results of DISTINCTROW queries are updateable. 

**Exercise 8** :computer: 

Access the **Human Resources** database :arrow_forward: 

1. List the department names and the number of distinct managers in that department. 

<details><summary>Solution 1:</summary>

```SQL
SELECT DEPARTMENT_NAME, COUNT(DISTINCT E.MANAGER_ID) AS MGRCOUNT
FROM EMPLOYEES E JOIN DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
GROUP BY DEPARTMENT_NAME;
```

</details>

## Aggregate functions

Aggregate Function | Description
--- | ---
MIN | Returns the smallest value in a given column
MAX | Returns the largest value in a given column
SUM | Returns the sum of the numeric values in a given column
AVG | Returns the average value of a given column
COUNT | Returns the total number of values in a given column
COUNt(*) | Returns the number of rows in a table
FIRST | Returns the first value
LAST | Returns the last value

> For example, listing out the biggest salary from the **API_EMPLOYEE** table would be as follows:

```SQL
SELECT MAX(SALARY) AS "BIGGEST SALARY" FROM API_EMPLOYEE;
```

:book: [Read more](https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions003.htm#SQLRF20035) about aggregate functions.

**Exercise 9** :computer: 

Access the **Human Resources** database :arrow_forward: 

1. Use as many aggregate functions as you can on the **Employees** table. Get creative!

<details><summary>Solution 1:</summary>

```SQL
SELECT COUNT(*) as NumEmp, ROUND(AVG(SALARY)) as AvgSal, COUNT(MANAGER_ID) Minions, 
  MAX(SALARY) as Rich, MIN(SALARY) as Poor, DEPARTMENT_ID
FROM EMPLOYEES
GROUP BY DEPARTMENT_ID
HAVING COUNT(*) > 3;
```

</details>

## Group by & Having

### Group by clause 

The `GROUP BY` clause will group the results into subsets determined by the distinct combinations of columns being grouped on. Each such subset will become one row. This is typically used in combination with aggregate expressions. Each aggregate function will return a single value.

The `GROUP BY` expression is placed after the `WHERE` clause and before the `ORDER BY` clause. 

> For example, the query below returns the total amount ordered by customers that are not named 'Johnson'.

```SQL
SELECT C.LAST_NAME, C.FIRST_NAME, SUM(CO.TOTAL_PRICE)
FROM API_CUSTOMER C JOIN API_CUSTOMER_ORDER CO 
ON C.CUSTOMER_ID = CO.CUSTOMER_ID
WHERE C.LAST_NAME <> 'Johnson'
GROUP BY C.LAST_NAME, C.FIRST_NAME
ORDER BY C.LAST_NAME ASC, C.FIRST_NAME ASC;
```

:book: [Read more](https://docs.oracle.com/javadb/10.8.3.0/ref/rrefsqlj32654.html) about `GROUP BY`.

**Exercise 10** :computer: 

Access the **Pet Store** database :arrow_forward:

1. List the biggest salaries by state.
2. List the average bonuses by title. 
3. List the average salary of employees by manager. 
4. Calculate the total price for an order. **Hint**: Multiply price by count to get the total price. 
5. Build upon the previous query. In addition, to the total price, list the order ID, the customer name who placed the order as well as the number of itemlines that made up the order. 
6. Display all the relevant aggregate functions for salary along with first and last names. Group the results by first and last name and for ease of readability, order them in ascending order of first and last name.

<details><summary>Solution 1:</summary>

```SQL
SELECT STATE_CODE AS "STATE", MAX(SALARY) AS "BIGGEST SALARY" 
FROM API_EMPLOYEE
GROUP BY STATE_CODE;
```

</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT TITLE, AVG(BONUS_PERCENT) AS "BONUS" 
FROM API_EMPLOYEE
GROUP BY TITLE 
ORDER BY BONUS DESC;
```

</details>

<details><summary>Solution 3:</summary>

```SQL
SELECT AVG(EMP.SALARY) AS AVG_SALARY, MGR.FIRST_NAME , MGR.LAST_NAME
FROM API_EMPLOYEE EMP INNER JOIN API_EMPLOYEE MGR
ON EMP.MANAGER_ID = MGR.EMPLOYEE_ID
GROUP BY MGR.FIRST_NAME,MGR.LAST_NAME
ORDER BY AVG_SALARY DESC;
```

</details>

<details><summary>Solution 4:</summary>

```SQL
SELECT OI.CUSTOMER_ORDER_ID, SUM(OI.ITEM_COUNT * I.PRICE) AS "TOTAL"
FROM API_CUSTOMER_ORDER_ITEM OI INNER JOIN API_ITEM I
ON OI.ITEM_ID = I.ITEM_ID
GROUP BY OI.CUSTOMER_ORDER_ID;
```

</details>

<details><summary>Solution 5:</summary>

```SQL
SELECT 
    C.FIRST_NAME || ' ' || C.LAST_NAME as "Customer", 
    OI.Customer_Order_ID as "OrderNum", 
    SUM(OI.ITEM_COUNT * I.PRICE) as "Total", 
    COUNT(*) as ItemLines
FROM API_ITEM I INNER JOIN API_CUSTOMER_ORDER_ITEM OI
ON I.ITEM_ID = OI.ITEM_ID
INNER JOIN API_CUSTOMER_ORDER O
ON OI.CUSTOMER_ORDER_ID = O.CUSTOMER_ORDER_ID
INNER JOIN api_customer C
ON O.CUSTOMER_ID = C.CUSTOMER_ID
GROUP BY OI.CUSTOMER_ORDER_ID, C.FIRST_NAME, C.LAST_NAME;
```

</details>

<details><summary>Solution 6:</summary>

```SQL
SELECT 
   SUM(SALARY) AS "TOTAL_SALARY",
   AVG(SALARY) AS "AVG_SALARY",
   MIN(SALARY) AS "MIN_SALARY",
   MAX(SALARY) AS "MAX_SALARY",
   COUNT(SALARY) AS "SALARY_COUNT",
   FIRST_NAME, LAST_NAME  --  NOT AGGREGATE FN SO MUST BE IN GROUP BY
FROM API_EMPLOYEE
GROUP BY FIRST_NAME,LAST_NAME
ORDER BY FIRST_NAME,LAST_NAME;
```

</details>

### Having clause 

Used after the `GROUP BY` clause, the keyword `HAVING` allows us to put filter conditions on aggregate functions. 

:warning: SQL does **not** allow `WHERE` to be used with aggregate functions. 

> For example, the query below refines the previous example by adding the count of the number of ordered items and only returning data for customers that have ordered over $50.

```SQL
SELECT C.LAST_NAME, C.FIRST_NAME, SUM(CO.TOTAL_PRICE), SUM(COI.ITEM_COUNT)
FROM API_CUSTOMER C JOIN API_CUSTOMER_ORDER CO 
ON C.CUSTOMER_ID = CO.CUSTOMER_ID
JOIN API_CUSTOMER_ORDER_ITEM COI 
ON CO.CUSTOMER_ORDER_ID = COI.CUSTOMER_ORDER_ID
WHERE C.LAST_NAME <> 'Johnson'
GROUP BY C.LAST_NAME, C.FIRST_NAME
HAVING SUM(CO.TOTAL_PRICE) >= 50
ORDER BY C.LAST_NAME ASC, C.FIRST_NAME ASC;
```

:book: [Read more](https://docs.oracle.com/javadb/10.8.3.0/ref/rrefsqlj14854.html) about `HAVING`.

**Exercise 11** :computer: 
1. Access the **Pet Store** database :arrow_forward: List the states where the average salary is greater than $100,000. Also list the average salary amount.

Access the **Human Resources** database :arrow_forward: 

2. How many people were hired on each date? List only those dates where 2 or more people were hired.
3. Which managers have employees who average greater than 10,000 in salary? List the names of the managers along with the average salary of their employees. 

<details><summary>Solution 1:</summary>

```SQL
SELECT STATE_CODE, AVG(SALARY)
FROM API_EMPLOYEE
GROUP BY STATE_CODE
HAVING AVG(SALARY) > 100000;
```

</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT HIRE_DATE, COUNT(*) as Hired
FROM EMPLOYEES
GROUP BY HIRE_DATE
HAVING COUNT(*) > 1;
```

</details>

<details><summary>Solution 3:</summary>

```SQL
SELECT M.FIRST_NAME || ' ' || M.LAST_NAME AS EMPMGR, AVG(E.SALARY) AS AVGSAL
FROM EMPLOYEES E JOIN EMPLOYEES M ON E.MANAGER_ID = M.EMPLOYEE_ID
GROUP BY M.FIRST_NAME || ' ' || M.LAST_NAME 
HAVING AVG(E.SALARY) > 10000;
```

</details>

:bulb: It is possible to have more than one aggregate expression for a given `GROUP BY` clause. 
Additionally the aggregate expression(s) can be used to limit the results returned by adding a `HAVING` clause to the query after the `GROUP BY` clause but before the `ORDER BY` clause.

> For example, the query below reports on the total sales (by dollars and by number of items sold) of each product line. For each product line, it reports the total per product line and the breakdown per package type.

```SQL
SELECT I.NAME, PT.CODE, SUM(CO.TOTAL_PRICE), SUM(COI.ITEM_COUNT)
FROM API_CUSTOMER_ORDER CO JOIN API_CUSTOMER_ORDER_ITEM COI 
ON CO.CUSTOMER_ORDER_ID = COI.CUSTOMER_ORDER_ID
JOIN API_ITEM I 
ON I.ITEM_ID = COI.ITEM_ID
JOIN API_PACKAGE_TYPE PT 
ON PT.PACKAGE_TYPE_ID = I.PACKAGE_TYPE_ID
GROUP BY ROLLUP(I.NAME, PT.CODE)
ORDER BY I.NAME, PT.CODE ASC;
```

:bulb: It is possible to report on multiple combinations of the columns participating in the `GROUP BY` in a single query. This is done using the `ROLLUP` keyword. It effectively does a bunch of different `GROUP BY` statements and adds the results to the query output. The extra `GROUP BY` statements correspond to the following column combinations: the first column in the group by, the first two columns in the group by, the first three columns in the group by...and so on.

:muscle: **Most importantly**, remember that the order of SQL Components is as follows:

![](https://i.imgur.com/ohqFmHd.png)
