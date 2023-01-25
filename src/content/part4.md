---
title: "More techniques"
nav_order: 6
hidden: false
---

# Properties of SQL

In SQL there are many same elements as in programming: Types, statements and functions. We have seen many examples of SQL commands, but let's loon into the language a bit deeper.

# Types

The types and their properties depend on the database system we are using. Usually there are several options even for integers and strings. In practice though, types `INTEGER` for integers and `TEXT` for strings go a long way.

## TEXT vs. VARCHAR
Traditional SQL type for saving a string is `VARCHAR`, where we give the maximum length of the string. For example `VARCHAR(100)` means a string, which can have at most 100 characters.

This is one remainder of the programming from old times: back then the strings were often saved as an array, with a fixed amount of characters. In practice `TEXT` is more versatile as we do not have to come up with the maximum amount of characters.

## DATE, DATETIME, TIME, TIMESTAMP...

Very useful types of data are the types used for saving `DATE` and `TIME`. The naming and usage vary from database system to another, so it is best to check the details from the documentation of the databasey system we are using.

# Statements

Statements are a part of SQL command, with a certain value. For example in query

```sql
SELECT price FROM Products WHERE name='radish';
```

Has two statements: `price` and `name='radish'`. In this the statement `price` defines the content of the result set, and the statement `name='radish'` is a conditional statement to limit the query.

We can also use calculations and other operators the same way as in programming. For example the query

```sql
SELECT price*2 FROM Products WHERE name='radish';
```

returns the price of the radish doubled.

A good way to test how SQL statements work is to discuss with the database by doing queries which do not search information from any tables, but only calculate values for statements. This could be something like

```
sqlite> SELECT 2*(1+3);
8
sqlite> SELECT 'tes' || 'ts';
tests
sqlite> SELECT 3 < 5;
1
```

The first query calculates the value for statement `2*(1+3)`. The second query combines with the operator `||` the strings 'tes' and 'ts'. into 'tests'. Third query defines the truth value for statement `3 < 5`. The practice in SQL is that the truth value is given as an integer: `1 is true`, and `0 is false`.

Many items for SQL statements are already familar from programming:

* calculations: `+`, `-`, `*`, `/`, `%`
* comparison: `=`, `<>`, `<`, `<=`, `>`, `>=`
* combining conditions: `AND`, `OR`, `NOT`

In addition to these SQL has some more special features, whose knowledge is useful. Next we will look into some of them:

## BETWEEN
Statement `X BETWEEN a AND b` is true, if `X` is at least `a` and at most `b`. For example the query

```sql
SELECT * FROM Products WHERE price BETWEEN 4 AND 6;
```

Returns the products, whose price is at least 6 and at maximum 6. We can also write the same query like this:

```sql
SELECT * FROM Products WHERE price >= 4 AND price <= 6;
```

## CASE
A `CASE` structure enables conditional statements. It can have one or more `WHEN` and a possible `ELSE`. For example the query

```sql
SELECT name, 
  CASE WHEN price>5 THEN 'expensive' 
    ELSE 'cheap' 
  END 
FROM Products;
```

Retrieves the name for all the products and the information about them, if they are expensive or cheap. In this query the product is expensive if the price is over 5, otherwise it is cheap.

## IN
Statement `x IN (...)` is true, if `x` is some of the given values. For example the query

```sql
SELECT * FROM Products WHERE name IN ('turnip','cucumber','celery');
```

Returns the products whose name is turnip, cucumber or celery.

## LIKE
Statement `s LIKE p` is true, if the string `s` matches to the description `p`. In the description we can use special characters `_` (any single character) and `%` (any amount of any characters). For example

```sql
SELECT * FROM Products WHERE name LIKE '%er%';
```

Returns the products whose name contain the string `er` (such as cucumber and celery).

## NULL
Handling `NULL` values we have a separate syntax. For example

```sql
SELECT * FROM Products WHERE price IS NULL;
```

retrieves the products to which no price has been set, and the query

```sql
SELECT * FROM Products WHERE price IS NOT NULL;
```

retrieves the products to which the price has been set.

## Functions

As a part of statements we can have functions, just like in programming. As with types, the functions available and their usage are dependant on the used database system and additional information should be checked from the database system documentation.

Here are some useful SQLite functions:

```
name          function
--------      ----------
ABS(x)        returns the absolute value for x
COALESCE(...) returns the first value from the list, which is not NULL
LENGTH(s)     returns the length of the string s
LOWER(s)      changes the characters in string s to lower case
MAX(x,y)      returns the greater of integers x and y
MIN(x,y)      returns the smaller of integers x and y
RANDOM()      returns a random number
ROUND(x,d)    returns x rounded to d decimals
UPPER(s)     changers the characters in string s to upper case
```

For example the query

```sql
SELECT * FROM Products WHERE LENGTH(name)=6;
```

Returns the products whose name are six characters long (such as carrot, radish, turnip and celery). The query

```sql
SELECT * FROM Products ORDER BY RANDOM();
```

Returns the all the rows in random order, since the order is not based on any column but on a random number.


# Subqueries
Subqueries are a statement used as a part of a SQL command, whose value is determined by a query. We can build subqueries in the same manner as main queries and produce queries with them, which could be difficult to achieve otherwise. 

## Example

Let's look at a situation, where the database has a table for players' scores in the table `Results`. We assume the table is following:

```
id          name        score     
----------  ----------  ----------
1           Uolevi      120       
2           Maija       80        
3           Liisa       120       
4           Aapeli      45        
5           Kaaleppi    115    
``` 

Now we want to know the players who have achieved the top score, and the query should return Uolevi and Liisa. We can achieve this by subquery like this:

```sql
SELECT name, score FROM Results 
WHERE score = (SELECT MAX(score) FROM Results);
```

And we get:

```
name        score     
----------  ----------
Uolevi      120       
Liisa       120       
```

In this scenario the subquery is `SELECT MAX(score) FROM Results`, which gives the largest score in the table, in this case 120. Notice, that the subquery has to be enclosed brackets, so that it does not mix up with the outer query.

Here's a bit more comlex query:

```sql
SELECT name, score FROM Results 
WHERE score >= 0.9*(SELECT MAX(score) FROM Results);
```

This query shows, that we can use the value from the subquery as part of a statement, just like any other value. The query retrieves the players, whose score is at most 10 percent lower than the the best score:

```
name        score     
----------  ----------
Uolevi      120       
Liisa       120       
Kaaleppi    115     
```

## Correlated or synchronized subquery

A subquery is also possible to create so, that its content is dependant or a row in the outer query. For example:

```sql
SELECT name, score, 
  (SELECT COUNT(*) FROM Results WHERE score > R.score) better 
  FROM Results R;
```

The idea for this query is to calculate for each player, how many players have a better score than them. For example for Maija the answer is 3, since Uolevi, Liisa and Kaaleppi have better scores. We get the following result set:

```
name        score       better
----------  ----------  ----------
Uolevi      120         0                                                    
Maija       80          3                                                    
Liisa       120         0                                                    
Aapeli      45          4                                                    
Kaaleppi    115         2                                                   
```

Because the table Results is in two roles in the subquery, we have given the table Results additional name R. With this in the subquery it is clear that we want to count rows, whose score is better than the row score from the outer query.

## Multiple values in subquery

Subquery can also return multiple values, as long as the result from the subquery are used in a location where this is allowed. This works in for example:

```sql
SELECT name FROM Products
WHERE id IN (SELECT product_id FROM Purchases WHERE customer_id = 1);
```

This query retrieves all the names for products in customer 1's shopping cart. The subquery returns the id values for the products, which can be combined with the IN syntax.

Notice, that we could have done the query also like this:

```sql
SELECT P.name
FROM Products P, Purchases O
WHERE P.id = O.product_id AND O.customer_id = 1;
```

Often a subquery is an alternative way to produce a query, which could just as well be done with for example properly designed multiple table query.

# Limiting results
SQL query by default returns all the rows mathcing its conditions, but we can ask for only a part of the rows when needed. This is useful for example in applications, where we only want to show a part of the results per page.

## Ways of limiting

When we add `LIMIT x` to the end of a query, the query only returns `x` first rows. For example `LIMIT 3` means, that the query only shows the first three rows of the result set.

A more common form is `LIMIT x OFFSET y`, which means that we want `x` rows starting from position `y` (with 0 indexing, of course). For example `LIMIT 3 OFFSET 1` means that the result set contains second, third and fourth rows.

## Example

Let's see an example query, which returns the products from cheapest to most expensive:

```sql
SELECT * FROM Products ORDER BY price;
```

We get the following result set:

```
id          name        price     
----------  ----------  ----------
3           cucumber    2
5           celery      4         
2           carrot      5         
1           radish      7         
4           turnip      8         
```

We can get the cheapest three as follows:

```sql
SELECT * FROM Products ORDER BY price LIMIT 3;
```

And the result is:

```
id          name        price     
----------  ----------  ----------
3           cucumber    2         
5           celery      4         
2           carrot      5      
```

The next query in turn gets the three cheapest products, starting from the second cheapest:

```sql
SELECT * FROM Products ORDER BY price LIMIT 3 OFFSET 1;
```

And the result is:

```
id          name        price     
----------  ----------  ----------
5           celery      4         
2           carrot      5         
1           radish      7      
```

## Limiting subquery

Let's look at a situation, where we want the combined price of the three cheapest products. The following query does not work like we would want:

```sql
SELECT SUM(price) FROM Products ORDER BY price LIMIT 3;
```

This returns the price of all the products in the table:

```
SUM(price)
----------
26
```

The problem is that the query forms the return set, where only one row containing the value 26 (sum of all products), after which we select the first three rows of the result set (in practice, the only row there is).

We can solve this with a subquery, where we get the three cheapest prices, and calculate the sum of these:

```sql
SELECT SUM(price) FROM (SELECT price FROM Products ORDER BY price LIMIT 3);
```

Now we get the desired result:

```
SUM(price)
----------
11
```
