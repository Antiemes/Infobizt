# A setuid bit

## Setuid root jogú programok

Van néhány olyan program, amit a nem privilegizált felhasználók is futtathatnak, maga a program mégis meg tud változtatni olyan fájlokat, amiket csak a rendszergazda írhat. Ilyen többek közt a `passwd` is. Hogyan lehet képes erre ez a program?

Először is keressük meg, hogy hol van ez a program:

```bash
which passwd
```

A program helye a `/usr/bin/passwd`. Most listázzuk ki a fájl adatait, beleértve annak jogait:

```bash
ls -l /usr/bin/passwd
```

Ezt kapjuk:

```
-rwsr-xr-x 1 root root 68248 Mar 23  2023 /usr/bin/passwd
```

A bal oldalon a `-` a fájl típusát jelzi (ebben az esetben ez egy sima fájl), a következő 9 karakter pedig a jogosultsági biteket. Általában itt háromszor egymás után megjelenhet az `rwx` sorozat, vagy a megfelelő jogosultság hiányában egy `-` jel aszerint, hogy a tulajdonos (első 3 karakter), a csoport (második 3 karakter), vagy mindenki más (utolsó 3-as csoport) rendelkezik-e olvasási (`r), írási (`w`), illetve végrehajtási (`x`) joggal. Itt viszont az első csoportban az `x` helyett `s` betűt találunk. Ez az `s` betű a `setuid` jog, illetve bit.

Azok a programok, amikben be van kapcsolva a `setuid` bit, nem az adott felhasználó jogosultságaival futnak, hanem átveszik a fájl tulajdonosának jogait. Ha a tulajdonos a `root`, akkor `setuid root` jogú programról beszélünk.

## Készítsünk saját setuid root jogú programot

A `setuid root` kipróbálásához egy nagyon egyszerű, C nyelven írt programra van szükségünk. Fontos, hogy ezt a tesztet ne a home könyvtárunkban végezzük el, hanem a `/tmp`-ben. Lépjünk be ide:

```bash
cd /tmp
```

Készítsünk egy forrásfájlt `sr.c` néven:

```bash
mcedit sr.c
```

Majd írjuk bele a következő forráskódot:

```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
  setuid(0);
  printf("Main program started\n");
  execve("/bin/bash", NULL, NULL) == -1;
  return 1;
}
```

Ahhoz, hogy le tudjuk fordítani ezt a programot, szükségünk van a `gcc` programra. Tulajdonképpen egyszerűbb, ha a `build-essential` csomagot tesszük fel.

```bash
apt-get install build-essential
```

Majd lefordíthatjuk a programot:

```bash
gcc sr.c -o sr
```

A `setuid root` beállításához először a `root` tulajdonába kell juttatni a fájlt:

```bash
sudo chown root sr
```

Majd hozzáadjuk a `setuid` bitet:

```bash
sudo chmod u+s sr
```

Végül elindítjuk a programot:

```bash
./sr
```

A prompt végén most nem `$` jel áll, hanem `#`, ami jelzi a `root` jogunkat, de emellett az `id` paranccsal is meg tudunk győződni erről.

## A nosuid opció

Most végezzük el ugyanezt a home-unkban is. Ide a `cd` paranccsal tudunk visszalépni. Láthatjuk, hogy itt ugyanez nem működik.

A kérdésre a megoldás a telepíŧéskor kiválasztott `nosuid` opcióban rejlik. Ez ugyanis nem engedélyezi a `setuid root` mechanizmust. A `nosuid` opciórót a `mount` paranccsal tudjuk ellenőrizni.

## Setuid root programok

Milyen programok rendelkeznek `setuid root` joggal? Erről a következő paranccsal kaphatunk egy listát:

```bash
cd /usr/bin
ls -l | grep '^...s'
```

Ezek a programok tehát lényegében `root` joggal futnak. Ha egy támadó valahogy ki tudja valamelyik itt szerplő programot úgy akasztani, hogy ennek mellékhatásaként az indítson el egy shellt, akkor az illető `root` jogot tud szerezni.


