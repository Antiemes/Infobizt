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

## A PATH változó

Linux (és egyéb Unix-like rendszerek) alatt egy végrehajtható fájlt annak nevével akkor
tudunk futtatni, ha az azt tartalmazó könyvtár szerepel a `PATH` környezeti változóban.
Például a gyakran használt `ls` parancs vagy a `/bin`, vagy a `/usr/bin` könyvtárban van. A
`PATH mindkettőt tartalmazza. Erről a következő paranccsal tudunk meggyőződni:

```bash
echo $PATH
```

Itt az egyes könyvtárak `:` jellel vannak elválasztva.

A `PATH`-ben nem szereplő program futtatása az elérési útjával lehetséges. Az aktuális könyvtárban
levő `valami` nevű fájl futtatása az aktuális könyvtárból:

```bash
./valami
```

Ez egy apró kellemetlenség. Mi lenne, ha például betennénk az aktuális könyvárat is a `PATH`-ba?
A lista végére fűzhetjük:

```bash
export PATH=$PATH:"."
```

Vagy a lista elejéhez:

```bash
export PATH=".":$PATH
```

Ekkor az elérési út része lenne az aktuális könyvtár is, így a futtatható fájlok elérése az
elérési út beírása nélkül is működni fog.

De mi van akkor, ha egy támadó mondjuk egy `ls` nevű rosszindulatú programot helyez el egy olyan
könyvtárban, amire van írási joga és potenciálisan kiadhatjuk ott az `ls` parancsot? (Például
a `/tmp`-ben.) Ekkor, ha az aktuális könyvtárat a `PATH` elejére tettük, a támadó `ls`
programja fog elindulni. Az önmagában nem ad kellő védelmet, ha nem a `PATH` elejéhez,
hanem a végéhez adjuk hozzá az aktuális könyvtárat, hiszen például egy elgépelt parancs
beírása (amilyen nevűt a támadó a megfelelő könyvtárban elhelyezett) nem várt
következménnyel járhat.

Ez az oka annak, hogy a `PATH`-hez soha nincs hozzáadva az aktuális könyvtár.

## Véletlen rm -r

