# PrimeCam

Játékoskamerák továbbítása esport közvetítéshez. A játékosok gépén futó
**PrimeCam adó** (`primecam-sender.exe`) a webkamera képét kis méretre vágva,
H.264-be tömörítve, titkosítva (HTTPS/WebSocket) küldi a stúdió szerverére.
A szerver OBS böngészőforrásként adja tovább a képeket (slot, játékos Steam
azonosító vagy gép szerint), a stúdió pedig böngészőből, az admin felületen
kezeli: a stúdió gépein semmit nem kell telepíteni.

Ez a repó a nyilvános részeket tartalmazza:

- `config/primecam.toml` – a központi beállítások (képméret, fps, bitráta, …)
- [Releases](https://github.com/antal-k/prime-cam-config/releases) – a
  letölthető programok

## Telepítés játékosoknak

1. Töltsd le a legfrissebb `primecam-sender.exe` fájlt a
   [Releases](https://github.com/antal-k/prime-cam-config/releases/latest)
   oldalról, és tedd egy állandó helyre (pl. `C:\PrimeCam\`).
2. Indítsd el. Ha a Windows SmartScreen figyelmeztet, válaszd a
   **További információ → Futtatás mindenképp** lehetőséget.
3. Másold be a szervezőtől kapott **csatlakozási kódot**
   (`primecam://…` kezdetű sor), majd kattints a **Csatlakozás** gombra.
4. Válaszd ki a kamerát. Az előnézetben látod a teljes képet és a kivágott
   részt; a nagyítást és a kivágás helyét itt is beállíthatod, de a stúdió
   felül is bírálhatja.
5. Indítsd el a Steamet és jelentkezz be a saját fiókodba – ez alapján
   azonosít a közvetítés.
6. Kapcsold be az **Indítás a Windowszal** opciót, hogy a meccsnapokon ne
   kelljen külön elindítani.

A program csak kimenő HTTPS-kapcsolatot használ (443-as port), a routeren
semmit nem kell beállítani. Magától nem frissül: új verziónál zárd be, és cseréld le az exe-t
a Releases oldalról letöltöttre (a beállítások megmaradnak). Ha a stúdió
újabb verziót vár, a program figyelmeztet.

### Mit küld a program?

- a kamera kivágott, kicsinyített képét (alapból 300×168, 30 fps, ~300 kbps)
- a bejelentkezett Steam fiók Steam64 azonosítóját és nevét
- a gép nevét, a program verzióját, a kamera listáját és a kapcsolat
  minőségi adatait (késleltetés, eldobott képkockák, bitráta)
- a teljes, kivágatlan kameraképet másodpercenként egyszer, de **csak akkor**,
  ha a stúdió épp a kivágást állítja

A Steam azonosítót a Windows beállításjegyzékéből és a Steam
`loginusers.vdf` fájljából olvassa ki; a Steam folyamat memóriájához nem nyúl.

### Hibaelhárítás

- **Nincs kép / „Kamera foglalt”**: zárd be a kamerát használó más
  programokat (Logitech G HUB/Capture, Discord videó, böngésző).
- **„Steam nem fut”**: indítsd el a Steamet és jelentkezz be.
- **Nem csatlakozik**: ellenőrizd, hogy a teljes csatlakozási kódot
  másoltad-e be. Ha a program szerint *„a stúdió szervere nem érhető el”*
  (régebbi verzióban: *vevője*), a hiba a stúdió oldalán van – szólj nekik.
- **„Ez a régi, IP-alapú kód”**: a 0.2-es verziótól új csatlakozási kód kell,
  kérd el a stúdiótól.
- **Szaggat, késik**: a program magától lejjebb veszi a bitrátát, ha a
  hálózat nem bírja; ha tartósan rossz, szólj a stúdiónak. Ilyenkor érdemes a
  gépen futó nagy feltöltéseket (játékfrissítés, stream, felhőmentés)
  szüneteltetni.

## Központi beállítások

A `config/primecam.toml` minden mezője kommentezve van. Egy commit a `main`
ágra legfeljebb `poll_minutes` percen belül minden adón és a szerveren életbe
lép; az admin felület **Konfig újratöltése** gombjával azonnal is
kikényszeríthető.

Az érvényes beállítás mezőnként, ebben a sorrendben áll össze (a későbbi
felülírja a korábbit):

1. beépített alapértékek
2. ez a repó (`config/primecam.toml`)
3. az adó saját beállítása – csak a kameraválasztás és a kivágás
   (nagyítás, eltolás, tükrözés)
4. a stúdió felülbírálása az admin felületen, adónként

A programok mezőnként mutatják, honnan jön az érték, és a felülbírálások
egy gombbal visszaállíthatók a központira.

## Stúdió

A stúdió a szervert böngészőből kezeli: `https://<a stúdió címe>/admin`,
belépés az admin jelszóval.

- **Csatlakozás**: a csatlakozási kód (`primecam://…#key=…`) tartalmazza az
  eseménykulcsot – csak a játékosoknak add oda, ne tedd közzé. Az **Új
  eseménykulcs** gombbal lecserélhető; utána mindenkinek az új kód kell.
- **Adók**: minden csatlakozott gép élő előnézettel és a kapcsolat
  minőségével. A **Beállítások** gombbal az adott adón a kivágás (a keret
  húzásával, görgetéssel nagyítva) és minden központi mező felülbírálható.
- **Konfig újratöltése**: a központi beállítások azonnal, minden adón.

### Közvetítés OBS-ben

OBS-ben adj hozzá **Böngésző** forrást, és illeszd be az admin felületről
másolt címet. Háromféle cím van:

- **OBS slot** (`/stream/1`, `/stream/2`, …): azt mutatja, akit az admin
  felületen a slothoz rendelsz, így cserénél az OBS-jelenethez nem kell
  nyúlni.
- **Játékos** (`/stream/7656119…`, Steam64 azonosító): a játékost követi
  akkor is, ha másik gépre vált.
- **Gép** (`/stream/sender/…`): mindig az adott gép kamerája.

A címek tartalmazzák a nézői kulcsot (`?key=…`), ezért ne tedd őket közzé. Az
**Új nézői kulcs** gombbal lecserélhető, utána az OBS-forrásokat is frissíteni
kell. A cím végére írható kiegészítők:

- `&fit=contain` – teljes kép kivágás nélkül (alapból `cover`)
- `&mirror=1` – tükrözés csak ebben a nézetben
- `&label=1` – a játékos nevének kiírása

Ha a forrásban *„a böngésző nem tudja lejátszani”* felirat jelenik meg,
frissítsd az OBS-t.
