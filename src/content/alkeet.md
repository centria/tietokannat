---
title: "SQL:n alkeet"
nav_order: 4
hidden: false
---

# Basic commands

Tässä luvussa tutustumme tavallisimpiin SQL-komentoihin, joiden avulla voimme lisätä, hakea, muuttaa ja poistaa tietokannan sisältöä. Nämä komennot muodostavat perustan tietokannan käyttämiselle. Yleisesti alalla näitä toimintoja kutsutaan englaninninkielisillä nimillä.  **C**reate, **R**ead, **U**pdate and **D**elete, eli lyhenteenä **CRUD**, muodostavat tietokannan perustoiminnot, erityisesti dokumentaatiossa.

## Taulun luonti

Komento `CREATE TABLE` luo taulun, jossa on halutut sarakkeet. Esimerkiksi seuraava komento luo taulun `Products`, jossa on kolme saraketta:


```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);
```

Voimme nimetä taulun ja sarakkeet haluamallamme tavalla. Tällä kurssilla käytäntönä on, että kirjoitamme taulun nimen suurella alkukirjaimella ja monikkomuotoisena. Sarakkeiden nimet puolestaan kirjoitamme pienellä alkukirjaimella.

Jokaisesta sarakkeesta ilmoitetaan nimen lisäksi tyyppi. Tässä taulussa sarakkeet `id` ja `price` ovat kokonaislukuja (INTEGER) ja sarake `name` on merkkijono (TEXT). Sarake `id` on lisäksi taulun pääavain (PRIMARY KEY), mikä tarkoittaa, että se yksilöi jokaisen taulun rivin ja voimme viitata sen avulla kätevästi mihin tahansa riviin.


## Pääavain

Tietokannan taulun pääavain on jokin sarake (tai sarakkeiden yhdistelmä), joka yksilöi taulun jokaisen rivin eli millään kahdella rivillä ei ole samaa pääavainta. Käytännössä hyvin tavallinen valinta pääavaimeksi on kokonaislukumuotoinen id-numero.

Usein haluamme lisäksi, että id-numerolla on juokseva numerointi. Tämä tarkoittaa, että kun tauluun lisätään rivejä, ensimmäinen rivi saa automaattisesti id-numeron 1, toinen rivi saa id-numeron 2, jne. 

Juoksevan numeroinnin toteuttaminen riippuu tietokantajärjestelmästä. Esimerkiksi SQLite-tietokannassa `INTEGER PRIMARY KEY` -tyyppinen sarake saa automaattisesti juoksevan numeroinnin.


## Tiedon lisääminen

Komento `INSERT` lisää uuden rivin tauluun. Esimerkiksi seuraava komento lisää rivin äsken luomaamme tauluun `Products`

```sql
INSERT INTO Products (name,price) VALUES ('radish',7);
```

Tässä annamme arvot lisättävän rivin sarakkeille `name` ja `price`. Kun sarakkeessa `id` on juokseva numerointi, se saa automaattisesti arvon 1, kun kyseessä on taulun ensimmäinen rivi. Niinpä tauluun ilmestyy seuraava rivi:


```
id          name        price     
----------  ----------  ----------
1           radish     7         
```

Jos emme anna arvoa jollekin sarakkeelle, se saa oletusarvon. Tavallisessa sarakkeessa oletusarvo on `NULL`, mikä tarkoittaa tiedon puuttumista. Esimerkiksi seuraavassa komennossa emme anna arvoa sarakkeelle `price`:


```sql
INSERT INTO Products (name) VALUES ('radish');
```

Tällöin tauluun ilmestyy rivi, jossa hinta on `NULL` (eli tyhjä):



```
id          name        price     
----------  ----------  ----------
1           radish     
```

## Esimerkkitaulu

Oletamme tämän osion tulevissa esimerkeissä, että olemme lisänneet tauluun `Products` seuraavat viisi riviä:

```sql
INSERT INTO Products (name,price) VALUES ('radish',7);
INSERT INTO Products (name,price) VALUES ('carrot',5);
INSERT INTO Products (name,price) VALUES ('turnip',4);
INSERT INTO Products (name,price) VALUES ('cucumber',8);
INSERT INTO Products (name,price) VALUES ('celery',4);
```

Taulun sisältö on siis seuraavanlainen:

```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4       
```  

## Tiedon hakeminen

Komento `SELECT` suorittaa *kyselyn* (*query*) eli hakee tietoa taulusta. Yksinkertaisin tapa tehdä kysely on hakea kaikki tiedot taulusta:

```sql
SELECT * FROM Products;
```

Tässä tapauksessa kyselyn tulos on seuraava:

```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4       
```  

Kyselyssä tähti `*` ilmaisee, että haluamme hakea kaikki sarakkeet. Kuitenkin voimme myös hakea vain osan sarakkeista. Esimerkiksi seuraava kysely hakee vain tuotteiden nimet:

```sql
SELECT name FROM Products;
```

Kyselyn tulos on seuraava:

```
name      
----------
radish    
carrot             
turnip               
cucumber               
celery    
```

Tämä kysely puolestaan hakee nimet ja hinnat:

```sql
SELECT name, price FROM Products;
```

Kyselyn tulos on nyt seuraavanlainen:

```sql
name        price     
----------  ----------
radish      7         
carrot      5         
turnip      4         
cucumber    8         
celery      4         
```

Kyselyn tuloksena olevat rivit muodostavat taulun, jota kutsutaan nimellä *tulostaulu* (`result set`). Sen sarakkeet ja rivit riippuvat kyselyn sisällöstä. Esimerkiksi äskeinen kysely loi tulostaulun, jossa on kaksi saraketta ja viisi riviä.

Tietokannan käsittelyssä esiintyy siis kahdenlaisia tauluja: tietokannassa kiinteästi olevia tauluja, joihin on tallennettu tietokannan sisältö, sekä kyselyjen muodostamia väliaikaisia tulostauluja, joiden tiedot on koostettu kiinteistä tauluista.

## Hakuehdot (Search clauses)

Liittämällä `SELECT`-kyselyyn `WHERE`-osan voimme valita vain osan riveistä halutun ehdon perusteella. Esimerkiksi seuraava kysely hakee tiedot kurkusta:

```sql
SELECT * FROM Products WHERE name='cucumber';
```

Kyselyn tulos on seuraava:

```
id          name        price     
----------  ----------  ----------
4           cucumber    8        
```

Ehdoissa voi käyttää vertailuja ja sanoja `AND` ja `OR` samaan tapaan kuin ohjelmoinnissa. Esimerkiksi seuraava kysely etsii tuotteet, joiden hinta on välillä 4...6:

```sql
SELECT * FROM Products WHERE price>=4 AND price<=6;
```

Kyselyn tulos on seuraava:

```
id          name        price     
----------  ----------  ----------
2           carrot      5         
3           turnip      4         
5           celery      4         
```

## Järjestäminen

Oletuksena kyselyn tuloksena olevien rivien järjestys voi olla mikä tahansa. Voimme kuitenkin määrittää halutun järjestyksen `ORDER BY` -osan avulla. Esimerkiksi seuraava kysely hakee tuotteet järjestyksessä nimen mukaan:

```sql
SELECT * FROM Products ORDER BY name;
```

Kyselyn tulos on seuraava:  

```
id          name        price     
----------  ----------  ----------
2           carrot      5
5           celery      4
4           cucumber    8         
1           radish      7
3           turnip      4         
```

Järjestys on oletuksena pienimmästä suurimpaan (**ASC**ENDING). Kuitenkin jos haluamme järjestyksen suurimmasta pienimpään, voimme lisätä sanan `DESC` (**DESC**ENDING) sarakkeen nimen jälkeen:

```sql
SELECT * FROM Products ORDER BY name DESC;
```

Tämän seurauksena kyselyn tulos on seuraava:

```
id          name        price     
----------  ----------  ----------
3           turnip      4  
1           radish      7
4           cucumber    8
5           celery      4
2           carrot      5
```

SQL-kielessä on myös avainsana `ASC`, joka tarkoittaa nousevaa järjestystä. Seuraavat kyselyt toimivat siis samalla tavalla:

```sql
SELECT * FROM Products ORDER BY name;
SELECT * FROM Products ORDER BY name ASC;
```

Käytännössä sanaa `ASC` käytetään kuitenkin äärimmäisen harvoin.

Voimme myös järjestää rivejä usealla eri perusteella. Esimerkiksi seuraava kysely järjestää rivit ensisijaisesti kalleimmasta halvimpaan hinnan mukaan ja toissijaisesti aakkosjärjestykseen nimen mukaan:

```sql
SELECT * FROM Products ORDER BY price DESC, name;
```

Kyselyn tulos on seuraava:

```
id          name        price     
----------  ----------  ----------
4           cucumber    8         
1           radish      7         
2           carrot      5         
5           celery      4
3           turnip      4    
```     

Tässä tapauksessa turnipsi ja selleri järjestetään nimen mukaan, koska ne ovat yhtä kalliita.

## Erilliset tulosrivit (Distinct result rows)

Joskus tulostaulussa voi olla useita samanlaisia rivejä. Näin käy esimerkiksi seuraavassa kyselyssä:

```sql
SELECT price FROM Products;
```

Koska kahden tuotteen hinta on 4, kahden tulosrivin sisältönä on 4:

```
price     
----------
7         
5         
4         
8         
4  
```

Jos kuitenkin haluamme vain erilaiset tulosrivit, voimme lisätä kyselyyn sanan `DISTINCT`:

```sql
SELECT DISTINCT price FROM Products;
```

Tämän seurauksena kyselyn tulos muuttuu näin:

```
price     
----------
7         
5         
4         
8         
```

##  Tiedon muuttaminen (Changing information)

Komento `UPDATE` muuttaa taulun rivejä, jotka täsmäävät haluttuun ehtoon. Esimerkiksi seuraava komento muuttaa tuotteen `turnip` hinnaksi 6:

```sql
UPDATE Products SET price=6 WHERE name='turnip';
```

Useita sarakkeita voi muuttaa yhdistämällä muutokset pilkuilla. Esimerkiksi seuraava komento muuttaa tuotteen `turnip` nimeksi `pineapple` ja hinnaksi 9:

```sql
UPDATE Products SET name='pineapple', price=9 WHERE name='turnip';
```

Muutos voidaan myös laskea aiemman arvon perusteella. Esimerkiksi seuraava komento kasvattaa turnipsin hintaa yhdellä:

```sql
UPDATE Products SET price=price+1 WHERE name='turnip';
```

Jos komennossa ei ole ehtoa, se vaikuttaa *kaikkiin* riveihin. Esimerkiksi seuraava komento kasvattaa jokaisen tuotteen hintaa yhdellä:

```sql
UPDATE Products SET price=3;
```

## Tiedon poistaminen (Removing information)

Komento `DELETE` poistaa taulusta rivit, jotka täsmäävät annettuun ehtoon. Esimerkiksi seuraava komento poistaa taulusta tuotteen nimeltä `carrot`:

```sql
DELETE FROM Products WHERE name='carrot';
```

Kuten muuttamisessa, jos ehtoa ei ole, niin komento vaikuttaa kaikkiin riveihin. Seuraava komento siis poistaa kaikki tuotteet taulusta:

```sql
DELETE FROM Products;
```

Komento `DROP TABLE` poistaa tietokannan taulun (ja kaiken sen sisällön). Esimerkiksi seuraava komento poistaa taulun `Products`:


```sql
DROP TABLE Products;
```






# Yhteenveto ja ryhmittely (Aggregate queries)  

Yhteenvetokysely laskee jonkin yksittäisen arvon taulun riveistä, kuten taulun rivien määrän tai sarakkeen kaikkien arvojen summan. Tällaisen kyselyn tulostaulussa on vain yksi rivi. VOimme myös ryhmitellä rivejä sarakkeiden mukaan ja tehdä yhteenvetoja jokaisesta ryhmästä.

## Koostefunktiot (Aggregate functions)

Yhteenvetokyselyn perustana on koostefunktio, joka laskee yhteenvetoarvon taulun riveistä. Tavallisimmat koostefunktiot ovat seuraavat:

```
name          function
---------     ---------------------------
COUNT()       laskee rivien määrän
SUM()         laskee summan rivien arvoista
MIN()         noutaa pienimmän arvon
MAX()         noutaa suurimman arvon
AVG()         laskee keskiarvon
```
## Esimerkkejä

Tarkastellaan taas taulua `Products`:

```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           carrot      5         
3           turnip      4         
4           cucumber    8         
5           celery      4       
```        

Seuraava kysely hakee taulun rivien määrän:

```sql
SELECT COUNT(*) FROM Products;
```

```
COUNT(*)
----------
5
```

Seuraava kysely hakee niiden rivien määrän, joissa hinta on 4:

```sql
SELECT COUNT(*) FROM Products WHERE price=4;
```

```
COUNT(*)
----------
2
```

Seuraava kysely puolestaan laskee summan tuotteiden hinnoista:

```sql
SELECT SUM(price) FROM Products;
```

```
SUM(price)
----------
28
```

## Rivien valinta (Selecting rows)

Jos koostefunktion sisällä on tähti `*`, kysely laskee kaikki rivit. Jos taas funktion sisällä on sarakkeen nimi, kysely laskee rivit, joissa sarakkeessa on arvo (eli sarake ei ole `NULL`).

Tarkastellaan esimerkkinä seuraavaa taulua, jossa rivillä 3 ei ole hintaa:

```
id          name        price     
----------  ----------  ----------
1           radish      7         
2           turnip      4         
3           cucumber               
4           celery      4         
```

Seuraava kysely hakee rivien yhteismäärän:

```sql
SELECT COUNT(*) FROM Products;
```

```
COUNT(*)  
----------
4
```

Seuraava kysely taas hakee niiden rivien määrän, joilla on hinta:

```sql
SELECT COUNT(price) FROM Products;
```

```
COUNT(price)
------------
3
```

Voimme myös käyttää sanaa `DISTINCT`, jotta saamme laskettua, montako eri arvoa jossakin sarakkeessa on:

```sql
SELECT COUNT(DISTINCT price) FROM Products;
```

```
COUNT(DISTINCT price)
---------------------
2
```

## Ryhmittely (Grouping)

Ryhmittelyn avulla voimme yhdistää rivikohtaista ja koostefunktion antamaa tietoa. Ideana on, että rivit jaetaan ryhmiin `GROUP BY` -osassa annettujen sarakkeiden mukaan ja tämän jälkeen koostefunktion arvo lasketaan jokaiselle ryhmälle erikseen.


Tarkastellaan esimerkkinä seuraavaa taulua `Sales`, jossa on eri vuosien myyntitietoja:

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

Seuraava kysely palauttaa kokonaismyynnin jokaiselta vuodelta ryhmiteltynä:


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

Esimerkiksi vuoden 2017 kokonaismyynti on 120 + 30 + 75 = 225.

Toisaalta, voimme saada tuotteiden kokonaismyynnin näin:

```sql
SELECT product, SUM(amount) FROM Sales GROUP BY product;
```

Kyselyn tulos on seuraava:

```
product       SUM(amount)
----------  ----------
cucumber    255       
turnip      75        
radish      355
```

Esimerkiksi kurkun kokonaismyynti on 75 + 100 + 80 = 255.

## Tulossarakkeen nimentä

Oletuksena tulostaulun sarake saa nimen suoraan kyselyn perusteella, mutta voimme halutessamme antaa myös oman nimen `AS`-sanan avulla. Tämän ansiosta voimme esimerkiksi selventää, mistä yhteenvetokyselyssä on kyse.

Esimerkiksi seuraavassa kyselyssä toisen sarakkeen nimeksi tulee `total`:

```sql
SELECT product, SUM(amount) AS total FROM Sales GROUP BY product;
```

Kysely palauttaa seuraavaa:

```
product     total
----------  --------
cucumber    255       
turnip      75        
radish      355
```

Itseasiassa sana `AS` ei ole pakollinen, eli voisimme kirjoittaa kyselyn myös näin:

```sql
SELECT product, SUM(amount) total FROM Sales GROUP BY product;
```

## Rajaus ryhmittelyn jälkeen (Limitation after grouping)

Voimme lisätä avainsanan `HAVING` kyselyymme, joka rajoittaa tuloksia ryhmittelyn *jälkeen*. Esimerkiksi seuraava kysely palauttaa tuotteet, joiden myynti on vähintään 200:

```sql
SELECT product, SUM(amount) AS total
FROM Sales
GROUP BY product
HAVING total >= 200;
```

Kysely palauttaa seuraavasti:

```
product     total
----------  --------
cucumber    255       
radish      355     
```

## Kyselyn yleiskuva (Query overview)

Kyselyissämme voimme käyttää useita avainsanoja joita olemme tähän mennessä oppineet, kunhan ne ovat seuraavassa järjestyksessä:


```
SELECT – FROM – WHERE – GROUP BY – HAVING – ORDER BY
```

Seuraavassa on esimerkki kyselystä, joka sisältää yhtä aikaa kaikki nämä osat.

```sql
SELECT product, SUM(amount) AS total
FROM Sales
WHERE year < 2019
GROUP BY product
HAVING total >= 100
ORDER BY product;
```

Kysely palauttaa tuotteiden myyntitiedot ennen vuotta 2019, näyttää ainoastaan tuotteet joiden myynti näinä vuosina on yli 100, ja järjestää ne nimen mukaan. Kysely palauttaa seuraavaa:

```
product       total
----------  --------
cucumber    175       
radish      205      
```

Huomaa ero `WHERE` ja `HAVING` välillä: `WHERE` rajoittaa rivejä *ennen* ryhmittelyä, kun `HAVING` rajoittaa tuloksia ryhmittelyn *jälkeen*.


# SQLite-tietokanta

*SQLite* on yksinkertainen avoimesti saatavilla oleva tietokantajärjestelmä, joka soveltuu hyvin SQL-kielen opetteluun. Voit kokeilla helposti SQL-kieleen liittyviä asioita SQLiten avulla, ja käytämme sitä tämän kurssin esimerkeissä.

## Tietokantajärjestelmät (Database systems)

SQLite on mainio valinta SQL-kielen harjoitteluun, mutta siinä on tiettyjä rajoituksia, jotka voivat aiheuttaa ongelmia todellisissa sovelluksissa. 

Muita suosittuja avoimia tietokantajärjestelmiä ovat *MySQL* ja *PostgreSQL*. Niissä on suuri määrä ominaisuuksia, jotka puuttuvat SQLitestä, mutta toisaalta niiden asentaminen ja käyttäminen on vaikeampaa.

Eri tietokantajärjestelmien välillä siirtyminen on onneksi helppoa, koska kaikissa on samantapainen SQL-kieli. 

## SQLite-tulkki (SQLite interpreter)

`SQLite-tulkki` on ohjelma, jonka kautta voidaan käyttää SQLite-tietokantaa. Tulkki käynnistyy antamalla komentorivillä komento `sqlite3`. Tämän jälkeen tulkkiin voi kirjoittaa joko suoritettavia SQL-komentoja tai pisteellä alkavia SQLite-tulkin omia komentoja.

Jos käyttämälläsi koneella ei ole vielä SQLite-tulkkia, voit asentaa sen tästä:
[https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)

Valitse oman käyttöjärjestelmäsi mukainen paketti, jonka vieressä on otsikko *command-line tools* (eli komentorivityökalut). Tarvittava tiedosto on se, jonka nimi alkaa *sqlite3*.

## Esimerkki

SQLite-tulkissa tietokanta on oletuksena muistissa (*in-memory database*). Tämä tarkoittaa, että se on aluksi tyhjä ja katoaa, kun tulkki suljetaan. Tämä on hyvä tapa testailla SQL-kielen ominaisuuksia. Keskustelu tulkin kanssa voi näyttää vaikkapa tältä (tähän on lisätty ylimääräisiä rivinvaihtoja luettavuuden takia):


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

Esimerkissä luomme aluksi taulun `Products` ja tarkastamme sitten komennolla `.tables`, mitä tauluja tietokannassa on. Ainoa taulu on `Products`, mikä kuuluu asiaan.

Tämän jälkeen lisäämme tauluun rivejä ja haemme sitten kaikki rivit taulusta. SQLite-tulkin oletustapa näyttää tulosrivit pystyviivoin erotettuina ei ole kovin tyylikäs, minkä vuoksi parannamme tulostusta komennoilla `.mode column` (jokaisella sarakkeella on kiinteä leveys) ja `.headers on` (sarakkeiden nimet näytetään).

Lopuksi suoritamme komennon `.quit`, joka sulkee SQLite-tulkin.

## Tietokanta tiedostossa


Käynnistyksen yhteydessä SQLite-tulkille voi antaa parametrina tiedoston, johon tietokanta tallennetaan. Tällöin tietokannan sisältö säilyy tallessa tulkin sulkemisen jälkeen.

Seuraavassa esimerkissä tietokanta tallennetaan tiedostoon `test.db`. Tämän ansiosta tietokannan sisältö on edelleen tallessa, kun tulkki käynnistetään uudestaan.


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

## Komennot tiedostosta

Voimme myös ohjata SQLite-tulkille tiedoston, jossa olevat komennot suoritetaan peräkkäin. Tämän avulla voimme automatisoida komentojen suorittamista. Esimerkiksi voimme laatia seuraavan tiedoston `commands.sql`:

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

Tämän jälkeen voimme ohjata komennot tiedostosta tulkille näin:

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