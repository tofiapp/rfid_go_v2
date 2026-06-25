# RFID Go

Android aplikace (verze **3.0**) pro čtečku **Chainway C5** (vestavěný UHF UART modul, RSCJA/Chainway SDK).
Slouží k **přiřazení parametrů** (TUDU, výhybka, část) k UHF tagům a jejich **zápisu do tabulky CSV**, včetně volitelného **zaheslování** a **zamčení** tagu. **EPC kód na tagu se nepřepisuje.**

---

## Co aplikace umí

Hlavní obrazovka vede operátora třemi kroky (**TUDU → Načtení → Hotovo**). Výběr TUDU a výhybky je vždy nahoře; pokročilé funkce (parametry, CSV, heslo, zamčení) jsou ve vyjížděcím panelu **Pokročilé**.

### Průběh práce (3 kroky)

| Krok | Název | Popis |
|------|-------|-------|
| 1 | TUDU | Vybrán zdrojový soubor, TUDU a výhybka |
| 2 | Načtení | Načtení tagu, zápis parametrů do tabulky, heslo a zamčení |
| 3 | Hotovo | Tag úspěšně zpracován – potvrzení nebo opakování |

### 1. Zdroj dat – výběr TUDU
- Načte úseky **TUDU** ze souboru `.CSV` nebo `.SQL`.
- Po načtení souboru se automaticky otevře dialog s **vyhledáváním TUDU**.
- TUDU a výhybku lze kdykoli změnit klepnutím na náhledový panel (TUDU / Výhybka).
- Výběr výhybky zohledňuje **již zapsané části v CSV** – dokončené výhybky jsou v seznamu zašedlé a nevybíratelné.
- Při výběru výhybky se automaticky nastaví **první chybějící část** podle CSV.

Formát vstupního souboru (oddělovač `;` nebo `,`, hlavička volitelná):

```
TUDU;VYHYBKA;CAST_MIN;CAST_MAX
1501J1;1;1;3
1501J1;10;1;4
1501A;5;1;3
```

Stačí i jen `TUDU;VYHYBKA` – části se doplní na `1–3`.
Vzorové soubory jsou ve složce [`sample_data/`](sample_data).

**Nápověda k části výhybky** – u výhybek se třemi částmi (1–3) se pod výběrem zobrazí textová nápověda:
- část 1 → *jazyk*
- část 2 → *levé rameno*
- část 3 → *pravé rameno*

### 2. Parametry a zápis do tabulky
Parametry se zadávají přímo – **bez převodu do hex EPC kódu**:

| # | Kategorie | Popis | Příklad |
|---|-----------|-------|---------|
| 1 | Rok | fixně, lze přepsat | `2026` |
| 2 | TUDU 1.–4. znak | první 4 znaky TUDU | `1501` |
| 3 | TUDU 5. znak | 5. znak TUDU | `J` |
| 4 | TUDU 6. znak | 6. znak TUDU | `1` |
| 5 | Výhybka | číslo výhybky | `10` |
| 6 | Část výhybky | část 1–4 | `1` |
| 7 | ID_RFID | pořadové číslo, +1 po každém tagu | `30001` |

- **Názvy kategorií** i hodnoty Rok / Část / ID_RFID lze ručně přepsat.
- Tlačítko **NAČÍST TAG A ULOŽIT** přečte tag v dosahu (EPC + TID) a uloží parametry do CSV.
- EPC tagu se **nemění** – v tabulce se uloží aktuální EPC přečtené z tagu.
- Po dokončení celého cyklu (viz níže) se automaticky:
  - `ID_RFID += 1` (hodnota se ukládá do aplikace),
  - posune **část výhybky** o 1; po překročení maxima se přepne na **další nedokončenou výhybku** v pořadí daného TUDU.

### 3. Tabulka CSV
Po každém načtení tagu se uloží řádek do `rfid_go_output.csv`:

| Sloupec | Zdroj |
|---------|-------|
| ID_RFID | přiřazené pořadové číslo |
| EPC | EPC přečtené z tagu (beze změny) |
| TID | přečtený z tagu |
| Rok | zadaný rok |
| TUDU | vybrané TUDU |
| Vyhybka | číslo výhybky |
| CastVyhybky | část výhybky |

Při zápisu stejného `ID_RFID` se daný řádek **přepíše**.
Tabulku lze sdílet tlačítkem **Sdílet / Export** nebo **vymazat poslední záznam** (obnoví se předchozí stav šablony).
Nad spodním panelem se zobrazuje náhled **posledního záznamu** (výhybka a část).

Soubor je uložen v `Android/data/com.rfidw.app/files/rfid_go_output.csv`.

### 4. Zaheslování – zápis access hesla
- **bank RESERVED**, `ptr 2`, `len 2` (access password, 8 hex znaků)
- Pole **ACCESS PWD** – aktuální heslo tagu (default `00000000`)
- Pole **NEW PWD** – nové heslo (8 hex znaků)
- Tlačítko **ZAPSAT HESLO** zapíše nové access heslo na tag v dosahu
- Při selhání zápisu s uživatelským heslem se automaticky zkusí **preset hesla** `11223344`, `11112222`

### 5. Zamčení tagu
- Pole **NEW ACCESS PWD** – heslo pro zamčení (po zápisu hesla se doplní automaticky)
- **Lock code** – pevná hodnota `008020`
- Tlačítko **ZAMKNOUT** zamkne tag v dosahu

### Spouště čtečky
Fyzické tlačítko (spouště) čtečky spouští **celý řetězec** v jednom kroku:

1. načtení tagu a zápis do tabulky → 2. zápis access hesla → 3. zamčení tagu

Po úspěšném dokončení se zobrazí přehled **Načetli jste** (výhybka + část) s volbami:
- **Pokračovat** – posune část/výhybku a připraví další tag (stejně jako po ručním dokončení cyklu),
- **Opakovat** – zůstane na stejné části pro nový pokus.

Během zobrazení tohoto dialogu lze **Pokračovat** potvrdit i fyzickým tlačítkem čtečky.

Jednotlivé akce (jen načtení a uložení, jen heslo, jen zamčení) lze spustit i ručně tlačítky v panelu **Pokročilé**.

---

## Sestavení

Projekt je standardní Android (Gradle). Otevřete v **Android Studiu** nebo přes Cursor a:

```bash
./gradlew assembleDebug
```

APK: `app/build/outputs/apk/debug/rfid_go.apk`

- `compileSdk 34`, `minSdk 21`, `targetSdk 34`, Java 17, Material 3
- Android Gradle Plugin **8.5.2**, Gradle **8.7**
- Knihovny čtečky a Excelu jsou v `app/libs/`:
  - `DeviceAPI_ver20251103_release.aar` – Chainway/RSCJA UHF SDK (obsahuje i nativní `.so`),
  - `poi-*`, `jxl.jar`, `xUtils-*` – ponechány pro budoucí export do XLSX.

> Pozn.: Při prvním otevření Android Studio vygeneruje `local.properties` s cestou k SDK.

### CI (GitHub Actions)

Při pushi nebo PR na `main` se automaticky sestaví debug APK (workflow [`.github/workflows/android.yml`](.github/workflows/android.yml)).
Výsledné APK je k dispozici jako artefakt **rfid-go-debug-apk**.

---

## Struktura kódu

```
app/src/main/java/com/rfidw/app/
├─ epc/EpcModel.java       – model parametrů (rok, TUDU, výhybka, část, ID)
├─ data/Tudu.java          – model TUDU + výhybky
├─ data/TuduLoader.java    – načítání z .csv / .sql
├─ csv/CsvStore.java       – výstupní CSV s přepisem podle ID_RFID
├─ rfid/UhfManager.java    – obal nad RFIDWithUHFUART (čtení tagu, heslo, zamčení)
└─ ui/
   ├─ MainActivity.java    – obrazovka, workflow a propojení všeho
   └─ CsvAdapter.java      – zobrazení tabulky CSV v RecyclerView

app/src/main/res/layout/
├─ activity_main.xml           – hlavní obrazovka (krok 1, indikátor, spodní panel)
├─ bottom_sheet_workflow.xml   – panel Pokročilé (parametry, karty 2–5)
├─ dialog_tudu_picker.xml      – dialog výběru TUDU s vyhledáváním
└─ row_*.xml                   – řádky parametrů a CSV
```

## Možné další kroky
- Export do `.xlsx` (knihovny POI/jxl jsou už přibalené).
- Nastavení (uložení access pwd, výchozí rok, výkon).
- Hromadné vymazání celé CSV tabulky.
