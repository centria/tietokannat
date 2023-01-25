---
title: "SQL Basics"
nav_order: 4
hidden: false
---

# Basic commands

Next we will familiarize ourselves with the basic SQL commands. With these commands we add, search, change and delete contents from the database. Usually the combination of these commands are known with other names, however. **C**reate, **R**ead, **U**pdate and **D**elete, or **CRUD**, forms the basic functionality of database usage, especially in documentation.

## Creating a table

The command `CREATE TABLE` indeed creates a table, with the desired columns. For example the following command creates the table `Products` with three columns:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);
```

We can name the table and the columns the way we want. Common practice (at least for this course) is, that the tables are written with capital first letter and in plural, and columns with small first letter and in singular form.

For each column, along with the name we declare the desired type. In this table the columns `id` and `price` are *integers* (INTEGER) and the column `name`is a string (TEXT). The column `id` is also the *primary key* (PRIMARY KEY) for the table. This means it creates an identification for each row in the table and with it we can easily refer to any row.

## Primary key

The primary key can be any column or combination of columns, which is unique to each row. In practice, a common way for a primary key is an id column with integer type.

We usually want the id to have a sequential numbering. This means that when we add rows to the table, the first row gets automatically id value of 1, second gets id value of 2, and so on.

The implementation of this depends on the database management system. For example in SQLite database `INTEGER PRIMARY KEY` column automatically gets a sequential numbering.

## Adding information

The command `INSERT` adds a new row into the table. For example the following command adds a row into the table `Products` we just created

```sql
INSERT INTO Products (name,price) VALUES ('radish',7);
```

Here we give the values to the columns `name` and `price` for the row. When we assume the column `id` to have a sequential numbering, it automatically gets the value 1, when the row in question is the first row for the table. Thus the table now contains the following:

```
id          name        price     
----------  ----------  ----------
1           radish     7         
```

If we do not give a value to a column, it gets a default value. In a regular column the default value is `NULL`, which means the data does not exist. For example in the following command we do not give a value to the column price:


```sql
INSERT INTO Products (name) VALUES ('radish');
```

Now the table gets a row, where the price is `NULL` (or empty):

```
id          name        price     
----------  ----------  ----------
1           radish     
```

## Example table

In this chapter we assume in our examples, that we have added the following lines into our table `Products`:

```sql
INSERT INTO Products (name,price) VALUES ('radish',7);
INSERT INTO Products (name,price) VALUES ('carrot',5);
INSERT INTO Products (name,price) VALUES ('turnip',4);
INSERT INTO Products (name,price) VALUES ('cucumber',8);
INSERT INTO Products (name,price) VALUES ('celery',4);
```

Now the table looks like this:

```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4       
```  

## Retrieving information

The command `SELECT` performs a *query*, or retrieves information from the table. The simplest way to perform a query is to get all the information from a table:

```sql
SELECT * FROM Products;
```

In this case the query returns as follows:

```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4       
```  

The asterisk `*` represents all the columns. We can of course get only certain columns instead. For example we can get only the names of the products:

```sql
SELECT name FROM Products;
```

In this case the query returns as follows:

```
name      
----------
radish    
carrot             
turnip               
cucumber               
celery    
```

This query gets the names and the prices:

```sql
SELECT name, price FROM Products;
```

Now the query returns as follows:

```sql
name        price     
----------  ----------
radish      7         
carrot      5         
turnip      4         
cucumber    8         
celery      4         
```

As a result of the query the rows form a table, which is called a `result set`. Its columns and rows are dependand on the content of the query. For example the previous query created a result set with two columns and five rows.

The result set is sort of a table. Thus while handling databases, there are two types of tables: Fixed tables in the database, and temporary tables created by the queries, whose content are fetched from the fixed tables.

## Search clauses

Adding `WHERE` to our `SELECT` query we can choose only a part of the rows according to our desired condition. For example the following query retrieves the information for cucumber:

```sql
SELECT * FROM Products WHERE name='cucumber';
```

In this case the query returns as follows:

```
id          name        price     
----------  ----------  ----------
4           cucumber    8        
```

We can also use `AND` and `OR` in the same way as other programming. For example the next query retrieves the products whose price is between 4...6:

```sql
SELECT * FROM Products WHERE price>=4 AND price<=6;
```

In this case the query returns as follows:

```
id          name        price     
----------  ----------  ----------
2           carrot      5         
3           turnip      4         
5           celery      4         
```

## Ordering

As a default the order of the returned rows can be anything. We can determine the desired order with `ORDER BY` in our query. For example the next query returns the results in alphabetical order by product name:

```sql
SELECT * FROM Products ORDER BY name;
```

In this case the query returns as follows:

```
id          name        price     
----------  ----------  ----------
2           carrot      5
5           celery      4
4           cucumber    8         
1           radish      7
3           turnip      4         
```

The default order is from smallest to largest (**ASC**ENDING). However we can reverse the order, we can add `DESC` (for **DESC**ENDING) after the column `name`:

```sql
SELECT * FROM Products ORDER BY name DESC;
```

In this case the query returns as follows:

```
id          name        price     
----------  ----------  ----------
3           turnip      4  
1           radish      7
4           cucumber    8
5           celery      4
2           carrot      5
```

If you want to be certain about the ascending order, you can also use `ASC` in your query. Thus, the following queries are identical:

```sql
SELECT * FROM Products ORDER BY name;
SELECT * FROM Products ORDER BY name ASC;
```

In practice, `ASC` is not very often used, as it is the default.

We can also order the rows with multiple criteria. For example the following query orders the rows primarily from most expensive to cheapest and secondarily by name:

```sql
SELECT * FROM Products ORDER BY price DESC, name;
```

In this case the query returns as follows:

```
id          name        price     
----------  ----------  ----------
4           cucumber    8         
1           radish      7         
2           carrot      5         
5           celery      4
3           turnip      4    
```     

In this case turnip and celery are ordered by their name (ascending), as they have the same price.

## Distinct result rows

Sometimes result sets can have similar rows. This happens for example with the next query:

```sql
SELECT price FROM Products;
```

As two products have a price of 4, two result rows have the content of 4:

```
price     
----------
7         
5         
4         
8         
4  
```

If we only want different results, we can add the keyword `DISTINCT`:

```sql
SELECT DISTINCT price FROM Products;
```

With this the result becomes the following:

```
price     
----------
7         
5         
4         
8         
```

## Changing information

The command `UPDATE` changes the content of the rows which match the selected condition. For example the next command changes the `price` for turnip to `6`:

```sql
UPDATE Products SET price=6 WHERE name='turnip';
```

You can change several values by combining the changes with comma. For example the following command sets the `name` of the turnip into `pineapple` and `price` into `9`:

```sql
UPDATE Products SET name='pineapple', price=9 WHERE name='turnip';
```

The change can also be calculated from a previous value. For example the following command increases the `price` of the turnip `by 1`:

```sql
UPDATE Products SET price=price+1 WHERE name='turnip';
```

If the command does not have a condition, the update affects all rows. For example the following command changes the `price` of all the products into `3`:

```sql
UPDATE Products SET price=3;
```

## Removing information

The command `DELETE` removes from the table the rows, which match the wanted condition. For example the following command removes `carrot` from products:

```sql
DELETE FROM Products WHERE name='carrot';
```

Alike in changing, if there are no conditionals, the command affects all rows. The following command removes all the rows from the table:

```sql
DELETE FROM Products;
```

The command `DROP TABLE` removes the table (and all its content). For example the following command removes the table `Products`:

```sql
DROP TABLE Products;
```






# Aggregate queries

An aggregate query calculates a single value from the rows of a table. For example we can count the amount of rows or the sum of all the values in a column. We can also group the rows by columns and run an aggregate query for each group.

## Aggregate functions

The aggregate queries are based on aggregate functions, which perform operations for the rows or columns. Common aggrecate functions are the following:

```
name          function
---------     ---------------------------
COUNT()       counts the amount of rows
SUM()         counts the sum of rows
MIN()         retrieves the smallest value
MAX()         retrieves the largest value
AVG()         counts the average
```
## Examples

Let's look at the table `Products` we created earlier:


```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4       
```        

The next query returns the count of rows:

```sql
SELECT COUNT(*) FROM Products;
```

```
COUNT(*)
----------
5
```

The following returns the count of those rows, whose price is 4:

```sql
SELECT COUNT(*) FROM Products WHERE price=4;
```

```
COUNT(*)
----------
2
```

The following query counts the sum of the prices:

```sql
SELECT SUM(price) FROM Products;
```

```
SUM(price)
----------
28
```

## Selecting rows

If the aggregate function contains asterisk `*`, the query selects all rows. If the function contains the name of a column, the query will choose the rows whose value *is not* `NULL`.

Let's look at the following table, whose row 3 has `NULL` for price:

```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           turnip      4         
3           cucumber               
4           celery      4         
```

The following query returns the count of rows:

```sql
SELECT COUNT(*) FROM Products;
```

```
COUNT(*)  
----------
4
```

The following query return those rows, whose price is not `NULL`:

```sql
SELECT COUNT(price) FROM Products;
```

```
COUNT(price)
------------
3
```

We can also use the keyword `DISTINCT` in aggregate queries. For example the following query informs us, how many different (not NUll) values for price the table has:

```sql
SELECT COUNT(DISTINCT price) FROM Products;
```

```
COUNT(DISTINCT price)
---------------------
2
```

## Grouping

With grouping we can combine information from rows and aggregate functions. The idea behind this is that rows are divided into groups with columns assigned to `GROUP BY`, after which the aggregate function is calculated for each group separately.

Let's have another example of table `Sales`, where we have information about sales amounts for different years:

```
id          product       year       amount
----------  ----------  ----------  ----------
1           radish      2017        120
2           radish      2018        85
3           radish      2019        150
4           turnip      2017        30
5           turnip      2018        35
6           turnip      2019        10
7           cucumber    2017        75
8           cucumber    2018        100
9           cucumber    2019        80
```

The next query returns the total sales per year by grouping:

```sql
SELECT year, SUM(amount) FROM Sales GROUP BY year;
```

The query returns as follows:

```
year       SUM(amount)
----------  ----------
2017        225       
2018        220       
2019        240       
```

For example the total sales of 2017 is 120 + 30 + 75 = 225.

On the other had, we can get the total sales by product like this:

```sql
SELECT product, SUM(amount) FROM Sales GROUP BY product;
```

The query returns as follows:

```
product       SUM(amount)
----------  ----------
cucumber    255       
turnip      75        
radish      355
```

For example the total sales for cucumber is 75 + 100 + 80 = 255.

## Naming the return column

By default the column in the return set gets its name direclty by the query, but we can name them ourselves with `AS` keyword. With this we can clarify, what the aggregate query is about.

For example in the following query the name of the second column is `total`:

```sql
SELECT product, SUM(amount) AS total FROM Sales GROUP BY product;
```

The query returns as follows:

```
product     total
----------  --------
cucumber    255       
turnip      75        
radish      355
```

Actually, the word `AS` is not compulsory, so we could write the query also like this:

```sql
SELECT product, SUM(amount) total FROM Sales GROUP BY product;
```

## Limitation after grouping

We can add `HAVING` to our query, which limits the results *after* the grouping. For example the following query returns the products, whose sale is at least 200:

```sql
SELECT product, SUM(amount) AS total
FROM Sales
GROUP BY product
HAVING total >= 200;
```

The query returns as follows:

```
product     total
----------  --------
cucumber    255       
radish      355     
```

## Query overview

In our queries we can use many of the clauses we have learnt so far, as long as they are in the following order:


```
SELECT – FROM – WHERE – GROUP BY – HAVING – ORDER BY
```

Here is an example of a query with all these parts:

```sql
SELECT product, SUM(amount) AS total
FROM Sales
WHERE year < 2019
GROUP BY product
HAVING total >= 100
ORDER BY product;
```

The query returns the sales of the products before the year 2019, only shows products whose sale in these years is over 100, and orders the results by name. The query returns as follows:

```
product       total
----------  --------
cucumber    175       
radish      205      
```

Notice the difference between `WHERE` and `HAVING`: `WHERE` limits the rows *before* grouping, whereas `HAVING` limits *after* the grouping.



# SQLite database

*SQLite* is a simple and openly availabe database system, which is suitable for learning SQL. You can try the basic functions of SQL with SQLite, and we will use it with some of the examples during this course. 

## Database systems

SQLite is a valid choice for learning SQL, but it does have some restrictions, which can cause problems in actual programs.

Widely used open database systems are *MySQL* and *PostgreSQL*. They have a large amount of features which are lacking from SQLite, but on the other hand their installation and often usage is more difficult.

Transferring data from different database systems is quite easy, as they all have similar SQL language.

## SQLite interpreter

*Interpreter* is a program, with which we can use a database. In this case, we are using one for SQLite. The interpreter can be run by giving the command `sqlite3` on command line. Now we can write and run SQL commands or commands beginning with a dot for the interpreter.

If the computer you are using does not have the SQLite interpreter, you can install it from here:
[https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)

Choose the according your operating system a packet, which is marked with the topic *command-line tools*. The file you need is the one whose name begins with *sqlite3*.

## Example

In the SQLite interpreter the database is by default in memory (being then an *in-memory database*). This means it is empty in the beginning and disappears when the interpreter is closed. This is a good way to test the properties of SQL. A set of commands with the interpreter coud look something like this (with some additional line breaks for readibility):

```
$ sqlite3
SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.

sqlite> CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);

sqlite> .tables
Products

sqlite> INSERT INTO Products (name,price) VALUES ('radish',7);

sqlite> INSERT INTO Products (name,price) VALUES ('carrot',5);

sqlite> INSERT INTO Products (name,price) VALUES ('turnip',4);

sqlite> INSERT INTO Products (name,price) VALUES ('cucumber',8);

sqlite> INSERT INTO Products (name,price) VALUES ('celery',4);

sqlite> SELECT * FROM Products;

1|radish|7
2|carrot|5
3|turnip|4
4|cucumber|8
5|celery|4

sqlite> .mode column

sqlite> .headers on

sqlite> SELECT * FROM Products;

id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4         

sqlite> .quit
```

In the example we begin by creating a table `Products` and then check with the command `.tables`, what tables exist in the database. The only table is `Products`, as it was supposed to.

After this we add rows to the table and retrieve all the rows from the table. The default of SQLite is to separate the columns with vertical lines. We make the results more readable with the command `.mode column` (each column has a fixed width) and `.headers on` (showing the names of the columns). Finally we run the command `.quit`, which closes the SQLite interpreter.

## Database in a file

When running the SQLite interpreter, we can give a filename as a parameter, into which the database is saved. Thus the content of the database is saved after the interpreter is closed.

In the following example the database is saved into the file `test.db`. With this the content of the database is still available, when the interpreter is run again.

```
$ sqlite3 test.db
SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.

sqlite> CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);

sqlite> .tables
Products

sqlite> .quit

$ sqlite3 test.db
SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.

sqlite> .tables
Products

sqlite> .quit
```

## Commands from a file

For the interpreter we can also redirect a file containing commands, which are run one after another. With this we can automate running the commands. For example we can run the commands from the following file `commands.sql`:

```
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);
INSERT INTO Products (name,price) VALUES ('radish',7);
INSERT INTO Products (name,price) VALUES ('carrot',5);
INSERT INTO Products (name,price) VALUES ('turnip',4);
INSERT INTO Products (name,price) VALUES ('cucumber',8);
INSERT INTO Products (name,price) VALUES ('celery',4);
.mode column
.headers on
SELECT * FROM Products;
```

After this we can redirect the commands from the file to the interpreter as follows:

```
$ sqlite3 < commands.sql
id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4 
```