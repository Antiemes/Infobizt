# Belépés, kilépés, lállítás

Amikor szöveges módban elindul a virtuális (vagy fizikai) gép, akkor a következő bejelentkező képernyő fogad minket:

```
Debian GNU/Linux 12 debian tty1

debian login:
```

A `debian` a gép hosztneve. Itt vagy a `root`, vagy az általunk létrehozott másik (például `user`) felhasználóval és a hozzá tartozó jelszóval léphetünk be.

A kijelentkezéshez a `logout`, vagy az `exit` parancsot, vagy a `Ctrl-D` billentyűkombinációt használhatjuk.

A rendszert leállítani alapesetben csak a root tudja a következő paranccsal:

```bash
shutdown now
```

Ez egyébként egy **virtuális konzol**, ezek közül is az első (`tty1`). Ezek közt váltani az `Alt-Fx`, vagy a `Ctrl-Alt-Fx` gombokkal tudunk, ahol `x` értéke alapesetben 1 és 6 között lehet. Így a szöveges felületen is be tudnk jelentkezni párhuzamosan több példányban, akár más-más felhasználóval is.

