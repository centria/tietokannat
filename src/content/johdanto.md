---
title: "Johdanto"
nav_order: 3
hidden: false
---

# Mikä on tietokanta?

<Note>Materiaalissa on usein mainittuna termien englanninkieliset selitykset, ja kurssin tehtävät ovat englanniksi.

Tämä siksi, että lähes kaikki dokumentaatio aiheesta on englanniksi, sekä itse SQL-kieli pohjaavat englantiin.</Note>

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

Tällainen tietokanta toimii hyvin, jos oletamme pankilla olevan vain muutama asiakas ja tapahtuma, ja vain yksi asiakas kerrallaan käyttää pankkia, eikä tietokoneet koskaan temppuile. Valitettavasti tämä ei useinkaan ole totta.

## Tiedon määrä

Mitä tapahtuu, jos pankilla on miljoona asiakasta, ja jokaisen asiakkaan tilillä on keskimäärin viisi tapahtumaa päivässä? Tällöin meidän tarvitsee lisätä 15 miljoonaa riviä tiedostoomme joka päivä, tai yli viisi miljardia riviä vuodessa.

Tietokantamme suunnittelussa on kriittiinen tehokkuuteen liittyvä virhe: kun haluamme tietää asiakkaan tilin saldon, meidän tarvitsee selata kaikki tiedoston rivit läpi. Kun tietokanta kasvaa, tästä tulee äärimmäisen hidasta.

Käytännössä ongelmat tehokkuudessa näkyvät asiakkaille tietokantaa käyttävän ohjelmiston hitautena. Esimerkiksi tässä tapauksessa Maijan pitäisi odottaa minuutti, ennen kuin hän näkee tilinsä saldon.


## Samanaikaisuus

Moni järjestelmä toimii hyvin kunhan sillä on vain yksi käyttäjä, mutta samanaikaiset käyttäjät auttavat ongelmia. Mitä tapahtuu, jos esimerkiksi yritämme luoda kaksi tiliä samaan aikaan?

Oletetaan että haluamme luoda tilin Liisalle ja Aapelille, ja kirjoitamme tiedostoon seuraavat rivit:

```
CREATE ACCOUNT
NUMBER: 185421761
OWNER: Liisa
CREATE ACCOUNT
NUMBER: 111562714
OWNER: Aapeli
```

Valitettavasti kun kaksi käyttäjää kirjoittaa tietokantaan samaan aikaan, rivit voisivat *mennä sekaisin* ja tiedostomme voisi näyttää tältä:

```
CREATE ACCOUNT
NUMBER: 185421761
CREATE ACCOUNT
NUMBER: 111562714
OWNER: Aapeli
OWNER: Liisa
```

Nyt Liisa on laittanut tietokantaan ensimmäiset kaksi riviä, sitten Aaapeli kaikki kolme riviään, ja lopulta Liisalta tulee vielä yksi rivi. Nyt on mahdotonta tietää mikä tili kuuluu kellekin, ja tietokantamme data on korruptoitunut (*corrupted*).

## Yllätykset

Yksi lisäongelma on tietokoneiden sammuminen *koska tahansa* (esimerkiksi sähkökatko, virtajohto irtoaa seinästä, jne). Mitä tapahtuu, jos juuri sillä hetkellä teemme muutoksia tietokantaamme?

Tarkastellaan seuraavaa tilannetta, missä Maija siirtää Uoleville 50 euroa:

```
MAKE TRANSACTION
ACCOUNT: 175299717
SUM: -50
MAKE TRANSACTION
ACCOUNT: 131778223
SUM: 50
```

Tietokone kuitenkin kaatuu ensimmäisen kolmen rivin jälkeen, ja vain ne tallentuvat tietokantaan:


```
MAKE TRANSACTION
ACCOUNT: 175299717
SUM: -50
```

Nyt Maija on menettänyt 50 euroa ja Uolevi ei saanut mitään, tai 50 euroa on vain *kadonnut* ja tätä ei edes voi nähdä tiedostosta, kun kone käynnistyy uudelleen.


## Miten korjaamme tämän?

Tietokannan käsittely on hankala tehtävä, ja tällä kurssilla emme yritä saavuttaa kaikkea itse, vaan luotamme olemassaoleviin tietokantajärjestelmiin.

# Relaatiomallit (Relational models)

Relaatiomallit korvasivat kilpailijansa 1970-luvulla ja ovat hallinneet tietokantojen maailmaa siitä lähtien. Malli perustuu kahteen perusideaan: kaikki data tallennetaan *riveihin* (row) jotka voivat *viitata* (reference) toisiinsa, ja tietokannan käyttäjä käsittelee tietoa *SQL* kielellä, joka piilottaa tietokannan sisäiset yksityiskohdat käyttäjältä.

## Tietokannan rakenne (Database structure)

Relaatiomallissa tietokanta muodostuu *tauluista* (*table*), joissa on kiinteät *sarakkeet* (*column*). Tauluihin tallennetaan tietoa *riveinä* (*row*), joilla on tietyt arvot sarakkeissa. Jokaisessa taulussa on kokoelma tiettyyn asiaan liittyvää tietoa.

Seuraavassa taulussa meillä on esimerkki tietokannasta, joka voisi olla osa verkkokauppaa. Tauluissa `Products`, `Customers` ja `Purchases` meillä on tietoa tuotteista, käyttäjistä ja heidän ostoskoreistaan.

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


Tauluissa `Products` ja `Customers` jokaisella rivillä on uniikki *id* numero, joilla niihin voidaan viitata. Tämän takia taulussa `Purchases`  voimme esittää numeerisesti, minkä esineen kukin asiakas on valinnut. Tässä esimerkissä Uolevilla on porkkanoita ja selleriä korissaan, ja Maijalla on retiisi, turnipsi ja selleri omassaan.

## SQL-kieli

*Structured Query Language* tai *SQL* on vakiintunut tapa käsitellä tietokannan sisältöä. Kielessä on komentoja (commands), joiden avulla tietokannan käyttäjä (esimerkiksi tietokantaa käyttävä ohjelmoija) voi lisätä, hakea, muuttaa ja poistaa tietoa.

SQL-komennot muodostuvat *avainsanoista* (*keywords*) kuten `SELECT` ja `WHERE`, taulujen ja sarakkeiden nimistä sekä muista arvoista. Esimerkiksi komento

```sql
SELECT price FROM Products WHERE name='radish';
```

hakee tietokannan tuotteista retiisin hinnan. Komentojen lopussa on puolipiste `;` ja voimme käyttää välilyöntejä ja rivinvaihtoja haluamallamme tavalla. Esimerkiksi voisimme kirjoittaa äskeisen komennon myös näin usealle riville:


```sql
SELECT 
    price 
FROM 
    Products 
WHERE 
    name='radish';
```

Koska SQL on luotu menneinä aikoina, siinä on paljon viitteitä sen aikaiseen ohjelmointiin. Esimerkiksi

* Avainsanat ovat kokonaisia englannin kielen sanoja, ja komennot muistuttavat englannin kielen lauseita.
* Avainsanoissa kirjainkoolla ei ole väliä. Esimerkiksi `SELECT`, `select` ja `SeLeCT` tarkoittavat samaa. Avainsanat on tapana kirjoittaa kokonaan suurilla kirjaimilla.
* Yhtäsuuruusmerkki `=` tarkoittaa sekä asetusta että yhtäsuuruusvertailua (ja `==` ei käytetä lainkaan).


SQL-kielestä on olemassa *standardeja*, jotka pyrkivät antamaan yhteisen pohjan kielelle. Käytännössä jokainen SQL-kielen toteutus toimii kuitenkin hieman omalla tavallaan. Tällä kurssilla keskitymme SQL:n ominaisuuksiin, jotka ovat yleisesti käytettävissä eri tietokannoissa.

## Tietokannan sisäinen rakenne

Käyttäjän ja tietokannan välissä on *tietokantajärjestelmä* (*relational database management system*), jonka tehtävänä on käsitellä käyttäjän antamat SQL-komennot. Esimerkiksi kun käyttäjä antaa komennon, joka hakee tietoa tietokannasta, tietokantajärjestelmän tulee löytää jokin hyvä tapa käsitellä komento ja toimittaa tulokset takaisin käyttäjälle mahdollisimman nopeasti.

SQL-kielen hienoutena on, että käyttäjän riittää *kuvailla*, mitä tietoa hän haluaa, minkä jälkeen tietokantajärjestelmä hoitaa likaisen työn ja hankkii tiedot tietokannan uumenista. Tämä on mukavaa käyttäjälle, koska hänen ei tarvitse tietää mitään tietokannan sisäisestä toiminnasta vaan voi luottaa tietokantajärjestelmään.

Tietokantajärjestelmän toteuttaminen on vaikea tehtävä, ja yleensä vain asiaan erikseen perehtyneet ovat tietoisia tietokannan sisäisestä toiminnasta. Tämä on pääasiassa kurssimme tavoitteiden ulkopuolella, mutta vilkaistaan niitä vähän kurssin loppupuolella.