---
title: "Tiedon eheys"
nav_order: 8
hidden: false
---

# Tiedon eheys (data integrity)

*Tiedon eheys (data integrity)* viittaa siihen, että tietokannassa oleva tieto on paikkansa pitävää ja ristiriidatonta. Päävastuu tiedon laadusta on toki käyttäjällä tai sovelluksella, joka muuttaa tietokantaa, mutta myös tietokannan suunnittelija voi vaikuttaa asiaan lisäämällä tauluihin ehtoja, jotka tarkkailevat tietokantaan syötettävää tietoa.

## Sarakkeiden ehdot (Column conditions)

Voimme määrittää taulun luonnin yhteydessä sarakkeisiin liittyviä ehtoja, joita tietokantajärjestelmä valvoo tiedon lisäämisen ja muuttamisen yhteydessä. Näillä ehdoilla voi ohjata sitä, millaista tietoa tietokantaan ilmestyy. Tyypillisiä ehtoja ovat seuraavat:

## UNIQUE

Ehto `UNIQUE` tarkoittaa, että kyseisessä sarakkeessa tulee olla eri arvo joka rivillä. Esimerkiksi seuraavassa taulussa vaatimuksena on, että joka tuotteella on eri nimi:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT UNIQUE, price INTEGER);
```

## NOT NULL ja DEFAULT
Ehto `NOT NULL` tarkoittaa, että kyseisessä sarakkeessa ei saa olla arvoa `NULL`. Esimerkiksi seuraavassa taulussa tuotteen hinta ei saa olla tyhjä:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER NOT NULL);
```

Tähän liittyy usein myös määre `DEFAULT`, jonka seurauksena sarake saa tietyn oletusarvon, jos sille ei anneta arvoa rivin lisäämisessä. Esimerkiksi voimme määrittää oletusarvon 0 näin:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER DEFAULT 0);
```


## CHECK

Yleisempi tapa luoda ehto on käyttää avainsanaa `CHECK`, jonka jälkeen voi kirjoittaa minkä tahansa ehtolausekkeen. Esimerkiksi seuraava komento luo taulun tuotteista, jossa rivin ehtona on `price >= 0` eli hinta ei saa olla negatiivinen:

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price CHECK (price >= 0));
```

## Ehtojen valvonta (Condition monitoring)

Ehtojen hyötynä on, että tietokantajärjestelmä valvoo niitä ja kieltäytyy tekemästä lisäystä tai muutosta, joka rikkoisi ehdon. Seuraavassa on esimerkki tästä SQLitessä.


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

Kun koetamme lisätä tauluun `Products` rivin, jossa hinta on negatiivinen, tämä rikkoo ehdon `price >= 0` ja SQLite ei salli rivin lisäämistä vaan antaa virheen `Error: CHECK constraint failed: Products`. Samalla tavalla käy, jos koetamme muuttaa olemassa olevan rivin sarakkeen hinnan negatiiviseksi jälkeenpäin.


## Viittausten ehdot (Reference conditions)

Voimme liittää myös tauluihin ehtoja, jotka pitävät huolen siitä, että tauluissa olevat viittaukset viittaavat todellisiin riveihin. Tämä tapahtuu luomalla *viiteavain (foreign key)*, joka ilmaisee, mihin taulussa oleva rivi viittaa.

Tarkastellaan esimerkkinä seuraavia tauluja:

```sql
CREATE TABLE Teachers (id INTEGER PRIMARY KEY, name TEXT);
CREATE TABLE Courses (id INTEGER PRIMARY KEY, name TEXT, teacher_id INTEGER);
```

Tässä tarkoituksena on, että taulun `Courses` sarake `teacher_id` viittaa taulun `Teachers` sarakkeeseen `id`, mutta tietokannan käyttäjä voi antaa sarakkeen `teacher_id` arvoksi mitä tahansa (esim. luvun 123), jolloin tietokannan sisältö muuttuu epämääräiseksi.

Voimme parantaa tilannetta kertomalla taulun `Courses` luonnissa, että sarake `teacher_id` on viiteavain tauluun `Teachers`:


```sql
CREATE TABLE Courses (id INTEGER PRIMARY KEY, name TEXT, teacher_id INTEGER REFERENCES Teachers);
```

Tämän jälkeen voimme luottaa siihen, että taulussa `Courses` sarakkeen `teacher_id` arvot viittaavat todellisiin riveihin taulussa `Teachers`.


## Viiteavaimet SQLitessä

Voimmeko oikeasti luottaa, että viiteavaimet toimivat halutulla tavalla?

Can we actually trust, that the reference keys work in the desired ways? Historiallisista syistä SQLite ei oletuksena valvo viiteavainten ehtoja, vaan meidän tulee ensin suorittaa seuraava komento:

```
sqlite> PRAGMA foreign_keys = ON;
```

Tämä on SQLiten erikoisuus, ja muissa tietokannoissa viiteavaimia *pitäisi* aina valvoa.

Tässä on esimerkki viiteavaimen käyttämisestä:

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

Taulussa `Teachers` on kaksi opettajaa, joiden `id`-numerot ovat 1 ja 2. Niinpä kun koetamme lisätä tauluun `Courses` rivin, jossa `teacher_id` on 123, SQLite ei salli tätä vaan saamme virheilmoituksen `Error: FOREIGN KEY constraint failed`.

## Viittaukset ja poistot

Viittausten ehtoihin liittyy tavallisia sarakkeiden ehtoja mutkikkaampia tilanteita, koska viittaukset ovat kahden taulun välisiä. Erityisesti mitä tapahtuu, jos taulusta yritetään poistaa rivi, johon viitataan toisen taulun rivillä?

Yleensä oletuksena tietokannoissa riviä ei voi poistaa, jos siihen on viittaus muualta. Esimerkiksi jos koetamme äskeisen esimerkin päätteeksi poistaa taulusta `Teachers` rivin 2, tämä ei onnistu, koska siihen viitataan taulussa `Courses`:


```sql
sqlite> DELETE FROM Teachers WHERE id=2;
Error: FOREIGN KEY constraint failed
```

Halutessamme voimme kuitenkin määrittää taulun luonnissa tarkemmin, mitä tapahtuu tässä tilanteessa. Esimerkiksi yksi vaihtoehto on `ON DELETE CASCADE`, mikä tarkoittaa, että rivin poistuessa myös siihen viittaavat rivit poistetaan. Saamme tämän aikaan näin:

```sql
CREATE TABLE Courses (id INTEGER PRIMARY KEY, name TEXT,
      teacher_id INTEGER REFERENCES Teachers ON DELETE CASCADE);
```   

Nyt jos tietokannasta poistetaan opettaja, niin samalla poistetaan automaattisesti kaikki kurssit, joita hän opettaa. Tämä voi kuitenkin olla kyseenalainen vaihtoehto, koska tämän seurauksena tietokannan tauluista voi kadota yllättäen tietoa.

## Poiston seuraukset

Mahdollisia vaihtoehtoja `ON DELETE` -osassa ovat:

* `NO ACTION`: “älä tee mitään” (oletus)
* `RESTRICT`: estä poistaminen
* `CASCADE`: poista myös viittaavat rivit
* `SET NULL`: muuta viittaukset arvoksi `NULL`
* `SET DEFAULT`: muuta viittaukset oletusarvoksi

Hämmentävä seikka on, että myös oletusvaihtoehto `NO ACTION` estää rivin poistamisen, vaikka nimestä voisi päätellä muuta. Vaihtoehdot `NO ACTION` ja `RESTRICT` toimivat käytännössä lähes samalla tavalla, mutta tietokannasta riippuen niiden toiminnassa voi olla eroja joissain erikoistilanteissa.

# Johdatus transaktioihin

*Transaktio (transaction)* on joukko peräkkäisiä SQL-komentoja, jotka tietokantajärjestelmä lupaa suorittaa yhtenä kokonaisuutena. Tietokannan käyttäjä voi luottaa siihen, että joko 
1. kaikki komennot suoritetaan ja muutokset jäävät pysyvästi tietokantaan tai
2. transaktio keskeytyy eivätkä komennot aiheuta mitään muutoksia tietokantaan.


## ACID

Transaktioiden yhteydessä esiintyy usein ihanteena kirjainyhdistelmä *ACID*, joka tulee seuraavista sanoista:

* Atomicity: Transaktiossa olevat komennot suoritetaan yhtenä kokonaisuutena.
* Consistency: Transaktio säilyttää tietokannan sisällön eheänä.
* Isolation: Transaktiot suoritetaan eristyksessä toisistaan.
* Durability: Loppuun viedyn transaktion tekemät muutokset jäävät pysyviksi.

## Transaktion vaiheet


Itse asiassa transaktio on hyvin arkipäiväinen asia tietokannan käyttämisessä, sillä oletuksena jokainen suoritettava SQL-komento on oma transaktionsa. Tarkastellaan esimerkiksi seuraavaa komentoa, joka kasvattaa jokaisen tuotteen hintaa yhdellä:

```sql
UPDATE Products SET price=price+1;
```

Koska komento suoritetaan transaktiona, voimme luottaa siihen, että joko jokaisen tuotteen hinta todella kasvaa yhdellä tai sitten minkään tuotteen hinta ei muutu. Jälkimmäinen voi tapahtua esimerkiksi silloin, kun sähköt katkeavat kesken päivityksen. Siinäkään tapauksessa ei siis voi käydä niin, että vain osa hinnoista muuttuu.

Usein kuitenkin sana transaktio viittaa erityisesti siihen, että kokonaisuuteen kuuluu useampi SQL-komento. Tällöin annamme ensin komennon `BEGIN`, joka aloittaa transaktion, sitten kaikki transaktioon kuuluvat komennot tavalliseen tapaan ja lopuksi komennon `COMMIT`, joka päättää transaktion.

Klassinen esimerkki transaktiosta on tilanne, jossa pankissa siirretään rahaa tililtä toiselle. Esimerkiksi seuraava transaktio siirtää 100 euroa Maijan tililtä Uolevin tilille:

```sql
BEGIN TRANSACTION;
UPDATE Accounts SET balance=balance-100 WHERE owner='Maija';
UPDATE Accounts SET balance=balance+100 WHERE owner='Uolevi';
COMMIT;
```

Transaktion ideana on, että mitään pysyvää muutosta ei tapahdu ennen komentoa `COMMIT`. Niinpä yllä olevassa esimerkissä ei ole mahdollista, että Maija menettäisi 100 euroa mutta Uolevi ei saisi mitään. Joko kummankin tilin saldo muuttuu ja rahat siirtyvät onnistuneesti tai molemmat saldot säilyvät entisellään.

Jos transaktio keskeytyy jostain syystä ennen komentoa `COMMIT`, kaikki transaktiossa tehdyt muutokset peruuntuvat. Yksi syy transaktion keskeytymiseen on jokin häiriö tietokoneen toiminnassa (kuten sähköjen katkeaminen), mutta voimme myös itse halutessamme keskeyttää transaktion antamalla komennon `ROLLBACK`.


## Transaktioiden kokeilu


Hyvä tapa saada ymmärrystä transaktioista on kokeilla käytännössä, miten ne toimivat. Tässä on esimerkkinä yksi keskustelu SQLiten kanssa:


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

Alkutilanteessa Uolevin tilillä on 350 euroa ja Maijan tilillä on 600 euroa. Ensimmäisessä transaktiossa poistamme ensin Maijan tililtä 100 euroa, mutta sen jälkeen tulemme toisiin ajatuksiin ja keskeytämme transaktion. Niinpä transaktiossa tehty muutos peruuntuu ja tilien saldot ovat samat kuin alkutilanteessa. Toisessa transaktiossa viemme kuitenkin transaktion loppuun, minkä seurauksena Uolevin tilillä on 450 euroa ja Maijan tilillä on 500 euroa.

Huomaa, että transaktion sisällä muutokset kyllä näkyvät, vaikka niitä ei olisi tehty vielä pysyvästi tietokantaan. Esimerkiksi ensimmäisen transaktion `SELECT`-kysely antaa Maijan tilin saldoksi 500 euroa, koska edellinen `UPDATE`-komento muutti saldoa.


## Transaktioiden sisäinen toiminta

Transaktioiden toteuttaminen on kiehtova tekninen haaste tietokannoissa. Tavallaan transaktion tulee tehdä muutoksia tietokantaan, koska komennot voivat riippua edellisistä komennoista, mutta toisaalta mitään ei saa muuttaa pysyvästi ennen transaktion viemistä loppuun.

Yksi keskeinen ajatus tietokantojen taustalla on tallentaa muutoksia kahdella tavalla. Ensin kuvaus muutoksesta kirjataan *lokitiedostoon* (`write-ahead log`), jota voi ajatella luettelona suoritetuista komennoista. Vasta tämän jälkeen muutokset tehdään tietokannan varsinaisiin tietorakenteisiin. Nyt jos jälkimmäisessä vaiheessa sattuu jotain yllättävää, muutokset ovat jo tallessa lokitiedostossa ja ne voidaan suorittaa myöhemmin uudestaan.

Transaktioiden yhteydessä tietokantajärjestelmän täytyy myös pitää kirjaa siitä, mitkä muutokset ovat minkäkin meneillään olevan transaktion tekemiä. Käytännössä tauluihin voidaan tallentaa rivimuutoksia, jotka näkyvät vain tietyille transaktioille. Sitten jos transaktio pääsee loppuun asti, nämä muutokset liitetään taulun pysyväksi sisällöksi.


# Rinnakkaiset transaktiot (Parallel transactions)

Lisämaustetta transaktioiden käsittelyyn tuo se, että tietokannalla voi olla useita käyttäjiä, joilla on meneillään samanaikaisia transaktioita. Missä määrin eri käyttäjien transaktiot tulisi eristää toisistaan?

Tämä on kysymys, johon ei ole yhtä oikeaa vastausta, vaan vastaus riippuu käyttötilanteesta ja myös tietokannan ominaisuuksista. Tavallaan paras ratkaisu olisi eristää transaktiot täydellisesti toisistaan, mutta toisaalta tämä voi haitata tietokannan käyttämistä.



## Transaktoiden eristystasot (Transaction isolation levels)
SQL-standardi määrittelee transaktioiden eristystasot seuraavasti:

* Taso 1 (read uncommitted)
On sallittua, että transaktio pystyy näkemään toisen transaktion tekemän muutoksen, vaikka toista transaktiota ei ole viety loppuun (dirty read).

* Taso 2 (read committed)
Toisin kuin tasolla 1, transaktio saa nähdä toisen transaktion tekemän muutoksen vain, jos toinen transaktio on viety loppuun.

* Taso 3 (repeatable read)
Tason 2 vaatimus ja lisäksi jos transaktion aikana luetaan saman rivin sisältö useita kertoja, joka kerralla saadaan sama sisältö.

* Taso 4 (serializable)
Transaktiot ovat täysin eristettyjä ja komennot käyttäytyvät samoin kuin jos transaktiot olisi suoritettu peräkkäin yksi kerrallaan jossain järjestyksessä.

## Esimerkki

Tarkastellaan tilannetta, jossa tuotteen 1 hinta on aluksi 8 ja kaksi käyttäjää suorittaa samaan aikaan komentoja transaktioiden sisällä (käyttäjän 1 komennot ovat vasemmalla ja käyttäjän 2 komennot ovat oikealla):

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
Tasolla 1 käyttäjä 1 voi saada kyselyistä tulokset 5 ja 7, koska käyttäjän 2 tekemät muutokset voivat tulla näkyviin heti, vaikka käyttäjän 2 transaktioita ei ole viety loppuun.

Tasolla 2 käyttäjä 1 voi saada kyselyistä tulokset 8 ja 7, koska ensimmäisen kyselyn kohdalla toista transaktiota ei ole viety loppuun, kun taas toisen kyselyn kohdalla se on viety loppuun.

Tasoilla 3 ja 4 käyttäjä 1 saa kyselyistä tulokset 8 ja 8, koska tämä on tilanne ennen transaktion alkua eikä välissä loppuun viety transaktio saa muuttaa luettua rivin sisältöä.

## Transaktiot käytännössä


Transaktioiden toteutustavat ja saatavilla olevat eristystasot riippuvat käytetystä tietokantajärjestelmästä. Esimerkiksi SQLitessä ainoa mahdollinen taso on 4, kun taas PostgreSQL toteuttaa tasot 2–4 ja oletuksena käytössä on taso 2.

## Miksi käyttää eri tasoja?

Eristystaso 4 on tavallaan selkeästi paras, koska silloin transaktioiden muutokset eivät voi näkyä mitenkään toisilleen. Miksi edes muut tasot ovat olemassa ja miksi esimerkiksi PostgreSQL:n oletustaso on 2?

Hyvän eristämisen hintana on, että se voi hidastaa tai estää transaktioiden suorittamista, koska transaktion vieminen loppuun voisi aiheuttaa ristiriitaisen tilanteen. Toisaalta monissa käytännön tilanteissa riittää mainiosti heikompikin eristys, kunhan tietokannan käyttäjä on siitä tietoinen.

Hyvää tietoa rinnakkaisten transaktioiden toiminnasta saa perehtymällä käytetyn tietokannan dokumentaatioon sekä testailemalla asioita itse käytännössä. Esimerkiksi voimme käynnistää itse *kaksi* SQLite-tulkkia, avata niillä saman tietokannan ja sen jälkeen kirjoittaa transaktioita sisältäviä komentoja ja tehdä havaintoja.

Seuraava keskustelu näyttää edellisen esimerkin tuloksen kahdessa rinnakkain käynnissä olevassa SQLite-tulkissa:

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

Tästä näkee, että ensimmäinen transaktio tosiaan saa kummastakin kyselystä tuloksen 8. Toista transaktiota ei sen sijaan saada vietyä loppuun, vaan tulee virheviesti `Error: database is locked`, koska tietokanta on lukittuna samanaikaisen transaktion takia. Eristys toimii siis hyvin, mutta toista transaktiota pitäisi yrittää viedä loppuun uudestaan.

Vertailun vuoksi tässä on vastaava keskustelu PostgreSQL-tulkeissa (tasolla 2):

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

Nyt toisen transaktion muuttama arvo 7 ilmestyy ensimmäiseen transaktioon, mutta toisaalta molemmat transaktiot saadaan vietyä loppuun ongelmitta.