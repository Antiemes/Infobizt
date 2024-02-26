# A csomagkezelő használata

A csomagkezelő segítségével tudunk programokat telepíteni a rendszerre. Van olyan csomag, ami több programot is tartalmaz egyben és gyakran előfordul az is, hogy egy program több csomagra van bontva (például egy csomag csak dokumentációt, vagy valamilyen plugint, vagy libraryt tartalmaz). A csomagok között függőségi reláció van, tehát bizonyos csomagok csak akkor tudnak megfelelően működni, ha bizonyos más csomagok is fel vannak telepítve. A csomagkezelő automatikusan fel tudja oldani ezeket a függőségeket.

A csomagkezelőhöz egy adatbázis tartozik, ami azt tárolja, hogy milyen csomagok érhetők el, ezek közül mik vannak már telepítve, melyikből mi a legfrissebb verzió, illetve hogy honnan lehet ezeket letölteni. A csomag adatbázis frissítése az

```bash
apt-get update
```

paranccsal történik. Ez a parancs tehát nem telepít fel semmit, csak magát az adatbázist frissíti. Kiadásához root jog kell, így vagy root-ként bejelentkezve, vagy sudo-val (lásd később) tudjuk futtatni.

Ha a telepített csomagokat szerenént frissíteni, azt a következő paranccsal tehetjük meg:

```bash
apt-get upgrade
```

Egy, vagy több csomag telepítése:

```bash
apt-get install csomag1 csomag2 ...
```

Ez a parancs a függőségeket is telepíti, illetve tájékozatt arról is, hogy milyen opcionális csomagok érhetőek el.

Ha egy csomag mégsem kell, akkor le tudjuk törölni:

```bash
apt-get remove csomag1 csomag2 ...
```

A csomagkezelő nyilvántartja a csomagokhoz tartozó konfigurációs fájlokat is. Eltávolításkor ezeket nem törli. Ha szeretnénk törölni ezeket is:

```bash
apt-get remove --purge csomag1 csomag2 ...
```

Ha azokat a csomagokat is törölni szeretnénk, amik az eltávolítandó csomagok függőségeként kerültek telepítésre:

```bash
apt-get autoremove csomag1 csomag2 ...
```

A telepített csomagok listázása:

```bash
apt-get list --installed
```

Az összes elérhető csomagot is listázhatjuk:

```bash
apt-get list
```

Illetve kereshetünk is kulcsszó alapján:

```bash
apt-cache search kulcsszo
```

