---
title: "Johdanto"
nav_order: 3
hidden: false
---

# Mikä on tietokanta?

Tietokanta *(database)* on tietokoneella oleva kokoelma tietoa, johon voidaan suorittaa hakuja ja jonka sisältöä voidaan muuttaa. Tietokantoja ovat esimerkiksi:

* nettisivuston käyttäjärekisteri
* verkkokaupan tuotteet ja varastotilanne
* pankin tiedot asiakkaista ja tilitapahtumista
* päivittäin mitatut säätiedot eri paikoissa
* lentoyhtiön lentoaikataulut ja varaustilanne

Tietokantoja on nykyään valtavasti, ja useimmat ihmiset ovat tavallisen päivän aikana yhteydessä lukuisiin tietokantoihin.

## Tietokantojen haasteet

Tietokantojen tekniseen toteutukseen liittyy monia haasteita, eikä hyvin toimivan tietokannan toteuttaminen ole helppo tehtävä. Keskeisiä haasteita ovat:

## Tiedon määrä (Amount of information)

Monessa tietokannassa on suuri määrä tietoa, johon kohdistuu jatkuvasti hakuja ja muutoksia. Miten toteuttaa tietokanta niin, että tietoon pääsee käsiksi tehokkaasti?

## Samanaikaisuus (Concurrency)

Usually a database has multiple users, who can change and get information at the same time. Why do we have to take this into account, when designing our database?

## Yllätykset (Surprises)

Tietokannan sisällön tulisi säilyä järkevänä myös yllättävissä tilanteissa. Esimerkiksi mitä tapahtuu, jos sähköt katkeavat juuri silloin, kun tietoa ollaan muuttamassa?

# Tietokantojen kehitys

Tietokantojen kehitys sai vauhtia 1970-luvulla jolloin vaihtoehtoja oli useita, mutta yhdestä tuli kaikkein suosituin: relaatiomalli ja **S**tructural **Q**uery **L**anguage, **SQL**. Relaatiotietokannat ovat olleet hyvin suosittuja siitä lähtien, ja suurin osa tietokannoista perustuvat edelleen relaatiotietokantamalleihin.

Tällä kurssille keskitymme relaatiotietokantoihin sekä tietokannan käyttäjien että teknisestä näkökulmasta. Tarkoituksena on vastata kysymykseen, *miksi* relaatiomalli on niin hyvä tapa luoda tietokantoja?

Vaikka relaatiotietokannat ovat edelleen vallalla, viimeaikoina on noussut hyviä haastajia. Yksi syistä on tarve erilaisille tietokannoille jotka ovat soveltuvampia hajautetuille järjestelmille, kuten valtaville nettisivuille.

Usein käytetty termi, *NoSQL*, viittaa tietokantoihin jotka perustuvat muuhun kuin relaatiomalliin tai SQL-kieleen. Erityisesti dokumenttitietokannat ovat olleet suosittuja viime vuosina. Vaikka keskitymmekin relaatiomalliin tällä kurssilla, on hyvä tietää muistakin vaihtoehdoista.

# Yksinkertainen tietokanta

Ennen tutustumista olemassa oleviin tietokantajärjestelmiin on hyvä miettiä, mitä tarvetta tällaisille järjestelmille on. Miksi emme voisi vain *toteuttaa tietokantaa itse* vaikkapa tallentamalla tietokannan sisällön tekstitiedostoon sopivassa muodossa?

Oletetaan että haluamme luoda tietokannan pankkia varten. Tietokannan tarkoitus on on tallettaa asiakastietoja ja tilisiirtoja heidän tileillään.

## Tietokannan rakenne (Database structure)

Tallennetaan tietokantamme tekstitiedostoon, `pankki.txt`, jonka rivit vastaavat tapahtumia tietokannassa. Meillä on kahdenlaisia tapahtumia: Voit luoda uuden tilin pankkiin tai olemassaolevan tilin saldo muuttuu. Tiedoston sisältö voisi olla vaikka seuraava:

```
CREATE ACCOUNT
NUMBER: 131778223
OWNER: Uolevi
CREATE ACCOUNT
NUMBER: 175299717
OWNER: Maija
MAKE TRANSACTION
ACCOUNT: 131778223
SUM: 500
MAKE TRANSACTION
ACCOUNT: 131778223
SUM: -100
MAKE TRANSACTION
ACCOUNT: 175299717
SUM: 100
```

Tässä tapauksessa olemme luoneet tilit Maijalle ja Uoleville. Tämän jälkeen Uolevin tilille siirretään 500 euroa. Seuraavaksi Uolevi siirtää 100 euroa Maijan tilille (eli Uolevin saldo laskee 100:lla ja Maijan saldo nousee vastaavasti 100:lla). Lopussa Uolevin saldo on 400 ja Maijan on 100 euroa.

Ideana siis on, että jokainen siirto lisää kolme riviä tiedoston loppuun, jotka yhdessä vastaavat tilitapahtumaa. Kun luemme tiedoston rivit, saamme selville kaikkien tilien tiedot ja pystymme laskemaan ajantasaiset saldot kaikille.

## "Tein itse ja säästin?"

<Note>TÄHÄN PÄÄSTIIN</Note>

This kind of a database works fine, if we can assume the bank having only a few customers and transaction, and only one customer uses the bank at a time and the computer is always reliable. Unfortunately, this is usually not the case.

## Amount of information

What happens, if the bank has a million customers, and the accounts of said customers have on average five transactions per day? This way we have to add 15 million lines to the file per day, or more than five billion lines in a year.

There is a critical design flaw conserning efficiency in our database: when we want to know the balance of a customer, we have to go through all the lines in the file. When the database grows, this becomes extremely slow.

In practice, problems with efficiency are shown to customers as the software using the database working slowly. For example in this case, Maija would probably wais for a minute, before they could see their balance from the bank.

## Concurrency

Many systems work fine as long as they have only one user, but concurrent users can cause problems. What happens for example, if two users try to create accounts at the same time?

Let's assume we want to create accounts for Liisa and Aapeli, and write the following lines into the file:

```
CREATE ACCOUNT
NUMBER: 185421761
OWNER: Liisa
CREATE ACCOUNT
NUMBER: 111562714
OWNER: Aapeli
```

Alas, as two users are writing into the database at the same time, the lines could *mix up* and we could end up with the following file:

```
CREATE ACCOUNT
NUMBER: 185421761
CREATE ACCOUNT
NUMBER: 111562714
OWNER: Aapeli
OWNER: Liisa
```

Now Liisa has put in the first two lines, then Aapeli all three of theirs and finally one more line from Liisa. With this it is impossible to know, which account is whose, and the data in our database is *corrupted*.

## Suprises

One more problem is that computers can go off *at any time* (for example a power outage, someone plugs it off the socket, etc). What happens, if at exactly that point we are making changes to our database?

Let's have the following scenario, where Maija transfers 50 euros to Uolevi:

```
MAKE TRANSACTION
ACCOUNT: 175299717
SUM: -50
MAKE TRANSACTION
ACCOUNT: 131778223
SUM: 50
```

However the computer dies after three lines, where only the following lines are saved:

```
MAKE TRANSACTION
ACCOUNT: 175299717
SUM: -50
```

Now Maija has lost 50 euros and Uolevi did not get anything, or 50 euros just *vanished* and this cannot even be seen from the file, as the power comes back.

## How do we fix this?

Handling a database is a tricky task, and on this course we will not try to achieve everything ourselves, but rely on existing database systems.

# Relational models

Relational models replaced their competitors in the 1970's and has ruled the database world ever since. The model is based on two basic concepts: all the data is stored in the database as *rows*, which can *reference* to one another, and the database user handles information with *SQL* language, which hides the internal details of the database from the user.

## Database structure

Relational database consists of *tables* which have fixed *columns*. The data is stored in the tables as *rows*, which have certain values in the columns. Each table contains information related to a certain collection.

In the following tables we have an example of a database, which could be used as a part of a web store. In the tables `Products`, `Customers` and `Purchases` we have information about products, customers and their shopping carts.

```
 Products 
 id  name     price
 --  ------   -----
 1   radish   7  
 2   carrot   5  
 3   turnip   4  
 4   parsley  8  
 5   celery   4  

 Customers 
 id  name  
 --  ------
 1   Uolevi
 2   Maija
 3   Aapeli

 Purchases 
 customer_id  product_id
 -----------  ----------
 1            2
 1            5
 2            1
 2            4
 2            5
```


In the tables `Products` and `Customers` each row has a unique *id* number, with which they can be referenced at. Because of this, the table `Purchases` we can represent in number form, which items which customer has chosen. In this example, Uolevi has carrots and celery in their basket, and Maija har redish, turnip and celery in theirs.

## SQL Language

The *Structured Query Language* or *SQL* is an established way to handle data in relational databases. The language as commands, with which each database user (for example the programmer or software using the database) can add, search, change and remove information.

SQL commands are formed with *keywords* such as `SELECT` and `WHERE`, names of tables and columns and other values. For example, the following command

```sql
SELECT price FROM Products WHERE name='radish';
```

Fetches the price for radish from the database described above. The command end with a semicolon `;` and we can use whitespace and linebreaks the way we want. For example, we could write the command above like this as well:


```sql
SELECT price 
FROM Products 
WHERE name='radish';
```

As SQL is from the ages of old, it has plenty of resemblance to the programming of that time. For example

* The keywords are complete English words, and the commands resemble English sentences
* The keywords are not casesensitive. For example, `SELECT`, `select` and `SeLeCT` are all the same. The keywords are usually written with all capital letters.
* The equality sign `=` means both equality and setting a value (and `== ` is not used at all).

SQL language has a *standard*, which aims to give common base for the langauge. In practice every implementation does something their own way. We try ton concentrate on the properties which are common for all SQL databases.

## Internal structure of a database

Between the user and the database there is a *relational database management system*, whose function is to handle the SQL commands from the user. For example when the user gives a command which retrieves information from the database, the database system has to find a good way to handle the command and deliver the results back to the user as fast as possible.

The nice idea behind SQL is that the user only has to *describe* what information they want, and the database system does all the work and fetches the information from the database. This is convenient for the user, because they don't have to know anything about the internal functionality but can rely on the database system to function.

Designing a database system is a complex task, and only those really interested are aware of the internal details of database systems. Most of this is outside the scope of this course, but we'll have a glimpse of that in parts 6 and 7.