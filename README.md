# PrimeCam

Játékoskamerák továbbítása esport közvetítéshez. A játékosok gépén futó
**PrimeCam adó** (`primecam-sender.exe`) a webkamera képét kis méretre vágva,
H.264-be tömörítve, titkosítva küldi a stúdióba. A stúdióban futó **PrimeCam
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

A program csak kimenő kapcsolatot használ, a routeren semmit nem kell
beállítani. Magától nem frissül: új verziónál zárd be, és cseréld le az exe-t
a Releases oldalról letöltöttre (a beállítások megmaradnak). Ha a stúdió
újabb verziót vár, a program figyelmeztet.

### Mit küld a program?

- a kamera kivágott, kicsinyített képét (alapból 300×168, 30 fps, ~300 kbps)
- a bejelentkezett Steam fiók Steam64 azonosítóját és nevét
- a gép nevét, a program verzióját, a kamera listáját és a kapcsolat
  minőségi adatait (késleltetés, csomagvesztés, bitráta)
- a teljes, kivágatlan kameraképet másodpercenként egyszer, de **csak akkor**,
  ha a stúdió épp a kivágást állítja

A Steam azonosítót a Windows beállításjegyzékéből és a Steam
`loginusers.vdf` fájljából olvassa ki; a Steam folyamat memóriájához nem nyúl.

### Hibaelhárítás

- **Nincs kép / „Kamera foglalt”**: zárd be a kamerát használó más
  programokat (Logitech G HUB/Capture, Discord videó, böngésző).
- **„Steam nem fut”**: indítsd el a Steamet és jelentkezz be.
- **Nem csatlakozik**: ellenőrizd, hogy a teljes csatlakozási kódot
  másoltad-e be. Céges/kollégiumi hálózaton a 8443-as TCP és 8189-es UDP
  kimenő port lehet tiltva.
- **Szaggat**: a program magától lejjebb veszi a bitrátát; ha tartósan rossz,
  szólj a stúdiónak.

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

- A vevő gépen a routeren továbbítsd a **8443/TCP** (WHIP, HTTPS) és a
  **8189/UDP** (média) portot a vevő gép helyi IP-címére. Ha ez nem
  lehetséges (pl. CGNAT), a játékosok és a vevő [Tailscale](https://tailscale.com)
  hálózaton is összeköthetők; ilyenkor a csatlakozási kódban a vevő
  Tailscale címét kell megadni.
- A csatlakozási kódot a vevő ablakában találod. Ez tartalmazza az
  eseménykulcsot és a tanúsítvány ujjlenyomatát – ne tedd közzé.
- OBS-ben adj hozzá **Böngésző** forrást, és illeszd be a vevőben a
  játékos sorában található **OBS URL másolása** gombbal kapott címet
  (pl. `http://127.0.0.1:8080/view/steam/7656119…`). A forrás a játékost
  követi akkor is, ha másik gépre vagy fiókra vált.
  - `?fit=contain` – teljes kép kivágás nélkül (alapból `cover`)
  - `&mirror=1` – tükrözés csak ebben a nézetben
  - `&label=1` – a játékos nevének kiírása
- A `http://127.0.0.1:8080/` oldalon az összes kamera egyszerre látható.
