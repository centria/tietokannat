---
title: "Lisää tekniikoita"
nav_order: 6
hidden: false
---

# SQL:n omiaisuudet

SQL-kielessä esiintyy tyyppejä ja lausekkeita samaan tapaan kuin ohjelmoinnissa. Olemme jo nähneet monia esimerkkejä SQL-komennoista, mutta nyt on hyvä hetki tutustua syvällisemmin kielen rakenteeseen.



# Tyypit (Types)

Jokainen tietokantajärjestelmä toteuttaa tyypit ja lausekkeet vähän omalla tavallaan ja tietokantojen toiminnassa on paljon pieniä eroja. Käytännössä kuitenkin tyypit `INTEGER` kokonaisluvuille ja `TEXT` tekstille riittävät melko pitkälle.


## TEXT vs. VARCHAR

Perinteikäs tapa tallentaa merkkijono SQL:ssä on käyttää tyyppiä `VARCHAR`, jossa annetaan suluissa merkkijonon maksimipituus. Esimerkiksi tyyppi `VARCHAR(100)` tarkoittaa, että merkkijonossa voi olla enintään 100 merkkiä.

Tämä on muistuma vanhan ajan ohjelmoinnista, jossa merkkijono saatettiin esittää kiinteän pituisena merkkitaulukkona. Tyyppi `TEXT` on kuitenkin mukavampi, koska siinä ei tarvitse keksiä maksimipituutta.

## DATE, DATETIME, TIME, TIMESTAMP...

Erittäin hyödyllisiä tyyppejä ovat ajan ja päivämäärän tallentamiseen tarkoitetut tyypit, kuten `DATE` ja `TIME`. Näiden nimeäminen riippuu vahvasti tietokantajärjestelmästä, joten parasta lienee tarkistaa yksityiskohdat käyttämäsi tietokantajärjestelmän dokumentaatiosta.


# Lausekkeet (Statements)

Lauseke on SQL-komennon osa, jolla on tietty arvo. Esimerkiksi kyselyssä

```sql
SELECT price FROM Products WHERE name='radish';
```

on neljä lauseketta: `name`, `price`, `'radish'` ja `name='radish'`. Lausekkeet hinta ja nimi saavat arvonsa rivin sarakkeesta, lauseke `'radish'` on merkkijonovakio ja lauseke `name='radish'` on totuusarvoinen.



Has two statements: `price` and `name='radish'`. In this the statement `price` defines the content of the result set, and the statement `name='radish'` is a conditional statement to limit the query.

Voimme rakentaa monimutkaisempia lausekkeita samaan tapaan kuin ohjelmoinnissa. Esimerkiksi kysely

```sql
SELECT price*2 FROM Products WHERE name='radish';
```

palauttaa retiisin hinnan kaksinkertaisena.

Hyvä tapa testata SQL:n lausekkeiden toimintaa on keskustella tietokannan kanssa tekemällä kyselyitä, jotka eivät hae tietoa mistään taulusta vaan laskevat vain tietyn lausekkeen arvon. Keskustelu voi näyttää vaikkapa seuraavalta:

```
sqlite> SELECT 2*(1+3);
8
sqlite> SELECT 'tes' || 'ts';
tests
sqlite> SELECT 3 < 5;
1
```

Ensimmäinen kysely laskee lausekkeen `2*(1+3)` arvon. Toinen kysely yhdistää `||`-operaattorilla merkkijonot 'tes' ja 'ti' merkkijonoksi 'testi'. Kolmas kysely puolestaan määrittää ehtolausekkeen `3 < 5` arvon. Tästä näkee, että SQLitessä kokonaisluku ilmaisee totuusarvon: `1 on tosi` ja `0 on epätosi`.

Monet SQL:n lausekkeisiin liittyvät asiat ovat tuttuja ohjelmoinnista:


* laskutoimitukset: `+`, `-`, `*`, `/`, `%`
* vertailut: `=`, `<>`, `<`, `<=`, `>`, `>=`
* ehtojen yhdistys: `AND`, `OR`, `NOT`

Näiden lisäksi SQL:ssä on kuitenkin myös erikoisempia ominaisuuksia, joiden tuntemisesta on välillä hyötyä. Seuraavassa on joitakin niistä:

## BETWEEN

Lauseke `X BETWEEN a AND b` on tosi, jos `X` on vähintään `a` ja enintään `b`. Esimerkiksi kysely

```sql
SELECT * FROM Products WHERE price BETWEEN 4 AND 6;
```

hakee tuotteet, joiden hinta on vähintään 4 ja korkeintaan 6. Voimme toki kirjoittaa samalla tavalla toimivan kyselyn myös näin:

```sql
SELECT * FROM Products WHERE price >= 4 AND price <= 6;
```

## CASE

Rakenne `CASE` mahdollistaa ehtolausekkeen tekemisen. Siinä voi olla yksi tai useampi `WHEN`-osa sekä mahdollinen `ELSE`-osa. Esimerkiksi kysely

```sql
SELECT name, 
  CASE WHEN price>5 THEN 'expensive' 
    ELSE 'cheap' 
  END 
FROM Products;
```

hakee kunkin tuotteen nimen sekä tiedon siitä, onko tuote kallis vai halpa. Tässä tuote on kallis, jos sen hinta on yli 5, ja muuten halpa. 

## IN
Lauseke` x IN (...)` on tosi, jos `x` on jokin annetuista arvoista. Esimerkiksi kysely



```sql
SELECT * FROM Products WHERE name IN ('turnip','cucumber','celery');
```

hakee tuotteet, joiden nimi on turnipsi, kurkku tai selleri.


## LIKE

Lauseke `s LIKE p` on tosi, jos merkkijono `s` vastaa kuvausta `p`. Kuvauksessa voi käyttää erikoismerkkejä `_` (mikä tahansa yksittäinen merkki) sekä `%` (mikä tahansa määrä mitä tahansa merkkejä). Esimerkiksi kysely

```sql
SELECT * FROM Products WHERE name LIKE '%er%';
```

hakee tuotteet, joiden nimen osana esiintyy merkkijono “er” (kuten cucumber ja celery, eli kurkku ja selleri).


## NULL

`NULL`-arvojen käsittelyyn on oma syntaksinsa. Esimerkiksi


```sql
SELECT * FROM Products WHERE price IS NULL;
```

hakee tuotteet joille ei ole vielä asetettu hintaa, ja kysely


```sql
SELECT * FROM Products WHERE price IS NOT NULL;
```
hakee tuotteet joille on hinta asetettuna.


## Funktiot

Lausekkeiden osana voi esiintyä myös funktioita samaan tapaan kuin ohjelmoinnissa. Tässä on esimerkkinä joitakin SQLiten funktioita. Kuten tyyppien kanssa, funktioiden saatavuus ja käyttö on riippuvainen tietokantajärjestelmästä, ja lisätiedot kannattaa tarkastaa käytössä olevan järjestelmän dokumentaatiosta.

Tässä on muutamia hyödyllisiä SQLiten funktioita:

```
nimi          funktio
--------      ----------
ABS(x)        antaa luvun x itseisarvon
COALESCE(...) palauttaa listasta ensimmäisen arvon, joka ei ole NULL
LENGTH(s)     antaa merkkijonon s pituuden
LOWER(s)      muuttaa merkkijonon s kirjaimet pieniksi
MAX(x,y)      antaa suuremman luvuista x ja y
MIN(x,y)      antaa pienemmän luvuista x ja y
RANDOM()      palauttaa satunnaisen luvun
ROUND(x,d)    antaa luvun x pyöristettynä d desimaalin tarkkuudelle
UPPER(s)      muuttaa merkkijonon s kirjaimet suuriksi
```

Esimerkiksi kysely

```sql
SELECT * FROM Products WHERE LENGTH(name)=6;
```

hakee tuotteet, joiden nimessä on kuusi kirjainta (kuten carrot, radish, turnip ja celery). Kysely

```sql
SELECT * FROM Products ORDER BY RANDOM();
```

puolestaan antaa rivit satunnaisessa järjestyksessä, koska järjestys ei perustu minkään sarakkeen sisältöön vaan satunnaiseen arvoon.

# Alikyselyt (Subqueries)

*Alikysely* on SQL-komennon osana oleva lauseke, jonka arvo syntyy jonkin kyselyn perusteella. Voimme rakentaa alikyselyjä samaan tapaan kuin varsinaisia kyselyjä ja toteuttaa niiden avulla hakuja, joita olisi vaikea saada aikaan muuten.


## Esimerkki

Tarkastellaan esimerkkinä tilannetta, jossa tietokannassa on pelaajien tuloksia taulussa `Results`. Oletamme, että taulun sisältö on seuraava:

```
id          name        score     
----------  ----------  ----------
1           Uolevi      120       
2           Maija       80        
3           Liisa       120       
4           Aapeli      45        
5           Kaaleppi    115    
``` 

Haluamme nyt selvittää ne pelaajat, jotka ovat saavuttaneet korkeimman tuloksen, eli kyselyn tulisi palauttaa Uolevi ja Liisa. Saamme tämän aikaan alikyselyllä seuraavasti:



```sql
SELECT name, score FROM Results 
WHERE score = (SELECT MAX(score) FROM Results);
```

Tuloksena on:

```
name        score     
----------  ----------
Uolevi      120       
Liisa       120       
```

Tässä kyselyssä alikysely on `SELECT MAX(score) FROM Results`, joka antaa suurimman taulussa olevan tuloksen eli tässä tapauksessa arvon 120. Huomaa, että alikysely tulee kirjoittaa sulkujen sisään, jotta se ei sekoitu pääkyselyyn.

Tässä vähän monimutkaisempi kysely:

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
