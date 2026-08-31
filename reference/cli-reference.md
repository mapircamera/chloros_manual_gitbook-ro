# Chloros CLI Referință

**Versiune:**

1.2.0**Generat:**29 iulie 2026, ora 19:19 ·**Revizuit:** 30 august 2026**Public țintă:** Optimizat pentru utilizarea de către LLM; ușor de citit de către oameni.**Domeniu de aplicare:** Fiecare subcomandă destinată utilizatorilor din `chloros-cli`, cu opțiuni și exemple care pot fi copiate și lipite.

Acest document reprezintă referința completă pentru instrumentul de linie de comandă `chloros-cli` livrat împreună cu MAPIR Chloros. Este intenționat exhaustiv, astfel încât un LLM (sau o persoană) să poată compune orice flux de lucru acceptat din listele de mai jos fără a inspecta codul sursă.

Dacă aveți nevoie doar de informațiile esențiale, treceți la:
- [Ghid de pornire rapidă în cinci minute](#five-minute-quickstart)
- [Fluxul de lucru pentru prima conectare a camerei LATTICE](#lattice-camera-first-connect-workflow)
- [Fluxul de lucru pentru prima conectare a senzorului DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Moduri de captură, înregistratoare și reprocesare offline](#capture-modes-recorders--offline-reprocess)

---

## Convenții

- Toate comenzile au prefixul `chloros-cli`. Pe Windows, fișierul binar este `chloros-cli.exe`; pe Linux /Jetson, acesta este `chloros-cli`.
- Argumentele opționale sunt afișate ca `--flag`. Argumentele poziționale obligatorii sunt afișate fără paranteze.
- În cazul în care este specificată o valoare implicită, omiterea indicatorului duce la utilizarea acelei valori.
- CLI este un client „thin” HTTP care rulează pe backend-ul Chloros (server Flask pe `127.0.0.1:5000`). Backend-ul este pornit automat de majoritatea comenzilor. `CHLOROS_BACKEND_URL=<url>` direcționează familiile de comenzi **`lattice`**,**`project`**și**`daq pool-*`** către un backend la distanță — comenzile de bază (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixează în mod deliberat `http://127.0.0.1:<port>` și îl ignoră (literalul IPv4 evită penalizarea de ~Windows `localhost`→`::1` ~2 s-per-request). Consultați [Variabilele de mediu](#environment-variables).
- Este necesară autentificarea cu un cont de tip Chloros+ pentru toate apelurile către SDK / CLI (rulați `chloros-cli login` o dată pe fiecare mașină; stocat în cache în `~/.chloros/`).
- Exemplele utilizează căile Linux; pe Windows înlocuiți `/home/user/...` cu `C:/Users/.../...`.

---

## Sinteză de nivel superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opțiuni globale

| Indicator | Descriere |
| --- | --- |
| `--backend-exe PATH` | Suprascrie executabilul backend detectat automat. |
| `--port N` | Portul backend-ului HTTP (implicit: `5000`). |
| `-v, --verbose` | Activează afișarea detaliată a ieșirii. |
| `--restart` | Forțează repornirea backend-ului (închide orice proces `backend_server.py` care rulează). |
| `--version` | Afișează versiunea (`Chloros CLI 1.2.0`). |
| `--help` | Afișează ajutorul de nivel superior. |

### Indexul comenzilor

| Comandă | Scop |
| --- | --- |
| [`process`](#chloros-cli-process) | Procesează un folder cu capturi „Survey3” sau „LATTICE” de la un capăt la altul. |
| [`login`](#chloros-cli-login) | Autentifică acest computer cu un cont Chloros+. |
| [`logout`](#chloros-cli-logout) | Șterge datele de autentificare stocate în cache. |
| [`status`](#chloros-cli-status) | Afișează starea curentă a licenței / autentificării. |
| [`export-status`](#chloros-cli-export-status) | Afișează progresul în timp real al exportului Thread-4 în timpul unei execuție a `process`. |
| [`language`](#chloros-cli-language) | Setează sau afișează lista limbilor de afișare dCLI (38 de limbi acceptate). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Dosarul implicit al proiectului (partajat cu interfața grafică). |
| [`update`](#chloros-cli-update) | Verifică și instalează actualizările CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnosticarea sistemului + teste de funcționare. |
| [`time-sync`](#chloros-cli-time-sync) | Starea și controlul grandmasterului PTP. |
| [`lattice`](#chloros-cli-lattice) | Controlul camerei LATTICE și captarea (peste 45 de subcomenzi). |
| [`daq`](#chloros-cli-daq) | Controlul senzorului spectral DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Deschiderea și rularea unui proiect salvat în format Chloros (camere + DAQ-uri). |

---

## Instalare

`chloros-cli` este inclus în programul de instalare pentru desktop Chloros pe toate platformele suportate — nu există o descărcare separată CLI. Instalarea pachetului de platformă adaugă `chloros-cli` la `PATH`, alături de aplicația desktop și de fișierul binar backend pe care îl controlează.

Cele mai recente descărcări: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Programul de instalare include, de asemenea, scripturi de lansare (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) care deschid un shell CLI gata de utilizare; acestea sunt descrise în [CLI Ghidul utilizatorului](../CLI.md) și nu sunt repetate aici.

### Windows (.exe)

1. Descărcați programul de instalare Windows de pe pagina de descărcări.
2. Rulați `Chloros-Setup-x.y.z.exe` și urmați instrucțiunile expertului de instalare. Calea de instalare implicită este `C:\Program Files\Chloros\` (CLI-ul se află în `C:\Program Files\Chloros\cli\`, pe care programul de instalare îl adaugă la PATH).
3. Deschideți un terminal nou (`cmd.exe`, PowerShell sau un terminal dWindows), astfel încât să fie detectat fișierul actualizat `PATH`.

```powershell
chloros-cli --version
```

Programul de instalare adaugă automat `chloros-cli.exe` în sistemul dvs. `PATH` și include runtime-ul Arena SDK necesar pentru camerele LATTICE.

### Linux amd64 (.deb)

Pentru stații de lucru x86_64 bazate pe Ubuntu 22.04 LTS sau o versiune mai recentă / Debian.

> **Ubuntu 20.04 nu este acceptat.** Lista de dependențe a pachetului este derivată din
> ceea ce leagă efectiv backend-ul, iar aceasta include `libc6 (>= 2.34)`;
> focal livrează glibc 2.31. `apt` refuză instalarea, în loc să permită eșuarea acesteia în
> timpul rulării.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Fișierul .deb instalează:
- `chloros-cli` la `/usr/bin/chloros-cli`
- Backend-ul compilat la `/usr/lib/chloros/chloros-backend`
- Runtime-ul Arena SDK (pentru camerele LATTICE)
- Modele de denoizare, pachete de calibrare și configurația canalului de actualizare

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Aceeași structură ca și fișierul .deb pentru amd64, cu o versiune CUDA optimizată pentru Jetson Orin / Orin NX / Orin Nano.

### Autentificare o singură dată pe mașină

Fiecare platformă necesită o autentificare unică la Chloros+ înainte ca apelurile SDK / CLI să funcționeze:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Datele de autentificare sunt stocate în cache în `~/.chloros/user_session.json`.

### Verificați instalarea

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Este necesar un abonament Chloros+.**CLI necesită un plan activ Chloros+.**Copper**este nivelul de bază Chloros+ — fiecare nivel plătit Chloros+ are acces la CLI / SDK; doar nivelul gratuit**Iron** nu are. (Corelare plan-ID: `0`=Iron/gratuit, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Fă upgrade la [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Această limită minimă este impusă de backend, nu doar de CLI: o cerere cu flag-urile SDK / CLI fără un plan plătit este respinsă cu codul `403 PLAN_UPGRADE_REQUIRED`, indiferent dacă provine de la `chloros-cli`, de la Python SDK sau de la un client HTTP creat manual. Un utilizator deconectat primește în schimb codul de eroare `401 AUTH_REQUIRED`. Accesul funcționează offline pe durata perioadei de grație a abonamentului (30 de zile pentru abonamentele lunare, până la expirare pentru cele anuale) și se întrerupe la expirarea acesteia; codul de eroare `chloros-cli status` rămâne activ, astfel încât motivul să fie vizibil (este singura rută SDK / CLI scutită de limitarea pe niveluri — `GET /api/license-status`).

---

## Ghid de pornire rapidă în cinci minute

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Procesează un folder de imagini prin întregul flux de lucru Chloros (detectarea țintei → calibrare → vignetă → reflectanță → export index).

### Sinteză

```
chloros-cli process INPUT [OPTIONS]
```

### Argumente de poziție

| Argument | Descriere |
| --- | --- |
| `INPUT` | Calea către folderul de intrare care conține `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) sau `.dng`. |

### Opțiuni comune

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `-o, --output PATH` | un folder nou cu marcaj temporal în calea implicită a proiectului (`~/Chloros Projects`, dacă nu este configurat altfel) | Folderul proiectului care urmează să fie creat sau reutilizat. Dacă folderul conține deja un fișier `project.json`, se creează un folder înrudit `_1`/`_2` în loc să se suprascrie. |
| `-n, --project-name NAME` | auto (marcă temporală) | Numele proiectului. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utilizează un algoritm de debayer neuronal de tip Chloros+; mai lent, dar de calitate superioară. |
| `--vignette / --no-vignette` | `--vignette` | Corecția vignetării. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibrarea reflectanței (utilizează ținta panoului dacă este găsită, calibrare NIST pe serie pentru LATTICE). Pentru LATTICE multispectral, aceasta servește și ca comutator de reflectanță **** — vezi [Comutatoare de export pe produs](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | dezactivat | Aplică corecțiile PPK GNSS din fișierele sidecar. |
| `--exposure-pin-1 MODEL` | dezactivat | Fixează modelul „pin-1” al unui sistem cu două camere „Survey3” . |
| `--exposure-pin-2 MODEL` | dezactivat | Fixează modelul „pin-2”. |
| `--recal-interval SECONDS` | 0 | Forțează rulararea din nou a calculelor de calibrare la fiecare N secunde din timpul de captură. |
| `--timezone-offset HOURS` | local | Ignoră decalajul de fus orar încorporat în metadatele de ieșire. |
| `--format FORMAT` | `TIFF (16-bit)` | Una dintre următoarele: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | niciunul | Indici de vegetație (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forțează punctul de intrare în pipeline pentru fișierele TIFF LATTICE (Survey3 .raw nu este afectat). De asemenea, se elimină complet opțiunea care permite procesarea unei capturi cu **fără raw** să fie procesată deloc — vezi [Cum arată un folder de capturi](#what-arată-un-dosar-de-capturi). |
| `--debayered / --no-debayered` | activat | Generează produsul debayerat liniar (`Debayered_Images`). Vezi [Opțiuni de export per produs](#comutatoare-de-export-per-produs-lattice-multispectral). |
| `--preview / --no-preview` | activat | Emite previzualizarea pe ecran (`Preview_Images`): RGB = balansul de alb (iluminant DAQ, dacă este disponibil, altfel „gray-world”) + gamma; multispec = extindere în culori false. |
| `--radiance / --no-radiance` | activat | Emite radianță de tip float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referință pentru produsul de reflectanță LATTICE: `auto` = QA- ținta care trece prin cadru este referința absolută, cu DAQ-downwelling (ρ = π·L/E) ca soluție de rezervă; `target` = strict (fără substituție DAQ); `daq` = DAQ-autoritar. A se vedea [Comutatoare de export per produs](#comutatoare-de-export-pe-produs-lattice-multispectral). |
| `--target-reflectance-dir DIR` | none | Directorul scanărilor de reflectanță **măsurată** a țintei pe unitate (`<serial>.csv`); în caz de eșec, se revine la spectrele nominale T3/T4P. |
| `--array-alignment / --no-array-alignment` | activat | Matrice LATTICE: aplică alinierea modul-la--modul înscris în fișierul XMP al fiecărei capturi (`Chloros:Alignment*`) la fiecare produs procesat (debayering / previzualizare / radianță / reflectanță / index). Nu se aplică pentru imaginile fără etichete. |
| `--array-alignment-crop / --no-array-alignment-crop` | decupare | Decupează exporturile aliniate la regiunea de suprapunere comună a matricei, astfel încât toate modulele să împărtășească o singură amprentă; `--no-…` păstrează întreaga suprafață a senzorului (umplere cu negru în afara sursei). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Reeșantionare pentru deformarea de aliniere. `nearest` păstrează valorile DN exacte ale sursei (fără amestecarea valorilor radiometrice între pixeli). |

### Opțiuni de detectare a țintelor

| Indicator | Descriere |
| --- | --- |
| `--min-target-size PIXELS` | Dimensiunea minimă a țintei pe panou (px) pentru detector. |
| `--target-clustering 0-100` | Sensibilitatea grupării. |
| `--target / --targets` | Tratează folderul de intrare ca fiind exclusiv pentru panouri-țintă (omite detectarea sondajelor). |

### Exemple

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Comutatoare de export pe produs (LATTICE multispectral)

Prelucrarea LATTICE se ramifică în **fiecare produs aplicabil într-o singură trecere**. Cele patru comutatoare pe tip — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sunt toate**activate în mod implicit**; utilizați formularul `--no-<type>` pentru a dezactiva una dintre ele. Camerele master RGB emit întotdeauna doar imagini debayered + previzualizare (fără radianță pe bandă/reflectanță), astfel încât `--radiance`/`--reflectance` nu au niciun efect asupra acestora. Comutatoarele sunt ignorate pentru Survey3 `.raw` (care urmează calea standard de reflectanță/țintă). *(Vechiul indicator `--radiometric-output {reflectance,radiance,sensor-response}` a fost **eliminat** și înlocuit cu aceste comutatoare; nu mai există nivelul `sensor-response`.)*

| Produs | Ieșire | Este necesară înregistrarea DAQ a radiației descendente? |
| --- | --- | --- |
| `--debayered` | Demosaic liniar (`Debayered_Images`). | Nu. |
| `--preview` | Previzualizare afișaj (`Preview_Images`): RGB = WB + gamma; multispec = extindere în culori false. | Nr. |
| `--radiance` | float32 W/m²/sr/nm din lanțul radiometric complet (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflectanță ρ (`32768` = 1,0), compatibil cu Pix4D. | **Da**, cu excepția cazului în care este ancorat de o țintă din cadru care a trecut controlul de calitate (vezi mai jos). |

`--reflectance-source` alege referința de reflectanță:**`auto`**(implicit) face ca o țintă din cadru care a trecut testul de control al calității să fie**referință absolută**— lanțurile de linii empirice ancorate la țintă sunt evaluate încrucișat pe panouri reținute, iar câștigătorul măsurat este aplicat — revenind la diviziunea descendentă DAQ (ρ = π·L/E) atunci când nu este prezentă nicio țintă sau când controlul de calitate eșuează;**`target`**este strict (fără substituție DAQ);**`daq`**renunță la comportamentul autoritar al DAQ. Geometria țintei (ArUco / ROI fix / bandă) provine din configurația țintei proiectului; `--target-reflectance-dir DIR` păstrează, pentruunitate scanările**măsurate** (`<serial>.csv`) identificate după numărul de serie/codul QR al unității țintă, cu spectrele nominale T3/T4P ca opțiune de rezervă.

Calea de reflectanță DAQ rezolvă automat **raza descendentă corelată cu marcajul temporal**dintr-o**`.daq`**(DAQ-U/M/E)**sau a unui `.csv` nativ DAQ-M**găsit alături de imagini. Dacă un pachet de calibrare per cameră sau DAQ nu este stocat local în cache, pipeline-ul**îl preia automat din AWS** la prima utilizare (necesită conexiune la internet o singură dată; stocat în cache sub `~/.chloros/`).

#### Citirea pixelilor de reflectanță (Pix4D / Metashape / scripturile proprii)

Reflectanța este stocată ca valoare întreagă DN, iar **valoarea DN care corespunde lui ρ = 1,0 depinde de camera sursă**:

| Sursă | ρ = 1,0 este | Cum se identifică |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (marjă până la ρ 2,0) | Fișierul poartă marcajul XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (tăiat la ρ 1,0) | Nu există etichete XMP `Chloros:*` — această absență *este* semnalul. |

**Citiți `Chloros:PixelScale` și împărțiți la această valoare**, în loc să presupuneți o constantă. Eticheta este definită în domeniul uint16, deci rămâne `32768` în toate formatele de ieșire care se redimensionează — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` și `TIFF (32-bit, Percent)` sunt toate autodescriptive (normalizați mai întâi tipul de date stocat înapoi la uint16: ×257 din 8 biți, ×65535 din float).

> **Un caz nu conține nicio scală, conform proiectării.** Când o captură cu sursă de 8 biți (BayerRG8) este scrisă cTIFF de 8 biți, pipeline-ul *tăie* la intervalul 0..255 în loc să rescalzeze, astfel încât orice valoare peste ρ≈0,008 se uniformizează la 255 și niciun factor de scalare nu descrie fișierul. Chloros omite în mod deliberat atât `Chloros:PixelScale`, cât și tuplul `MicaSense:RadiometricCalibration` de acolo și înregistrează motivul. **Dacă eticheta lipsește dintr-un fișier de reflectanță LATTICE, nu presupuneți existența unei scale — reexportați la 16 biți sau 32 de biți**, în loc să împărțiți pixeli care nu au fost niciodată divizibili.

#### EXIF preluat în export

`process` copiază **blocul GPS și ExifIFD-ul** capturii sursă în fiecare produs, astfel încât un
exportul conține `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` și
`CameraSerialNumber` alături de georeferențiere.

**`FocalLength` nu este opțional pentru fotogrammetrie.** Pix4D calculează distanța dintre eșantioanele la sol pe baza
distanța focală plus altitudine; în absența etichetei, se revine la o scară extrem de eronată. Într-un
zbor deasupra unei plantații de portocali cu 49 de capturi, lipsa etichetei a transformat un sit de 411 m × 160 m într-unul reconstruit
de 47,8 km × 13 km — o ortofotografie de 455 MP alcătuită în mare parte din „nodata”, care a fost interpretată inițial ca o problemă de aliniere a imaginilor și
o problemă cu fișierul BigTIFF înainte ca cineva verifice GSD-ul. Dacă ortofotografia ta are o scară neverosimilă
, rulați mai întâi `exiftool -FocalLength` asupra produsului exportat.

Copia **nu** este în mod deliberat `-all:all`: etichetele structurale ale IFD0 afectează ieșirea LATTICE atunci când
sunt copiate, iar `ExifImageWidth` / `ExifImageHeight` sunt excluse deoarece descriu
captura *sursă* — un export care a fost vreodată redimensionat ar conține altfel dimensiuni
contradictorii cu propriul raster. Fișierul XMP este scris direct, nu copiat, deoarece ExifTool
elimină etichetele XMP cu aceeași invocare atunci când blocul XMP este copiat (ceea ce ar elimina etichetele de calibrare MAPIR
).

### Unde sunt salvate rezultatele

Produsele sunt salvate **în folderul proiectului, grupate mai întâi după cameră și apoi după formatul fișierului**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folderul camerei este `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE (corespunzând EXIF-ului capturii
`Model`) și `<model>_<filter>` pentru Survey3 — două camere care împărtășesc același senzor și filtru, dar care diferă
în ceea ce privește obiectivul, au arbori separați, deoarece vignetarea, câmpul vizual și distorsiunea diferă. Formatul
dosarului urmează `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` sau `tiff32` pentru
`TIFF (32-bit, Percent)`.

> **Fiecare produs exportat păstrează numele fișierului SURSĂ.** Un export Radiance al
> `capture_…_raw.tif` se numește în continuare `capture_…_raw.tif` — doar că se află în
> `tiff32/Radiance_Images/`. **Folderul identifică produsul, nu numele fișierului**, așa că utilizarea operatorului glob
> pentru `*radiance*.tif` nu găsește nimic; căutați în schimb în director.

### Înregistrări ale senzorului de lumină — calibrate `.daq` + `.csv`

`process` gestionează, de asemenea, înregistrările `.daq` din folderul dvs. de intrare și **nu**
are nevoie de nicio imagine pentru a face acest lucru: un DAQ-U / DAQ-M / DAQ-E zburat singur reprezintă o
captură completă, iar un folder care conține doar fișiere `.daq` este o intrare validă.

Un DAQ poate fi înregistrat **fără** calibrarea sa — aceasta este ceea ce înregistrează
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fac în mod implicit: ele scriu valorile brute ale senzorului și marchează fișierul astfel încât
Chloros să preia calibrarea din fabrică a senzorului respectiv **după numărul de serie** (mai întâi din cache-ul local,
apoi din cloud-ul MAPIR) și o aplică. `process` scrie rezultatul înapoi:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` conține un rând pentru fiecare citire: marcaj temporal UTC, timp de integrare, putere totală,
lux fotopic/scotopic, PPFD (și defalcarea sa albastru/verde/roșu), lungimea de undă de vârf, apoi
spectrul complet pe grila de lungimi de undă proprie a senzorului. `.daq` reimportează fără a fi
calibrat a doua oară.

În caz de succes, execuția raportează `Light-sensor products written: N (calibrated .daq + .csv)`.
Textul dintre paranteze descrie ce a fost scris efectiv, astfel încât se citește
`(RAW COUNTS — this sensor has no calibration bundle)` pentru un senzor fără pachet și
`(N calibrated, M raw counts)` pentru un folder care conține ambele. Titlurile proprii ale backend-ului,
`[DAQ-EXPORT]` și `[RUN-SUMMARY]`, își derivă formularea în același mod — niciunul dintre
cele trei nu poate numi un export brut „calibrat”.

O înregistrare DAQ-U / DAQ-M / DAQ-E al cărei pachet de calibrare nu poate fi preluat — sunteți
offline sau senzorul respectiv nu are nicio calibrare înregistrată — este **omise cu menționarea unui motiv** pe o
linie `[DAQ-EXPORT]`, nefiind niciodată salvată ca fișier „calibrat” care conține numărătoarea brută.
Conectați-vă la internet și rulați din nou procesul. Motivul este cel pe care cititorul a stabilit-o efectiv
pentru acel fișier (schemă ilizibilă, lipsa pachetului, o eroare de scriere), iar rezumatul
execuției listează motive **distincte** — douăzeci de fișiere omise dintr-o singură cauză sunt afișate ca o singură
cauză, nu ca douăzeci de repetări ale acesteia.

#### Exportul înregistrărilor DAQ-A ca numere brute

Familia **DAQ-A** este anterioară sistemului de pachete pe serie și nu are un pachet de calibrare
de preluat — în schimb, este calibrată pe teren în raport cu o țintă de reflectanță, motiv pentru care
nu a avut niciodată nevoie de unul. Refuzarea acestor înregistrări le-a lăsat fără nicio posibilitate de a-și
extrage deloc datele, așa că se exportă sub un **nume diferit**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Un nume de fișier diferit, în loc de un indicator în interiorul fișierului, deoarece informația trebuie să rămână intactă
atunci când este trimisă prin e-mail doar ca nume simplu. Antetul `.csv` indică
`raw spectral sensor counts (NOT irradiance)` și avertizează că valorile sunt comparabile
**în cadrul** fișierului — ceea ce este exact scopul pentru care sunt utilizate în calibrarea bazată pe țintă — și
nu între senzori. Coloanele fotometrice dependente de putere (puterea totală, luxul fotopic și
scotopice, PPFD) sunt înregistrate ca **NULL** în loc să fie integrate din numărări, iar rezumatul
execuției indică `RAW COUNTS`, astfel încât datele „exportate” într-un jurnal nu pot fi interpretate ca iradianță.

Înregistrările vechi **v1.01 / v1.02** (un DAQ-A-SD le scrie) nu conțin o epocă pentru fiecare citire,
ci doar ora de scriere a fișierului. Programul de corelare imagine↔radiație descendentă încă le respinge — potrivirea unui
cadru cu o oră de scriere ar fi greșită, fără a se observa — dar exportatorul le citește, iar
CSVul afișează `clock=daq_created_on`, astfel încât produsul indică ce ceas utilizează.

### Note

- `process` detectează automat dacă folderul dvs. este de tip „Survey3”, „LATTICE” sau mixt.
- Progresul este transmis prin Server-Sent Events; CLI afișează în timp real progresul pe fiecare fir de execuție (Detectare, Analiză, Procesare, Export).
- Pentru Linux /Jetson, CLI verifică memoria swap și poate afișa un avertisment înainte de procesarea folderelor mari. Funcția de debayer sensibilă la textură aplică automat și o limită de frecvență a GPU-ului pe dispozitivele Jetson cu consum redus de energie (Nano, Orin Nano).
- În caz de succes, execuția raportează câte imagini a generat (`Image products written: N`).

#### O execuție care nu generează imagini eșuează

Dacă ați solicitat produse, iar execuția nu a scris **niciunul** — doar `project.json` și
`calibration_data.json` — `process` tratează acest lucru ca pe un eșec: afișează
`Processing finished but wrote no image products.` și **iese cu o valoare diferită de zero**, astfel încât un script să poată
detecta acest lucru. Mesajul menționează numele folderului proiectului și cauzele obișnuite:

- folderul de intrare nu a fost recunoscut ca o captură (verificați structura și `--input-level`), sau
- fiecare produs solicitat a fost omis ca fiind inaplicabil pentru acele camere (de exemplu, solicitarea
  radianței/reflectanței de la camere care suportă doar modul „RGB”).

Rulați din nou cu `--verbose` și verificați jurnalul backend-ului pentru liniile `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
care explică omisiunile specifice fiecărei camere care, altfel, nu ajung în ieșirea CLI.

O execuție deliberată numai cu metadate — toate produsele dezactivate și fără `--indices` — este totuși un
**succes**, deoarece o ieșire de imagini goală este rezultatul corect în acest caz.

La fel este și o **execuție numai cu senzorul de lumină**: un folder cu înregistrări `.daq` nu conține imagini de exportat
prin definiție, iar execuția este evaluată pe baza fișierelor calibrate `.daq` / `.csv` pe care a scris în schimb.

---

## `chloros-cli login`

Autentificați acest computer cu un cont cloud de tipul Chloros+. Datele de autentificare sunt stocate în cache în mod securizat în `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemple

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (eliminând sau duplicând părți din parolă). La un cod de eroare 401, CLI încearcă automat din nou cu `$$` adăugat la sfârșit, apoi cu jumătate din parolă fără duplicări; dacă o nouă încercare reușește, vă autentifică și afișează sintaxa corectă cu ghilimele simple pe care să o utilizați data viitoare.

> **Utilizare fără interfață grafică/prin script: lipsa unei sesiuni stocate în cache înseamnă un prompt interactiv, nu o eroare rapidă.** Orice comandă care generează un proces de fundal (`process`, `status`, `export-status`, `time-sync`, …) care rulează fără o licență/sesiune stocată în cache va afișa un prompt interactiv `Email:` / `Password:` pe stdin înainte de a continua. Prin urmare, o sarcină nesupravegheată fără sesiune stocată în cache se va bloca așteptând o intrare — rulați `chloros-cli login EMAIL PASSWORD` o dată pe fiecare mașină înainte de a programa sarcini fără interfață grafică.

---

## `chloros-cli logout`

Șterge sesiunea stocată în cache și forțează o autentificare nouă la următoarea apelare.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Afișează nivelul actual de licență (Iron/Copper/Bronze/Silver/Gold), utilizatorul autentificat și numărul de asocieri cu dispozitive.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Verifică progresul în timp real al exportului Thread-4. Poate fi apelat în siguranță **în timpul** executării comenzii `process` dintr-un alt shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Setează limba de afișare a „CLI” (sunt acceptate 38 de limbi, inclusiv CJK, RTL și limbile indiene). Revine în mod corespunzător la limba engleză pe consolele vechi care nu pot reda scriptul.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exemple

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comenzi pentru folderul de proiect

Acestea gestionează locația implicită a folderului de proiect (comună cu interfața grafică).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Numai pentru Jetson. Verifică `version_url` din `/etc/chloros/update.conf` și oferă posibilitatea de a descărca și instala fișierul corespunzător `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Pe Linux / Jetson, CLI efectuează, de asemenea, o **verificare automată a actualizărilor la fiecare pornire** (fără blocare, nu întârzie niciodată comanda): citește `/etc/chloros/update.conf`, stochează rezultatul în cache timp de 1 oră în `~/.chloros/update_cache.json` și afișează `Update available: vX.Y.Z / Run: chloros-cli update` atunci când există o versiune mai nouă. Se omite în mod silențios în cazul oricărei erori și în cazul fișierului „Windows”.

---

## `chloros-cli selftest`

Efectuează un test de funcționare în 7 pași: versiune, disponibilitatea porturilor, pornirea backend-ului, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), prezența modelului de denoizare, disponibilitatea CUDA+denoizare.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Starea și controlul grandmasterului PTP. Gazda Chloros rulează grandmasterul PTP; camerele LATTICE și unitățile DAQ-E funcționează ca slave ale acestuia pentru marcaje temporale între dispozitive.

| Subcomandă | Descriere |
| --- | --- |
| `status` | Afișează starea grandmasterului, prioritățile BMCA, identitatea ceasului. |
| `peers` | Listează dispozitivele slave detectate prin Delay_Req (camere + senzori DAQ-E). |
| `cameras` | Starea PTP pentru fiecare cameră (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Repornirea procesului grandmaster. |
| `set-priority --priority1 N --priority2 N` | Ignorarea priorităților BMCA. |

### Exemple

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controlul camerei LATTICE. Fiecare subcomandă trece prin backend-ul Chloros; backend-ul deține grupul de camere, astfel încât apelurile ulterioare CLI reutilizează același identificator deschis.

### Opțiuni comune (împărtășite de majoritatea subcomenzilor)

| Indicator | Descriere |
| --- | --- |
| `-d, --device N` | Indicele camerei (implicit: 0). |
| `-s, --serial SN` | Număr de serie specific; are prioritate față de `--device`. |
| `--serials SN1,SN2,…` | Seriale separate prin virgulă pentru funcționarea cu mai multe camere. |
| `--all` | Funcționează pe fiecare cameră detectată. |
| `--exposure US` | Timpul de expunere în microsecunde. |
| `--gain DB` | Amplificare în dB. |
| `--pixel-format FMT` | deg. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimensiunile imaginii. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplică o presetare de setări. Toate funcționează în mod liber, cu excepția `triggered`, care armează camera pentru o margine de semnal hardware pe linia 2 — dacă nimic nu activează acea linie, camera va aștepta la nesfârșit în loc să captureze. |
| `-o, --output DIR` | Director de ieșire (implicit: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Dimensiunea pachetului GVSP. `auto` execută sonde ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Fluxul de lucru pentru prima conectare a camerelor LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referință subcomenzi

#### Descoperire și informații

| Subcomandă | Scop |
| --- | --- |
| `lattice info` | Afișează lista camerelor conectate (fabricant, model, număr de serie, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analizează sistemul gazdă pentru configurarea optimă a camerei. `--no-discover` omite (mai rapidă, analiză numai pe baza plăcii de rețea). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Verifică/corectează setările plăcii de rețea; estimează lățimea de bandă/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Capacitate de rețea a backend-ului cu schemă stabilă + recomandare de matrice (returnează `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` păstrează rezoluția solicitată, dar limitează numărul de cadre pe secundă (fps) țintă — citește `recommended.recommended_target_fps` și transmite-l ca țintă de conectare; tratează-l ca pe un succes, nu ca pe o eroare. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analiză ipotetică fără deschiderea camerelor. **`--n-active` reprezintă numărul total de camere conectate, nu doar pe cele din acest tablou**— creșteți-l atunci când camerele independente transmit simultan, sau când bugetul rețelei este calculat pe baza unei cereri care le subestimează numărul (implicit: `len(--models)`). Afișează întotdeauna valoarea agregată `Wire budget:` (MB/s solicitați vs. plafonul fără coliziuni) și `Max cameras:`, și semnalează `** OVER-SUBSCRIBED**` când matricea suprasolicită cablul — vezi [Modelul fps și burst al matricei](#array-fps--burst-model). |
| `lattice gpu` | Afișează starea GPU-ului. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verifică sau actualizează firmware-ul camerei. Selecția locală `.fwa` este fixată: fișierul din `firmware/<MODEL_PREFIX>/` care corespunde cu `MIN_FIRMWARE_VERSION` al versiunii de compilare este flash-uit atunci când este prezent (versiunea cea mai recentă doar ca soluție de rezervă), astfel încât o imagine de la furnizor mai nouă stocată pe disc rămâne inactivă până când acea fixare este modificată — versiunile mai noi ajung în mod deliberat la unități prin intermediul manifestului AWS semnat, care este preferat atunci când este mai nou. |
| `lattice presets [--apply NAME]` | Afișează sau aplică presetările camerei. |
| `lattice status` | Starea camerei în timp real. |

#### Captură

| Subcomandă | Scop |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Cadru unic. **Salvează implicit fiecare tip de export** (`--processing all`); consultați [Niveluri de export pentru captură](#capture-export-levels-the-all-default). `--levels` salvează un subset explicit (înlocuiește `--processing`); `--force-daq` scrie valoarea DAQ atribuită ca fișier `.daq` chiar și în cazul unei capturi exclusiv în format brut. `--jpeg-quality` = JPEG calitate 1–100 (implicit 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Transmite pe disc până la apăsarea Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Previzualizare MJPEG live în browser. `--ae-damping` setează amortizarea expunerii automate (0,4–100). |

#### Reglarea senzorului

| Subcomandă | Scop |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Citește/scrie orice nod GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Expunere și AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Câștig și câștig automat. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Zona de interes (ROI) a senzorului și binning. |
| `lattice format [--set FMT] [--list]` | Formatul pixelilor. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Declanșare hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (fără indicatori = compensare de alb cu o singură expunere) | Operațiuni de compensare de alb. Numai pentru camereleRGB/Bayer; o operație nulă (omise) pe M3M monocrom. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB pipeline de culoare pentru afișare. `natural` (implicit) reprezintă finisajul live ieftin; `enhanced` adaugă eliminarea franjurilor + vibranță + contrast local CLAHE pentru aspectul complet de paritate a hub-ului la ~2× costul de finisare pe cadru, deci o ****live** — capturile salvate beneficiază întotdeauna de finisajul complet, indiferent de opțiune. Numai pentru camerele RGB /Bayer; omis pe M3M mono. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Afișează saturația/contrastul (camere cu filtru RGB). Se omite pe M3M mono. |
| `lattice filter [--set NAME] [--list]` | Setează modelul de filtru al camerei (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Sondează nodurile de alimentare/termice; comută modul de repaus cu consum redus de energie. |

#### Calibrare și senzori

| Subcomandă | Scop |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrare folosind o țintă de reflectanță. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Comenzi pentru senzorul integrat de lumină descendentă. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplică corecția de vignetare imaginilor existente. |

#### Multi-cameră (sesiuni tranzitorii)

| Subcomandă | Scop |
| --- | --- |
| `lattice multi-info` | Afișează lista tuturor camerelor cu roluri de sincronizare. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Un cadru sincronizat de la fiecare cameră. Salvează **toate tipurile de export în mod implicit**atunci când este conectat un array persistent; opțiunea de rezervă tranzitorie fără array este**doar debayerat** (rulați mai întâi `array-connect` pentru restul). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Transmite cadre sincronizate (tranzitorie). |
| `lattice multi-test [--count N]` | Test de sincronizare GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Detectare automată a cablării GPIO master/slave. |

#### Aliniere

| Subcomandă | Scop |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus butoane de detectare/potrivire `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, butoane RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinare multi-cadru `[--averaging mean\|median\|inlier_weighted]`, constrângeri geometrice `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restricție spațială `[--roi X0,Y0,X1,Y1] [--mask PATH]` și suprascrieri per-slave `[--per-cam-override SN:KEY=VALUE]` (repetabil) | Calculează profilul de aliniere din camerele live. `--prefilter` utilizează implicit `gradient` (hartă de margini; corespunde aliniatorului GUI/matrice — marginile se mențin pe toate benzile spectrale). `--matcher flann` dă rezultate bune la peste ~5000 de caracteristici; `--averaging median` este robust față de o captură defectuoasă, `inlier_weighted` ponderează în funcție de numărul de potriviri; `--lock-scale` proiectează către cea mai apropiată rotație (fără scalare), `--lock-axis` pune la zero o componentă de translație; `--mask` se aplică tuturor camerelor (utilizați `--per-cam-override` pentru setări specifice fiecărei camere, de exemplu, `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` refuză să salveze o calibrare a cărei valoare RMS a reproiectării depășește pragul. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Capturează un cadru multibandă aliniat. `--bit-depth` se potrivește implicit cu camera; `--no-crop` păstrează cadrul complet (completând cu negru); `--interpolation` (implicit `linear`) și `--border-mode`/`--border-value` (implicit `constant`/0) controlează deformarea CPU — calea GPU este oricum biliniară. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Cadre multibandă aliniate la flux (aceleași comenzi de warp ca `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Afișează detaliile profilului. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Modifică ordinea straturilor. |

#### Index / Calculele privind vegetația

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Set complet de indicatori: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (repetabil), `--capture-level raw|debayered|radiance|reflectance|unknown` (înlocuiește nivelul de captură înregistrat în sursa TIFF; implicit: citit din metadatele TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. În cazul modelului `--live`, se aplică și butoanele de aliniere: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Simbolurile `--channel` sunt sensibile la majuscule și minuscule.** Partea cu simbolul trebuie să corespundă exact cu numele canalelor din preset (presetările folosesc litere mici, de exemplu NDVI = `red`,`nir` — verificați `--list-presets`), iar partea cu banda trebuie să corespundă unui nume de bandă din stiva aliniată (sau să fie un index de bandă începând de la 0 în modul offline). `--channel red=Red_660 --channel nir=NIR_850` funcționează; `--channel RED=660` eșuează cu o eroare `channel_map missing entries`.

#### Conexiuni persistente (Smart-Prep, flux echivalent GUI)

Aceste comenzi mențin camerele deschise în grupul din backend între invocările `CLI`.

| Subcomandă | Scop |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Adăugarea unei camere în grup (o singură cameră, fără matrice). |
| `lattice cam-disconnect [--serial SN] [--all]` | Eliberare. |
| `lattice cam-list` | Afișează lista camerelor din grup. |
| **`lattice array-connect`**|**Conectează o matrice sincronizată persistentă (punctul de intrare recomandat).** Rulează fluxul complet de pregătire inteligentă prin interfața grafică. |
| `lattice array-disconnect [--array-id ID] [--all]` | Eliberare matrice. |
| `lattice array-list` | Listare matrice conectate. |
| `lattice array-status [--array-id ID]` | FPS în timp real, PTP, ultima eroare. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | O captură sincronizată de pe matricea live — Singură / Continuă / La intervale / Cea mai rapidă. **Implicit: `all`** (un fișier pentru fiecare tip de export aplicabil, pentru fiecare cameră). Camerele omise (de ex., RGB excluse din radianță/reflectanță) sunt raportate cu `Skipped: SN:<serial> (<reason>)`; valoarea DAQ utilizată pentru reflectanță este salvată împreună și raportată cu `DAQ: <path>`. A se vedea [Moduri de captură, înregistratoare și reprocesare offline](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Înregistrează vizualizarea live a indicelui combinat în format video/GIF (calitate de monitorizare; necesită deschiderea fluxului combinat). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Rafală raw-Bayer cu fps ridicat (calitate de analiză; reprocesare offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocesarea unei serii raw salvate în videoclipuri calibrate. |

##### Opțiuni `array-connect`

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--serials SN1,SN2,…` | descoperire automată a tuturor camerelor LATTICE (sunt necesare ≥2) | Primul număr de serie este MASTER. Când este omis, detectarea filtrează modelele LATTICE (`TRI032*`) și le conectează pe toate. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linie de sincronizare GPIO. |
| `--target-fps F` | auto | Rata de declanșare a Masterului. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ignoră selectorul de nivel. |
| `--wire-ceiling-mbps MB_PER_S` | detectat automat | **Bugetul de lățime de bandă susținută al gazdei, în MB/s — valoarea de care depinde întreaga alocare a matricei.** Reduceți-o atunci când matricea raportează cadre GVSP: valoarea implicită este derivată din rata de legătură anunțată de placa de rețea, care supraestimează adaptorii USB, benzile PCIe subțiri și structurile partajate aglomerate. Este stocată înblocul de captură a matricei, astfel încât o reconectare prin „reopen”, „CLI” sau „SDK” o restabilește. Consultați [Starea matricei](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Grupare hardware. |
| `--no-recommend` | dezactivat | Omite etapa de analiză a rețelei. |
| `--no-ptp` | dezactivat | Dezactivează PTP (marcajele temporale între camere **nu** sunt atunci comparabile). |

### Smart-AE / Captură inteligentă

Matricile LATTICE rulează AE continuu în fundal imediat ce sunt conectate, dar o scenă nou configurată necesită un moment pentru a converge. `array-capture --smart` este **soluția practică**: așteaptă ca AE să se stabilizeze pe toate camerele din matrice, apoi declanșează capturarea. Folosiți-l când schimbați scenele în timpul sesiunii.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Politica de stabilizare este conservatoare în mod implicit: timp de așteptare de 5 s, fereastră de stabilitate de 1,5 s, toleranță de variație a expunerii de ±5 %. Reglați prin SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) dacă aveți nevoie de un comportament diferit din partea automatizării.

### Niveluri de export al capturilor (implicit `all`)

Începând cu această versiune, `lattice capture`, `lattice multi-capture` și `lattice array-capture` **au ca setare implicită `--processing all`** — un singur fișier salvat pentru fiecare tip de export, valabil pentru fiecare cameră, corespunzând comportamentului opțiunii „Capture All” (Captură totală) din interfața grafică. Nivelurile sunt:

| Nivel | Ieșire | Se aplică la |
| --- | --- | --- |
| `raw` | Bayer monocanal (camere monocrome: banda unică) direct de la senzor. | Toate camerele. |
| `debayered` | Demosaic BGR cu 3 canale (camere monocrome: 1 canal în tonuri de gri). | Toate camerele. |
| `radiance` | float32 W/m²/sr/nm prin lanțul radiometric complet. | Multispectral (M3C/M3M) numai — **omise pentru camerele cu filtru „RGB”**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatibil cu Pix4D. | Numai multispectral, și **numai când un DAQ este asociat + camera este calibrată**; altfel este omis. |
| `preview` / `display` | Lanț complet de previzualizare GUI (CCM + WB + gamma conform profilului camerei). `lattice capture` denumește acest `preview`; `array-capture`/`multi-capture` utilizează `display`. | Toate camerele. |

Treceți un singur nivel pentru a salva doar acela (`--processing debayered`). Când solicitați `all`, nivelurile care nu se aplică unei anumite came sunt omise (și raportate), nu sunt semnalate ca erori — o cameră neconectată sau necalibrată primește totuși `raw` / `debayered` / `preview`.

Pentru orice cadru de reflectanță, citirea de către DAQ a este scrisă într-un fișier **`.daq`** atașat lângă imagini (astfel încât captura să poată fi reprocesată ulterior) și raportată pe o linie `DAQ:`.

### Cum arată un folder de capturi

Fiecare tip de export ajunge în **propriul subfolder** sub `-o`, astfel încât o captură pe mai multe niveluri nu amestecă niciodată tipurile:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` reprezintă marca temporală a capturii, iar `<serial>` numărul de serie al camerei, astfel încât un grup sincronizat are aceeași
marcă temporală comună pentru toate camerele. **Rețineți singura asimetrie:** nivelul `display` este stocat într-un folder
numit `preview/`, în timp ce fișierele în sine păstrează `_display` în nume — numele folderului și sufixul diferă
numai pentru acel nivel. Nivelurile necunoscute sunt plasate într-un dosar cu numele propriu, iar dacă subfolderul
nu poate fi creat, fișierul este scris în rădăcina de ieșire, în loc să se piardă.

**Reprocesarea unui folder de capturi:**indicați `chloros-cli process` către**rădăcina capturilor**
(`output/`). `process` importă în mod normal doar folderul pe care îl numiți, dar când acel folder nu conține
imagini și are subfoldere, acesta coboară automat — astfel încât subfolderele de la nivelul rădăcinii și
`.daq` de la rădăcină sunt preluate toate dintr-o singură mișcare. Fiecare nivel al unei capturi se importă ca o singură imagine, iar
celelalte niveluri sunt disponibile ca moduri, în loc să fie câte o imagine pe nivel.

Denumirea directă a unui **subfolder de nivel** (de exemplu, `output/raw/`) funcționează, de asemenea. Procedând astfel, rădăcina
`.daq` este omisă, așa că copiați sau indicați citirea DAQ alături atunci cândderivați un produs radiometric
din `raw/` — altfel, potrivirea marcajului temporal nu are cu ce să se raporteze.

**Prelucrarea începe întotdeauna de la `raw`.** În cadrul fiecărei capturi, cadrul brut este sursa pipeline-ului;
`debayered`, `radiance`, `reflectance` și `preview` apar ca moduri vizualizabile, dar nu sunt niciodată redirecționate
înapoi prin pipeline. Reprocesarea unui produs derivat ar reaplica vignetarea, CCM și
calculele de radianță care sunt deja integrate în pixelii săi, astfel încât Chloros preferă să renunțe decât să
o dublă procesare. Două consecințe demne de reținut:

- Renderizările `index/` și `composite/` nu sunt **niciodată** procesate. Acestea sunt ieșiri, nu capturi —
  un render LUT „NDVI” nu are o interpretare semnificativă a radianței.
- Un export al folderului „captures”**fără** `raw` (de exemplu, `array-capture --processing reflectance`) nu are
  nicio sursă legitimă în pipeline. Aceste capturi se importă și se afișează normal, dar `process` le omite
  și indică acest lucru:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Dacă aveți cu adevărat nevoie să treceți un produs derivat — o sesiune hub capturată cu
  `demosaic` activat sau un folder vechi — `--input-level {raw,debayered,processed}` forțează punctul
  de intrare și anulează omiterea. Acest indicator este o cale de ieșire deliberată; `auto` (implicit)
  nu procesează niciodată o captură care nu are date brute.

### Capturi omise în matrice cu filtre mixte

Când combinați camere dRGBă și multispectrale într-o singură matrice, `array-capture --processing radiance` (sau `reflectance`) salvează cadrele multispectrale și **omite** camerele „RGB” — radianța pe pixel Bayer nu are sens pentru un senzor de bandă largă. Comanda „CLI” afișează explicit fiecare fișier salvat (împreună cu nivelul său de export), fiecare fișier `.daq` scris și fiecare omisiune, astfel încât numărul de fișiere nu este surprinzător:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokenurile care indică motivul omiterii urmează modelul `<level>-not-applicable-to-rgb-cam`. Reflectanța poate fi, de asemenea, omisă cu `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` și cu `dls-uncalibrated-band-<nm>` atunci când banda se află în mare parte în afara intervalului calibrat radiometric al senzorului de lumină DAQ (~374–974 nm) — dintre codurile de produs disponibile, doar F988, a cărui cale acceptată este fluxul de lucru cu panoul de reflectanță.

Utilizați `--processing debayered` (sau `display`) pentru a include toate camerele, indiferent de tipul de filtru, sau comanda implicită `all` pentru a obține toate nivelurile aplicabile pentru fiecare cameră dintr-o singură operațiune.

---

## Moduri de captură, înregistratoare și reprocesare offline

Toate acestea funcționează pe un **tablou persistent** (rulați mai întâi `array-connect`). Ele reflectă panoul de captură din interfața grafică.

### Modurile `array-capture`

`array-capture` este o singură comandă cu patru moduri de declanșare, plus un set de opțiuni de export:

| Mod | Indicator | Comportament |
| --- | --- | --- |
| **Unic** *(implicit)* | (niciunul) | Un singur grup de capturi sincronizate, apoi ieșire. |
| **Continuu** | `--continuous` | Treceri consecutive până la `Ctrl+C`, `--count N` sau `--duration S`. |
| **Interval** | `--interval S` | O trecere la fiecare `S` secunde (măsurat de la începutul fiecărei treceri), aceleași limite. |
| **Cel mai rapid** | `--fastest` | Doar date brute + citirea DAQ atribuită + compozitul cu indice combinat; omite calculele de radianță/reflectanță/afișare, astfel încât cadrul să se încarce rapid. Implică `--processing raw --force-daq`. Reprocesați ulterior `.daq`-ul salvat în produse calibrate. |

Opțiuni de export (se pot combina cu orice mod; toate utilizează interfața grafică/punctul final SDK):

| Indicator | Efect |
| --- | --- |
| `--processing LEVEL` | Un singur nivel de export sau `all` (implicit). |
| `--levels L1,L2,…` | Subset explicit de tipuri de export (de ex. `raw,radiance,reflectance`); **înlocuiește `--processing`**. |
| `--aligned` / `--no-aligned` | Aliniază exportul non-raw al fiecărui element la [profilul de aliniere](#alignment) (co-înregistrat). Datele brute rămân nealiniate, dar poartă transformarea în metadate. Se revine la nealiniat (cu un avertisment) dacă matricea nu are profil. |
| `--index` / `--no-index` | Salvează / omite suprapunerea indicelui de vegetație pentru fiecare cameră, acolo unde este configurată una. Implicit: o redă. |
| `--force-daq` | Salvează citirea DAQ/DLS atribuită ca fișier sidecar `.daq` chiar și atunci când niciun nivel selectat nu are nevoie de aceasta (de exemplu, o captură doar cu date brute), astfel încât cadrele să poată fi reprocesate offline în reflectanță/indice. |
| `--smart` | Așteaptă stabilizarea AE pe toate camerele înainte de declanșare (vezi [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compresie de pixeli „TIFF”. `deflate` (implicit) = zlib L1 fără pierderi + predictor orizontal, ~4,1 MB pe cadru la rezoluție maximă; `none` = fără, scriere de ~5 ori mai rapidă la ~6,3 MB pe cadru — se utilizează pentru o rată susținută maximă atunci când spațiul pe disc permite acest lucru. Ambele sunt fără pierderi și se citesc identic la import. |

> **TIFF cu o singură scriere + modelul cu rată susținută.**Capturile sunt scrise într-o**singură**trecere a fișierului TIFF, care conține pixeli + XMP + IFD0 Marcă/Model (măsurat pe Mono12 la rezoluție maximă: 36 ms comprimat / 6,5 ms necomprimat, față de ~148 ms pentru vechea metodă de scriere urmată de rescriere cu ExifTool); singura operațiune ExifTool rămasă (finisarea sub-IFD-ului EXIF) rulează într-un procesor de fundal asincron, iar un cadru este complet și gata de import chiar dacă acesta nu rulează niciodată. Rețineți că compresia DEFLATE menține GIL-ul Python, așa că scrierile comprimate**nu**se paralelizează între firele de scris ale fiecărei camere — captarea susținută la rezoluție maximă cu 8 camere la rata senzorului (~10,4 fps) necesită `--compression none`**și** un disc de clasă NVMe (~500 MB/s de scrieri susținute). Același parametru este expus ca `compression` pe `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — video/GIF cu index combinat (de calitate pentru monitorizare)

Înregistrează tot ceea ce afișează **vizualizarea live cu index combinat** pe un `.avi` (și, opțional, un `.gif`). Deoarece preia semnalul compozit în timp real, fluxul combinat trebuie să fie deschis (de exemplu, matricea este previzualizată în interfața grafică) pentru ca cadrele să fie înregistrate. Verifică progresul la fiecare 2 s și se oprește pe `--duration`, `Ctrl+C` sau când înregistratorul se oprește automat.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--array-id ID` | numai matrice | Matrice țintă (se omite dacă este conectat doar unul). |
| `-o, --output DIR` | `output` | Director de ieșire (local pe backend). |
| `--fps F` | `10` | Rata de cadre a înregistrării. |
| `--duration S` | până la Ctrl+C | Oprire automată după `S` secunde. |
| `--gif` | dezactivat | Se scrie și un GIF animat. |
| `--gif-only` | dezactivat | Se scrie doar un GIF (fără `.avi`). |

### `array-burst` — rafală raw-Bayer cu fps ridicat (calitate de analiză)

Citește direct bufferul grupului sincronizat al buclei de captură — **nu este necesar niciun lanț de calibrare, nici exiftool, nici vizualizare live** — astfel încât rulează la rata maximă de captură a camerei. Scrie cadre raw + un manifest per cadru + un `.daq` pentru fiecare citire DLS distinctă sub `<output>/bursts/<base>/`. Reprocesați offline (comanda următoare) sau transmiteți `--build` pentru a efectua reprocesarea imediat la oprire.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--array-id ID` | numai matrice | Matrice țintă. |
| `-o, --output DIR` | `output` | Director de ieșire (burst-ul ajunge în `<DIR>/bursts/<base>/`). |
| `--duration S` | până la Ctrl+C | Oprire automată după `S` secunde. |
| `--max-frames N` | nelimitat | Oprire automată după `N` de cadre brute. |
| `--build` | dezactivat | După oprire, reprocesează imediat seria (la fel ca `array-build-video`). |
| `--products …` | `combined:index` | Cu `--build`: ce fișier(e) video să se creeze (vezi mai jos). |
| `--fps F` | `10` | Cu `--build`: fps-ul videoclipului de ieșire. |
| `--save-tiffs` | dezactivat | Cu `--build`: salvează și fișiere TIFF calibrate pentru fiecare cadru. |
| `--gif` | dezactivat | Cu `--build`: scrie și GIF-uri animate. |

### `array-build-video` — reprocesare offline a unei serii salvate

Sincronizează temporal fiecare cadru brut cu cea mai apropiată valoare `.daq` salvată și îl trece prin **același lanț de radianță / reflectanță / indice ca și fluxul de import**, generând unul sau mai multe videoclipuri.

`--products` este o listă separată prin virgulă de elemente `kind:level`, unde `kind` ∈ `per_cam` | `combined` și `level` ∈ `radiance` | `reflectance` | `index`. Un `level` simplu (fără `kind:`) are ca valoare implicită `per_cam`. Valoarea implicită este `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--burst-dir DIR` | (obligatoriu) | Calea către folderul de burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, ca mai sus. |
| `--fps F` | `10` | Număr de cadre pe secundă (fps) pentru video. |
| `--save-tiffs` | dezactivat | Salvează, de asemenea, fișiere TIFF calibrate pentru fiecare cadru alături de videoclipuri. |
| `--gif` | dezactivat | Salvează, de asemenea, fișiere GIF animate. |

> **Alegeți înregistratorul potrivit.** `array-record` este de *nivel de monitorizare* — captează semnalul compozit live așa cum este afișat și necesită ca fluxul deschis. `array-burst` → `array-build-video` este de *nivel de analiză* — salvează datele brute ale senzorului la viteză maximă și reconstruiește ulterior videoclipuri calibrate de radianță/reflectanță/indice, fără a fi necesară vizualizarea în timp real.

### Camere monocrome (M3M) cu o singură bandă

Seria **M3M**este varianta mono a modelului Bayer**M3C**: un singur filtru de interferență în bandă îngustă per cameră (`M3M-<lens>-F<wavelength>`, de exemplu `M3M-L87-F685`), astfel încât senzorul oferă o**singură bandă în tonuri de gri**, fără mozaic Bayer . Nu există nimic de demosaic, nici o interferență între canale de separat și niciun balans de alb de setat — întregul proces de afișare a culorilor „RGB” pur și simplu nu se aplică.

Ce înseamnă asta pentru CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detectează o cameră monocromă și**trec mai departe cu un mesaj de o singură linie**, în loc să aplice setări fără sens. Acestea funcționează în continuare normal cu o cameră RGB /Bayer M3C în aceeași sesiune.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** funcționează în continuare — radianța și reflectanța sunt hărți radiometrice *pe bandă* și sunt perfect definite pentru o singură bandă. Cadrele monocrome conțin o matrice **de identitate** a răspunsului senzorului (fără amestec 3×3), astfel încât planul trece prin calculele de calibrare fără modificări.
- **O singură cameră monocromă nu poate produce un indice de vegetație.**NDVI / NDRE /etc. necesită cel puțin două benzi (de ex. Red + NIR). Pentru a obține un indice de la un sistem mono, orientați**mai multe** camere M3M pe lungimi de undă diferite, aliniați-le într-un singur stack multibandă și calculați indicele *acestuia*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` simbolurile trebuie să corespundă **exact** cu numele canalelor din presetare (se ține cont de majuscule și minuscule; cele de la NDVI sunt cu litere mici `red`,`nir` — a se vedea `--list-presets`), iar numele de bandă indică o bandă din stiva aliniată (modul offline acceptă și indici de bandă începând de la 0, de exemplu `--channel red=0 --channel nir=1`).

Elementul de discriminare în întregul teanc este tokenul `M3M` din șirul modelului (acesta nu apare niciodată într-un șir `M3C`), afișat în GUI/SDKul ca `is_mono`.

---

## Configurarea și reglarea plăcii de rețea a gazdei (matrice LATTICE)

Camerele LATTICE transmit GVSP prin adaptorul Ethernet al gazdei, astfel încât, pentru matricele cu mai multe camere, **driverul**adaptorului și**dimensiunea inelului de recepție** sunt la fel de importante ca și rata de transfer. Setările incorecte apar sub forma unei porți `FRAMES WILL DROP` / `Reduce ROI to enable` în panoul Setări matrice (și în `lattice network-analysis` / `analyze_array_network()` din „SDK”), chiar și atunci când camerele în sine funcționează corect.

### Adaptoare USB 10GbE — Realtek RTL8157 („Controler din familia Realtek USB 10GbE”)

| Element | Valoare necesară | De ce este important |
| --- | --- | --- |
| **Versiunea driverului**|**≥ v10.67 (ianuarie 2026)**, INF `rtump64x64sta.inf` | Driverul vechi**2016**(v10.65, `rtump64x64.inf`) gestionează incorect oprirea și verificările de erori cu**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**la oprire/repornire/modul de repaus. Tranziția se blochează (timp de așteptare de ~5 minute), utilizatorul oprește forțat sistemul, iar închiderile necorespunzătoare repetate**corup depozitul WMI**(PowerShell/instrumentele încep să dea erori cu `Invalid class`) și**blochează stiva USB** la următoarea pornire (placa de rețea nu se activează; unitățile USB nu mai sunt enumerate). Efectuați o actualizare de pe realtek.com (sau de la furnizorul dongle-ului) înainte de a vă baza pe reporniri corecte. |
| **Bufferele de recepție**— cuvânt-cheie `ReceiveBufferLen` |**256**(maximul driverului) | Inelul RX al plăcii de rețea. Valoarea implicită a driverului,**32**lasă doar ~0,26 MB de inel utilizabil — mult prea mic pentru o rafală multi-cam — așa că panoul matricei raportează `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` și blochează conexiunile. La**256**, inelul este mare (**~13,5 MB măsurat pe gazda de 10GbE din laborator**), oferind canalului de recepție (RX pipeline) o marjă reală pentru rafale GVSP cu mai multe camere. (Dacă o anumită configurație se *conectează* efectiv este decis de două verificări — verificarea de admitere **sensibilă la epuizare**și verificarea**supra-abonării agregate** — nu o comparație brută între rafală și inel; vezi [Modelul fps și rafale pentru matrice](#array-fps--burst-model).) |
| **URB-uri de recepție**— cuvânt cheie `PendingReceives` |**64** (max) | Blocuri de cereri USB în tranzit; se măresc odată cu tampoanele de recepție pentru absorbția burst-urilor. |
| **Cadru Jumbo** — cuvânt cheie `*JumboPacket` | **9014** | Necesar pentru pachetele GVSP de 9000 de octeți (de 6 ori mai puține pachete/cadru decât 1500). |

> ⚠️ **O actualizare a driverului plăcii de rețea RESETEAZĂ aceste proprietăți avansate la valorile implicite.**După actualizarea sau înlocuirea driverului adaptorului,**reaplicați** `ReceiveBufferLen=256` și `PendingReceives=64`, altfel panoul matricei se va bloca din nou, chiar dacă „nu s-a schimbat nimic la nivel de hardware”. Aceasta este cauza principală pentru care o platformă care funcționa anterior refuză brusc să se conecteze.

Aplicați dintr-un **PowerShell cu drepturi ridicate** (înlocuiți cu numele adaptorului dvs., de ex. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` se referă la adaptoarele USB 10GbE.** Acum detectează tipul adaptorului și setează cuvântul-cheie corect pentru inelul de recepție: `*ReceiveBuffers`→2048 pentru plăci de rețea PCIe (Intel I219 etc.) sau `ReceiveBufferLen`→256 + `PendingReceives`→64 pentru controlerul **USB** 10GbE (care nu expune `*ReceiveBuffers`). Valorile țintă sunt limitate la maximul raportat de fiecare driver (`NumericParameterMaxValue`), astfel încât nu se scrie niciodată o valoare în afara intervalului. Rulați-l dintr-un terminal **cu drepturi ridicate**; ca orice optimizare bazată pe registru, modificarea intră în vigoare după o repornire a adaptorului sau a sistemului. Comenzile manuale `Set-NetAdapterAdvancedProperty` de mai sus rămân o alternativă bună — ele se aplică în timp real (reconfigurează adaptorul) fără a fi necesară o repornire.

### Noțiuni de bază despre rețea (toate legăturile LATTICE)

- **Adresare:** adresă locală de legătură (GigE Vision LLA). Gazda utilizează o adresă statică; camerele și DAQ-E își alocă automat adrese în același interval. Nu este necesar DHCP/gateway.
- **Dimensiunea pachetului:**preferați jumbo (9000), dar lăsați sonda automată să o determine — aceasta reevaluează la fiecare conectare și ignoră deja limita de 1500- prin intermediul unei sonde GVSP, astfel încât ajunge la jumbo oriunde cablul îl suportă cu adevărat. Fixați cu `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` numai atunci când știți mai bine decât sonda și preferați setarea pe comandă în locul celei permanente: o fixare ocolește sonda, așa că, dacă traseul nu poate transporta efectiv 9000,**fiecare** captură va expira cu `SC_ERR_TIMEOUT -1011` (vezi [Variabile de mediu](#environment-variables)).
- **Inelul RX se scalează cu `ReceiveBufferLen`:**la valoarea implicită `32`, inelul utilizabil este de ~0,26 MB (prea mic pentru orice rafală multi-cameră); la valoarea maximă `256` este mare (~13,5 MB măsurat pe gazda de laborator 10GbE), oferind o marjă reală de manevră. Dacă o configurație se conectează sau nu este decis apoi de verificarea admiterii bazată pe consum**și** de verificarea suprasubscrierei agregate de mai jos — nu printr-o simplă comparație brută între rafală și inel.

### Modelul de fps și rafale al matricei

Cum se interpretează panoul „Array Settings” (și `lattice analyze-array` / `analyze_array_network` din „SDK”):

- **Burst-ul este însumat pe cameră, la formatul real de pixeli al fiecărei camere.**Camerele mono**M3M**transmit**Mono12 (2 biți/px)**; camerele Bayer**M3C**transmit 8 sau 12-biți (TRI032S transmite în mod silențios BayerRG12 chiar și atunci când este solicitat BayerRG8). Așadar, un cadru la rezoluție maximă cu 4 camere are**~12,6 MB dacă toate sunt de 8 biți, dar ~25 MB cu trei camere mono de 12 biți**. Proiecția determină formatul fiecărei camere pe baza modelului său (cache de identitate), astfel încât seria de cadre corespunde cu ceea ce transmite efectiv cablul — nu o presupunere universală de tip BayerRG8.
- **Un adaptor USB-Ethernet are o limită maximă de 200 MB/s, indiferent de specificațiile tehnice.** Tabelul de eficiență care transformă rata de legătură într-o valoare susținută este derivat din PCIe; o placă de rețea USB anunță rata de legătură *Ethernet*, dar este limitată de magistrala USB și de driverul acesteia. Un dongle USB 10GbE a înregistrat ~1063 MB/s „susținut” — o valoare care nu a fost niciodată verificată — iar ritmul de transfer rezultat a corupt 6–18 % din cadre, în timp ce raporta în continuare o rată de cadre pe secundă (fps) țintă corespunzătoare. Plăcile de rețea conectate prin USB sunt acum limitate la **200 MB/s** ca valoare absolută (limita este reprezentată de magistrală, deci nu se adaptează la specificațiile tehnice; un adaptor USB 1 GbE atinge ~80 MB/s și nu este afectat). `wire_ceiling_source` din înregistrarea de capacități menționează acest lucru în mod explicit, iar `nic_is_usb` semnalează acest lucru. Se poate suprascrie în orice caz cu `--wire-ceiling-mbps`.
- **Admitanța ține cont de consumul de energie, nu de raportul între rafală completă și inel.** Un burst simultan trebuie să se încadreze doar în *backlog-ul tranzitoriu* = `max(0, Σ per-cam arrival − host drain) × emit_window`, nu în întregul burst. Pe o structură cu gazdă rapidă / camere lente (o gazdă **PCIe**de 10G + 4 camere de 1 GbE: sosire ≈ 320 MB/s, evacuare ≈ 1063 MB/s), gazda evacuează mai repede decât se umplu camerele, stocul de tranziții ≈ 0, astfel încât simularea la rezoluție maximă**admite**, chiar dacă rafala de 25 MB depășește inelul de 13,5 MB. Dacă se conectează aceleași patru camere la un adaptor**USB**de 10 GbE, viteza de evacuare este de 200 MB/s, nu 1063 — sosirea depășește viteza de descărcare, iar pierderea se manifestă sub formă de cadre corupte, mai degrabă decât printr-o rată de cadre mai mică. Pe o gazdă de 1 GbE, pragul DLThr de 31,25 MB/s al camerelor face ca sosirea să depășească viteza de descărcare → aceasta**blochează** (pentru *această* clasă de blocare, reduceți ROI sau utilizați binning ≥ 2). Admiterea este una dintre **cele două** porți de conectare — cealaltă este verificarea agregată a suprasubscrierii de mai jos.
- **FPS-ul proiectat reprezintă un plafon conservator pentru recuperarea serială.**Bucla de preluare a gazdei extrage în prezent bufferul fiecărei camere**serial**(~o fereastră de emisie per cameră fiecare), astfel încât ciclul este limitat de `max(readout+emit, N × emit)`, emisia per cameră fiind limitată la**legătura de acces**a camerei (1 GbE ≈ 80 MB/s), nu de legătura ascendentă a gazdei. Pentru o matrice de 4 camere la rezoluție maximă de 12 biți, aceasta înseamnă**~2,8 fps**, ceea ce corespunde valorilor măsurate de ~2,7–3,0. fps este în mod deliberat**independent de expunere**, astfel încât, în scenele slab iluminate, valoarea reală poate scădea ușor sub limita maximă pe măsură ce expunerea se prelungește. Recuperarea serială este adevăratul limitator al fps-ului; paralelizarea acesteia ar ridica limita maximă spre rata de emisie unică.
- **Supra-abonarea agregată este un factor care blochează conexiunea.**Alocarea lățimii de bandă per cameră are o limită minimă de**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), astfel încât, odată ce limita minimă este atinsă, cererea agregată (`per_cam × N`) poate depăși**limita maximă a canalului de transmisie fără coliziuni**(`sustained × sim_emit_factor`). Limitele practice la rezoluție maximă pe 1 GbE:**6 camere la 1500 MTU, 9 cu jumbo**. Această limită maximă este determinată exclusiv de proprietățile cablului și de pragul minim — este**independentă de dimensiunea cadrului**, așa că**gruparea în binuri și o zonă de interes (ROI) mai mică NU ajută** (acestea reduc numărul de octeți pe *cadru*, nu numărul de octeți pe *secundă* stabilit de GevSCPD); singurele soluții sunt un număr mai mic de camere, cadre jumbo de la un capăt la altul, sau o placă de rețea (NIC) mai rapidă. Simptomul ar fi pierderea de pachete GVSP, nu o reducere graduală a fps-urilor, așa că `analyze-array` resetează la zero valorile fps-urilor realizabile și afișează `**OVER-SUBSCRIBED**`, iar `array-connect` cu o rezoluție fixă **refuză să se conecteze** (în caz contrar, „walk-down” grupează cadrele în categorii mai mici, ceea ce nu elimină nici această clasă de blocuri). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` transformă refuzul într-un avertisment puternic pentru lucrările de testare — vezi [Variabilele de mediu](#environment-variables).

### Starea matricei — care subsistem pierde cadre

Un `GET /api/camera/array/<array_id>/capability` al unei matrice conectate conține un
bloc `health` activ, reevaluat într-o fereastră **10 secunde**. Acesta împarte pierderea de cadre
în cele două cauze care necesită remedieri opuse, în loc să raporteze o singură rată „incompletă”
care nu specifică niciuna dintre ele:

| Câmp | Ce înseamnă | Care subsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pe serial) | Cadrul **a sosit și era defect din punct de vedere structural**— pierdere de pachete GVSP. |**Rețea**: lățime de bandă, ritm de transmisie, inelul de recepție al plăcii de rețea, MTU |
| `never_arrived_rate_pct` (pe serial) | Cadrul **nu a sosit deloc**— camera nu s-a declanșat sau nu a transmis nimic. |**Declanșare / sincronizare**: cablu M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | **Cea mai slabă rată a camerei**pentru fiecare. | — |
| `per_cam_rate_pct` | Rata combinată de cadre incomplete pe cameră (ambele cauze împreună). | — |
| `stable_for_seconds` | Cât timp a rămas fiecare cameră sub 0,01 %. | — |

Peste 5 %, backend-ul înregistrează o linie `[array-health <id>] WARN` care menționează diviziunea — la
prima încălcare, la o schimbare a benzii de gravitate, o dată pe minut cât timp aceasta persistă și o dată când
se remediază. Jumătatea coruptă afișează `[gvsp-corrupt <SN>]` la prima depășire pentru fiecare cameră și
motiv, apoi un raport cumulativ la fiecare 60 de secunde. Fiecare evaluare este totuși înregistrată în fișierul jurnal al backend-ului;
contoarele se actualizează la fiecare buffer, indiferent de ceea ce este afișat.

Aceeași înregistrare raportează numărul de la care depinde întreaga alocare:

| Câmp | Ce înseamnă |
| --- | --- |
| `wire_ceiling_mbps` | Bugetul de bandă susținut al gazdei, în vigoare, MB/s. |
| `wire_ceiling_source` | De unde provine acea valoare, în cuvinte — de ex. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` sau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` atunci când `--wire-ceiling-mbps` (sau câmpul **Wire Budget** din interfața grafică) îl setează. |
| `nic_is_usb` | `true` pentru un adaptor USB Ethernet — vezi limita de 200 MB/s de mai sus. |

**Interpretare:** o valoare diferită de zero pentru `gvsp_corrupt_rate_pct`, cu `never_arrived_rate_pct` la 0,
înseamnă că declanșarea și sincronizarea cablului sunt perfecte, iar 100 % din pierderi se datorează traseului de rețea
— reduceți valoarea `--wire-ceiling-mbps` și reconectați. Modelul invers indică mai degrabă
cablul de sincronizare sau linia de declanșare.

> **`--target-fps` nu este factorul determinant pentru cadrele corupte.** Ritmul GevSCPD este stabilit
> o singură dată la conectare, astfel încât reducerea ratei de declanșare modifică ciclul de lucru și nu
> rata de transmisie în rafale simultane. O reducere măsurată a cererii de 5× nu a produs nicio îmbunătățire;
> scăderea limitei maxime a cablului de la 240 la 200 MB/s a dus aceeași configurație de la 10,4 %
> de cadre corupte la 0,00 %.

> **Reducerea automată în timpul transferului nu este disponibilă pe firmware-ul TRI032S.** O matrice în funcțiune
> nu poate remedia singură această problemă; deconectați și reconectați, astfel încât selectorul de timp de conectare să poată
> replanifica cu noul prag maxim.

### Simptom → remediere

| Simptom (Setări matrice / conectare / `analyze_array_network`) | Cauză | Remediere |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` resetat la 32 (de obicei după o actualizare a driverului) | Setați `ReceiveBufferLen`→256, `PendingReceives`→64; redeschideți panoul (reporniti backend-ul dacă a stocat în cache vechea dimensiune a inelului) |
| Repornirea/oprirea se blochează; ulterior apar erori WMI `Invalid class`, placa de rețea nu se activează, unitățile USB lipsesc | Vechiul driver Realtek USB 10GbE din 2016 → BSOD `0x9F` → opriri forțate | Actualizați driverul adaptorului la versiunea ≥ v10.67 (2026), apoi reaplicați setările de inel de recepție de mai sus |
| Conectarea reușește, dar returnează o rezoluție sub cea nativă | Smart-prep a redus automat cadrul pentru a se potrivi cu cablul | Actualizați legătura / acceptați reducerea / `--force-tier slip-emit-and-capture` |
| Matricea raportează o frecvență țintă a cadrelor (fps) corectă, dar livrează doar o fracțiune din aceasta; `health.gvsp_corrupt_rate_pct` diferit de zero, `never_arrived_rate_pct` 0 | Bugetul de bandă dedus al gazdei supraestimează ceea ce susține de fapt (tipic pentru un adaptor Ethernet USB, o bandă PCIe subțire sau o structură partajată) | Reconectați-vă cu o valoare mai mică pentru `--wire-ceiling-mbps` și-verificați blocul de stare. **Nu** `--target-fps` — Ritmul GevSCPD este fixat la conectare |
| Camere lipsă din grupurile publicate; `health.never_arrived_rate_pct` diferit de zero, `gvsp_corrupt_rate_pct` 0 | Cale de declanșare / sincronizare — camerele nu se declanșează, nu este o problemă de rețea | Verificați cablul de sincronizare M8 și `--line`; confirmați că fiecare element este activat (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` depășit în `analyze-array`, sau refuz de conectare cu rezoluție fixată (`array over-subscribes the wire`) | Cererea agregată pe cameră (8 MB/s minim × N camere) depășește limita maximă a cablului fără coliziuni — 6 camere la rezoluție maximă pe 1 GbE la1500 MTU, 9 cu cadre jumbo | Mai puține camere, cadre jumbo de la un capăt la altul sau o placă de rețea mai rapidă. **ROI/binning NU vor ajuta** (limita maximă este independentă de dimensiunea cadrului). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` se aplică prioritar pe banc (acceptă pierderea de pachete) |

---

## `chloros-cli daq`

Comenzi pentru senzorul spectral. Două clase:
- **`pool-*`**— clienți „HTTP” ușori care controlează senzorul prin intermediul pool-ului persistent al backend-ului.**Aceasta este calea suportată, și singura prezentă în pachetul livrat CLI.** Backend-ul deține controlul asupra transportului, astfel încât interfața grafică, scripturile CLI și SDK împart toate un singur identificator activ, în loc să se lupte pentru portul serial.
- **Toate celelalte**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — acces direct la hardware, documentat mai jos pentru exhaustivitate. Acestea necesită pachetul `daq` Python, care**nu este inclus în niciun artefact livrat**: pachetul compilat CLI îl exclude (`scripts/Build-CLI.ps1` setează `--nofollow-import-to=daq`, iar transporturile `pyserial` / `bleak` / `zeroconf` împreună cu acesta), iar pachetul PyPI SDK nu îl conține nici el. Acestea rulează doar dintr-un checkout al sursei, așa că tratați-le ca pe o cale de dezvoltare internă MAPIR, mai degrabă decât ca pe ceva la care să apelați.
- **`discover` / `list`** se situează între cele două: sunt comenzi directe către hardware provenite dintr-o copie a codului sursă, dar într-o versiune livrată se recurg la `pool-discover`, iar scanarea este efectuată de backend. Așadar, scanarea funcționează peste tot — ceea ce este important, deoarece este singura modalitate de a afla adresa MAC BLE a unui DAQ-M.

> **`chloros-cli daq --help`** (și `-h` / `help`) listează subcomenzile `pool-*` — ajutorul este redirecționat în mod deliberat către clientul pool, astfel încât să reflecte comenzile care se execută efectiv. Dacă invocați o subcomandă directă la nivel de hardware pe o versiune livrată, aceasta se închide cu o eroare explicită care menționează pachetul lipsă și te redirecționează către `pool-*`; nimic nu eșuează în tăcere. (`discover` / `list` reprezintă excepția — acestea redirecționează către `pool-discover` și funcționează fără probleme.)
>
> **Tot ce are nevoie un client este accesibil prin `pool-*`** — conectare, transmisie în flux, înregistrare de fișiere `.daq` calibrate și schimbarea profilurilor de condensatoare. DAQ-ul poate fi, de asemenea, controlat de lPython, cu `chloros_sdk.connect_daq_sensor()`, care utilizează aceeași cale comună.

### Fluxul de lucru pentru prima conectare a senzorului DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referință `pool-*`

| Subcomandă | Scop |
| --- | --- |
| `daq pool-connect` (smart-detect) | Deschide un senzor din grupul din fundal. |
| `daq pool-connect --port PORT` | DAQ-U pe un port serial specific. |
| `daq pool-connect --ble` | DAQ-M prin BLE, scanare automată a adresei MAC. |
| `daq pool-connect --mac MAC` | DAQ-M prin BLE la o adresă MAC cunoscută (implică `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E prin Ethernet la o gazdă cunoscută. |
| `daq pool-connect --eth` | DAQ-E prin Ethernet, gazdă descoperită automat (mDNS + fallback ARP; funcționează cu un cache ARP gol pe Windows și Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Reglare fereastră de integrare / stare AE. |
| `daq pool-connect --no-stream` | Conectare, dar fără a începe încă transmisia (se reia cu `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil de corecție Cap. Valoarea implicită la nivel de backend este `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scanează fiecare transport pentru senzori la care te-ai putea conecta, fără a vă conecta. **Astfel găsiți adresa MAC BLE a unui DAQ-M.** `daq discover` / `daq list` sunt redirecționate automat aici în versiunile livrate. Senzorii deja deschisi în pool nu sunt listați — un DAQ-M conectat încetează să mai emită anunțuri — așa că folosiți `pool-list` pentru aceștia. |
| `daq pool-list` | Afișează fiecare senzor din pool-ul backend. |
| `daq pool-disconnect --sensor-id ID [--all]` | Eliberează. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Cele mai recente cadre din spectrul N. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reluare / pauză streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Pornește / oprește o înregistrare .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Schimbă profilul de corecție a capacității în timpul rulării. |

### Subcomenzi directe pentru hardware (disponibile doar în codul sursă — nu în versiunile livrate)

> Enumerate pentru exhaustivitate. Acestea necesită pachetul `daq` Python plus `pyserial` / `bleak` / `zeroconf`, dintre care niciuna nu este inclusă în versiunile compilate CLI sau în PyPI SDK — acestea rulează numai dintr-o descărcare a sursei de la MAPIR. **Dacă utilizați o versiune lansată Chloros, folosiți în schimb comenzile `pool-*` de mai sus**; acestea acoperă conectarea, transmisia, înregistrarea și selectarea capturilor.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Deschideți, conectați-vă la și controlați un proiect Chloros salvat (un folder conținând `cameras.json` + `sensors.json` + `project.json`). Totul se tranzitează prin backend, astfel încât interfața grafică și CLI să genereze o stare identică a hardware-ului.

### Referință subcomenzi

| Subcomandă | Scop |
| --- | --- |
| `project open PATH` | Afișează lista dispozitivelor din proiect (camere, matrice, senzori). |
| `project devices PATH [--reconnect]` | Afișează lista sau reexecută procesul de detectare. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Conectează fiecare cameră / matrice / senzor salvat. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Captură unică de la o cameră sau matrice specificată. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Serie de N cadre de la o cameră sau matrice specificată (`-n/--count` implicit 5; `-i/--interval` secunde între cadre, valoare implicită 0). Seriile de cadre din matrice elimină duplicatele din grupurile sincronizate repetate (mecanism de detectare a datelor învechite), astfel încât o matrice cu ciclu parțial nu poate returna N copii ale unui singur cadru; afișează rezultatele pentru fiecare iterație. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Transmitere în flux către disc printr-o sarcină de fundal. `--poll-interval` = secunde între interogările `/stats` (implicit 2,0). |
| `project sensor read PATH NAME [--json]` | Cel mai recent cadru de spectru. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Înregistrare .daq. |
| `project run PATH RECIPE.yaml` | Executare a unei rețete de captură YAML/JSON. `--dry-run` validează fără a rula. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcul alinierea pentru un șir — consultați [tabelul de opțiuni de mai jos](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Afișează profilul de aliniere curent. |
| `project align clear PATH NAME` | Șterge profilul din cache. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Modifică ușor transformarea unui slave. |
| `project align export PATH NAME --to FILE` | Salvează profilul în fișierul „JSON”. |
| `project align import PATH NAME --from FILE [--no-validate]` | Încarcă un profil salvat. |

#### Opțiuni `project align calibrate`

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda de aliniere. **Aceste denumiri diferă de `lattice align-calibrate`**, care acceptă formele scurte `orb` / `akaze` / `phase`; cele două comenzi nu sunt interschimbabile pentru acest indicator. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformă modelul pentru a se potrivi. |
| `--frames N` | `1` | Instantanee ale cadrelor sincronizate la media. |
| `--reference SN` | camera principală | Numărul de serie al camerei de referință; toate celelalte elemente sunt deformate pentru a se potrivi cu aceasta. |
| `--max-features N` | `5000` | Limită pentru numărul de caracteristici ORB. |
| `--ratio-threshold F` | `0.75` | Testul raportului Lowe. |
| `--ransac-threshold-px F` | `3.0` | Pragul de puncte interioare RANSAC. |
| `--min-matches N` | `15` | **Prag de calitate** — respinge soluția dacă numărul de potriviri cu punctele din interior este mai mic decât această valoare. |
| `--max-reproj-err-px F` | `4.0` | **Criteriu de calitate** — respinge soluția dacă depășește această eroare RMS de reproiectare. |
| `--checkerboard RxC` | — | Geometria plăcii pentru `--method checkerboard`, de exemplu `9x6`. |
| `--name PROFILE` | gol | Numele profilului încorporat în fișierul „JSON” salvat. **Nu este numele matricei** — acesta este `NAME` pozițional. |

Cele două praguri de calitate sunt motivul pentru care o calibrare poate reuși să rezolve problema și totuși
să refuze salvarea: un profil care nu îndeplinește unul dintre ele ar înregistra în mod eronat, fără avertisment, fiecare
captură ulterioară, așa că este refuzat în loc să fie păstrat.

### Exemple

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL pentru rețete

`project run RECIPE.yaml` acceptă un fișier YAML sau JSON care descrie o secvență de acțiuni:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Acțiuni acceptate: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Acțiunea `burst` necesită `name` (obligatoriu), `count` (implicit 5), `interval` (secunde, implicit 0), `output`, `format` și `settings` (la fel ca pentrusetările fiecărei camere ca `apply`); rafalele de tip matrice utilizează același watchdog de grup proaspăt sincronizat ca `project burst`.

Executați:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Variabile de mediu

| Variabilă | Efect |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Suprascrie backend-ul URL (implicit `http://127.0.0.1:5000`) — **respectată doar de `lattice`, `project`, și `daq pool-*`.** Comenzile de bază (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) conectează `http://127.0.0.1:<port>` și ignoră această variabilă (literalul IPv4 ocolește Windows `localhost`→`::1` ~2 s-penalizare pe cerere), astfel încât vizează întotdeauna mașina locală. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` reduce nivelul refuzului de conectare din cauza suprasubscrierii matricei (cererea agregată pe cameră &gt; plafonul de bandă fără coliziuni cu `pin_resolution`) la un „avertisment puternic și continuă”, acceptând pierderea de pachete GVSP. Numai pentru utilizare în mediul de testare — vezi [Modelul de fps și rafale al matricei](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Setat de către „CLI” însuși; indică backend-ului să activeze procesarea paralelă. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` omite sonda de rezervă GVSP (doar rezultate ICMP). **Aceasta dezactivează pachetele jumbo, nu doar reduce volumul jurnalului** — camera răspunde la ping-urile DF doar până la 1500 pe fiecare traseu, așa că această sondă este singura care poate detecta pachetele jumbo. Economisește ~1 s pe cameră per conexiune; costă ~1,45× lățimea maximă a cablului dacă rețeaua *ar fi putut* să transporte pachete jumbo. SDK te avertizează când o setezi. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixează dimensiunea pachetului GVSP la N octeți; omite complet sondarea. Preferă setarea pe comandă (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) în locul setării permanente: o dimensiune fixată nu se mai adaptează la rețeaua din fața ei, iar fixarea valorii 9000 pe o cale care nu poate transporta pachete jumbo face ca **toate** capturile să depășească timpul de așteptare cu `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Suprascrie directorul de extragere onefile al Nuitka. CLI utilizează automat `/mnt/ssd/tmp` dacă este prezent. |

---

## Coduri de ieșire

| Cod | Semnificație |
| --- | --- |
| `0` | Succes. |
| `1` | Eșec generic (majoritatea erorilor de subcomandă). |
| `2` | Eroare de argument. |
| `130` | Întrerupt cu Ctrl+C. |

---

## Indicații pentru depanare

- **„Este necesară autentificarea”** → Rulați o dată `chloros-cli login EMAIL PASSWORD` pe acest computer.
- **„backend inaccesibil”** → Porniți aplicația desktop Chloros, sau rulați direct fișierul binar al backend-ului (`chloros-backend`), sau verificați `CHLOROS_BACKEND_URL` dacă este vorba de o conexiune la distanță.
- **Comenzile `lattice` eșuează cu mesajul „Drivere cameră LATTICE neidentificate”** → Runtime-ul Arena SDK nu este instalat; CLI vine cu `win32api` inclus pe Windows, dar runtime-ul C face parte din programul de instalare GUI.
- **„Array connect” / „Array Settings” afișează „FRAMES WILL DROP” sau „Reduceți ROI pentru a activa”** → Inelul de recepție al plăcii de rețea a gazdei este prea mic (de obicei se resetează la 32 după o actualizare a driverului plăcii de rețea). Consultați [Configurarea și reglarea plăcii de rețea a gazdei](#host-nic-setup--tuning-lattice-arrays) — setați `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Mașina se blochează la repornire/oprire, apoi WMI `Invalid class` / placa de rețea nu se activează / lipsesc unitățile USB** → Driverul învechit al adaptorului USB 10GbE provoacă `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Actualizați driverul adaptorului — consultați [Configurarea și optimizarea plăcii de rețea a gazdei](#host-nic-setup--tuning-lattice-arrays).
- **Avertisment privind swap-ul Jetson** → Adăugați un fișier; comanda „CLI” afișează exact comenzile `fallocate` / `swapon`.
- **Comenzi directe DAQ lipsă** → Așteptat: pachetul livrat `chloros-cli` exclude în mod deliberat pachetul `daq`, astfel încât este prezent doar `pool-*` (nici PyPI SDK nu îl include). Utilizați `pool-*`, care controlează același senzor prin intermediul backend-ului, sau `chloros_sdk.connect_daq_sensor()` de pe Python.

---

## Vezi și

- [Python Referință SDK](sdk-reference.md) — echivalentul programatic al fiecărei comenzi CLI.
- [Ghidul senzorilor DAQ](../daq/README.md) — cablare și calibrare specifice senzorilor.
- Documentație online: `https://mapir.gitbook.io/chloros/cli`
