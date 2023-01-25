---
title: "Data integrity"
nav_order: 8
hidden: false
---

# Data integrity

With *data integrity* we mean that the data in the database is current and does not have conflicts. Main responsibility of the data quality lies of course on the user or the application using the database, but also the database designer can affect the integrity by adding conditions to the tables, which monitor the information added to the database.

## Column conditions

We can define conditions for the columns when we create columns, which the database systems monitors when the data is added or changed. With these conditions we can control what sort of data the database contains. Let's look at some of the most typical conditions.

## UNIQUE

Condition `UNIQUE` means, that the column must have a different value on each row. For example in the following table the requirement is, that each product has a different name:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT UNIQUE, price INTEGER);
```

## NOT NULL and DEFAULT
The condition `NOT NULL` means, that the column cannot contain the value `NULL`. For example in the next command the price cannot be empty:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER NOT NULL);
```

The definiton `DEFAULT` is often coupled with this, which gives the column a certain default value, if a value is not given when a row is added. We can give for example a default of 0 like this:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER DEFAULT 0);
```


## CHECK

Another common way to create a condition using the keyword `CHECK`, after which you can write any conditional. For example the next command creates a table of products whose price condition is `price >= 0`, which means the price cannot be negative:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price CHECK (price >= 0));
```

## Condition monitoring

The benefit of condition is that the database system monitors them and refuses to add a row or make a change, which would violate the condition. Here's an example of this in action with SQLite.


```
sqlite> CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price CHECK (price >= 0));
sqlite> INSERT INTO Products(name,price) VALUES ('radish',4);
sqlite> INSERT INTO Products(name,price) VALUES ('celery',7);
sqlite> INSERT INTO Products(name,price) VALUES ('cucumber',-2);
Error: CHECK constraint failed: Products

sqlite> SELECT * FROM Products;
1|radish|4
2|celery|7

sqlite> UPDATE Products SET price=-2 WHERE id=2;
Error: CHECK constraint failed: Products
```

When we try to add a row to the table `Products` with a negative price, this violates the condition `price >= 0` and SQLite does not allow adding the row but gives an `Error: CHECK constraint failed: Products`. Same happens when we try to change the price of existing lines to negative afterwards.

## Reference conditions

We can also add conditions to the tables which make sure, that the references in the tables refer to actual rows. This is done by creating `foreing key` which describes, where does the row refer to.

Let's look at an example:

```sql
CREATE TABLE Teachers (id INTEGER PRIMARY KEY, name TEXT);
CREATE TABLE Courses (id INTEGER PRIMARY KEY, name TEXT, teacher_id INTEGER);
```

Here the purpose is that the table `Courses` column `teacher_id` references to the column `id` from table `Teachers`, but the database user can give any value for the column (for example number 123), when the database content is broken. We can make this better while creating the table `Courses` by telling that the column `teacher_id` is a `foreign key` to the table `Teachers`:

```sql
CREATE TABLE Courses (id INTEGER PRIMARY KEY, name TEXT, teacher_id INTEGER REFERENCES Teachers);
```

Now we can trust that the values in the `Courses` column `teacher_id` refer to actual rows in table `Teachers`.

## Reference keys in SQLite

Can we actually trust, that the reference keys work in the desired ways? For historical reasons, SQLite does not monitor the reference key konditions by default, but first we have to run the following command:

```
sqlite> PRAGMA foreign_keys = ON;
```

This is specific to SQLite, and on other database systems the reference key monitoring *should* be always monitored.

Here's an example of using a reference key:

```
sqlite> PRAGMA foreign_keys = ON;
sqlite> CREATE TABLE Teachers (id INTEGER PRIMARY KEY, name TEXT);
sqlite> CREATE TABLE Courses (id INTEGER PRIMARY KEY, name TEXT, teacher_id INTEGER
   ...>                       REFERENCES Teachers);
sqlite> INSERT INTO Teachers (name) VALUES ('Ahonen');
sqlite> INSERT INTO Teachers (name) VALUES ('Isohanni');
sqlite> SELECT * FROM Teachers;
1|Ahonen
2|Isohanni
sqlite> INSERT INTO Courses (name, teacher_id) VALUES ('Basic programming',2);
sqlite> INSERT INTO Courses (name, teacher_id) VALUES ('Algebra',123);
Error: FOREIGN KEY constraint failed   
```

In the table `Teachers` we have two teachers whose `id` are 1 and 2. When we try to add a row to `Courses` where the `teacher_id` is 123, SQLite does not allow it and we get an error `Error: FOREIGN KEY constraint failed`.

## References and removals

Reference conditions have more complex situations than regular columns, as the references are between two tables. Especially important is to notice, what happens when we try to remove a row, which is being referenced to from another table?

Usually the default is that we cannot remove a row from a database, if the row is being references from elsewhere. For example, if we try to remove the row 2 from the table `Teachers` above, this does not work, as it is being referenced from the table `Courses`:

```sql
sqlite> DELETE FROM Teachers WHERE id=2;
Error: FOREIGN KEY constraint failed
```

If we want, we can define in table creation more cleary what happens in a situation like this. For example one option is `ON DELETE CASCADE`, which means that when a row is removed, so are the rows referencing to it. We can achieve it as follows:

```sql
CREATE TABLE Courses (id INTEGER PRIMARY KEY, name TEXT,
      teacher_id INTEGER REFERENCES Teachers ON DELETE CASCADE);
```   

Now if we remove a teacher from the database, the courses they teach are also automatically removed. This though is often not the desired option, as this means we might lose too much information from the database.

## Consequences of removal

Possible options in `ON DELETE` are:

* `NO ACTION`: "don't do anything" (default)
* `RESTRICT`: prevent removal
* `CASCADE`: also remove the referencing rows
* `SET NULL`: change the referencing value to `NULL`
* `SET DEFAULT`: change the reference to default value

A puzzling notice is, that `NO ACTION` prevents the removal, even though the name suggests otherwise. In practice, `NO ACTION` and `RESTRICT` work almost similarly, but on some database systems and special cases there might be differences.

# Introduction to transactions

Transaction is a group of consecutive SQL commands, which the database promises to execute as a whole. The database user can trust, that either
1. All the commands are executed and the changes are permanent on the database
2. Transaction is aborted and the commands do not cause any changes to the database

## ACID

With transactions we come across the acronym *ACID* quite often, which is derived from the following:

* Atomicity: The commands in the transaction are run as a single whole.
* Consistency: The transaction keeps the database content intact.
* Isolation: Transactions are run separated from each other.
* Durability: When a transaction is completed the change is permanent.

## Transaction phases

A transaction is actually quite a mundane part of database usage, as per default each SQL command is a transaction as such. Let's look for example the next command, which increases the price of each product by one:

```sql
UPDATE Products SET price=price+1;
```

As the command is actually a transaction, we can trust that indeed the price of each product is increased by one, or then the price of any product is not changed. The latter happens for example if the connection to database is lost during the update. Even in that situation only part of the prices would be updated.

Usually still the word transaction is used to refer to a whole of several SQL commands. Then we begin with the command `BEGIN TRANSACTION`, which starts the transaction. We continue by running all the necessary commands as usual, and finish with the command `COMMIT`, which closes the transaction.

A classic example of a transaction is a situation, where we transfer money from one bank account to another. For example the following transaction moves 100 euros from Maija's account to Uolevi's:

```sql
BEGIN TRANSACTION;
UPDATE Accounts SET balance=balance-100 WHERE owner='Maija';
UPDATE Accounts SET balance=balance+100 WHERE owner='Uolevi';
COMMIT;
```

The concept behind transactions is that no permanent change is done before the command `COMMIT`. Thus in the example above it is not possible for Maija to lose 100 euros and Uolevi getting nothing (remember the example from the beginning of the course?). Either the balance of both accounts are changed and the money is transferred successfully, or both balances remain unchanged.

If a transaction is interrupted for any reason before the `COMMIT`, all the changes in the transaction are cancelled. One reason for a transaction interruption could be a malfunction with the computer, but we can also ourselves terminate the transaction with the command `ROLLBACK`.

## Testing a transaction

A good way to gain knowledge of transactions is to try in practice, how they work. Here's an example of a conversation with SQLite (with added linebreaks):

```
sqlite> CREATE TABLE Accounts (id INTEGER PRIMARY KEY, owner TEXT, balance INTEGER);
sqlite> INSERT INTO Accounts (owner,balance) VALUES ('Uolevi',350);
sqlite> INSERT INTO Accounts (owner,balance) VALUES ('Maija',600);

sqlite> SELECT * FROM Accounts;
1|Uolevi|350
2|Maija|600

sqlite> BEGIN TRANSACTION;
sqlite> UPDATE Accounts SET balance=balance-100 WHERE owner='Maija';
sqlite> SELECT * FROM Accounts;
1|Uolevi|350
2|Maija|500
sqlite> ROLLBACK;

sqlite> SELECT * FROM Accounts;
1|Uolevi|350
2|Maija|600

sqlite> BEGIN TRANSACTION;
sqlite> UPDATE Accounts SET balance=balance-100 WHERE owner='Maija';
sqlite> UPDATE Accounts SET balance=balance+100 WHERE owner='Uolevi';
sqlite> COMMIT;

sqlite> SELECT * FROM Accounts;
1|Uolevi|450
2|Maija|500
```

In the beginning, Uolevi has 350 euros on their account and Maija has 600 on theirs. In the first transaction we first remove 100 euros from Maija, but then we have a second thought and terminate the transaction. Thus the change made in the transaction is cancelled and the balances remain the same as in the beginning. We do complete the second transaction, for which Uolevi now has 450 euros and Maija has 500 euros on the account.

Notice, that we can see the changes *inside the transaction*, even though they have not been permanently saved to the database. For example the `SELECT` query inside the first transaction shows Maija's balance as 500 euros, since the previous `UPDATE` changed the balance.


## Internal implementation

Implementing transactions is a fascinating technical challenge in databases. In a way the transaction has to make changes to the database as the commands may be reliant on previous commands, but on the other hand nothing can be permanently changed before the transaction is completed.

One central idea behind databases is to save changes in two ways. First the description of the change is written into a *log file* called `write-ahead log`, which can be thought of as a list of commands that have been run. Only after this the changes are done to the actual data structures of the database. Now if something surprising happens in the second phase, the changes are already saved in the log and can be run again later.

When making a transaction the database system also has to keep track of which ongoing changes are done by which transaction. In practice we can only make changes which are visible to certain transactions. If and only if the transaction is completed, the changes are made to the table row permanently.


# Parallel transactions

Additional mix to transaction handling is created when a database can have several users, whom all have transactions going on at the same time. How much should we isolate the transactions from one another?

Unfortunately we do not have a single answer for this, but the answer is dependant on the use case as well as the database qualities. In a way the best solution would be to completely isolate the transaction, but in practice this can affect the database usage negatively.

## Transaction isolation levels
The SQL standard defines the transaction isolation levels as follows:

* Level 1 (read uncommitted)
It is permitted that a transaction can see the changes done by another transaction, even though the other transaction is not yet committed (dirty read).

* Level 2 (read committed)
Unlike on level 1, the transaction can only see the changes made by another transaction, if the other transaction has been committed.

* Level 3 (repeatable read)
Requirements from level 2, and in addition if during a transaction a row is read multiple times, the content stays unchanged on each reading.

* Level 4 (serializable)
Transactions are totally isolated and the commands act as if the transactions were run one at a time in some order.

## Example

Let's examine a situation where the price for product 1 is 8 in the beginning and two users are running commands at the same time inside transactions (user 1 on the left, user 2 on the right):

```sql
BEGIN TRANSACTION;
                                       BEGIN TRANSACTION
                                       UPDATE Products SET price=5 WHERE id=1;
SELECT price FROM Products WHERE id=1;
                                       UPDATE Products SET price=7 WHERE id=1;
                                       COMMIT;
SELECT price FROM Products WHERE id=1;
COMMIT;
```
On level 1, the user 1 can get results 5 and 7 on their query, as the changes of user 2 can be shown immediately, even though the transaction of user 2 has not yet been committed.

On level 2, user 1 can get results 8 and 7, as during the first query the transaction for user 2 has not been yet committed, but has been during the second query.

On levels 3 and 4, user 1 gets the query results as 8 and 8, as this is the situation before the transaction began, and a transaction done in between cannot change the content of the read row.

## Transactions in practice

The implementation and the available isolation levels depend on the database system used. For example in SQLite the only level is 4, whereas PostgreSQL has levels 2 to 4 available, defaulting to 2.

## Why use different levels?

The isolation level 4 is clearly the best, as the changes from transactions are not shown to each other. Why do the other levels exist, and why does PostgreSQL default to level 2?

The price of good isolation is slowing down the database, or even blocking transactions, as committing transactions might cause a conflict. On the other hand in many situations a lower level of isolation is more than enough, as long as the database user is aware of it.

Best source for knowledge on transactions is once again reading the database system documentation, as well as testing it yourself. For example we could ourselves run *two* SQLite interpreters, open the same database and run transaction commands on both and observe the results.

The following conversation was run with the above setting:

```
sqlite> BEGIN TRANSACTION;
                                 sqlite> BEGIN TRANSACTION;
                                 sqlite> UPDATE Products SET price=5 WHERE id=1;
sqlite> SELECT price FROM Products WHERE id=1;
8
                                 sqlite> UPDATE Products SET price=7 WHERE id=1;
                                 sqlite> COMMIT;
                                 Error: database is locked
sqlite> SELECT price FROM Products WHERE id=1;
8
sqlite> COMMIT;
```

Here we can see the first transaction indeed gets 8 as a result for both queries. We cannot infact even commit the second transaction but get an error message `Error: database is locked` as the database is locked due to the other transaction being uncommitted. The isolation indeed works well, but we would have to try again to commit the second transaction.

For comparison, here's the same conversation with PostgreSQL and level 2:

```
user=> BEGIN TRANSACTION;
                                 user=> BEGIN TRANSACTION;
                                 user=> UPDATE Products SET price=5 WHERE id=1;
user=> SELECT price FROM Products WHERE id=1;
8
                                 user=> UPDATE Products SET price=7 WHERE id=1;
                                 user=> COMMIT;
user=> SELECT price FROM Products WHERE id=1;
7
user=> COMMIT;
``` 

Now the value changed to 7 by the second transaction is returned to the first transaction, but on the other hand both transactions are committed without a problem.