# Technická proveditelnost

## Varianta A: jeden telefon a dvoje libovolná Bluetooth sluchátka

Tato varianta je produktově atraktivní, ale technicky velmi problematická.

### Problém

Běžná mobilní aplikace na iOS ani Androidu obvykle nemá dostatečný přístup k nízkoúrovňovému Bluetooth audio routingu.

Aplikace typicky nemůže sama rozhodnout:

> stejný audio stream pošli současně do dvou různých Bluetooth A2DP zařízení.

To řeší operační systém nebo výrobce zařízení.

### iOS

iOS podporuje sdílení audia přes Apple Share Audio, ale pouze pro kompatibilní AirPods a Beats zařízení.

Univerzální podpora pro libovolná Bluetooth sluchátka není běžně dostupná pro aplikace třetích stran.

### Android

Někteří výrobci, například Samsung, mají vlastní funkce typu Dual Audio.

Nejde však o spolehlivou univerzální Android funkci dostupnou běžné aplikaci na všech zařízeních.

### Auracast / LE Audio

Bluetooth LE Audio a Auracast jsou budoucí směr, který může umožnit vysílání zvuku více přijímačům.

Podmínkou ale je podpora na straně telefonu, systému i sluchátek.

## Varianta B: dva telefony, každé zařízení vlastní sluchátka

Tato varianta je technicky mnohem realističtější.

Aplikace by nesdílela Bluetooth výstup z jednoho telefonu, ale synchronizovala by přehrávání mezi dvěma zařízeními.

### Výhody

- funguje s libovolnými sluchátky,
- nevyžaduje speciální Bluetooth oprávnění,
- je použitelná na iOS i Androidu,
- umožňuje vzdálený i lokální společný poslech,
- dá se rozšířit o sociální a vzdělávací funkce.

### Nevýhody

- každý uživatel potřebuje vlastní telefon,
- je nutné řešit synchronizaci přehrávání,
- u hudby je potřeba velmi přesná synchronizace,
- u podcastů a jazykových lekcí je tolerance větší.

## Doporučení

Pro MVP zvolit Variant B:

> synchronizovaný poslech mezi dvěma telefony.

Variantu A sledovat pouze jako budoucí možnost podle vývoje LE Audio / Auracast.
