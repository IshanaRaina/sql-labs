# Section 5

## Oracle Metadata

Oracle contains vast amounts of metadata. In fact, if you can think of it, Oracle is probably tracking it. The number of views and tables is very large, and only a very seasoned DBA would be expected to know what most of them even do. These tables and views are most often used for advanced troubleshooting and scripting. Even administration tools such as OEM use these tables and views behind the covers (OEM will actually show you the SQL it is using to pull the data).

:warning: A warning is in order: never directly update an Oracle owned table without an extremely good reason. Odds are that Oracle and the DBAs have it locked down, but just in case, do not try it. 

Oracle provides a large number of packages to assist with all types of database administration. These packages allow users to gather statistics, partition tables, refresh materialized views etc. 

One type of data that Oracle has is metadata on all schema objects in the database (tables, indexes, constraints, triggers, views, partitions, synonyms, sequences, packages...). This metadata can be queried and Oracle provides some user friendly views for this purpose. This is known as the **data dictionary.**

> To find information on the table **API_ITEM** execute the query below.

```SQL
SELECT * FROM ALL_TABLES WHERE TABLE_NAME = 'API_ITEM';
```

:bulb: Note the use of `all_` If multiple tables with that name existed (different owners) you would see them in the result as multiple rows. The prefix `all_` spans the entire database, but only returns those things the current user has permissions to see. 
Similarly, 
- the prefix `user_` will only consider those objects owned by the current user. 
- the prefix `dba_` will show everything across all schemas.

> To find information on the columns in the table **API_ITEM**
execute the query below.

```SQL
SELECT * FROM ALL_TAB_COLS WHERE TABLE_NAME = 'API_ITEM';
```

> To find the partitions on the table **API_EMPLOYEE_HASH_PART** run the query below.

```SQL
SELECT * FROM ALL_TAB_PARTITIONS WHERE TABLE_NAME = 'API_EMPLOYEE_HASH_PART';
```

> Suppose we wanted to drop all tables that start with **API_**. We can create a SQL query that will generate the drop statements for us. It will have all of the tables. 

```SQL
SELECT 'DROP TABLE ' || TABLE_NAME || ';' 
FROM ALL_TABLES 
WHERE TABLE_NAME LIKE 'API_%'
```

Note that the query does not look into the views with referential integrity information, so the order is random. As a result, if it is run, some of the drops will fail. The query could be enhanced, or you can just run it a couple of times.

:bulb: In general: 
- When you are looking to find some metadata, there is a way to get it, the only question is how. You can search online (if you use the right keywords you will find the correct answers without too much difficulty) or you can read through the documentation. It is worthwhile to skim through the documentation once to get a feel for what is available.
- The information in the data dictionary is useful for research, but it can also be used to speed up support tasks and even write automated scripts.
- Database maintenance scripts often make use of the data dictionary to ensure that they are generic (don't have hard-coded table names) and keep running even when the schema is altered.

:books: Read more about the following:
- A short list of data dictionary views [here](http://docs.oracle.com/cd/B28359_01/server.111/b28310/tables014.htm).
- or check out the full list [here](http://docs.oracle.com/cd/B28359_01/nav/catalog_views.htm).
