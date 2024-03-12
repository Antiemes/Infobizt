# SQL injection

## Rendszerterv

Az SQL injection teszteléséhez szükségünk lesz egy viszonylag egyszerű,
PHP nyelvű weblapra, ami egy MySQL szerver által kiszolgált adatbázist fog
elérni. Az adatbázis az egyszerűség kedvéért egyetlen táblából fog állni,
ami diákok adatait tartalmazza. A weboldal két lapból fog állni. Az egyiken
egy form lesz egy beviteli mezővel (egy név számára), és egy submit gombbal.
Ez a másik lapra irányít, ami magát a lekérdezést megvalósítja.

## A webszerver telepítése

A webszervert a következő paranccsal tudjuk telepíteni:

```bash
sudo apt-get install apache2
```

A telepítés után a webszerverünk már fut. Más disztribúciók esetén
esetleg olyan szokással is találkozhatunk, ahol az indítást,
illetve annak beállítását, hogy a szolgáltatás automatikusan
induljon el, kézzel kell beállítani.

A webszerverünk a virtuális gépünk IP címén a 80-as (alap)
porton fut. Elérése: `http://192.168.8.55/`. Az IP cím természetesen
változhat.

A webszerver konfigurációs fájljai a `/etc/apache2` könyvtárban vannak,
ahol már be van állítva egy alap elérés, ami a `/var/www/html`
könyvtárban feltételezi a szolgáltatott fájlokat.

## Az SQL szerver telepítése

Az SQL szerver a MariaDB (régi nevén MySQL) lesz. Telepítése:

```bash
sudo apt-get install mariadb-server
```

Majd futtassuk le a következő parancsot néhány alapbeállításhoz:

```
sudo mysql_secure_installation
```

Ez a script többek közt letiltja a szerver távoli elérését.

## A PHP és a kapcsolódó modulok telepítése

A PHP használatához szükségünk lesz magára a nyelvi értelmezőre,
a webszerver PHP moduljára, valamint a PHP nyelv MySQL moduljára,
aminek segítségével elérjük az adatbázist. Ezek telepítése:

```bash
sudo apt install php libapache2-mod-php php-mysql
```

## Az adatbáz is létrehozása és feltöltése

Először jelentkezzünk be az adatbázis szerverbe:

```bash
sudo mariadb -uroot
```

Itt hozzunk létre egy adatbázist és egy felhasználót, ami ezt
az adatbázist teljeskörűen használhatja:

```
create database kreta;
create user 'kretauser'@'%' identified by 'jelszo';
grant all on kreta.* to 'kretauser'@'%';
flush privileges;
```

Most már bejelentkezhetünk az új felhasználóval:

```bash
mariadb -ukretauser -pjelszo kreta
```

Itt hozzunk létre egy táblát és töltsük fel néhány tesztadattal:

```
CREATE TABLE jegyek (jegyid int not null auto_increment, nev varchar(255) not null, targy varchar(255) not null, jegy int, primary key (jegyid) );
INSERT INTO jegyek (nev, targy, jegy) VALUES ('Nagy Pal', 'matematika', 5);
INSERT INTO jegyek (nev, targy, jegy) VALUES ('Nagy Pal', 'magyar', 3);
INSERT INTO jegyek (nev, targy, jegy) VALUES ('Kis Peter', 'tortenelem', 4);
```

Ez a példa a minimumra szorítkozik. Természetesen egy valós esetben ezt több táblára bontanánk.

## A weblapok létrehozása

Legyen az egyik fájl neve `jegyek.php` a következő tartalommal:

```php
<form action="jegyq.php" method="POST">
<input type="text" id="nev" name="nev" size=60><br>
<input type="submit" value="Keres">
</form>
```

Ez a fentebb már említett form, amibe a diák nevét tudjuk beírni.
A `size=60` a mező méretét adja meg. Mivel egy szép hosszú bemenetet fogunk
majd beírni, nem árt, ha látjuk nagyjából egyben a beírt szöveget.

A fájl kiterjesztése egyébként lehetne HTML is, mivel nem tartalmaz dinamikus
elemeket.

Ez a lap a `jegyq.php`-re visz, aminek tartalma a következő:

```php
<?php

$nev=$_POST['nev'];
print($nev);
print("<br>");

mysqli_report(MYSQLI_REPORT_OFF);

$mysqli = new mysqli("localhost", "kretauser", "jelszo", "kreta");

$query = "SELECT * FROM jegyek where nev like '".$nev."';";
print("Query:<br>");
print($query);
print("<br>");

$result = $mysqli->query($query);
print("Error message: ".$mysqli->error."<br>");

while($obj = $result->fetch_object())
{
  $nev=$obj->nev;
  $targy=$obj->targy;
  $jegy=$obj->jegy;
  print($nev." ".$targy." ".$jegy."<br>");
}

$result->close();

?>

```

Ez a PHP fájl fogadja a formból származó nevet. Először elmenti azt
a `$nev` változóba, amit ki is ír. Majd az adatbázis-hozzáférést biztosító
`MYSQLI` modulon egy olyan beállítást végez el, aminek hatására exception
helyett megjelenik maga a hibaüzenet. Ez lényegében a mi dolgunkat
könnyíti meg: Látni fogjuk, hogy a elfogadta-e a lekérdezést az
SQL szerver.

Ezt követően a program összeállítja magát a lekérdezést. (Ez az a rész, ahol
maga a sebezhetőség rejlik.) A lekérdezést kiírja, majd végrehajtja. Ez
a kiírás is a mi kényelmünket szolgálja.

Végül végig iterálunk a lekérdezés eredményének sorain és kiírjuk a
`nev`, `targy` és `jegy` mezőket.

## Támasási formák

### A támadás alapja

A kódban, bár úgy tűnik, hogy a `nev` változóval egy egyszerű összehasonlítás történik,
valójában a stringet záró `'` jelet maga a változó is tartalmazhatja. Így például a

```
Nagy Pal
```

helyett ugyanezt az eredményt adja a következő is:

```
Nagy Pal' -- itt pedig akarmi lehet
```

Itt a bevitt adat tartalmazza a lezáró `'` jelet, a -- pedig SQL-ben egy komment.

### A feltétel ignorálása

Így tehát tetszőlegesen kiegészíthetjük az SQL mondatot. Például:

```
Nagy Pal' OR 1=1 -- 
```

Ügyeljünk a `--` utáni kötelező szóközre!

Ez a bemenet ignorálja az eredeti lekérdezésben levő feltételt és minden sort kiír.


