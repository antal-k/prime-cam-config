# PrimeCam

Játékoskamerák továbbítása esport közvetítéshez. A játékosok gépén futó
**PrimeCam adó** (`primecam-sender.exe`) a webkamera képét kis méretre vágva,
H.264-be tömörítve, titkosítva (HTTPS/WebSocket) küldi a stúdióba. A stúdióban futó **PrimeCam
vevő** (`primecam-receiver.exe`) fogadja a képeket, és OBS böngészőforrásként
adja tovább, a játékos Steam azonosítója alapján.

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
  másoltad-e be. Ha a program szerint *„a stúdió vevője nem érhető el”*, a
  hiba a stúdió oldalán van – szólj nekik.
- **„Ez a régi, IP-alapú kód”**: a 0.2-es verziótól új csatlakozási kód kell,
  kérd el a stúdiótól.
- **Szaggat, késik**: a program magától lejjebb veszi a bitrátát, ha a
  hálózat nem bírja; ha tartósan rossz, szólj a stúdiónak. Ilyenkor érdemes a
  gépen futó nagy feltöltéseket (játékfrissítés, stream, felhőmentés)
  szüneteltetni.

## Központi beállítások

A `config/primecam.toml` minden mezője kommentezve van. Egy commit a `main`
ágra legfeljebb `poll_minutes` percen belül minden adón és a vevőn életbe
lép; a vevőben az **Újratöltés minden adón** gombbal azonnal is
kikényszeríthető.

Az érvényes beállítás mezőnként, ebben a sorrendben áll össze (a későbbi
felülírja a korábbit):

1. beépített alapértékek
2. ez a repó (`config/primecam.toml`)
3. az adó saját beállítása – csak a kameraválasztás és a kivágás
   (nagyítás, eltolás, tükrözés)
4. a vevőben az adott adóra beállított felülbírálás

A programok mezőnként mutatják, honnan jön az érték, és a felülbírálások
egy gombbal visszaállíthatók a központira.

## Stúdió

A játékosok a stúdió saját domainjén keresztül, [Cloudflare
Tunnel](https://developers.cloudflare.com/tunnel/)lel érik el a vevőt: a
stúdióban sincs szükség porttovábbításra, nyilvános IP-címre vagy VPN-re, és
a stúdió IP-címe sehol nem jelenik meg.

### Egyszeri beállítás

Kell hozzá egy Cloudflare-en kezelt domain.

1. A [Cloudflare irányítópulton](https://dash.cloudflare.com/) nyisd meg a
   **Networking → Tunnels** oldalt, és válaszd a **Create Tunnel** gombot.
   Adj neki nevet (pl. `primecam`).
2. A **Setup Environment** résznél válaszd a Windowst, és a vevő gépén
   **rendszergazdai** parancssorban futtasd a megjelenő parancsokat (a végén
   `cloudflared.exe service install <token>`). A cloudflared ettől kezdve
   Windows-szolgáltatásként fut, újraindítás után is.
3. A tunnel **Routes** lapján: **Add route → Published application**. Adj meg
   egy aldomaint (pl. `cam`) és a domaint, **Service URL**:
   `http://127.0.0.1:8090`. Mentés.
4. A vevő ablakában, a **Csatlakozás** résznél írd be a teljes nevet (pl.
   `cam.example.hu`), és nyomd meg az **Ellenőrzés** gombot. Zöld jelzésnél
   minden rendben, hiba esetén az üzenet megmondja, melyik lépés hiányzik.
5. A **Csatlakozási kód** (`primecam://cam.example.hu#key=…`) tartalmazza az
   eseménykulcsot – csak a játékosoknak add oda, ne tedd közzé. Az **Új
   kulcs…** gombbal bármikor lecserélhető; utána mindenkinek az új kód kell.

Az adók kapcsolatait a vevő csak a gépen futó cloudflaredtől fogadja
(`127.0.0.1:8090`), ezért a routeren semmit nem kell beállítani.

### Közvetítés

- OBS-ben adj hozzá **Böngésző** forrást, és illeszd be a vevőben a
  játékos sorában található **OBS URL másolása** gombbal kapott címet
  (pl. `http://127.0.0.1:8080/view/steam/7656119…`). A forrás a játékost
  követi akkor is, ha másik gépre vagy fiókra vált.
  - `?fit=contain` – teljes kép kivágás nélkül (alapból `cover`)
  - `&mirror=1` – tükrözés csak ebben a nézetben
  - `&label=1` – a játékos nevének kiírása
- A `http://127.0.0.1:8080/` oldalon az összes kamera egyszerre látható.
