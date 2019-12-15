# Section 2

## Joins

So far, we've queried one table at a time but what if we want to retrieve data that lies in more than one table? Joins allow us to access data that resides in more than one table **inside of a single statement**.

There are several types of joins to support the different ways in which we might want to combine the tables. When specifying a join we must choose a join type and then specify the columns on which the join must happen. The process can be repeated and additional tables can be joined to other tables already being used in the statement. Note that the same table can appear multiple times, and it can even be joined to itself (this is known as a self join which we will learn about in just a bit).

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

:warning: Let's bring into focus the **HR** database as well, in order to understand and implement the different kinds of joins. 

### Inner Join

An inner join is one in which values from the two columns must exist and satisfy the join condition for either to be returned. If either side is missing the value, the other side will be not be included in the result set. 

> For example, on joining **API_CUSTOMER** to **API_CUSTOMER_ORDER** with an inner join on `CUSTOMER_ID`, if a customer does not have any orders, that customer will not appear in the query results.

**Exercise 1** :computer: 

1. List the employee first and last names along with their job titles.
2. List the employee first and last names along with their job titles for all employees whose last name is 'King'.
3. List employee names, jobs, salary and max_salary only if those employees earn the max_salary or more.
4. List department name, first and last name and job titles using the alternative join method that you saw above.

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

1. List departments which have no employees. 
2. List employees that don't belong to any department.
3. List employees that don't belong to any department without using a join.
4. List the department name, first and last name and job titles of those employees whose first name is `NULL`. Replace the `NULL` first names with 'Harvey' in the result.
5. List departments along with their manager's full names. 

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

Who let the dogs :dog2: out?! Oops, I meant, who bought the 'API Dog Food'?! List the item name and customer name.

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

**Exercise 5** :computer: 

1. List names of all managers and their employees, ordered by manager.
2. Let's take it one step further - list employee's full name, the department they belong to followed by their manager's full name.

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

:bulb: Note that the join conditions used in the previous examples are based on the equality operator, but other operators are also allowed. We could have used <= or <> (to list a few) instead of = in the join condition of one of the previous example queries. This is fairly uncommon, but it is needed on occasion.

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

List all salaries (ie employee salary, min salary and max salary) in a single column.

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

1. Access the HR database :arrow_forward: List the names of all employees followed by the names of all countries in a single column.
2. Access the API database :arrow_forward: List first names of all customers whose last name is 'Johnson' and the last names of all customers whose last name is not 'Johnson', in a single column.

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
