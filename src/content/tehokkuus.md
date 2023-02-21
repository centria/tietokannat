---
title: "Tehokkuus"
nav_order: 9
hidden: false
---

# Kyselyjen suoritus

SQL-kieli on tietokannan käyttäjälle mukava kyselyjen tekemisessä, koska käyttäjän riittää kuvata, mitä tietoa hän haluaa hakea, ja tietokantajärjestelmä hoitaa loput. Niinpä tietokantajärjestelmän on tärkeää pystyä löytämään jokin tehokas tapa toteuttaa käyttäjän antama kysely ja toimittaa kyselyn tulokset käyttäjälle.

## Kyselyn suunnitelma (Explain)

Monet tietokantajärjestelmät kertovat pyydettäessä suunnitelmansa, miten annettu kysely aiotaan suorittaa. Tämän avulla voimme tutkia tietokantajärjestelmän sisäistä toimintaa.

Tarkastellaan esimerkkinä kyselyä, joka hakee retiisin tiedot taulusta `Products`:

```
SELECT * FROM Products WHERE name='radish';
```

Kun laitamme SQLitessä kyselyn eteen sanan `EXPLAIN`, saamme seuraavan tapaisen selostuksen suunnitelmasta:

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

SQLite muuttaa kyselyn tietokannan sisäiseksi *ohjelmaksi*, joka hakee tietoa tauluista. Tässä tapauksessa ohjelman suoritus alkaa riviltä 12, jossa alkaa transaktio, ja sitten rivillä 14 rekisteriin 2 sijoitetaan hakuehdossa oleva merkkijono "radish". Tämän jälkeen suoritus siirtyy riville 1, jossa aloitetaan taulun `Products` käsittely, ja rivit 2–9 muodostavat silmukan, joka etsii hakuehtoa vastaavat rivit taulusta.

Voimme myös pyytää tiiviimmän suunnitelman laittamalla kyselyn eteen sanat `EXPLAIN QUERY PLAN`. Tällöin tulos voi olla seuraava:


```
sqlite> EXPLAIN QUERY PLAN SELECT * FROM Products WHERE name='radish';
0|0|0|SCAN TABLE Products
```

Tässä `SCAN TABLE Products` tarkoittaa, että kysely käy läpi taulun `Products` rivit.


## Kyselyn optimointi (Optimizing a query)

Jos kyselyssä haetaan tietoa vain yhdestä taulusta, kysely on yleensä helppo suorittaa, mutta todelliset haasteet tulevat vastaan usean taulun kyselyissä. Tällöin tietokantajärjestelmän tulee osata *optimoida (optimize)* kyselyn suorittamista eli muodostaa hyvä suunnitelma, jonka avulla halutut tiedot saadaan kerättyä tehokkaasti tauluista.

Tarkastellaan esimerkkinä seuraavaa kyselyä, joka listaa kurssien ja opettajien nimet:

```sql
SELECT C.name, T.name FROM Courses C, Teachers T WHERE C.teacher_id = T.id;
```

Koska kysely kohdistuu kahteen tauluun, olemme ajatelleet kyselyn toiminnan niin, että se muodostaa ensin kaikki rivien yhdistelmät tauluista `Courses` ja `Teachers` ja valitsee sitten ne rivit, joilla pätee ehto `C.teacher_id = T.id`. Tämä on hyvä ajattelutapa, mutta tämä ei vastaa sitä, miten kunnollinen tietokantajärjestelmä toimii.

Ongelmana on, että tauluissa `Courses` ja `Teachers` voi molemmissa olla suuri määrä rivejä. Esimerkiksi jos kummassakin taulussa on miljoona riviä, rivien yhdistelmiä olisi miljoona miljoonaa ja veisi valtavasti aikaa muodostaa ja käydä läpi kaikki yhdistelmät.

Tässä tilanteessa tietokantajärjestelmän pitääkin ymmärtää, mitä käyttäjä oikeastaan on hakemassa ja miten kyselyssä annettu ehto rajoittaa tulosrivejä. Käytännössä riittää käydä läpi kaikki taulun `Courses` rivit ja etsiä jokaisen rivin kohdalla jotenkin tehokkaasti yksittäinen haluttu rivi taulusta `Teachers`.

Voimme taas pyytää SQLiteä selittämään kyselyn suunnitelman:

```
sqlite> EXPLAIN QUERY PLAN SELECT C.name, T.name FROM Courses C, Teachers T WHERE C.teacher_id = T.id;
0|0|0|SCAN TABLE Courses AS C
0|1|1|SEARCH TABLE Teachers AS T USING INTEGER PRIMARY KEY (rowid=?)
```

Tämä kysely käy läpi taulun `Courses` rivit (`SCAN TABLE Courses`) ja hakee tietoa taulusta `Teachers` pääavaimen avulla (`SEARCH TABLE Teachers`). Jälkimmäinen tarkoittaa, että kun käsittelyssä on tietty taulun `Courses` rivi, kysely hakee tehokkaasti taulusta `Teachers` rivin, jossa pääavain `T.id` on sama kuin `C.teacher_id`.

Mutta miten käytännössä taulusta `Teachers` voi hakea tehokkaasti? Tämä onnistuu käyttämällä indeksiä, joihin tutustumme heti seuraavaksi.

# Indeksit

*Indeksi* on tietokannan taulun yhteyteen tallennettu hakemistorakenne, jonka tavoitteena on tehostaa tauluun liittyvien kyselyiden suorittamista. Indeksin avulla tietokantajärjestelmä voi selvittää tehokkaasti, missä päin taulua on rivejä, jotka täsmäävät tiettyyn hakuehtoon.

## Pääavaimen indeksi (Primary key index)

Kun tietokantaan luodaan taulu, sen pääavain saa automaattisesti indeksin. Tämän seurauksena voimme suorittaa tehokkaasti hakuja, joissa ehto liittyy pääavaimeen.

Esimerkiksi kun luomme SQLitessä taulun

```sql
CREATE TABLE Products (id INTEGER PRIMARY KEY, name TEXT, price INTEGER);
```

niin taululle luodaan indeksi sarakkeelle `id` ja voimme etsiä tehokkaasti tuotteita id-numeron perusteella. Tämän ansiosta esimerkiksi seuraava kysely toimii tehokkaasti:

```sql
SELECT price FROM Products WHERE id=3;
```

Voimme varmistaa tämän kysymällä kyselyn suunnitelman:


```
sqlite> EXPLAIN QUERY PLAN SELECT price FROM Products WHERE id=3;
selectid    order       from        detail                                                   
----------  ----------  ----------  ---------------------------------------------------------
0           0           0           SEARCH TABLE Products USING INTEGER PRIMARY KEY (rowid=?)
```

Suunnitelmassa näkyy `SEARCH TABLE`, mikä tarkoittaa, että kysely pystyy hakemaan taulusta tietoa tehokkaasti indeksin avulla.

## Indeksin luominen

Pääavaimen indeksi on kätevä, mutta voimme haluta myös etsiä tietoa jonkin muun sarakkeen perusteella. Esimerkiksi seuraava kysely hakee rivit sarakkeen `price` perusteella:


```sql
SELECT name FROM Products WHERE price=4;
```

Tämä kysely ei ole oletuksena tehokas, koska sarakkeelle `price` ei ole indeksiä. Näemme tämän pyytämällä taas selitystä kyselystä:

```
sqlite> EXPLAIN QUERY PLAN SELECT name FROM Products WHERE price=4;
selectid    order       from        detail             
----------  ----------  ----------  -------------------
0           0           0           SCAN TABLE Products
```

Nyt suunnitelmassa näkyy `SCAN TABLE`, mikä tarkoittaa, että kysely joutuu käymään läpi taulun kaikki rivit. Tämä on hidasta, jos taulussa on paljon rivejä.

Voimme kuitenkin luoda uuden indeksin, joka tehostaa saraketta `price` käyttäviä kyselyitä. Saamme luotua indeksin komennolla `CREATE INDEX` näin:


```sql
CREATE INDEX idx_price ON Products (price);
```

Tässä `idx_price` on indeksin nimi, jolla voimme viitata siihen myöhemmin. Indeksi toimii luonnin jälkeen täysin automaattisesti, eli tietokantajärjestelmä osaa käyttää sitä kyselyissä ja huolehtii sen päivittämisestä.


## Miten indeksi toimii?


Indeksi tarvitsee tuekseen hakemistorakenteen, josta voi hakea tehokkaasti rivejä sarakkeen arvon perusteella. Tämä voidaan toteuttaa esimerkiksi puurakenteena, jonka avaimina on sarakkeiden arvoja.

Indeksin luomisen jälkeen voimme kysyä uudestaan kyselyn suunnitelmaa:


```
sqlite> EXPLAIN QUERY PLAN SELECT name FROM Products WHERE price=4;
selectid    order       from        detail                                               
----------  ----------  ----------  -----------------------------------------------------
0           0           0           SEARCH TABLE Products USING INDEX idx_price (price=?)
```

Indeksin ansiosta suunnitelmassa ei lue enää `SCAN TABLE` vaan `SEARCH TABLE`. Suunnitelmassa näkyy myös, että aikomuksena on hyödyntää indeksiä `idx_price`.

## Lisää käyttötapoja

Voimme käyttää indeksiä myös kyselyissä, joissa haemme pienempiä tai suurempia arvoja. Esimerkiksi sarakkeelle `price` luodun indeksin avulla voimme etsiä vaikkapa rivejä, joille pätee ehto `price<3` tai `price>=8`.

Indeksi on myös mahdollista luoda usean sarakkeen perusteella. Esimerkiksi voisimme luoda indeksin näin:

```sql
CREATE INDEX idx_price ON Products (price,name);
```

Tässä indeksissä rivit on järjestetty ensisijaisesti hinnan ja toissijaisesti nimen mukaan. Indeksi tehostaa hakuja, joissa hakuperusteena on joko pelkkä hinta tai yhdessä hinta ja nimi. Kuitenkaan indeksi ei tehosta hakuja, joissa hakuperusteena on pelkkä nimi.


## Milloin luoda indeksi?

Periaatteessa voisi ajatella, että taulun jokaiselle sarakkeelle kannattaa luoda indeksi, jolloin monenlaiset kyselyt ovat nopeita. Tämä ei ole kuitenkaan käytännössä hyvä idea.

Vaikka indeksit tehostavat kyselyitä, niissä on myös kaksi ongelmaa: indeksin hakemistorakenne vie tilaa ja indeksi myös hidastaa tiedon lisäämistä ja muuttamista. Jälkimmäinen johtuu siitä, että kun taulun sisältö muuttuu, niin muutos täytyy myös päivittää kaikkiin tauluun liittyviin indekseihin. Indeksiä ei siis kannata luoda huvin vuoksi.

Hyvä syy indeksin luontiin on, että haluamme suorittaa usein tietynlaisia kyselyitä ja ne toimivat hitaasti, koska tietokantajärjestelmä joutuu käymään läpi turhaan jonkin taulun kaikki rivit kyselyn aikana. Tällöin voimme lisätä taululle indeksin, jonka avulla tällaiset kyselyt toimivat jatkossa tehokkaasti.

Indekseillä on käytännössä suuri vaikutus tietokantojen tehokkuuteen. Moni tietokanta toimii hitaasti sen takia, että siitä puuttuu oleellisia indeksejä. Tasapaino riittävien ja liian monen indeksin välillä on herkkä, ja pitäisi ottaa huomioon tietokantojen kehityksessä.


# Tiedon toisteisuus

Ideaalitilanteessa tietokantaa suunnitellessa tietokanta ei sisällä toisteista tietoa, joka pystyttäisiin päättelemään toisaalta tietokannassa. Joskus meidän täytyy hieman taivuttaa tätä sääntöä tehdäksemme kyselyistämme tehokkaampia.

## Esimerkki

Tarkastellaan pankin tietokantaa, jossa taulu `Transactions` sisältää tietoa tilisiirroista. Jokaiselle siirrolle on sarake `change` joka indikoi kuinka paljon tilin tase muuttuu (eli arvo voi olla positiivinen tai negatiivinen).

Seuraava esimerkki palauttaa tilin tämänhetkisen saldon tilille 123 laskemalla yhteen kaikki tiliin liittyvät muutokset:

```sql
SELECT SUM(change) FROM Transactions WHERE account_id=123;
```

Tämä on sinänsä hyvä kysely, mutta sen pitää käydä läpi kaikki tiliin 123 liittyvät rivit taulusta `Transactions` löytääkseen nykyisen saldon. Tämä on melkolailla ylimääräistä työtä, sillä meitä ei kiinnosta historia vaan ainoastaan saldo.

Voimme tehdä kyselystä tehokkaamman luomalla uuden taulun `Balances`, joka sisältää tilien saldot. Tämän taulun avulla voimme hakea saldon tilille 123 näin helposti:

```sql
SELECT balance FROM Balances FROM account_id=123;
```

Tässä rikomme periaatetta, että tietokannassa ei pitäisi olla tietoa joka voidaan laskea muualta tietokannasta, sillä taulu `Balances` pystytään laskemaan taulusta `Transactions`. Tällä tavoin tosin voimme tehdä todella paljon tehokkaamman kyselyn.

Mitä vikaa on periaatteen rikkomisessa? No, teimme juuri tietokannan päivittämisestä hankalampaa. Joka kerta kun nyt lisäämme uuden rivin tauluun ``Transactions`, joudumme myös päivittämään tietoa taulussa `Balances`. Aiemmin saldo laskettiin suoraan tilisiirroista, joka oli aina ajantasalla.

## Denormalisointi (Denormalization)

Tietokantojen teoriassa käytetään termiä *denormalisointi (denormalization)*, mikä tarkoittaa kyselyiden tehostamista lisäämällä toisteista tietoa. Tällä kertaa teemme siis melko päinvastaista kuin normalisoidessa.

## Muutokset vastaan kyselyt

Usein esiintyvä ilmiö tietotekniikassa on, että joudumme tasapainoilemaan sen kanssa, haluammeko muuttaa vai hakea tehokkaasti tietoa ja paljonko tilaa voimme käyttää. Tämä tulee tietokantojen lisäksi vastaan esimerkiksi algoritmien suunnittelussa.

Jos tietokannassa ei ole toisteista tietoa, muutokset ovat helppoja, koska jokainen tieto on vain yhdessä paikassa eli riittää muuttaa vain yhden taulun yhtä riviä. Myös hyvänä puolena tietokanta vie vähän tilaa. Toisaalta kyselyt voivat olla monimutkaisia ja hitaita, koska halutut tiedot pitää kerätä kasaan eri puolilta tietokantaa.

Kun sitten lisäämme toisteista tietoa, pystymme nopeuttamaan kyselyjä mutta toisaalta muutokset hidastuvat, koska muutettu tieto pitää päivittää useaan paikkaan. Samaan aikaan myös tietokannan tilankäyttö kasvaa toisteisen tiedon takia.

Valitettavasti ei ole mitään yleistä sääntöä, paljonko toisteista tietoa kannattaa lisätä, vaan tämä riippuu tietokannan sisällöstä ja halutuista kyselyistä. Yksi hyvä tapa on aloittaa tilanteesta, jossa toisteista tietoa ei ole, ja lisätä sitten toisteista tietoa tarvittaessa, jos osoittautuu, että kyselyt eivät muuten ole riittävän tehokkaita.

Huomaa, että indeksointi on myös yksi esimerkki kuinka toisteinen tieto tekee kyselyistä tehokkaampia. Tällöin toisteinen tieto ei ole talletettu tauluihin, vaan taulujuen ulkopuolelle omiin tietorakenteisiinsa.
