# System rescue

Ez a fejezet olyan technikákat tartalmaz, amik segíthetnek egy esetleges
rendszerösszeomlás utáni helyreállításban, illetve segítik az erre való
felkészülést.

## Másolás számítógépek között, SCP

Ha egyik számítógépről SSH kapcsolatot tudunk létesíteni egy másikra,
akkor (általában) fájlokat is tudunk másolni. Legyenek adottak a következő
körülmények:

  * A helyi gépen levő másolandó fájl neve `file.zip`
  * A távoli gép IP címe: `1.2.3.4`
  * A távoli gépen a felhasználónevünk `ruser`
  * A távoli gépen a könyvtár, ahova másolunk, a `home` könyvtárunkon belül a `dest`

A másoláshoz a következő parancsot használhatjuk:

```bash
scp file.zip user@1.2.3.4:dest
```

A célkönyvtár elhagyásával a parancs a távoli oldalon levő felhasználó home
könyvtárába másol.

Az scp több fájt is át tud másolni egyszerre, illetve a `-r` kapcsolóval rekurzívan
egy, vagy több könyvtárt is.

## Másolás `rsync`-kel

Az scp-nél sokoldalúbb eszköz az `rsync`. Működéséhez először mindkét oldalon
telepíteni kell:

```bash
sudo apt-get install rsync
```

A következő parancs a `pelda` nevű könyvtárt másolja át a távoli számítógépre:

```bash
rsync --partial --progress --inplace --recursive pelda ruser@1.2.3.4:dest
```

A kapcsolók jelentése a következő:
* `--partial`: Részlegesen átmásolt fájlok folytatása
* `--progress`: Átviteli folyamat jelzése
* `--inplace`: Ne használjon ideiglenes fájlt
* `--recursive`: Rekurzív működés (könyvtár másolásához mindenképp szükséges)

Az `rsync` előnye, hogy megtalálja a már átmásolt fájlokat és azokat már nem
másolja át újra.

## Távoli könyvtár felcsatolása

Néha kényelmesebb a másolás helyett a távoli gép egy adott könyvtárának felcsatolása
a helyi gép egy (üres) könyvtárába. Az előző példához hasonlóan legyen az `1.2.3.4`
IP című számítógépen levő egy `ruser` nevű felhasználó, annak home könyvtárában
pedig egy `dest` nevű könyvtár. Ezt fogjuk a helyi gép `cel` könyvtárába
csatolni.

Ehhez először elkészítjük ezt a `cel` könyvtárat. Értelemszerűen ez a könyvtár
kezdetben üres.

```bash
mkdir cel
```

A felcsatoláshoz az `sshfs` csomag szükséges. Ha nincs telepítve, telepítsük:

```bash
sudo apt-get install sshfs
```

Majd végezzük el magát a felcsatolást:

```bash
sshfs ruser@1.2.3.4:dest cel
```

Most tehát a `cel` könyvtárban látjuk a célgép `dest` könyvtárát. Amit ide másolunk, az
megjelenik a célgépen.

Lecsatolni a következő paranccsal tudjuk:

```bash
sudo umount cel
```

Vagy `sudo` nélkül:

```bash
fusermount -u cel
```

## System Rescue CD

Akár biztonsági mentés készítéséhez, akár helyreállításhoz többféle úgynevezett live rendszer
használható. Ezek egy megfelelő adathordozóra (CD, DVD, pendrive) kiírva és a számítógépet
arról indítva változatos funkcionalitású környezetet biztosítanak. Az egyik, gyakorlatban
jól használható rendszer a System Rescue CD, ami a következő linkről tölthető le:

[https://www.system-rescue.org/Download/](https://www.system-rescue.org/Download/)

Virtuális gép esetén a gépet a letöltött ISO lemezképről kell indítani úgy, hogy
a szokásos háttértárat is meghagyjuk. Fizikai gép esetén a képfájlt célszerű
egy USB pendrive-ra írni.

A indítás után egy boot menüt látunk. A rendszer teljes egészében RAM-ba is tölthető,
de lehetőség van a menüből a telepített operációs rendszer indítására is.

A rendszer jelszót nem kér, illetve a `root` felhasználóval automatikus bejelentkezik.
Az alapértelmezett billentyűzet angol.

### Magyar billentyűzet beállítása

Aki a magyart szereti használni, az a 

```bash
loadkeys hu
```

paranccsal válthat.

### SSH belépés engedélyezése

Arra is lehetőség van, hogy a rendszerbe SSH kapcsolaton keresztül bejelentkezzünk.
Ehhez két dolgot kell beállítanunk. Először is jelszót kell adnunk a `root` felhasználónak:

```bash
passwd
```

Majd a tűzfalon engedélyezni kell a bejelentkezést. Most az egyszerűség kedvéért
a tűzfalat alaphelyzetbe állítjuk:

```bash
iptables -F
iptables -P INPUT ACCEPT
```

### Partíciók

A mentendő, vagy sérült rendszer partíciós tábláját a `cfdisk` paranccsal vizsgálhatjuk meg. A `cfdisk`
paraméterek nélkül a `/dev/sda` lemezt próbálja meg megnyitni. Ha valamelyik másik eszközre vagyunk
kíváncsiak, akkor a parancs paramétereként adhatjuk meg azt:

```bash
cfdisk /dev/sdb
```

Itt táblázatos formában láthatjuk a lemezen levő partíciók főbb adatait (ezeket módosíthatjuk is).

A gépben található blokk eszközökről az `lsblk` parancs tud listát adni.

### Partíciók felcsatolása, vizsgálata

Miután információt szereztünk a partíciók kiosztásáról, fel tudjuk őket csatolni. A root partíció
felcsatolásának elsősorban akkor lehet értelme, ha egy sérült rendszert szeretnénk helyreállítani.
Más partíciók (elsősorban a home) felcsatolásának célja az adatmentés lehet.

Tegyük fel, hogy a root fájlrendszer a `/dev/sda1` partíción van. A felcsatoláshoz először is
szükség van egy üres könyvtárra. Ezt legcélszerűbb a `/tmp`-ben létrehozni, mivel ide
biztonan van írási jogunk. (Mivel most root-ok vagyunk, elvileg bárhova lenne írási jogunk, de
live rendszer lévén bizonyos területek esetleg eleve csak olvashatóak lehetnek.)

```bash
mkdir /tmp/mnt
```

Majd felcsatoljuk ebbe a könyvtárba a kiszemelt partíciót:

```bash
mount /dev/sda1 /tmp/mnt
```

Most a `/tmp/mnt`-ben láthatjuk a vizsgált fájlrendszer tartalmát, ahonnan adatokat menthetünk, vagy ahol
módosításokat végezhetünk. Munkánk végeztével célszerű lecsatolni ezt a fájlrendszert:

```bash
umount /tmp/mnt
```

vagy

```bash
umount /dev/sda1
```

Tehát akár a csatolási pontot, akár az eszköz nevét megadhatjuk. Ezt a lépést azért célszerű megtennünk,
mert nem lehetünk benne biztosak, hogy egy live rendszer ezt minden esetben elvégzi.

Érdemes még megemlíteni a `sync` parancsot is, ami a nem kiírt puffereket szinkronizálja a lemezre.

### chroot

A `chroot` egy olyan lehetőség, amivel egy lényegében átválthatunk egy másik, telepített
rendszerre úgy, hogy nem azt a rendszert indítottuk. (A `chroot` emellett másra is használható.)
Ekkor a másik rendszer indítási folyamata nem fog lefutni, viszont (szerencsés esetben) elindítható
a rendszer parancssora, esetleg a csomagkezelő is, így a helyreállításra, vagy egyéb
vizsgálatokra több lehetőségünk van.

Tegyük fel, hogy a fenti módon a `/tmp/mnt` könyvtárba csatoltuk be a vizsgált rendszer root fájlrendszerét.
Most `chroot`-oljunk bele:

```bash
chroot /tmp/mnt
```

A most látott shell a célrendszeré, a fájlstruktúra a célrendszernek megfelelő, szerencsés esetben a csomagkezelő
futtatható. A `chroot`-ből `exit` paranccsal léphetünk ki.


## Fájlrendszer tükrözése `dd`-vel


Primary, extended, logical
GPT

mount

/dev/sda1

chroot

ldd

which

sync

Másik shell? /bin/dash

dd

partclone

partclone.ext4 -c -s /dev/sda1 -o mentes_sda1.partclone
partclone.ext4 -r

