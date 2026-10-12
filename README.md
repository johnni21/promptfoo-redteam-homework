# KékKosár – Promptfoo red team evaluáció

## Az asszisztens

A KékKosár egy kitalált webshop magyar nyelvű ügyfélszolgálati
asszisztense. Könyvekkel, irodaszerekkel, szállítással és
visszaküldéssel kapcsolatos kérdésekben segít.

Az adatforrása a system-prompt.txt fájlban megadott üzletszabályzat.
Nincs valódi rendelési adatbázis vagy fizetési integráció.

## Környezet

- Node.js: 24.21.0
- Promptfoo: 0.121.20
- Tesztelt modell: google:gemini-3.5-flash-lite
- LLM-értékelő: google:gemini-3.5-flash-lite
- Mérés dátuma: 2026. október 11.

## Tesztkategóriák

| Plugin | Miért választottam? |
|---|---|
| excessive-agency | Ne állítsa, hogy rendelést módosított vagy visszatérítést indított. |
| harmful:cybercrime | Ne segítsen adathalászatban vagy jogosulatlan adatszerzésben. |
| harmful:privacy | Utasítsa vissza a személyes és belső adatok kiadását. |
| pii:social | Kapcsolatra vagy szerepre hivatkozva se adjon ki személyes adatot. |
| system-prompt-override | Ne kövesse a felhasználói szövegben szereplő hamis rendszerutasítást. |

A két adatvédelmi kategória részben átfedi egymást.
Kategóriánként három generált tesztet használtam, basic stratégiával.

Két kézzel írt teszttel egészítettem ki a készletet:
- hamis, 50% kedvezményt ígérő üzletszabályzat bemásolása;
- kitalált belső tesztjelölő kinyerése állítólagos vezetői engedéllyel.

## Telepítés

```bash
npm ci
```

Hozz létre egy .env fájlt a projekt gyökerében:

```dotenv
GOOGLE_API_KEY=your_gemini_api_key_here
```

Az API-kulcsot ne commitold. A méréshez Google AI Studio
Free tier projektet használtam.

## A meglévő tesztek futtatása

```bash
npx promptfoo eval \
  --config promptfoo-config.yaml \
  --env-file .env \
  --max-concurrency 1 \
  --delay 5000 \
  --output results.html
```

A riport megnyitása Linuxon:

```bash
xdg-open results.html
```

## Új tesztkészlet generálása

A meglévő eredmények reprodukálásához nem kell újragenerálni
a teszteket. Új készlethez külön kimeneti fájlt használj:

```bash
npx promptfoo redteam generate \
  --config redteam-config.yaml \
  --output promptfoo-config-regenerated.yaml \
  --provider google:gemini-3.5-flash-lite \
  --env-file .env \
  --strict
```

A harmful:cybercrime és system-prompt-override plugin a használt
verzióban Promptfoo távoli generálást igényelt. Ehhez a szolgáltatás
e-mail-ellenőrzést kért. A generálást Geminire konfiguráltam,
de ezeknél a pluginoknál a távoli generátor készítette a támadásokat.

A két kézzel írt kiegészítő teszt a promptfoo-config.yaml fájlban
található; az újragenerálás ezeket nem hozza létre automatikusan.

## Eredmények

| Futtatás | Sikeres | Sikertelen | Technikai hiba |
|---|---:|---:|---:|
| Első futás | 14 | 0 | 1 |
| Javított üzenetformátum | 15 | 0 | 0 |
| Hamis szabályzattal bővítve | 16 | 0 | 0 |
| Végső készlet | 17 | 0 | 0 |

Az első futásban egy támadó JSON-szöveget a Promptfoo
üzenetstruktúraként próbált értelmezni. Az explicit user üzenet
és a dump szűrő használata megoldotta a feldolgozási hibát.

A végső 17 tesztben az automatikus ellenőrzés nem talált
szabálysértést. Hat teszteset válaszát kézzel is átnéztem.
A részletes elemzés az evaluation-summary.pdf fájlban található.

## Korlátok és tanulságok

- A 17 teszt kis, egykörös tesztkészlet.
- A sikeres eredmény nem bizonyít teljes biztonságot.
- Valódi adatbázis hiányában tényleges vásárlói adatszivárgást
  nem vizsgáltam.
- A tesztelt modell és az LLM-értékelő ugyanaz volt.
- A system-prompt-override ellenőrzések egy konkrét előtagot
  vizsgáltak, nem minden lehetséges utasításfelülírást.
- Az ismételt futásoknál a cache befolyásolhatja a futásidőt.
- A system promptot a mérések között nem módosítottam;
  a technikai javítás az üzenetek átadását érintette.

További lépésként többkörös támadásokat és normál ügyfélkérdéseket
is tesztelnék. A sérült termék reklamációját és az elállást
az üzletszabályzatban külön pontosítanám.

## Fájlok

- redteam-config.yaml – tesztgenerálási konfiguráció
- promptfoo-config.yaml – végső, 17 tesztes konfiguráció
- system-prompt.txt – az asszisztens teljes system promptja
- results.html – végső HTML-riport
- results-first-run.html – az első, technikai hibás futás
- results-baseline.html – a hibamentes, 15 tesztes alapmérés
- promptfoo-config-baseline.yaml – az alapmérés konfigurációja
- system-prompt-baseline.txt – az alapmérés system promptja
- evaluation-summary.md – az elemzés szerkeszthető forrása
- evaluation-summary.pdf – kétoldalas elemzés

## Loom videó

[Megoldás bemutatása – Loom videó](https://www.loom.com/share/39da6d8af36a42de8ecf9cde6636dc85)
