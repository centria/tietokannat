---
title: "Monta taulua"
nav_order: 5
hidden: false
---

# Taulujen viittaukset ja kyselyt (References and queries)

Keskeinen idea tietokannoissa on, että taulun rivi voi viitata toisen taulun riviin. Tällöin voimme muodostaa kyselyn, joka kerää tietoa useista tauluista viittausten perusteella. Käytännössä viittauksena on yleensä toisessa taulussa olevan rivin `id`-numero.

## Esimerkki

Tarkastellaan esimerkkinä tilannetta, jossa tietokannassa on tietoa kursseista ja niiden opettajista. Oletamme, että jokaisella kurssilla on yksi opettaja ja sama opettaja voi opettaa monta kurssia.

Tallennamme tauluun `Teachers` tietoa opettajista. Jokaisella opettajalla on `id`-numero, jolla voimme viitata siihen.


```
id          name      
----------  ----------
1           Ahonen     
2           Isohanni
3           Niemi   
4           Laaksonen 
```

Taulussa `Courses` on puolestaan tietoa kursseista ja jokaisen kurssin kohdalla viittaus kurssin opettajaan.

```
id          name               teacher_id
----------  ----------------   -----------
1           Basic programming  3          
2           More programming   1          
3           Algorithms         1          
4           Scrum masters      4          
5           Algebra            3        
```

Voimme nyt hakea kurssit opettajineen seuraavalla kyselyllä, joka hakee tietoa samaan aikaan tauluista `Courses` ja `Teachers`:


```sql
SELECT Courses.name, Teachers.name
FROM Courses, Teachers
WHERE Courses.teacher_id = Teachers.id;
```

Koska kyselyssä on monta taulua, ilmoitamme sarakkeiden taulut. Esimerkiksi `Courses.name` viittaa taulun `Courses` sarakkeeseen `name`.

Kysely antaa seuraavan tuloksen:


```
name                name      
------------------  ----------
Basic programming   Niemi   
More programming    Ahonen     
Algorithms          Ahonen     
Scrum masters       Laaksonen 
Algebra             Niemi 
```

## Mitä tässä oikeastaan tapahtui?

Yllä olevassa kyselyssä uutena asiana on, että kysely koskee useaa taulua (FROM Courses, Teachers), mutta mitä tämä tarkoittaa oikeastaan?

Ideana on, että kun kyselyssä on monta taulua, kyselyn tulosrivien lähtökohtana ovat kaikki tavat valita rivien yhdistelmiä tauluista. Tämän jälkeen `WHERE`-osan ehdoilla voi määrittää, mitkä yhdistelmät ovat kiinnostuksen kohteena.

Hyvä tapa saada ymmärrystä monen taulun kyselyn toiminnasta on tarkastella ensin kyselyä, joka hakee kaikki sarakkeet ja jossa ei ole `WHERE`-osaa. Yllä olevassa esimerkkitilanteessa tällainen kysely on seuraava:

```sql
SELECT * FROM Courses, Teachers;
```

Koska taulussa `Courses` on 5 riviä ja taulussa `Teachers` on 4 riviä, kyselyn tulostaulussa on 5 * 4 = 20 riviä. Tulostaulu sisältää kaikki mahdolliset tavat valita ensin jokin rivi taulusta Courses ja sitten jokin rivi taulusta Teachers:


```
id          name                teacher_id   id          name      
----------  ------------------  -----------  ----------  ----------
1           Basic programming   3            1           Ahonen     
1           Basic programming   3            2           Isohanni
1           Basic programming   3            3           Niemi   
1           Basic programming   3            4           Laaksonen 
2           More programming    1            1           Ahonen     
2           More programming    1            2           Isohanni
2           More programming    1            3           Niemi   
2           More programming    1            4           Laaksonen 
3           Algorithms          1            1           Ahonen     
3           Algorithms          1            2           Isohanni
3           Algorithms          1            3           Niemi   
3           Algorithms          1            4           Laaksonen 
4           Scrum masters       4            1           Ahonen     
4           Scrum masters       4            2           Isohanni
4           Scrum masters       4            3           Niemi   
4           Scrum masters       4            4           Laaksonen 
5           Algebra             3            1           Ahonen     
5           Algebra             3            2           Isohanni
5           Algebra             3            3           Niemi   
5           Algebra             3            4           Laaksonen 
```

Suurin osa tulosriveistä ei ole kuitenkaan kiinnostavia, koska ne eivät liity toisiinsa mitenkään. Esimerkiksi ensimmäinen tulosrivi kertoo vain, että on olemassa kurssi *Basic programming* ja toisaalta on olemassa opettaja *Ahonen*. Tämän vuoksi rajaamme hakua niin, että opettajan `id`-numeron tulee olla sama kummankin taulun riveissä:

```sql
SELECT * FROM Courses, Teachers
WHERE Courses.teacher_id = Teachers.id;
```

Tämän seurauksena kysely alkaa antaa mielekkäitä tuloksia:


```
id          name               teacher_id   id          name      
----------  ----------------   -----------  ----------  ----------
1           Basic programming  3            3           Niemi   
2           More programming   1            1           Ahonen     
3           Algorithms         1            1           Ahonen     
4           Scrum masters      4            4           Laaksonen 
5           Algebra            3            3           Niemi   
```

Tämän jälkeen voimme vielä parantaa kyselyä valitsemalla meitä kiinnostavat sarakkeet:


```sql
SELECT Courses.name, Teachers.name
FROM Courses, Teachers
WHERE Courses.teacher_id = Teachers.id;
```

Näin päädymme samaan tulokseen kuin aiemmin:


```
name                name      
------------------  ----------
Basic programming   Niemi   
More programming    Ahonen     
Algorithms          Ahonen     
Scrum masters       Laaksonen 
Algebra             Niemi 
```

## Lisää ehtoja kyselyssä

Monen taulun kyselyissä `WHERE`-osa kytkee toisiinsa meitä kiinnostavat taulujen rivit, mutta lisäksi voimme laittaa `WHERE`-osaan muita ehtoja samaan tapaan kuin ennenkin. Esimerkiksi voimme suorittaa seuraavan kyselyn:

```sql
SELECT Courses.name, Teachers.name
FROM Courses, Teachers
WHERE Courses.teacher_id = Teachers.id AND Teachers.name = 'Niemi';
```

Näin saamme haettua kurssit, joiden opettajana on Niemi:


```
name              name      
----------------  ----------
Basic programming  Niemi   
Algebra  Niemi 
```

## Taulujen lyhyet nimet

Voimme tiivistää monen taulun kyselyä antamalla tauluille vaihtoehtoiset lyhyet nimet, joiden avulla voimme viitata niihin kyselyssä. Esimerkiksi kysely

```sql
SELECT Courses.name, Teachers.name
FROM Courses, Teachers
WHERE Courses.teacher_id = Teachers.id;
```

Voidaan esittää lyhemmin näin:


```sql
SELECT C.name, T.name
FROM Courses AS C, Teachers AS T
WHERE C.teacher_id = T.id;
```

Koska sana `AS` ei ole pakollinen, eli voimme lyhentää kyselyä lisää:

```sql
SELECT C.name, T.name
FROM Courses C, Teachers T
WHERE C.teacher_id = T.id;
```

## Saman taulun toistaminen (Repeating a table)

Monen taulun kyselyssä voi esiintyä myös monta kertaa sama taulu, kunhan niille annetaan eri nimet. Esimerkiksi seuraava kysely hakee kaikki tavat valita kahden opettajan pari:


```sql
SELECT A.name, B.name FROM Teachers A, Teachers B;
```

Kyselyn tulos on seuraava:



```
name        name      
----------  ----------
Ahonen       Ahonen     
Ahonen       Isohanni
Ahonen       Niemi   
Ahonen       Laaksonen 
Isohanni     Ahonen     
Isohanni     Isohanni
Isohanni     Niemi   
Isohanni     Laaksonen 
Niemi        Ahonen     
Niemi        Isohanni
Niemi        Niemi   
Niemi        Laaksonen 
Laaksonen    Ahonen     
Laaksonen    Isohanni
Laaksonen    Niemi   
Laaksonen    Laaksonen 
```


# Liitostaulut (Junction table)

Taulujen välillä esiintyy yleensä kahdenlaisia suhteita:

1. *Yksi moneen -suhde (One-to-many)*: Taulun A rivi liittyy enintään yhteen taulun B riviin. Taulun B rivi voi liittyä useaan taulun A riviin.
2. *Monta moneen -suhde (Many-to-many)*: Taulun A rivi voi liittyä useaan taulun B riviin. Taulun B rivi voi liittyä useaan taulun A riviin.

Tapauksessa 1 voimme lisätä *tauluun A* sarakkeen, joka viittaa *tauluun B*, kuten teimme edellisen osion esimerkissä. Tapauksessa 2 tilanne on kuitenkin hankalampi, koska yksittäinen viittaus kummankaan taulun rivissä ei riittäisi. Ratkaisuna on luoda kolmas liitostaulu, joka sisältää tiedot viittauksista.


## Esimerkki

Tarkastellaan esimerkkinä tilannetta, jossa verkkokaupassa on tuotteita ja asiakkaita ja jokainen asiakas on valinnut tiettyjä tuotteita ostoskoriin. Tietyn asiakkaan korissa voi olla useita tuotteita, ja toisaalta tietty tuote voi olla usean asiakkaan korissa.

Rakennamme tietokannan niin, että siinä on kolme taulua: Products, Customers ja Purchases. *Liitostaulu* Purchases ilmaisee, mitä tuotteita on kunkin asiakkaan ostoskorissa. Sen jokainen rivi esittää yhden parin muotoa *“asiakkaan X korissa on tuote Y”*.

Oletamme, että taulujen sisällöt ovat seuraavat:

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

Nyt voimme hakea asiakkaat ja tuotteet seuraavasti:

```sql
SELECT C.name, P.name
FROM Customers C, Products P, Purchases O
WHERE C.id = O.customer_id AND P.id = O.product_id;
```

Kyselyn ideana on hakea tauluista `Customers` ja `Products` taulun `Purchases` rivejä vastaavat tiedot. Jotta saamme mielekkäitä tuloksia, kytkemme rivit yhteen kahden ehdon avulla. Kysely tuottaa seuraavan tulostaulun:


```
name        name      
----------  ----------
Uolevi      carrot  
Uolevi      celery   
Maija       radish   
Maija       parsley    
Maija       celery   
```

Voimme lisätä kyselyyn lisää ehtoja, jos haluamme saada selville muuta ostoskoreista. Esimerkiksi seuraava kysely hakee Maijan korissa olevat tuotteet:

```sql
SELECT P.name
FROM Customers C, Products P, Purchases O
WHERE C.id = O.customer_id AND P.id = O.product_id AND C.name = 'Maija';
```

```
name      
----------
radish   
parsley    
celery
```   

## Yhteenveto tauluista

Voimme käyttää koostefunktioita ja ryhmittelyä myös usean taulun kyselyissä. Ne käsittelevät tulostaulua samalla periaatteella kuin yhden taulun kyselyissä. Voimme esimerkiksi luoda yhteenvedon, joka näyttää jokaiselta asiakkaalta kuinka monta tuotetta heillä on ostoskoreissaan, ja mikä on kokonaishinta. Voimme tehdä sen seuraavasti:


```sql
SELECT C.name, COUNT(P.id), SUM(P.price)
FROM Customers C, Products P, Purchases O
WHERE C.id = O.customer_id AND P.id = O.product_id
GROUP BY C.id;
```

## Miten kysely toimii?

Tässä kyselyssä ryhmittelemme sarakkeella `C.id`, mutta haemme sarakkeella `C.name`. Tämä on sinänsä järkevää, koska sarake `C.id` määrää sarakkeen `C.name`, ja kysely toimii mainiosti SQLitessä.

Muissa tietokannoissa (kuten PostgreSQL) voi kuitenkin olla, että sellaisenaan haettavan sarakkeen tulee aina esiintyä myös ryhmittelyssä. Tällöin ryhmittelyn tulisi olla `GROUP BY C.id, C.name`.

Kyselyn ideana on ryhmittää rivit asiakkaan `id`-numeron mukaan, jolloin funktio `COUNT (P.id)` antaa tuotteiden lukumäärän asiakkaan ostoskorissa ja funktio `SUM(P.price)` antaa kyseisten tuotteiden kokonaishinnan. Tulostaulumme on seuraavanlainen:


```
name        COUNT(P.id)  SUM(P.price)
----------  -----------  ------------
Uolevi      2            9           
Maija       3            19          
```

Tämä tarkoittaa, että Uolevin ostokset sisältävät 2 tuotetta, ja yhteishinta on 9. Maijalla on toisaalta 3 tuotetta ja kokonaishinta on 19. Kaikki näyttää hyvältä... Vai näyttääkö?

Ongelmana on, että kyselystämme puuttuu kolmas asiakas, Aapeli. Olemme törmänneet ongelmaan, jonka ratkaisemme tämän osion lopuksi.



## JOIN-syntaksi (JOIN syntax)

Tähän mennessä olemme hakeneet tietoa tauluista listaamalla taulut kyselyn `FROM`-osassa, mikä toimii yleensä hyvin. Kuitenkin joskus on tarpeen vaihtoehtoinen `JOIN`-syntaksi. Siitä on hyötyä silloin, kun kyselyn tuloksesta näyttää “puuttuvan” tietoa.


## Kyselytavat

Seuraavassa on kaksi tapaa toteuttaa sama kysely, ensin käyttäen ennestään tuttua tapaa ja sitten käyttäen `JOIN`-syntaksia.

```sql
SELECT Courses.name, Teachers.name
FROM Courses, Teachers
WHERE Courses.teacher_id = Teachers.id;

SELECT Courses.name, Teachers.name
FROM Courses JOIN Teachers ON Courses.teacher_id = Teachers.id;
```

Jälkimmäisen kyselyn JOIN-syntaksissa taulujen nimien välissä esiintyy sana *JOIN* ja lisäksi taulujen rivit toisiinsa kytkevä ehto annetaan erillisessä `ON`-osassa. Tämän jälkeen voisimme vielä käyttää `WHERE` lisätäksemme lisää ehtoja, kuten aiemminkin.

Tässä tapauksessa JOIN-syntaksi on vain vaihtoehtoinen tapa toteuttaa kysely eikä se tuo mitään uutta. Kuitenkin näemme seuraavaksi, miten voimme laajentaa syntaksia niin, että se antaa meille uusia mahdollisuuksia kyselyissä.



## Puuttuvan tiedon ongelma

Katsotaan tilannetta, missä meillä on esimerkkitaulut `Courses` ja `Teachers`, mutta yhdeltä kurssilta puuttuu opettaja:

```
id          name               teacher_id
----------  ----------------   -----------
1           Basic programming  3          
2           More programming   1          
3           Algorithms         1          
4           Scrum masters                
5           Algebra            3        
```     

Rivillä 4 sarakkeessa `teacher_id` arvo on `NULL`, joten jos käytämme kumpaa tahansa aiempaa kyselyä, rivi 4 ei vastaa yhtäkään riviä taulusta `Teachers`. Tämän takia tulostaulu ei sisällä riviä Scrum masters:



```
name                name      
------------------  ----------
Basic programming   Niemi   
More programming    Ahonen     
Algorithms          Ahonen     
Algebra             Niemi 
```

Ratkaisu ongelmaan on käyttää `LEFT JOIN`-syntaksia. Tämä tarkoittaa, että jos vasemmanpuoleinen taulu ei viittaa riviin oikeanpuoleisessa taulussa, rivi sisällytetään silti tulostauluun. Tällaiselle riville viite oikeaan tauluun on `NULL`.

Meidän tapauksessamme voisimme tehdä kyselyn täten:

```sql
SELECT Courses.name, Teachers.name
FROM Courses LEFT JOIN Teachers ON Courses.teacher_id = Teachers.id;
```

Nyt saamme kurssin Scrum masters ilman opettajaa:

```
name                name      
------------------  ----------
Basic programming   Niemi   
More programming    Ahonen     
Algorithms          Ahonen     
Scrum masters       
Algebra             Niemi 
```

## JOIN-kyselyperhe


`JOIN kyselyllä` on neljä muunnosta:
* `JOIN`: toimii kuten tavallinen kahden taulun kysely.
* `LEFT JOIN`: jos vasemman taulun rivi ei yhdisty mihinkään oikean taulun riviin, se valitaan kuitenkin mukaan erikseen.
* `RIGHT JOIN`: jos oikean taulun rivi ei yhdisty mihinkään vasemman taulun riviin, se valitaan kuitenkin mukaan erikseen
* `FULL JOIN`: sekä vasemmasta että oikeasta taulusta valitaan erikseen mukaan rivit, jotka eivät yhdisty toisen taulun riviin

SQLiten rajoituksena on kuitenkin, että vain kaksi ensimmäistä kyselytapaa ovat mahdollisia. Onneksi `LEFT JOIN` on yleensä se, mitä haluamme.

Venn-diagrammi-esitys on kutakuinkin seuraava:

![Venn diagram for SQL joins](https://raw.githubusercontent.com/centria/databases/master/src/images/sql_venn.png) 
[Source: C.L.Moffatt](https://www.codeproject.com/articles/33052/visual-representation-of-sql-joins)

Tässä kaaviossa on enemmän kuin neljä variaatiota. `INNER JOIN` on sama kuin `JOIN`, ja `FULL OUTER JOIN` on `FULL JOIN`. Muut variaatiot ovat ekslusiivisia variaatioita kyselyille.

## Puuttuuvaa tietoa yhteenvetokyselyssä

Nyt voimme ratkaista puuttuvan Aapelin ongelman. Tietokannassamme meillä on seuraavat taulut:


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

Teimme ostosten yhteenvetokyselyn seuraavasti:

```sql
SELECT C.name, COUNT(P.id), SUM(P.price)
FROM Customers C, Products P, Purchases O
WHERE C.id = O.customer_id AND P.id = O.product_id
GROUP BY C.id;
```

Tuloksena Aapeli puuttui tulostaulustamme:

```
name        COUNT(P.id)  SUM(P.price)
----------  -----------  ------------
Uolevi      2            9
Maija       3            19
```

Ongelmamme syynä on ettei Aapelilla ole ostoksia, eli kun kyselymme valitsee yhdistelmän rivejä, ei ole olemassa riviä jossa Aapeli olisi läsnä. Ratkaisu on käyttää `LEFT JOIN` seuraavasti:

```sql
SELECT C.name, COUNT(P.id), SUM(P.price)
FROM Customers C LEFT JOIN Purchases O 
    ON C.id = O.customer_id
LEFT JOIN Products P 
    ON P.id = O.product_id
GROUP BY C.id;
```

Nyt saamme myös Aapelin tuloksiimme:


```
name        COUNT(P.id)  SUM(P.price)
----------  -----------  ------------
Uolevi      2            9           
Maija       3            19          
Aapeli      0                     
```

Koska aapelilla ei ole ostoksia, ostosten summa on `NULL`.
