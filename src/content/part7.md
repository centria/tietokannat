---
title: "Efficiency"
nav_order: 9
hidden: false
---

# Executing a query

SQL is nice to the user for making queries, as the user only has to describe what information they wish to retrieve, and the database system does the rest. Thus it is crucial for the database system to find a reasonably efficient way to implement the user's query and deliver the results to the user.

## Explain! Explain!

Many database systems have a choice of explaining the plan, how it intends to run the given query. With this we can examine the interal functionality of the system.

Let's examine a query, which retrieves the information for `radish` from the table `Products`:

```
SELECT * FROM Products WHERE name='radish';
```

When we add the word `EXPLAIN` before the query in SQLite, we can get the following kind of representation, how the query is planned to run:

```
sqlite> EXPLAIN SELECT * FROM Products WHERE name='radish';
addr  opcode         p1    p2    p3    p4             p5  comment      
----  -------------  ----  ----  ----  -------------  --  -------------
0     Init           0     12    0                    00  Start at 12  
1     OpenRead       0     2     0     3              00  root=2 iDb=0; Products
2     Rewind         0     10    0                    00               
3       Column         0     1     1                    00  r[1]=Products.name
4       Ne             2     9     1     (BINARY)       52  if r[2]!=r[1] goto 9
5       Rowid          0     3     0                    00  r[3]=rowid   
6       Copy           1     4     0                    00  r[4]=r[1]    
7       Column         0     2     5                    00  r[5]=Products.price
8       ResultRow      3     3     0                    00  output=r[3..5]
9     Next           0     3     0                    01               
10    Close          0     0     0                    00               
11    Halt           0     0     0                    00               
12    Transaction    0     0     1     0              01  usesStmtJournal=0
13    TableLock      0     2     0     Products       00  iDb=0 root=2 write=0
14    String8        0     2     0     radish        00  r[2]='radish'
15    Goto           0     1     0                    00 
```

SQLite transforms the query into an internal *program*, which retrieves the information from the tables. In this case the program execution begins from the line 12, where we begin the transaction, and on row 14 we place the string "radish" from our query into the register 2. After this the execution moves to row 1, where it begins the handling for table `Products`, and rows from 2 to 9 form a loop, which searches for a matching row from the table.

We can ask for more compact plan with the keywords `EXPLAIN QUERY PLAN`. Then the result is something in the line of:

```
sqlite> EXPLAIN QUERY PLAN SELECT * FROM Products WHERE name='radish';
0|0|0|SCAN TABLE Products
```

Here `SCAN TABLE Products` means that the query goes through the table `Products`.

## Optimizing a query
If a query retrieves data from only a single table, the query is usually quite simple to execute, but the real problems come when we make queries to multiple tables. Then the database system should be able to optimize query execution, or form a good plan, with which the desired information can be collected from the tables efficiently.

Let's examine a query which lists course and teacher names:

```sql
SELECT C.name, T.name FROM Courses C, Teachers T WHERE C.teacher_id = T.id;
```

As the query focuses on two table, we have thought about the query functionality so, that it first creates the combination of all the rows from the tables `Courses` and `Teachers` and then selects those rows, which satisfy the condition `C.teacher_id = T.id`. This is a good way to think, but this does not actually reflect the functionality of a proper database system.

The issue is that both `Courses` and `Teachers` could have a large amount of rows. For example if both tables have a million rows, the combination of both would be million millions, and it would take an enormous time to go through all the combinations.

In this situation the database system has to understand, what the user is actually searching for and how the condition limits the result set. Practically it is enough to go through all the rows from table `Courses` and with each row search somehow efficiently the desired row from the table `Teachers`.

We can once more ask SQLite to explain the plan for the query:
```
sqlite> EXPLAIN QUERY PLAN SELECT C.name, T.name FROM Courses C, Teachers T WHERE C.teacher_id = T.id;
0|0|0|SCAN TABLE Courses AS C
0|1|1|SEARCH TABLE Teachers AS T USING INTEGER PRIMARY KEY (rowid=?)
```

This query iterates through the table `Courses` rows (`SCAN TABLE Courses`) and retrieves information from `Teachers` with the primary key (`SEARCH TABLE Teachers`). The latter means that while processing a certain row from `Courses`, the query efficiently searches from `Teachers` where the primary key `T.id` is same as `C.teacher_id`.

But how can we search `Teachers` efficiently? This is done by indeksin, which we will look into next.

# Indexes

*Index* is a datastructure saved alongside the database table, whose function is to make the queries to said table more efficient. With an index a database system can efficiently solve, where in the table are the rows which match a certain query condition.

## Primary key index

When a table is created in a database, the primary key automatically receives an index. Because of this we can efficiently perform queries, where the condition involves the primary key.

For example if we crate this table in SQLite

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);
```

An index is created for the column `id` and we can search efficiently products based on their id. Because of this for example the next query is efficient:

```sql
SELECT price FROM Products WHERE id=3;
```

We can confirm the efficiency with the explanation:

```
sqlite> EXPLAIN QUERY PLAN SELECT price FROM Products WHERE id=3;
selectid    order       from        detail                                                   
----------  ----------  ----------  ---------------------------------------------------------
0           0           0           SEARCH TABLE Products USING INTEGER PRIMARY KEY (rowid=?)
```

The plan contains `SEARCH TABLE` which means the query can search the table with an index efficiently.

## Creating an index

The primary key index is handy, but we might want to search information with other columns as well. For example the following query retrieves the rows with the `price` column:

```sql
SELECT name FROM Products WHERE price=4;
```

This is not efficient by default, as the column `price` does not have an index. We can once again check this with an explanation:

```
sqlite> EXPLAIN QUERY PLAN SELECT name FROM Products WHERE price=4;
selectid    order       from        detail             
----------  ----------  ----------  -------------------
0           0           0           SCAN TABLE Products
```

Now the plan contains `SCAN TABLE` which means the query has to iterate through all the rows in the table. This is slow, if the table contains many rows.

We can create a new index, which makes the queries via column `price` more efficient. We can create an index with `CREATE INDEX` as follows:

```sql
CREATE INDEX idx_price ON Products (price);
```

Here the `idx_price` is the name of the index, with which we can refer to it later. The index works automatically after the creation, so the database system can use it in queries and takes care of updating it.

## How does the index work?

Index requires a data structure, which can be efficiently searched through based on the values for the rows in the column. This can be achieved with for example a tree structure, where the keys are column values.

After creating the index, we can again ask for the plan:

```
sqlite> EXPLAIN QUERY PLAN SELECT name FROM Products WHERE price=4;
selectid    order       from        detail                                               
----------  ----------  ----------  -----------------------------------------------------
0           0           0           SEARCH TABLE Products USING INDEX idx_price (price=?)
```

Because of the index the `SCAN TABLE` has been replaced with `SEARCH TABLE`. In the plan you can also see, that it uses the new index `idx_price` in the search.

## More uses

We can also use indexes in queries, where we retrieve smaller or larger values. For example with the index created for the column `price` we can for example search for rows with which applies the condition `price<3` or `price>=8`.

An index can also be created based on several columns. We could create an index with for example:

```sql
CREATE INDEX idx_price ON Products (price,name);
```

In this index the rows are ordered primarily by `price` and secondarily by `name`. The index enhances the queries, where the search condition is either just `price` or the combination of `price` and `name`. This index does not help queries where the search is based purely on `name`.

## When to create an index?

In theory you might think that it is good to create an index for all the columns of all the tables, and boost all the queries possible. In practice this is not a good idea.

Even though indexes boost queries, they come with two drawbacks: The data structure required for a tree takes space, and indexing slows down adding and changing data. The latter is because when the data in the table is changed, the change has to be updated also to all teh indexes associated with the table. We shouldn't be creating indexes just because we can.

A good reason for indexing is, if we want to run certain types of queries often and they are slow because the database system has to go through all the rows in a table during the query. In this situation we could add an index for the table, so this sort of query would be faster in the future.

Indexing has quite an impact on database efficiency. Many a databases work slowly, because they are missing crucial indexes. The balance between enough indexes and too many indexes is a delicate one, but should be taken into consideration while developing databases.

# Repetitive information

Idealistic situation in database design is that databases do not contain repetitive information, which could be deducted from other information in the database. Sometimes we have to bend a little of this principle to make queries more efficient.

## Example

Let's look at a bank database, where the table `Tranactions` contains information about account transactions. With each transaction there is a column `change` which indicates how much the balance changes (so it can be either positive or negative).

The following example returns the current balance of account number 123 by counting together all the changes associated with the account:

```sql
SELECT SUM(change) FROM Transactions WHERE account_id=123;
```

This is a nice query per se, but it has to go through all the rows associated with the account 123 from the table `Transactions` to find the current balance. This is kind of unneccessary work, as we are not interested in the history but only the balance.

We can make the query more efficient by creating a new table `Balances` which contains the balance of each account. With this account we can retrieve the balance of account 123 this easily:

```sql
SELECT balance FROM Balances FROM account_id=123;
```

Here we break the principle that a database should not contain information that can be counted from other information in the database, as the information in table `Balances` could be calculated from a row in the table `Transactions`. This way though we can make much more efficient query.

What's wrong with breaking the principle? Well, we just made updating the database harder. Every time we now add a new row to table `Transactions`, we also have to make a change to table `Balances`. Previously the balance was calculated directly from the transactions, why it was always current.

## Denormalization

In database theory the term *denormalization* is used, which means making queries more efficient by adding repetitive information. This time we are doing quite the oppisite than with normalization.

## Changes vs Queries

Often occuring phenomenon in computer science is that we have to balance if we prefer to retrieve or update data efficiently, and how much space can we use. This is familiar from for example algorithms.

If a database does not contain repetitive information the, changes are easy as each piece of information is stored only in one place, so we only need to change one row. The database also takes up little space, which is desired. On the other hand the queries can be complicated and slow, as the information has to be gathered from different parts of the database.

When we add repetitive information, we can speed up the queries but then again the changes get slower, as the changed information has to be updated to several places. At the same time the space requirement for the database grows.

Unfortunately there is no general rule, how much repetitive information is necessary to add, but it depends on the usage and content of the database. One approach could be starting with no repetitive information, and add repetition when it is apparent the required queries are not efficient enough.

Notice, that indexing is also one example of how repetitive information can make queries more efficient. In those the repetitive information is not stored in the tables, but outside the table in a data structure of their own.
