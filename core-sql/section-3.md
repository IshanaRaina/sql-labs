# Section 3

## Subqueries

One of the most powerful features of SQL is the ability to create complex queries by using the results of simpler queries. A subquery is just a query within a query. Subqueries can be found in the `FROM`, `SELECT` and `WHERE` clauses. Sub-queries can themselves contain subqueries.

> For example, a subquery in the `SELECT` clause.

```SQL
SELECT CUSTOMER_ORDER_ITEM_ID, ITEM_ID,
(SELECT NAME FROM API_ITEM I WHERE I.ITEM_ID = COI.ITEM_ID) AS ITEM_NAME
FROM API_CUSTOMER_ORDER_ITEM COI;
```

> A subquery in the `FROM` clause.

```SQL
SELECT CUSTOMER_ORDER_ITEM_ID, COI.ITEM_ID, ITEM_NAMES.NAME AS ITEM_NAME
FROM API_CUSTOMER_ORDER_ITEM COI,
(SELECT ITEM_ID, NAME FROM API_ITEM I) ITEM_NAMES
WHERE COI.ITEM_ID = ITEM_NAMES.ITEM_ID;
``` 

> A subquery in the `WHERE` clause.

```SQL
SELECT I.ITEM_ID, I.NAME
FROM API_ITEM I
WHERE I.PACKAGE_TYPE_ID IN (
SELECT PACKAGE_TYPE_ID FROM API_PACKAGE_TYPE WHERE CODE IN ('ea', 'bx')
);
```

:bulb: When a subquery uses values from the outer query it is called a correlated subquery. Usually that implies that the subquery is executed once per row in the outer query. 
Recall that the optimizer will evaluate alternative ways to execute the query and it will sometimes transform subqueries into other equivalent expressions (typically as some type of join).

### Subquery Restrictions

When subqueries are used in `SELECT` clauses they are subject to a few restrictions: 

- To avoid runtime errors they must return one and only one row. The value may be `NULL` but it must return a row. This is the main restriction that must be kept in mind when working with subqueries.

- Oracle will inform you if your query is attempting to use a subquery in an unsupported way (E.g. “ORA-22818: subquery expressions not allowed here”). When that happens do a search using your error string, and the location where you are using the subquery. You should be able to find the underlying issue or restriction as well as the suggested workarounds.

### SQL-99 With Clause

As queries get progressively more complex, with more joins and subqueries they become more difficult to understand, maintain and tune. The SQL-99 `WITH` clause (aka subquery factoring) allows the SQL to be organized in a way much closer to how we think of queries. The idea is to factor out the simpler subqueries, run them first, give the result an alias, and thereafter treat that name as if it were a table in the outer queries that use it. This allows you to develop and troubleshoot the resulting query starting at the smaller statements and progressing to the increasingly complex statements.

The structure of a WITH clause query is as follows:

```SQL
WITH 
subquery1Alias AS (subquery 1 SQL),
subquery2Alias AS (subquery 2 SQL),
…
subqueryNAlias AS (subquery n SQL)
<SELECT expression using any schema object or alias defined in the WITH clause>
```

> For example, the query below uses a `WITH` clause. The goal is to find those customers that have had at least one order shipped outside of the 'USA' where that order contained at least one item that was a pallet. It could have been done as one monster join, however, the result using a `WITH` clause is much easier to read. 

:bulb:Note: 
- how the subqueries were given meaningful names and help make the query easier to understand.
- each subquery can use any schema object or any previously defined subquery alias. The final query can use any schema objects and any combination of previously defined subqueries. 
- a given subquery can be used more than once, but must be used at least once somewhere in the `WITH` clause query.

```SQL
WITH 

FOREIGN_ORDERS AS (
SELECT CO.CUSTOMER_ORDER_ID, CO.CUSTOMER_ID 
FROM API_CUSTOMER_ORDER CO JOIN API_ADDRESS A 
ON CO.SHIP_TO_ADDRESS_ID = A.ADDRESS_ID 
WHERE A.COUNTRY <> 'USA'),

PALLET_ORDERS AS (
SELECT DISTINCT CO.CUSTOMER_ORDER_ID 
FROM API_CUSTOMER_ORDER CO JOIN API_CUSTOMER_ORDER_ITEM COI 
ON COI.CUSTOMER_ORDER_ID = CO.CUSTOMER_ORDER_ID 
JOIN API_ITEM I 
ON I.ITEM_ID = COI.ITEM_ID 
JOIN API_PACKAGE_TYPE PT 
ON PT.PACKAGE_TYPE_ID = I.PACKAGE_TYPE_ID 
WHERE PT.CODE = 'sd'),

FOREIGN_PALLET_ORDERS AS (
SELECT FO.CUSTOMER_ORDER_ID, FO.CUSTOMER_ID 
FROM FOREIGN_ORDERS FO JOIN PALLET_ORDERS PO 
ON PO.CUSTOMER_ORDER_ID = FO.CUSTOMER_ORDER_ID) 

SELECT C.FIRST_NAME, C.LAST_NAME 
FROM API_CUSTOMER C JOIN FOREIGN_PALLET_ORDERS FPO 
ON FPO.CUSTOMER_ID = C.CUSTOMER_ID 
ORDER BY C.LAST_NAME ASC, C.FIRST_NAME ASC;
```

If there is a problem with the query, you can start with the simplest subquery, make sure that is working as expected (performance, results or anything else) and then move out to more complex subqueries.

> For example, create a copy of the query and start selecting only subquery 1.

```
WITH subquery1Alias as (subquery 1 SQL)
select * from  subquery1Alias;
``` 
> If that works, expand to include subquery 2.

```
WITH subquery1Alias as (subquery 1 SQL),
subquery2Alias as (subquery 2 SQL)
select * from subquery2Alias;
``` 
And so on.

:warning: Keep in mind that you can't have subqueries that are unused, so in this example, if subquery 2 did not use subquery 1 then we would need to use a query that uses both in the final select or we could just remove subquery 1 when troubleshooting subquery 2. 

If both have no problem, we could add them both back in when checking subquery 3 (assuming it was dependent on the two of them).


> For example, in the foreign orders example above, we would have started by testing subquery 1.

```SQL
WITH 

FOREIGN_ORDERS AS (
SELECT CO.CUSTOMER_ORDER_ID, CO.CUSTOMER_ID 
FROM API_CUSTOMER_ORDER CO JOIN API_ADDRESS A 
ON CO.SHIP_TO_ADDRESS_ID = A.ADDRESS_ID 
WHERE A.COUNTRY <> 'USA')

SELECT * FROM FOREIGN_ORDERS;
```

> If the previous query returned what was expected, we would move on to subquery 2. Since subquery 2 does not use subquery 1, we remove subquery 1 for now and test.

```SQL
WITH 

PALLET_ORDERS AS (
SELECT DISTINCT CO.CUSTOMER_ORDER_ID 
FROM API_CUSTOMER_ORDER CO JOIN API_CUSTOMER_ORDER_ITEM COI 
ON COI.CUSTOMER_ORDER_ID = CO.CUSTOMER_ORDER_ID 
JOIN API_ITEM I 
ON I.ITEM_ID = COI.ITEM_ID 
JOIN API_PACKAGE_TYPE PT 
ON PT.PACKAGE_TYPE_ID = I.PACKAGE_TYPE_ID 
WHERE PT.CODE = 'sd')

SELECT * FROM PALLET_ORDERS;
```

> If the previous query returned what was expected, we would move on to subquery 3. Note that subquery 3 uses both subqueries 1 and 2, so we need them now.

```SQL
WITH 

FOREIGN_ORDERS AS (
SELECT CO.CUSTOMER_ORDER_ID, CO.CUSTOMER_ID 
FROM API_CUSTOMER_ORDER CO JOIN API_ADDRESS A 
ON CO.SHIP_TO_ADDRESS_ID = A.ADDRESS_ID 
WHERE A.COUNTRY <> 'USA'), 

PALLET_ORDERS AS (
SELECT DISTINCT CO.CUSTOMER_ORDER_ID 
FROM API_CUSTOMER_ORDER CO JOIN API_CUSTOMER_ORDER_ITEM COI 
ON COI.CUSTOMER_ORDER_ID = CO.CUSTOMER_ORDER_ID 
JOIN API_ITEM I 
ON I.ITEM_ID = COI.ITEM_ID 
JOIN API_PACKAGE_TYPE PT 
ON PT.PACKAGE_TYPE_ID = I.PACKAGE_TYPE_ID 
WHERE PT.CODE = 'sd'), 

FOREIGN_PALLET_ORDERS AS (
SELECT FO.CUSTOMER_ORDER_ID, FO.CUSTOMER_ID 
FROM FOREIGN_ORDERS FO JOIN PALLET_ORDERS PO 
ON PO.CUSTOMER_ORDER_ID = FO.CUSTOMER_ORDER_ID) 

SELECT * FROM FOREIGN_PALLET_ORDERS;
```

For troubleshooting purposes, if that returned what was expected, we would know that the problem lies in the final `SELECT`.

:bulb: The SQL-99 query is a very powerful mechanism to help with complex queries. If you are creating a complicated query and are struggling with the logic, the debugging or the tuning you should strongly consider re-writing it as a query using a `WITH` clause. Even if you fully understand the query logic, if it is complex enough to create problems for other people, you may want to consider using SQL-99.

:books: Read more about:
- [subqueries](https://docs.oracle.com/cd/B19306_01/server.102/b14200/queries007.htm)
- [`WITH` clause](https://oracle-base.com/articles/misc/with-clause)

**Exercises** :computer: 

Acess the **HR** database :arrow_forward: 

1. List out all employee details on the employee who earns the least salary.
2. Building on the last query, who is the manager whose employee earns the least salary?
3. Using a subquery, list the first 10 employees with the least salaries in ascending order.
4. List all details on those employees who earn more than the average salary.
5. List all details on 2 employees - the one who earns the most and the one who earns the least.
6. List the employee first and last name, salary, average salary for his/her particular position and job title. Filter on those employees who make more than the average salary for their job. 

<details><summary>Solution 1:</summary>

```SQL
SELECT * FROM EMPLOYEES
WHERE SALARY = (SELECT MIN(SALARY) FROM EMPLOYEES);
```

</details>

<details><summary>Solution 2:</summary>

```SQL
SELECT FIRST_NAME, LAST_NAME FROM EMPLOYEES WHERE EMPLOYEE_ID IN
    (SELECT MANAGER_ID FROM EMPLOYEES WHERE SALARY =
       (SELECT MIN(SALARY) FROM EMPLOYEES)
    );
```

</details>

<details><summary>Solution 3:</summary>

```SQL
SELECT * FROM (SELECT * FROM EMPLOYEES ORDER BY SALARY)
WHERE ROWNUM < 11;
```

</details>

<details><summary>Solution 4:</summary>

```SQL
SELECT * FROM EMPLOYEES 
WHERE SALARY > (SELECT AVG(SALARY) FROM EMPLOYEES);
```

</details>

<details><summary>Solution 5:</summary>

```SQL
SELECT * FROM EMPLOYEES 
WHERE SALARY IN
( 
    SELECT MIN(SALARY) FROM EMPLOYEES
    UNION
    SELECT MAX(SALARY) FROM EMPLOYEES
);
```

</details>

<details><summary>Solution 6:</summary>

```SQL
SELECT FIRST_NAME, LAST_NAME, SALARY, AVGSAL, JOB_TITLE
FROM EMPLOYEES E  JOIN
(
    SELECT  JOB_TITLE, AVG(SALARY) AS AVGSAL, E.JOB_ID
    FROM EMPLOYEES E JOIN JOBS 
    ON E.JOB_ID = JOBS.JOB_ID
    GROUP BY JOB_TITLE, E.JOB_ID) T
ON E.JOB_ID = T.JOB_ID AND E.SALARY > AVGSAL;
```

</details>
