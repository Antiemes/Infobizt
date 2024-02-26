# IP-cím kiderítése és távoli bejelentkezés:

A virtuális gép konzol ablaka nem túl kényelmes. Nem tudunk benne kijelölni, ide beilleszteni, nem tudjuk az ablakot, a betűméretet egyszerű módszerrel módosítani stb. Ehelyett célszerűbb SSH-kapcsolaton keresztül belépni. Az SSH szervert a rendszerrel együtt telepítettük, feltéve, hogy kiválasztottuk a megfelelő opciót. Ha esetleg nem, akkor pótolhatjuk:

```bash
apt-get install task-ssh-server
```

(A `task-ssh-server` egyébként egy metacsomag, aminek a ténylegesen szükséges csomagok a függőségei.)

Az IP-cím a következő paranccsal deríthető ki:

```bash
ip addr show
```

Vagy röviden:

```bash
ip addr
```

Vagy még rövidebben:

```
ip a
```

A listában két adaptert látunk: a loopback (`lo`) és az `enp0s3` nevűt. Utóbbi azonosítója esetenként változhat. Ez alatt az `inet`-tel kezdődő sorban találjuk az IP-címet. Erre az IP címre kell majd belépnünk SSH protokollon keresztül, például a PuTTY programmal. A laborgépeken a PuTTY telepítve van, de a [https://www.putty.org/](https://www.putty.org/) címről is le tudjuk tölteni. A PuTTY-ban célszerű a munkamenetet (sessiont) elmenteni, valamint egy, a Courier New-nál használhatóbb betűtípust, például Incolsolata-t beállítani.

Fontos, hogy a `root` felhasználóval itt nem tudunk belépni. Ezért adtunk a felhasználónknak előzőleg `sudo` jogot.

