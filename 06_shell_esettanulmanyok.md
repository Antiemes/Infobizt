# Két esettanulmány egyszerű shell parancsokkal

## A veszélyes *

### Néhány egyszerű fájlmanipuláló parancs

Gyakorlásképpen hozzunk létre egy új könyvtárat `teszt` néven és lépjünk bele:

```bash
mkdir teszt
cd teszt
```

Mcedit segíŧségével készítsünk két szöveges fájlt. Az egyikben valami fontos legyen, mondjuk egy korszakalkotó ötletünk (`otlet.txt`), egy másik fájlban pedig valami kevésbé fontos tartalom, mondjuk a bableves receptje (`bableves.txt`).

```bash
mcedit otlet.txt
mcedit bableves.txt
```

Mindkét esetben F2-vel tudjuk elmenteni a tartalmat, F10-zel pedig ki tudunk lépni a szövegszerkesztőből.

Tegyük fel, hogy valahova le akarjuk menteni a fájlokat, mondjuk a `/tmp/backup` könyvtárba. Először hozzuk létre azt:

```bash
mkdir /tmp/backup
```

A másolás pedig így nézne ki:

```bash
cp * /tmp/backup
```

Ezt lefuttatva ellenőrizhetjük (pl. `mc`-vel), hogy valóban ott van a két átmásolt fájl a `/tmp/backup`-ban. De most csináljuk vissza ezt a műveletet (töröljük a `/tmp/backup`-ból a két fájlt) és térjünk vissza arra a pontra, amikor még csak eddig írtuk be a parancsot:

```bash
cp *
```

Most tegyük fel, hogy ezen a ponton véletlenül lenyomjuk az entert. Logikusan azt gondolhatjuk, hogy nincs semmi gond, mivel a félig bevitt parancs hibás, hiszen hiányzik a cél, ahova másoljuk a fájlokat. A parancs azonban lefut. A `cp` parancs ugyanis ,,buta'', nem tudna a `*`-gal mit kezdeni. A `*`-ot ehelyett a shell oldja fel és helyettesíti be a helyére az ott levő két fájl nevét. Tehát ami lefut, az a következő parancs lesz:

```bash
cp bableves.txt otlet.txt
```

Ami kérdés nélkül felülírja a második fájlt.

