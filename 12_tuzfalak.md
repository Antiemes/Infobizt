# Tűzfalak

## Netfilter tűzfal

### Általános működés

A Netfilter a Linux kernel beépített tűzfala. Alapesetben statikus csomagszűrő
tűzfal, de számos kiegészítéssel rendelkezik, amik ezt a funkcionalitást
kibővítik.

A Netfilter szolgáltatásaihoz az aktuális paracssori eszköz az `nftables`, vagy `nft` paranccsal érhetőek el; az újabb rendszereken
ez az alapértelmezett. Az az `iptables` jelenleg még támogatott, de a jövőben kivezetésre fog kerülni. Most mégis ezen keresztül
vizsgáljuk meg a Netfilter működését. Ennek egyik fő oka az, hogy az elérhető dokumentációk túlnyomó többsége is erre épül.
Egy ideig még biztosan támogatott marad és csomagból a következő módon telepíthető:

```bash
apt-get install iptables
```

A Netfilter tűzfal struktúrája a következő. A legfelső hierarchiai szintet a *táblák* képviselik. A következő táblák léteznek:
* filter
* nat
* mangle
* RAW
* security

Csak a *filter* táblával fogunk foglalkozni. Ide kerülnek az általános csomagszűrő szabályok. Ez az alapértelmezett tábla is.
A *nat* tábla értelemszerűen akkor hasznos, ha NAT-ot
(Network Address Transformation-t, hálózati címfordítást) szeretnénk beállítani. A *mangle* táblában az IP fejléc módosítása lehetséges (például
itt tudjuk a TTL mezőt átírni). A *RAW* tábla szabályai segítségével a kapcsolatállapot figyelését tudjuk befolyásolni. Végül a *security* tábla
a SElinux-szal tud összedolgozni.

Minden táblában *láncok* (chain) vannak. 5 beépített lánc van (nem mindegyik tábla tartalmazza az összeset), de ezek mellett sajátokat is létrehozhatunk.
A beépített láncok a következők:
* PREROUTING
* INPUT
* FORWARD
* OUTPUT
* POSTROUTING

A kívülről érkező csomagok először a PREROUTING láncba kerülnek. Ezután zajlanak le a forgalomirányítási döntések és
kerülnek át a csomagok az INPUT láncba. A rendszerben keletkező csomagok az OUTPUT láncba kerülnek, majd ezt követik
a forgalomirányítási döntések és a POSTROUTING lánc. Az átmenő csomagok (tehát amelyeknek se nem feladója,
se nem címzettje nem az aktuális számítógép) sorrendben a PREROUTING, FORWARD és POSTROUTING láncokon mennek keresztül.

A számítógép hálózati szolgáltatásainak védelmét tehát elsősorban az INPUT és az OUTPUT láncokkal szabályozhatjuk.

Mindegyik lánc *szabályokból*, illetve egy alapértelmezett szabályból (*policy*) áll. Az adott láncon végighaladó
csomag sorban végighalad a lánc szabályain. Minden esetben eldől, hogy a szabály *illeszkedik-e* a csomagra,
vagy nem. Illeszkedő esetben a szabály végrehajtásra kerül. A szabályok tartalmaznak egy-egy *akciót* is.
Az akciók egy része *terminális akció* (ilyen például az ACCEPT, a REJECT és a DROP), ezek esetében a
csomagot az adott láncban nem vizsgálja tovább a tűzfal. A terminális akciók tehát döntést hoznak
a csomag sorsával kapcsolatban. A nem terminális akciók esetében a vizsgálat a következő szabállyal folytatódik.

Ha a lánc egyik szabálya sem volt illeszkedő és terminális akcióval rendelkező, akkor a *policy*
(nevezzük alapértelmezett szabálynak) lép éltbe. A policy lehet ACCEPT, REJECT és DROP.

Az említett akciók működése a következő:
* Az *ACCEPT* elfogadja a vizsgált csomagot.
* A *REJECT* elutasíŧja a vizsgált csomagot és erről egy hálózati üzenetet küld vissza a csomag feladójának.

A REJECT akció által visszaadott üzenet arról tájékoztatja a másik oldalt, hogy az adott szolgáltatás létezik,
de a hozzáférést tűzfal szűri. Ez egy támadónak hasznos információ lehet. Ennek kivédésére létezik a harmadik,

* *DROP* akció, ami visszajelzés nélkül utasítja el a vizsgált csomagot.

### A szabályok listázása.

Az aktuális tűzfal szabályokat listázhatjuk tömör formában (a parancs szintaxisának megfelelően):

```bash
iptables -S
```

Eredménye:
```
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
```

Ez azt jelenti, hogy az INPUT, FORWARD és OUTPUT láncok csak az alapértelmezett szabályt (a policy-t) tartalmazzák,
ami mindhárom esetben ACCEPT. Tehát jelen helyzetben a tűzfal minden csomagot minden irányban átenged.

A szabályokat listázhatjuk hosszabb, részletesebb formában is:

```bash
iptables -L -n -v
```

Az alapértelmezett tábla a filter. Ha valamelyik másikat (mondjuk a nat-ot) szeretnénk listázni, azt a `-t` kapcsolóval
tehetjük meg.

```bash
iptables -t nat -L -n -v
```

Eredménye:
```
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
```

A fenti példából látható, hogy a filter táblából hiányzik a PREROUTING és a POSTROUTING, a nat táblából pedig a FORWARD lánc.

### Policy beállítása

A policy beállításakor meg kell adnunk:
  * A táblát (ez alapértelmezetten a filter).
  * A láncot
  * Magát a policy-t.

A policy-t a `-P` kapcsolóval állíthatjuk be, melynek két paramétere sorrendben a lánc és maga a policy. Például állítsuk be
a filter táblában az INPUT és a FORWARD láncok policy-jét DROP-ra, az OUTPUT-ét pedig ACCEPT-re:

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

Ez a beállítás, egyéb szabályok nélkül minden kimenő csomagot engedélyezni fog, minden egyebet pedig el fog dobni.

A tűzfalak tervezésénél két általános alapelv a policy-vel kapcsolatban:
  * Deny-everything policy: Minden tiltott, amit nem engedélyezünk külön (biztonsági szempontból előnyösebb).
  * Accept-everything policy: Minden engedélyezett, amit külön nem tiltunk (általában egyszerűbb).

### Láncok ürítése

Ahhoz, hogy tiszta lappal indulunk, egy lehetséges tűzfal script elején célszerű a láncokat kiüríteni. Erre a -F (flush)
kapcsoló szolgál.

```bash
iptables -F INPUT
iptables -F OUTPUT
iptables -F FORWARD
```

A fenti három sor a felsorolt láncokat üríti.

### Szabályok hozzáadása

A szabályok hozzáadásánál a következőket kell specifikálni:
  * Melyik táblához adjuk a szabályt (alapértelmezetten ez a filter).
  * Melyik lánchoz adjuk a szabályt.
  * Milyen feltételnek feleljen meg az adott csomag a vizsgált csomag (match).
  * Mi történjen a csomaggal (action, target).

A táblát a `-t` kapcsoló után adhatjuk meg. Mivel csak a filter táblával foglalkozunk, ami az alapértelmezett,
ezt a kapcsolót elhagyhatjuk.

A `-A` kapcsoló után adhatjuk meg, hogy melyik lánc végére tesszük az új szabályt. Lehetőség van a `-I`
kapcsoló használatára is, amely esetben tetszőleges helyre beszúrhatjuk a szabályt.

Az illesztés esetén számos lehetőségünk van. Illeszthetünk a bejövő, vagy a kimenő interface-re,
a forrás, vagy a cél IP címre, a protokollra,
TCP, vagy UDP esetén a forrás, vagy a cél portra, vizsgálhatjuk valamelyik header valamely mezőjét stb.

### Példák

Egy adott forrás IP címről (a.b.c.d) történő forgalom eldobása

```bash
iptables -A INPUT -s a.b.c.d -j DROP
```

A helyi interface-re (lo) érkező és oda irányuló forgalmat elfogadjuk:

```bash
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
```

A 22-es porton levő szolgáltatás (SSH) felé irányuló csomagok eldobása (tehát kívülről nem lehet majd belépni a 22-es porton futó szerverre):

```bash
iptables -A INPUT -p tcp --dport 22 -j DROP
```

A 22-es porton levő szolgáltatások tiltása erről a számítógépről (tehát erről a gépről nem lehet belépni más gépekre a 22-es porton):

```bash
iptables -A OUTPUT -p tcp --sport 22 -j DROP
```

A gép nem fog válaszolni a PING-re (mind a bejövő ICMP echo-request-et, mind a kimenő echo-reply-t tiltjuk):

```bash
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
iptables -A INPUT -p icmp --icmp-type echo-reply -j DROP
```


## Firewalld

https://blog.myhro.info/2021/12/configuring-firewalld-on-debian-bullseye

## ufw

