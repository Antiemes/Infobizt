# Debian Linux telepítése

A következőkben a gyakorláshoz használt Debian Linuxot fogjuk telepíteni grafikus felület nélkül, nagyrészt alapbeállításokkal. Természetesen telepíthető grafikusan is, lényegesen nagyobb helyfoglalás mellett.

## CD Image letöltése

A CD image-et a következő helyről tudjuk letölteni:

(https://www.debian.org/distrib/netinst#smallcd)[https://www.debian.org/distrib/netinst#smallcd]

Itt válasszuk az *amd64* verziót.

Ez a jelenlegi, 12.5-ös verzió esetén 629 MB. Léteznek ennél kisebb telepítők is, amik magát a telepítő rendszert is letöltik maguknak, illetve elérhető / készíthető teljes, sok CD-ből, vagy több DVD-ből álló offline telepítőkészlet is.

## A telepítés megkezdése

Amikor a telepítő boot menüje megjelenik, válasszuk az Install (és NE a Graphic install) pontot, hogy a telepítés szöveges módban fusson. Figyelem! Ha pár másodpercig nem nyomunk meg egy gombot sem, akkor a grafikus telepítő indul el. (Egy otthoni rendszerre egyébként az lenne az ajánlott.)

## Nyelv és terület kiválasztása

Ki kell választanunk a nyelvet (**Language**), itt válasszuk az angolt (**English**). Lehetőség lenne magyar, vagy egy sor más nyelvű telepítésre is. Ez egyébként a feltelepített rendszerre nincs hatással, a tekintetben később tudunk választani.

Meg kell adnunk, hogy hol vagyunk (**Country**). Ez is független magától a nyelvtől, de például hatással van az időzónára. Magyarország nincs a listában, így válasszuk az **other**-t azon belül Európát (**Europe**), azon belül Magyarországot (**Hungary**).

A területhez kapcsolható beállítás a **locales**, ami például az idő- és számformátumot befolyásolja. Itt válasszuk az USA-t (**United States**).

A fentiektől független lehet a billentyűzet kiosztás (**Keymap**), ami egyéni ízlés kérdése. Én a magyart (**Hungarian**) szoktam használni.

## A gép neve

A gépünknek nevet is kell adni (**Hostname**). Itt lehetünk kreatívak is, de most maradhat az alapértelmezett **debian** is. Ennek főleg akkor van jelentősége, ha több gépen is dolgozunk és szeretnénk látni, hogy éppen hova léptünk be.

A **Domain** maradjon üres.

## Jelszavak

A telepítő kérni fogja a **root** (rendszergazda) felhasználó jelszavát egymás után kétszer is.

Majd mindenképp létre kell hoznunk egy normál felhasználót is. Először adjuk meg a nevét (ide beírhatjuk a saját nevünket), majd kell egy loginnév is (ez az egyszerűség kedvéért lehet például **user**). Ennek a felhasználónak is kell jelszó.

## Partícionálás

Most következik a merevlemez felosztása. Itt egy fizikai rendszeren rengetegféle opciónk lehetne. Lehet, hogy osztozni kellene más operációs rendszerrel a tárhelyen, esetleg szóba jöhetne olyan opció is, hogy nem egy, hanem több merevlemezünk van, illetve a boot betöltésre is gondolni kellene (utóbbit egyébként az esetek nagy részében jól kezeli a telepítő).

Többféle lehetőségünk van olyan tekintetben is, hogy milyen módszerrel hajtjuk végre a partícionálást. Lehetőségünk van vezetett / segített (guided), vagy teljesen kézi (manual) módszerre.

Használhatunk Logical Volume Managert (LVM-et), akár titkosított, akár titkosítatlan formában. Erre is itt van lehetőségünk.

Végül magának a lemeznek a felosztása is itt történik. A végcélunk az, hogy a gyökér (`/`), a `/tmp` és a `/home` külön partíción legyen, illetve a `/home`-ra tudjuk beállítani a `nosuid` kapcsolót, illetve minden lemezre a `discard` kapcsolót.

Mi most a legegyszerűbb módszert, a vezetett (**Guided**) partícionálást választjuk. Válasszuk ki a felajánlott (1 db) merevlemezt.

Itt több sémát is felajánl a telepítő. A célunkhoz legközelebbi a **Separate /home, /var, /tmp**.

Most törölni fogjuk a `Swap` és a `/var` partíciókat, azok helyére pedig kiterjesztjük a `/`-t.

A **Swap**-ot válasszuk ki, enterrel belépünk a menübe, majd ott ``Delete this partition``.
A **/var**-on is együk meg ugyanezt.

Ahhoz, hogy a `/`-t ki tudjuk terjeszteni, először azt is törölni kell a fenti módon. Ekkor megjelenik egy üres terület (**Free space**), nyomjunk rajta egy entert, majd hozzuk létre a most már nagyobb méretű `/`-t: **Create new partition**, megjelenik a maximális elérhető méret (ezen is üssünk entert), válasszuk a **Primary**-t, majd, az **options**-nál aktiváljuk a **discard** opciót (ezt space-szel tehetjük meg). **Continue** és **Done setting up the partition**.

A fentihez hasonló módon a **/home** esetében aktiváljuk a **discard** és **nosuid** opciókat, majd a **/tmp**-nél a **discard**-ot.

Ha ezzel megvagyunk, választhatjuk a **Finish** gombot.

A telepítő panaszkodni fog, hogy nincs `swap` terület. és szeretnénk-e mégis beállítani ilyet. Válasszk a **No**-t. (Egyébként SSD-re eleve nem ajánlott swap-elni.

A következő panelen véglegesíthetjük mindazt, amit most beállítottunk a **Yes** megnyomásával.

Ezen a ponton a telepítő dolgozni fog egy darabig...

## Telepítés

A merevlemez előkészítése után kezdődik a tulajdonképpeni telepítés.

Ha rendelkeznénk még telepítő lemezekkel, azokat most tudnánk hozzáadni. Mivel ilyenünk nincs, válasszuk a **No**-t.

Ehelyett online tüköroldalakról fogjuk letölteni a csomagokat. Ezt választhatjuk ki most. Itt megfelel a magyar, onnan is pl. az első. Egyébként előfordulhat, hogy esetleg bizonyos tökröket nem frissítenek rendszeresen.

Ha van (nem transzparens) **Proxy**, akkor azt a következő panelen állíthatjuk be. Ha nincs, akkor egyszerűen nyomjunk a **Continue**-ra.

Ma már a fizikai lemezes telepítőknek jobbára akkor van jelentőségük, ha olyan helyre telepítünk rendszert, ahol nincs Internet kapcsolat (vagy lassú, forgalmi díjas, vagy korlátos stb.). Ehhez kapcsolódik egy adatgyűjtő rendszer, ami a csomagokról statisztikát készít. Ennek bekapcsolására kérdez rá a következő panel. Itt választhatjuk akármelyik opciót.

A következő panelen kiválaszthatjuk, hogy miket telepítünk. Itt be van már jelölve a **Debian desktop** és a **Gnome** felület. Ezeket vegyük ki (space), az **SSH servert** (alul) pedig tegyük be. Végül válasszuk a **Continue**-t. Egyébként bármit feltelepíthetünk később is, a csomagok választéka ennél eleve lényegesen nagyobb.

A tényleges telepítés most fog végbe menni, így ezen a ponton egy kis ideig megint dolgozni fog a telepítő.

## Boot loader

A telepítés utolsó fontos lépése a boot loader beállítása. Ez jelen esetben igen egyszerű. Az **Install the GRUB boot loader?** kérdésre válaszoljunk **Yes**-t, válasszuk ki a **/dev/sda**-t, majd nyomjunk a **Continue** -ra.

Ez után újraindul a rendszer és kész is vagyunk.

