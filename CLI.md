# CLI : Linia de comandă

> **Referință completă:**[Referința CLI](reference/cli-reference.md) documentează**fiecare opțiune a fiecărei subcomenzi** și este optimizată pentru asistenții AI — lipiți adresa URL în asistentul dvs. și solicitați o comandă funcțională: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Sfat pentru instrumentele AI:** orice pagină din acest manual este disponibilă în format Markdown brut prin adăugarea lui `.md` la URL-ul acesteia (de ex. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), iar `https://mapir.gitbook.io/chloros/llms.txt` indexează întregul manual pentru utilizarea de către LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Ce esteCLI


`chloros-cli` este interfața de linie de comandă pentru același motor de procesare pe care îl utilizează aplicația desktopChloros
. Este un client „HTTP
” ușor, care utilizează backend-ulChloros
(un server local pe `127.0.0.1:5000`) — majoritatea comenzilor pornesc automat backend-ul, astfel încât un singur apel către `chloros-cli process …` este tot ce are nevoie un script.

Funcționează pe **Windows
10/11 (x64)**și**Linux
(x86_64 și NVIDIA Jetson arm64 pe JetPack 6)**, în orice terminal, fără a fi necesară o interfață grafică. Verificați instalarea cu:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Prezentare generală a familiilor de comenzi:

* **Procesare și cont** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 de limbi — consultați [Limbi acceptate](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (numai pentruLinux
/Jetson)
* **Hardware activ** — `lattice` (control cameră LATTICE, peste 45 de subcomenzi), `daq pool-*` (senzori de lumină DAQ), `time-sync` (PTP)
* **Automatizare** — `project` (executare fără interfață grafică a unui proiect salvat înChloros
, inclusiv rețete de captură YAML)

Opțiuni globale utile: `--port N` (port backend, implicit `5000`), `-v/--verbose`, `--restart` (repornire forțată a backend-ului), `--backend-exe PATH`. Consultați [ReferințaCLI
](reference/cli-reference.md) pentru lista completă.

***

## Instalare

CLI
**este inclus în programul de instalareChloros** pe toate platformele — nu există o descărcare separatăCLI
. Descărcați programul de instalare de pe pagina [Descărcare](download.md).

###Windows


Programul de instalare plaseazăCLI
în:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

și adaugă acel folder la sistemul dumneavoastră `PATH` — **deschideți un terminal nou**după instalare, astfel încât `PATH` actualizat să fie recunoscut. Programul de instalare plasează, de asemenea, scripturi de lansare (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) în directorul rădăcină al instalării, precum și o**scurtăturăChloros
CLI
** comandă rapidă în meniul Start, fiecare dintre acestea deschizând un terminal cu `chloros-cli` gata de utilizare.

###Linux


Instalați `.deb` pentru arhitectura dvs.:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Aceasta instalează `chloros-cli` până la `/usr/bin/chloros-cli` (deja la versiunea `PATH`) și backend-ul la versiunea `/usr/lib/chloros/chloros-backend`, împreună cu runtime-ul ArenaSDK
necesar pentru camerele LATTICE. Consultați [InstalareaLinux
](linux/linux-installation.md) pentru detalii.

### Verificare

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Autentificare și licențiere

CLI
(șiPython
SDK
) accesul necesită un **plan plătitChloros
+**— orice nivel plătit îl include; nivelul gratuit nu îl include. Limita minimă este aplicată**pe partea de server** de către backend, nu de către binarulCLI
: un apel fără autentificare este respins cu codul `401 AUTH_REQUIRED`, iar o solicitare efectuată cu contul conectat în planul gratuit cu codul `403 PLAN_UPGRADE_REQUIRED`, indiferent dacă provine de la `chloros-cli`, de laSDK
sau de la un clientHTTP
creat manual. Efectuați actualizarea la [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Conectați-vă **o singură dată pe fiecare mașină**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Parole cu caractere speciale**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` este distorsionată de shell (CLI
-ul detectează acest lucru la un 401 și reîncearcă automat, dar ghilimelele simple evită complet problema).
{% endhint %}

Sesiunea este stocată în cache la `~/.chloros/user_session.json` și continuă să funcționeze offline pe durata perioadei de grație a abonamentului (30 de zile pentru abonamentele lunare, până la expirare pentru cele anuale). `chloros-cli status` funcționează chiar și fără un plan plătit, astfel încât motivul refuzului este întotdeauna vizibil.

{% hint style="danger" %}
**Programați sarcini fără interfață grafică? Autentificați-vă mai întâi.**O comandă de lansare a backend-ului (`process`, `status`, `export-status`, …) executată**fără sesiune stocată în cache**nu eșuează rapid — trece într-un prompt interactiv `Email:` / `Password:` pe stdin. Prin urmare, o sarcină cron nesupravegheată sau un pas CI va**rămâne blocat în așteptarea unei intrări**. Rulați `chloros-cli login EMAIL 'PASSWORD'` o dată pe mașină înainte de a programa orice.
{% endhint %}

***

## Prima dvs. execuție de procesare

Direcționați `process` către un folder cu capturi — acesta detectează automatSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` sau o combinație a acestora:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Progresul este afișat în timp real pentru fiecare fir al pipeline-ului (Detectare, Analiză, Procesare, Export), iar o execuție reușită se încheie cu raportarea numărului de produse imagistice generate (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Unde sunt stocate rezultatele

`process` scrie într-un **dosar de proiect**, nu în dosarul dvs. de intrare:

* Fără `-o`: proiectul este creat în folderul dvs. de proiect implicit (partajat cu interfața grafică; gestionați-l cu `get-project-folder` / `set-project-folder`, opțiune de rezervă `~/Chloros Projects`), denumit după `-n/--project-name` sau o marcă temporală (`YYYYMMDD_HHMMSS`) atunci când este omis.
* Cu `-o PATH`: acel folder **este** folderul proiectului. Dacă acesta conține deja un `project.json`, se creează un fișier înrudit cu sufixul `_1`/`_2`… în loc să se suprascrie.

În cadrul proiectului, produsele sunt grupate **după cameră, apoi după formatul fișierului**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folderul camerei este `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE (care corespunde cu EXIF-ul capturii `Model`) și `<model>_<filter>` (de exemplu, `Survey3N_RGN`) pentruSurvey3
. Dosarul de format urmează după `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` sau `tiff32` pentru `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Fiecare produs exportat păstrează numele fișierului SURSĂ.**Un export Radiance al fișierului `capture_..._raw.tif` se numește în continuare `capture_..._raw.tif` — doar că se află în `tiff32/Radiance_Images/`.**Folderul identifică produsul, nu numele fișierului**, așa că folosiți un filtru glob pentru director, nu pentru sufixul `*radiance*`.
{% endhint %}

### Opțiunile pe care le veți folosi efectiv

| Opțiune | Implicit | Ce face |
| --- | --- | --- |
| `-o, --output PATH` | folderul implicit al proiectului | Locația folderului proiectului (vezi mai sus). |
| `-n, --project-name NAME` | marcaj temporal | Numele proiectului. |
| `--format FMT` | `TIFF (16-bit)` | Una dintre următoarele: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | niciunul | Indici de vegetație de exportat (a se vedea [Indici de vegetație](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neuronal, mai lent, calitate maximă (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | activat | Corecția efectului de vignetă. |
| `--reflectance / --no-reflectance` | activat | Calibrarea reflectanței; pentru LATTICE, aceasta este și comutatorul pentru produsul de reflectanță. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forțează punctul de intrare în pipeline pentru fișierele TIFF LATTICE. |

Pentru toate celelalte opțiuni — reglarea detectării țintelor, PPK, pini de expunere, indicatori de aliniere a matricei — consultați [secțiunea `process` din ReferințaCLI
](reference/cli-reference.md).

***

## Alegerea elementelor de exportat (produse LATTICE)

Procesarea LATTICE se ramifică către **fiecare produs aplicabil într-o singură trecere**. Cele patru comutatoare specifice fiecărui produs sunt toate**activate implicit**; utilizați formularul `--no-` pentru a dezactiva unul dintre ele:

| Comutator | Produs |
| --- | --- |
| `--debayered` | Demosaic liniar → `Debayered_Images/` |
| `--preview` | Previzualizare afișaj (balans de alb + gamma; extindere în culori false pentru imagini multispectrale) → `Preview_Images/` |
| `--radiance` | radianță float32, W/m²/sr/nm → `Radiance_Images/` (întotdeauna `tiff32/`) |
| `--reflectance` | uint16 reflectanță, compatibilă cu Pix4D → `Reflectance_Calibrated_Images/` |

RGB
Camerele master emit întotdeauna doar date debayered + previzualizare — radianța/reflectanța pe bandă nu are relevanță pentru un senzor de bandă largă, astfel încât aceste comutatoare nu au niciun efect pentru acestea.Survey3
`.raw` ignoră comutatoarele și urmează calea standard de reflectanță/țintă.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (implicit `auto`) alege referința de reflectanță: `auto` creează o [țintă de calibrare](calibration-targets.md) în cadru care trece de controlul calității ca referință absolută și revine la raportul de divizare a luminii descendente a senzorului DAQ (ρ = π·L/E) atunci când nu este prezentă nicio țintă; `target` este strict (fără substituție DAQ); `daq` are prioritate asupra DAQ. Scanările țintelor măsurate pe unitate pot fi furnizate cu `--target-reflectance-dir`.

{% hint style="info" %}
**Citirea pixelilor de reflectanță:**valoarea DN care înseamnă ρ = 1,0 este**pe sursă** — Fișierele LATTICE încorporează `Chloros:PixelScale=32768` în XMP; fișiereleSurvey3
utilizează 65535 (și nu conțin etichete `Chloros:*`). Citiți eticheta și împărțiți la valoarea acesteia, în loc să presupuneți o constantă. Detaliile și singurul caz marginal deliberat fără scalare se găsesc în [CLI
Referință](reference/cli-reference.md).
{% endhint %}

**Procesarea începe întotdeauna de la `raw`.** Produsele derivate (exporturi debayered/radianță/reflectanță) nu sunt niciodată reintroduse în fluxul de procesare — reimportarea și procesarea lor ar duce la aplicarea dublă a calculelor de calibrare, așa căChloros
le omite și menționează acest lucru. `--input-level` este o soluție de rezervă intenționată pentru cazurile în care este cu adevărat necesar să forțați un punct de intrare.

***

## Când o execuție eșuează

Începând cu versiunea 1.2.0, `process` eșuează în mod evident, în loc să „reușească” fără a afișa nimic:

* O execuție care **a solicitat produse, dar nu a scris niciunul**— doar `project.json` și `calibration_data.json` — afișează `Processing finished but wrote no image products.` și**se închide cu un cod de ieșire diferit de zero**, astfel încât scripturile să o poată detecta. Cauzele obișnuite: folderul de intrare nu a fost recunoscut ca o captură (verificați structura și `--input-level`) sau fiecare produs solicitat era inaplicabil pentru acele camere (de exemplu, solicitarea de radianță/reflectanță de la camere care înregistrează doarRGB
).
* O **execuție deliberată numai cu metadate** (toate produsele dezactivate, fără `--indices`) este totuși un succes — o ieșire de imagine goală este rezultatul corect în acest caz.
* Rulați din nou cu `--verbose` și verificați jurnalul backend-ului pentru liniile `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, care explică omisiunile specifice fiecărei camere.

Coduri de ieșire: `0` succes · `1` eșec generic · `2` eroare de argument · `130` întrerupt de Ctrl+C.

***

## Indici de vegetație

Rulați `--indices` cu unul sau mai multe nume de presetări; fiecare indice este plasat în propriul său folder `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

Cele 22 de nume prestabilite acceptate de `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Există trei liste de index — nu le confundați.**Meniul derulant „Setări proiect” din interfața grafică conține 27 de formule (adaugă `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — aceste cinci sunt disponibile numai în interfața grafică și**nu** sunt valabile pentru `--indices`). Comanda live/offline `lattice index --preset` utilizează propria listă separată cu 22 de presetări. Formulele și calculele matematice ale benzilor sunt documentate în [Formule pentru indici multispectrali](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Senzori de lumină DAQ: o scurtă prezentare

Familia `daq pool-*` controlează senzorii spectrali DAQ dMAPIR
(DAQ-U prin USB, DAQ-M prin BLE, DAQ-E prin Ethernet) prin intermediul pool-ului persistent al backend-ului — GUI-ul,CLI
șiSDK
împărtășesc toate un singur handle live. **`pool-*` este calea DAQ acceptată în fișierul „CLI
” livrat**; alte subcomenzi `daq` la care s-ar putea face referire sunt o interfațăMAPIR
-internă, destinată exclusiv sursei- și se închid cu o eroare explicită care vă îndrumă către `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` fără `--duration` rulează până la `pool-record --stop`; directorul de ieșire implicit este `~/Documents/DAQ Live View/` **pe mașina backend-ului**. Profilul de corecție a capacului este ales la momentul conectării (`--cap-id`, implicit pentru backend `sunshine_cosine`) și poate fi schimbat în timp real cu `pool-set-cap` — profilurile de plafonare și intervalul calibrat al senzorului sunt descrise în capitolele dedicate DAQ din acest manual.

{% hint style="warning" %}
**DAQ-E pe o gazdă cu mai multe plăci de rețea (NIC):** prima detectare automată a profilului `pool-connect --eth` după pornire poate eșua chiar și cu un senzor funcțional. `--eth-host <ip-or-hostname>` este varianta fiabilă — folosiți-o ori de câte ori detectarea nu dă rezultate.
{% endhint %}

***

## Camere LATTICE, PTP și automatizarea proiectelor

Familia `lattice` (peste 45 de subcomenzi) acoperă funcționarea camerelor LATTICE de la un capăt la altul: detectare, capturi individuale, matrice sincronizate persistente cu fluxul de conectare „smart-prep” al GUI-ului, previzualizare live în browser, aliniere, calcule de indexare și diagnosticare a plăcilor de rețea ale gazdei. Un exemplu:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

În paralel: `chloros-cli time-sync` raportează despre grandmaster-ul PTP pe care îl rulează gazdaChloros
(camerele LATTICE și senzorii DAQ-E funcționează ca slave pentru acesta, pentru marcaje temporale între dispozitive), iar `chloros-cli project` deschide un proiectChloros
salvat și controlează camerele, matricele și senzorii acestuia fără interfață grafică — inclusiv rețete de captură YAML scriptate.

Aceste trei familii (`lattice`, `project`, `daq pool-*`) sunt, de asemenea, singurele care acceptă `CHLOROS_BACKEND_URL` pentru controlul unui backend **la distanță**; comenzile de bază vizează întotdeauna mașina locală.

Ghiduri complete sunt disponibile în capitolele dedicate LATTICE din acest manual; fiecare opțiune se regăsește în [CLI
Referință](reference/cli-reference.md).

***

## Depanare: Top 5

| Simptom | Soluție |
| --- | --- |
| `Login required` sau o sarcină programată se blochează la promptul `Email:` | Rulați o dată `chloros-cli login EMAIL 'PASSWORD'` pe această mașină — comenzile fără o sesiune stocată în cache vor interacționa în mod interactiv, în loc să eșueze rapid. |
| `backend unreachable` | Porniți aplicația desktop „Chloros
” sau rulați direct fișierul executabil backend (`chloros-backend`). Dacă direcționați `lattice`/`project`/`daq pool-*` către un backend la distanță, verificați `CHLOROS_BACKEND_URL`. |
| Conectarea la matrice blocată: `FRAMES WILL DROP` / `Reduce ROI to enable` | Inelul de recepție al plăcii de rețea (NIC) a gazdei a fost resetat la valorile implicite — cauza nr. 1 pentru care un sistem care funcționa anterior refuză să se conecteze, de obicei după o actualizare a driverului plăcii de rețea. Rulați `chloros-cli lattice network --fix` dintr-un terminal **cu privilegii ridicate** (sau setați `ReceiveBufferLen=256`, `PendingReceives=64`); consultați secțiunea *Configurarea și reglarea plăcii de rețea a gazdei* din documentația de referință. |
| Subcomanda `daq` se închide cu mesajul: „necesită pachetul DAQ complet…” | Situație așteptată la versiunile livrate — pachetulCLI
compilat include doar familia de comenzi `daq pool-*`, care acoperă conectarea, transmisia, înregistrarea și selectarea cap-urilor. Utilizați `pool-*` (sau `chloros_sdk.connect_daq_sensor()` de laPython
). |
| Jetson afișează un avertisment privind memoria swap înainte de folderele mari | Adăugați memoria swap bazată pe fișiere —CLI
afișează exact comenzile `fallocate`/`swapon` care trebuie rulate. |

***

## Obținerea de ajutor

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Fiecare opțiune, fiecare subcomandă:** [CLI
Referință](reference/cli-reference.md)
* **EchivalentPython
:** [Python
SDK
](api-python-sdk.md) și [SDK
Referință](reference/sdk-reference.md)
* **Asistență:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
