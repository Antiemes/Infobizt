# A sudo parancs

A `root` felhasználóval alapesetben nem tudunk távolról belépni a rendszerünkre és nem is célszerű engedélyezni azt. Nem privilegizált felasználóval viszont értelemszerűen nem tudnk rendszeradminisztrációs feladatokat végrehajtani. Ezt hivatott áthidalni a `sudo`, amivel kijelölt felhasználók tudnak `root` joggal parancsokat futtatni.

A `sudo` használatához először fel kell telepíteni magát a programot (értelemszerűen `root`-ként):

```bash
apt-get install sudo
```

Majd be kell állítanunk a `sudo` használatának jogát. Ehhez egy konfigurációs fájlt kell szerkeszteni, amihez az `mcedit` editort fogjuk használni (de használható például a `nano` is). Az `mcedit` az `mc` csomag része, ami egyébként is hasznos lesz később. Telepítése:

```bash
apt-get install mc
```

Most két lehetőségünk van. Megnyitva a `/etc/sudoers` fájlt:

```bash
mcedit /etc/sudoers
```

láthatjuk, hogy a `root` felhasználó számára korlátozás nélkül engedélyezett a `sudo` használata:

```
root ALL=(ALL:ALL) ALL
```

Hasonló módon felvehetjük ide a saját felhasználónkat is a következő sor beírásával:

```
user ALL=(ALL:ALL) ALL
```

A fájl tartalmát az F2-vel menthetjük el.

Látható, hogy szerepel a következő sor is a fájlban:

```
%sudo ALL=(ALL:ALL) ALL
```

Ez azt jelenti, hogy a `sudo` csoport számára engedélyezett a `sudo` parancs használata. Ha a felhasználót hozzáadjuk ehhez a csoporthoz, akkor neki is engedélyezetté válik a `sudo` használata. Ehhez a következő parancs futtatása szükséges:

```bash
usermod -a -G sudo user
```

Vagy alternativ módon kézzel is hozzá tudjuk adni a csoporthoz a felhasználót. Ehhez nyissuk meg a `/etc/group` fájlt:

```bash
mcedit /etc/group
```

Ott keressük meg a `sudo` csoport sorát és a sor végéhez adjuk hozzá a felhasználót. A sornak valahogy így kell kinéznie:

```
sudo:x:27:user
```

