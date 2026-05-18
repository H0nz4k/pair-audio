# Pair Audio

Aplikace pro společný poslech dvou lidí, kde každý používá vlastní sluchátka.

## Základní myšlenka

Původní idea byla vytvořit aplikaci, která umožní spárovat dvoje Bluetooth sluchátka k jednomu telefonu bez ohledu na značku.

Po prvotním technickém průzkumu se ukazuje, že univerzální varianta:

> jeden telefon → dvoje libovolná Bluetooth sluchátka

je na iOS/Android velmi omezená kvůli řízení Bluetooth audio routingu operačním systémem.

Reálně proveditelnější směr je:

> dva telefony → každé zařízení vlastní sluchátka → aplikace synchronizuje poslech.

## Hlavní scénáře

- společný poslech podcastu,
- playlist na cesty,
- společné učení jazyků,
- poslech na dálku pro páry,
- offline balíček na cestování.

## MVP

1. Jeden uživatel vytvoří místnost.
2. Druhý se připojí přes QR kód / odkaz / kód místnosti.
3. Oba mají vlastní telefon a vlastní sluchátka.
4. Aplikace synchronizuje přehrávání.
5. První podporovaný obsah: podcast / lokální audio / vzdělávací lekce.

## Pracovní slogan

Každý svoje sluchátka. Jeden společný zážitek.
