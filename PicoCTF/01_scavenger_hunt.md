a[href="#spoiler"] {
  text-decoration: none !important;
  cursor: default;
  margin-bottom: 10px;
  padding: 10px;
  background-color: #FFF8DC;
  border-left: 2px solid #ffeb8e;
  display: inline-block;
}
a[href="#spoiler"]::after {
  content: attr(title);
  color: #FFF8DC;
  padding: 0 0.5em;
}
a[href="#spoiler"]:hover::after,
a[href="#spoiler"]:active::after {
  cursor: auto;
  color: black;
  transition: color .5s ease-in-out;
}

# Scavenger Hunt

Ez a példa azt mutatja be, hogy a webes tartalmak mögött nyílt szöveges elemek működnek,
amiket a felhasználó közvetlenül nem lát, de lehetősége van belenézni.

Nézzük meg a weblap forráskódját! (View source menüpont.) Nézzük meg, hogy milyen más fájlokra hivatkozik a weboldal! (Egy `.css` és egy `.js` fájlt fogunk találni.)

A webszerveren még egyéb fájlok is lehetnek, amikről közvetlenül általában nem kaphatunk listát,
viszont ha tudjuk a fájlok nevét, akkor le tudjuk őket kérni.

A leggyakrabban szereplő ilyen fájl a webrobotoknak ad utasításokat (amiket azok nem feltétlenül tartanak be.) Derítsük ki, hogy mi ennek a neve!

>! `robots.txt`

Bizonyos webszerverek bizonyos beállításokat a kiszolgált fájlok mellett tárolni. Ezek
a fájlok jó beállítások esetén nem olvashatóak bárki számára, viszont ezek korrekt védelméről
esetleg megfeledkezhet a webszerver üzemeltetője. Az utalás azt mondja, hogy egy
Apache webszerverrel van dolgunk. Derítsük ki, hogy ott milyen néven lehetnek
beállításokat tároló fájlok.

>! .htaccess

<p>
  <a href="#spoiler" title="Spoiler Filled Text"></a>
</p>
