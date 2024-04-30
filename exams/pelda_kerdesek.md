# Példa ZH kérdések

## Kriprográfia

- Mi a kriptográfia 4 célja?
- Mit jelent sértetlenség fogalma?
- A kriptográfiai hash algoritmusok milyen méretű
bemenetet fogadnak és milyen méretű kimenetet
adnak?
- Mire használható egy kriprográfiai hash algoritmus?
- Adott egy (modern) kriptográfiai hash algoritmussal
előállított hash. Belátható időn belül előállítható-e olyan fájl, amiből ugyanez a hash állítható elő? Röviden indokolja a választ!
- Adott egy fájl, aminek a bináris formáján nem változtathatunk. Belátható időn belül előállítható-e olyan, tőle eltérő fájl, ami vele azonos hash-t ad?
- Mi a születésnap támadás?
- Nevezzen meg és ismertessen egy olyan blokk kódoló
működési módot, ami interaktív shellhozzáférés
titkosítására használható.
- Nevezzen meg és ismertessen egy olyan blokk kódoló
működési módot, ami véletlen elérésű nagy méretű adat (nagy fájl, egész partíció) titkosítására használható.
- Mi a blokk kódolók elektronikus kódkönyv módjának sérülékenysége?
- Hogy működik a digitális aláírás?
- Használhatóak-e (közvetlenül) a nyilvános kulcsú titkosítási algoritmusok nagy mennyiségű adat titkosítására? Válaszát indokolja! Hogyan lehet nagy mennyiségű adat titkosísása során a nyílt kulcsú titkosító algoritmusokat felhasználni?
- Egy informatikai rendszer belépési jelszavait nagy biztonságú, 2048 bites RSA algoritmussal titkosítjuk. Okoz-e ez a megoldás valamilyen nehézséget, vagy rejt-e valamilyen sebezhetőséget? Ha igen, milyen megoldás javasolt ehelyett?
- Egy informatikai rendszer belépési jelszavait nagy biztonságú, 256 bites AES algoritmussal titkosítjuk. Okoz-e ez a megoldás valamilyen nehézséget, vagy rejt-e valamilyen sebezhetőséget? Ha igen, milyen megoldás javasolt ehelyett?

## Sebezhetőségek

- Milyen megoldásokkal védekezik az operációs rendszer, illetve a fordító a stack buffer overflow típusú sebezhetőségek ellen?
- Milyen módszert használnak a stack buffer overflow típusú exploitok a program futásának átirányítására?
- Mit jelent, ha egy program 'setuid root' jogosultságú? Milyen potenciális veszélyt ad ez?
- Miért nem tartalmazza a PATH környezeti változó az aktuális könyvtárat?
- Mit jelent a privilégiumszint növelés (privilege escalation)?
- Mit jelent a race condition?
- Mit jelent a side channel attack?
- Mit jelent a mitigation kifejezés?
- Milyen hibákat eredhetnek a véletlenszám-generátorok gyengeségeiből, illetve azok nem megfelelő használatából?

## Tűzfalak

- Adott egy kiszolgáló, amelynek 22-es portján egy SSH szolgáltatás fut. A következő 4 beállítás közül melyik fogja megakadályozni, hogy egy külső eszközről SSH kapcsolatot létesítsünk a kiszolgálóval? Indokolja!
  * Az INPUT láncban elhelyezett tiltás, ahol a protokoll TCP és a forrás (source) port 22
  * Az INPUT láncban elhelyezett tiltás, ahol a protokoll TCP és a cél (destination) port 22
  * Az OUTPUT láncban elhelyezett tiltás, ahol a protokoll TCP és a forrás (source) port 22
  * Az OUTPUT láncban elhelyezett tiltás, ahol a protokoll TCP és a cél (destination) port 22
- Mit jelent a tűzfal beállításánál a POLICY?
- Mit jelentenek az ACCEPT és a DROP akciók?
- Hogyan működik a port knocking?

## Backup és helyreállítás

- Mik a fő különbségek az SCP és az RSYNC között?
- Mit jelent a chroot? Mire tudjuk használni?
- Mire használható a dd parancs? Mik a fő különbségek a dd és a partclone között?

## Egyéb általános kérdések

- Mit jelnt a social engineering? Írjon néhány példát!
- Mit jelent a security by obscurity kifejezés?

