# Audio routing a návrh architektury Pair Audio

Tento dokument popisuje technickou úvahu nad tím, jak by aplikace Pair Audio mohla fungovat, jak se dá obejít omezení iOS/Android audio routingu a kde má smysl použít softwarový audio mixer.

## Základní verdikt

Softwarový mixer audia udělat lze.

Univerzálně poslat výsledek do dvou libovolných Bluetooth sluchátek z jedné běžné mobilní aplikace většinou nelze, protože výběr Bluetooth audio výstupů řídí operační systém.

Proto hlavní směr aplikace nemá být:

```text
jeden telefon -> aplikace -> Bluetooth sluchátka A + Bluetooth sluchátka B
```

ale:

```text
telefon A -> sluchátka A
telefon B -> sluchátka B
aplikace synchronizuje společný poslech mezi telefony
```

## Správný produktový model

Uživatelé mají každý vlastní telefon a vlastní sluchátka.

Sluchátka nejsou přímo uložená ani řízená aplikací jako speciální výstupní zařízení. Každý uživatel si svoje sluchátka normálně spáruje se svým telefonem přes iOS nebo Android.

Aplikace pouze ověřuje, že uživatel má aktivní audio výstup, a přehrává do něj standardní cestou přes operační systém.

Uživatelský scénář:

1. Oba uživatelé mají nainstalovanou aplikaci Pair Audio.
2. Každý má vlastní sluchátka připojená ke svému telefonu.
3. Jeden uživatel klikne na tlačítko „Pustit spolu“.
4. Druhému přijde žádost o připojení ke společnému poslechu.
5. Druhý uživatel žádost přijme.
6. Aplikace na obou telefonech spustí stejný obsah a drží ho časově synchronizovaný.

Produktová formulace:

```text
Každý svoje sluchátka. Jeden společný poslech.
```

Technická formulace:

```text
Pair Audio není Bluetooth splitter.
Pair Audio je synchronizovaný poslechový prostor pro dva lidi.
```

## Proč neposílat hlavní audio přes Bluetooth mezi telefony

Bluetooth mezi telefony může dávat smysl pro rychlé nalezení druhého zařízení, handshake nebo předání pozvánky.

Neměl by však být hlavní cestou pro audio stream.

Důvody:

- Bluetooth audio je primárně řešené jako spojení telefon -> sluchátka.
- BLE má omezenou datovou propustnost a je vhodnější pro krátká data než pro stabilní kvalitní audio stream.
- Cross-platform komunikace iOS <-> Android přes Bluetooth je komplikovaná.
- Operační systémy nedávají běžným aplikacím plnou kontrolu nad Bluetooth audio routováním.

Lepší architektura:

```text
QR / link / Bluetooth / nearby discovery = spárování session
Internet / Wi-Fi / WebRTC = synchronizace nebo stream
Lokální OS audio = každý telefon hraje do vlastních sluchátek
```

## Nejlepší varianta: neposílat audio, ale posílat stav

Nejčistší MVP varianta je, že si telefony neposílají samotné audio, ale pouze stav přehrávání.

Příklad synchronizační zprávy:

```json
{
  "track_id": "podcast_001",
  "position_ms": 723500,
  "state": "playing",
  "server_time_ms": 1730000000000
}
```

Každý telefon si přehrává stejný obsah lokálně nebo ze stejného zdroje:

```text
Telefon A přehrává podcast z URL/cache.
Telefon B přehrává stejný podcast z URL/cache.
Pair Audio synchronizuje pozici, stav a čas.
```

Výhody:

- nízký datový tok,
- menší spotřeba baterie,
- vyšší stabilita,
- lepší kvalita zvuku,
- funguje s libovolnými sluchátky,
- obejde se bez nízkoúrovňového Bluetooth routingu.

Vhodné pro:

- podcasty,
- jazykové lekce,
- audioknihy,
- vlastní obsah,
- offline balíčky,
- cestovní playlisty mimo uzavřená DRM prostředí.

Nevhodné nebo komplikované pro:

- Spotify,
- Apple Music,
- Netflix,
- DRM obsah,
- libovolné audio z jiné aplikace.

## Druhá varianta: host streamuje audio druhému telefonu

Pro některé scénáře může jeden telefon fungovat jako host a streamovat zvuk do druhého telefonu.

Architektura:

```text
Telefon A:
audio soubor / lekce / mikrofon
        ↓
softwarový mixer
        ↓
lokální přehrávání do sluchátek A
        ↓
encoder
        ↓
stream přes síť do telefonu B

Telefon B:
přijme stream
        ↓
jitter buffer
        ↓
decoder
        ↓
přehrávání do sluchátek B
```

Možné technologie:

- WebRTC,
- RTP/Opus,
- WebSocket/QUIC stream,
- lokální Wi-Fi,
- server relay.

Výhody:

- lze posílat vlastní mix,
- lze přidat mikrofonní komentář,
- lze vytvořit řízené lekce nebo guided listening,
- lze podporovat vlastní lokální audio.

Nevýhody:

- vyšší latence,
- vyšší spotřeba baterie,
- složitější stabilita,
- nutnost řešit bufferování,
- nevhodné pro DRM obsah.

Tato varianta se hodí až jako druhá fáze projektu.

## Role softwarového mixeru

Softwarový mixer má smysl, ale ne jako cesta k obejití Bluetooth routingu operačního systému.

Mixer by měl sloužit k vytvoření společného audio zážitku uvnitř aplikace.

Příklady vstupů:

```text
podcast / lekce
+ hlasový komentář
+ systémové zvuky aplikace
+ reakce
+ intro/outro
+ jazykové opakování
= výsledný audio stream
```

Mixer může řešit:

- volume,
- balance,
- fade in / fade out,
- crossfade,
- ducking,
- normalizaci,
- limiter,
- delay compensation,
- EQ,
- mikrofonní komentář,
- notifikace nebo reakce.

Výstup mixeru:

```text
mixed PCM -> lokální přehrávání
```

nebo ve druhé fázi:

```text
mixed PCM -> encoder -> stream do druhého telefonu
```

## Doporučené MVP

První prototyp by měl být postavený na synchronizaci, ne na streamování audia.

Návrh MVP:

```text
Flutter aplikace
Firebase / Supabase realtime room
AVPlayer na iOS
ExoPlayer / Media3 na Androidu
Podcast RSS nebo demo MP3
QR code pairing
Synchronizace stavu přehrávání
Drift correction
```

Základní moduly:

```text
Mobile App iOS/Android
├── Pairing
│   ├── QR code
│   ├── invite link
│   └── room code
│
├── Audio Player
│   ├── local playback
│   ├── podcast RSS
│   ├── local/cache file
│   └── lessons
│
├── Sync Engine
│   ├── clock sync
│   ├── play/pause/seek events
│   ├── drift detection
│   └── soft correction
│
├── Session Server
│   ├── rooms
│   ├── presence
│   ├── current track state
│   └── reconnect
│
└── Couple Layer
    ├── reactions
    ├── shared notes
    ├── playlist na cesty
    └── společné učení
```

## Synchronizační algoritmus

Host nastaví:

```text
track_id
position_ms
play_state
server_timestamp
start_at
```

Guest spočítá očekávanou pozici:

```text
expected_position = position_ms + (now - server_timestamp)
```

Základní korekce:

```text
pokud rozdíl < 100 ms:
    neřešit

pokud rozdíl 100–500 ms:
    jemně upravit playback speed, například 0.98–1.02

pokud rozdíl > 500 ms:
    seek na správnou pozici
```

Pro podcasty a jazykové lekce je tato tolerance většinou použitelná. Hudba bude výrazně citlivější a měla by přijít až později.

## Rychlé připojení uživatelů

### Varianta 1: QR kód nebo odkaz

Nejjednodušší a nejspolehlivější cross-platform řešení.

Flow:

```text
Uživatel A klikne „Pustit spolu“.
Aplikace vytvoří session.
Zobrazí QR kód nebo odkaz.
Uživatel B naskenuje QR / otevře odkaz.
Oba vstoupí do stejné poslechové místnosti.
```

### Varianta 2: blízké zařízení

Možné použít pro lepší UX, ale je složitější.

Na iOS existuje Multipeer Connectivity.
Na Androidu existuje Nearby Connections.

Problém je, že nejde o jeden univerzální společný protokol pro iOS i Android.

Proto je pro první verzi vhodnější QR kód, odkaz nebo cloudová session.

### Varianta 3: uložený pár / oblíbený kontakt

Nejsilnější produktový scénář pro páry.

Flow:

```text
Moje polovička: Jana
[ Pustit spolu ]
```

Janě přijde notifikace:

```text
Honza s tebou chce poslouchat „Italština na cesty“.
[Připojit]
```

Toto je technicky jednodušší než Bluetooth discovery a produktově silnější.

## Co nedělat jako hlavní směr

Nepostavit MVP na slibu:

```text
Připojíme dvoje libovolná Bluetooth sluchátka k jednomu iPhonu.
```

Tento směr je velmi rizikový, protože aplikace nemá kontrolu nad systémovým Bluetooth audio routingem.

Také nestavět produkt na:

```text
Zachytíme zvuk z libovolné jiné aplikace.
```

Na Androidu je to v některých případech možné přes audio playback capture, ale není to spolehlivé a závisí to na oprávněních a nastavení zdrojové aplikace.

Na iOS je tento scénář pro běžnou aplikaci ještě omezenější.

Nevhodné hlavní směry:

- private iOS API,
- jailbreak,
- root Android,
- custom ROM,
- obcházení DRM,
- slib univerzálního Bluetooth splitteru.

## Budoucí směr: Auracast / LE Audio

Bluetooth LE Audio a Auracast mohou být v budoucnu důležité.

Ideální dlouhodobý scénář:

```text
telefon -> Bluetooth LE Audio broadcast -> více kompatibilních sluchátek
```

Problém je, že podpora závisí na:

- telefonu,
- operačním systému,
- sluchátkách,
- dostupnosti veřejných API.

Proto Auracast patří do roadmapy, ale ne do MVP.

Možné budoucí chování aplikace:

```text
pokud zařízení podporuje Auracast:
    nabídnout systémové sdílení zvuku
jinak:
    použít two-phone sync
```

## Možná hardwarová větev

Pokud by cílem bylo skutečně umožnit:

```text
jeden telefon -> dvoje libovolná Bluetooth sluchátka
```

nejčistší řešení by mohl být vlastní hardware dongle.

Architektura:

```text
telefon USB-C / analog / BLE
        ↓
Pair Audio dongle
        ↓
Bluetooth transmitter A
Bluetooth transmitter B
```

Telefon by viděl jedno audio zařízení. Dongle by řešil rozdělení do dvou Bluetooth výstupů.

Výhody:

- obchází omezení OS,
- telefon řeší jen jeden výstup,
- lze podporovat dvě libovolná Bluetooth sluchátka,
- silná produktová diferenciace.

Nevýhody:

- vývoj hardwaru,
- certifikace,
- baterie,
- latence,
- výrobní náklady,
- komplikace kolem iPhone/USB-C/MFi.

Tato větev patří až do pozdější fáze, ne do prvního softwarového MVP.

## Doporučená roadmapa

### Fáze 1: synchronizovaný poslech

- dva telefony,
- každé zařízení vlastní sluchátka,
- QR / link pairing,
- podcast nebo demo MP3,
- synchronizace play/pause/seek,
- základní drift correction.

### Fáze 2: párová vrstva

- uložený partner,
- push notifikace „Pustit spolu“,
- reakce,
- společné poznámky,
- playlist na cesty,
- jazykové lekce.

### Fáze 3: vlastní audio stream

- host streamuje audio guestovi,
- softwarový mixer,
- mikrofonní komentář,
- WebRTC / Opus,
- offline nebo lokální režim.

### Fáze 4: Auracast / LE Audio

- detekce podpory zařízení,
- využití systémových možností,
- případná integrace s kompatibilním hardwarem.

### Fáze 5: hardware dongle

- samostatný audio splitter,
- jeden telefon jako jeden zdroj,
- dvě libovolná Bluetooth sluchátka,
- komerční hardware produkt.

## Shrnutí

Nejlepší obchvat omezení OS je nechat každý telefon přehrávat do svých vlastních sluchátek a synchronizovat pouze stav přehrávání.

Aplikace by tedy fungovala takto:

```text
oba mají Pair Audio
každý má vlastní sluchátka
jeden klikne „Pustit spolu“
druhý přijme pozvánku
oba telefony spustí stejný obsah
aplikace drží časovou synchronizaci
```

Softwarový mixer má smysl pro vlastní obsah, lekce, komentáře a budoucí streamování mezi telefony.

Nemá být hlavním nástrojem pro obejití Bluetooth výstupů v jednom mobilu.
