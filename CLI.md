# CLI: Linia de comandă

> **Referință completă:**[Referința CLI](reference/cli-reference.md) documentează**fiecare opțiune a fiecărei subcomenzi** și este optimizată pentru asistenții AI — lipiți URL-ul acesteia în asistentul dvs. și solicitați o comandă funcțională: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Sfat pentru instrumentele AI:** orice pagină din acest manual este disponibilă în format Markdown brut prin adăugarea lui `.md` la URL-ul acesteia (de exemplu, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), iar `https://mapir.gitbook.io/chloros/llms.txt` indexează întregul manual pentru utilizarea de către modelele LLM.

> **Doriți ca asistentul dvs. să ruleze chiar Chloros?** `chloros-mcp`, un server MCP instalat cu Python SDK, funcționează cu Claude Desktop, Claude Code, Cursor și alți clienți MCP: consultați [Utilizarea Chloros cu asistenții AI](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Ce este CLI


`chloros-cli` este interfața de linie de comandă pentru același motor de procesare pe care îl folosește aplicația desktop Chloros
. Este un client HTTP
ușor care comunică cu backend-ul Chloros
(un server local pe `127.0.0.1:5000`) — majoritatea comenzilor pornesc automat backend-ul, astfel încât un singur apel către `chloros-cli process …` este tot ce are nevoie un script. Backend-ul rămâne apoi în funcțiune pentru următoarea comandă: se oprește la `chloros-cli logout` (cu excepția cazului în care aplicația este deschisă sau o execuție sau o compilare este în curs), la `chloros-cli backend stop` sau automat după 30 de minute fără client, execuție sau dispozitiv conectat. `chloros-cli backend status` o afișează.

Funcționează pe **Windows
10/11 (x64)**și**Linux
(x86_64 și NVIDIA Jetson arm64 pe JetPack 6)**, în orice terminal, fără a fi necesară o interfață grafică. Verificați instalarea cu:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Prezentare generală a familiilor de comenzi:

* **Procesare și cont** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 de limbi — consultați [Limbi acceptate](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (numai Linux
/Jetson)
* **Hardware activ** — `lattice` (control cameră LATTICE, peste 45 de subcomenzi), `daq pool-*` (senzori de lumină DAQ), `time-sync` (PTP)
* **Automatizare** — `project` (executare fără interfață grafică a unui proiect Chloros
salvat, inclusiv rețete de captură YAML)

Opțiuni globale utile: `--port N` (port backend, implicit `5000`), `-v/--verbose`, `--restart` (repornire forțată a backend-ului), `--backend-exe PATH`. Consultați [Referința CLI
](reference/cli-reference.md) pentru lista completă.

***

## Instalare

CLI
-ul **este inclus în programul de instalare Chloros** pe toate platformele — nu există o descărcare separată pentru CLI
. Descărcați programul de instalare de pe pagina [Descărcare](download.md).

### Windows


Programul de instalare plasează CLI
-ul în:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

și adaugă acel folder o singură dată la sistemul dvs. `PATH` (o actualizare elimină, de asemenea, intrările duplicate lăsate de programele de instalare anterioare) — **deschideți un terminal nou**după instalare, astfel încât să fie detectat `PATH` actualizat. Programul de instalare plasează, de asemenea, scripturi de lansare (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) în directorul rădăcină al instalării, precum și o**scurtătură Chloros
CLI
** în meniul Start, fiecare dintre acestea deschizând un terminal cu `chloros-cli` gata de utilizare.

### Linux


Instalați `.deb` pentru arhitectura dvs. cu `apt` (nu `dpkg -i`, care omite cerințele recomandate de SDK
Python
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Aceasta instalează `chloros-cli` până la `/usr/bin/chloros-cli` (deja la `PATH`), programele backend și CLI
despachetate în `/usr/lib/chloros/backend/` și `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` este un program de lansare care pornește backend-ul), mediul de execuție Arena SDK
de care are nevoie backend-ul pentru camerele LATTICE și Python
SDK
pentru sistemul `python3`. Consultați [Instalarea pe Linux
](linux/linux-installation.md) pentru detalii.

### Verificare

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Autentificare și licențiere

Accesul la CLI
(și la Python
SDK
) necesită un **plan Chloros
+ plătit**— orice nivel plătit îl include; nivelul gratuit nu îl include. Limita minimă este impusă**pe partea de server** de către backend, nu de către binarul CLI
: o apelare fără autentificare este respinsă cu `401 AUTH_REQUIRED`, iar o solicitare efectuată în timp ce utilizatorul este autentificat, dar în cadrul planului gratuit, cu codul `403 PLAN_UPGRADE_REQUIRED`, indiferent dacă provine de la `chloros-cli`, de la SDK
sau de la un client HTTP
creat manual. Efectuați actualizarea la [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Autentificați-vă **o singură dată pe fiecare mașină**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Parole cu caractere speciale**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` este distorsionat de shell (CLI
-ul detectează acest lucru la un cod 401 și reîncearcă automat, dar ghilimelele simple evită complet problema).
{% endhint %}

Sesiunea este stocată în cache la `~/.chloros/user_session.json` și continuă să funcționeze offline pe durata perioadei de grație a abonamentului (30 de zile pentru abonamentele lunare, până la expirare pentru cele anuale). `chloros-cli status` funcționează chiar și fără un plan plătit, astfel încât motivul refuzului este întotdeauna vizibil.

{% hint style="danger" %}
**Programați sarcini fără interfață grafică? Autentificați-vă mai întâi.**O comandă de lansare a backend-ului (`process`, `status`, `export-status`, …) executată**fără sesiune stocată în cache**nu eșuează rapid — trece într-un prompt interactiv `Email:` / `Password:` pe stdin. Prin urmare, o sarcină cron nesupravegheată sau un pas CI va**rămâne blocat în așteptarea unei intrări**. Rulați `chloros-cli login EMAIL 'PASSWORD'` o singură dată pe mașină înainte de a programa orice.
{% endhint %}

***

## Prima dvs. execuție de procesare

Direcționați `process` către un folder cu capturi — acesta detectează automat Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` sau o combinație:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Fluxurile de progres sunt transmise în timp real pentru fiecare fir de pipeline (Detectare, Analiză, Procesare, Export), iar o execuție reușită se încheie cu raportarea numărului de produse imagistice generate: `Image products written: N`, urmat de `(plus M clamp masks)` atunci când execuția a scris și [măști de clamp](output-image-formats.md#clamp-masks) (acestea nu sunt numărate ca produse).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Dacă un backend Chloros
rulează deja (de exemplu, cel al aplicației desktop), `process` îl utilizează și își deschide proiectul în acesta. Refuză să ruleze cât timp aplicația este deschisă pe un proiect pe care l-a deschis ea însăși: procesați din aplicație sau închideți proiectul respectiv. Dacă o execuție anterioară încă se finalizează, așteaptă până la 15 secunde înainte de a refuza. `--restart` forțează un backend nou. Pe Linux
, un backend rulat ca serviciu systemd nu este niciodată repornit: `process` îl utilizează așa cum este, iar în situațiile în care altfel l-ar repornit (`--restart` sau o importare care nu a găsit nimic), afișează `sudo systemctl restart <unit>` și iese din `1`.

### Unde ajung rezultatele

`process` scrie într-un **dosar de proiect**, nu în dosarul de intrare:

* Fără `-o`: proiectul este creat în folderul dvs. implicit pentru proiecte (partajat cu interfața grafică; îl puteți gestiona cu `get-project-folder` / `set-project-folder`; când nu este setat niciunul, `Documents/MAPIR/Chloros Projects` în folderul dvs. de utilizator), denumit după `-n/--project-name` sau o marcă temporală (`YYYYMMDD_HHMMSS`) când este omis.
* Cu `-o PATH`: acel dosar **este** dosarul proiectului. Dacă acesta conține deja un `project.json`, se creează un fișier înrudit cu sufixul `_1`/`_2`… cu sufixul respectiv, în loc să se suprascrie. Un XPTX000100 relativ este considerat relativ față de folderul din care rulați XPTX000101.

{% hint style="info" %}
**Căi relative (modificate în versiunea 1.3.0).** Fiecare cale deschisă de backend — `-o`/`--output` la orice comandă (inclusiv `output` implicit al comenzilor de captură), `--input`, `--burst-dir`, `--profile`, `--save` și `--mask` — este rezolvat în funcție de folderul din care rulați `chloros-cli`, astfel încât `-o captures/` scrie în `<current folder>/captures/`. În versiunea 1.2.0, backend-ul le rezolva în propriul său folder de instalare, ceea ce genera eroarea „Accesul este refuzat”. În cazul unui backend la distanță (`CHLOROS_BACKEND_URL`), căile sunt trimise așa cum au fost introduse și sunt rezolvate pe acea mașină.
{% endhint %}

{% hint style="warning" %}
**Windows
: păstrați folderul de proiect scurt, cu excepția cazului în care sunt activate căile lungi.** Cu căile lungi dezactivate în Windows
(setarea implicită a Windows
), nicio cale de fișier nu poate depăși 260 de caractere, iar produsele se află la o distanță de până la aproximativ 130 de caractere sub folderul de proiect. `process` verifică acest lucru înainte de a porni: un folder de proiect prea adânc este respins cu mesajul **„Folderul de proiect este prea adânc pentru acest computer”**, se afișează cel mai lung folder de proiect care se încadrează și se indică modul de activare a căilor lungi — nimic nu este procesat. Utilizați o versiune mai scurtă, `-o` (sau `set-project-folder`), sau activați căile lungi din Windows
(opțiunea **Enable Windows
long paths** din programul de instalare, bifată implicit) și reporniți Chloros
.
{% endhint %}

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

Folderul camerei este `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE (corespunzând câmpului EXIF al capturii `Model`) și `<model>_<filter>` (de exemplu, `Survey3N_RGN`) pentru Survey3
. Dosarul de format urmează `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` sau `tiff32` pentru `TIFF (32-bit, Percent)`.

În cazul unei camere multispectrale M3C, un dosar `clamp_masks/` poate apărea, de asemenea, într-un dosar de format, alături de dosarele de produse (de exemplu, `tiff16/clamp_masks/Reflectance_Calibrated_Images/` și `tiff32/clamp_masks/Radiance_Images/`). Acesta conține măștile pe pixel ale produselor de radianță și reflectanță în care procesul de separare a benzilor a limitat o valoare negativă la zero — a se vedea [Măști de limitare](output-image-formats.md#clamp-masks). Niciun dosar de produs nu conține vreodată o mască.

{% hint style="info" %}
**Fiecare produs exportat păstrează numele fișierului SURSĂ.**Un export de radianță al `capture_..._raw.tif` se numește în continuare `capture_..._raw.tif` — doar că se află în `tiff32/Radiance_Images/`.**Dosarul identifică produsul, nu numele fișierului**, așa că folosiți un filtru glob pentru director, nu pentru sufixul `*radiance*`.
{% endhint %}

### Opțiunile pe care le veți folosi efectiv

| Indicator | Implicit | Ce face |
| --- | --- | --- |
| `-o, --output PATH` | folderul implicit al proiectului | Locația folderului proiectului (vezi mai sus). |
| `-n, --project-name NAME` | marcaj temporal | Numele proiectului. |
| `--format FMT` | `TIFF (16-bit)` | Una dintre următoarele: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
nu mărește rezoluția: pe toate imaginile de 8 biți, un format cu adâncime mai mare este scris ca `TIFF (8-bit)`, iar acest lucru este menționat o singură dată. |
| `--indices NAME [NAME ...]` | niciunul | Indici de vegetație de exportat (vezi [Indici de vegetație](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neuronal, mai lent, calitate maximă (Chloros
+, necesită un GPU CUDA: fără acesta, procesul utilizează `standard`; respins pentru capturile LATTICE, care primesc întotdeauna `standard`). |
| `--vignette / --no-vignette` | activat | Corecție vignetă. |
| `--reflectance / --no-reflectance` | activat | Calibrare reflectanță; pentru LATTICE, aceasta este și comutatorul pentru produsul de reflectanță. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forțează punctul de intrare în pipeline pentru fișierele TIFF LATTICE. |

Pentru toate celelalte opțiuni — reglarea detectării țintelor, PPK, pini de expunere, indicatori de aliniere a matricei — consultați [secțiunea `process` din Referința CLI
](reference/cli-reference.md).

***

## Alegerea elementelor de exportat (produse LATTICE)

Procesarea LATTICE se ramifică către **toate produsele aplicabile într-o singură trecere**. Cele patru comutatoare per produs sunt toate**activate implicit**; utilizați formularul `--no-` pentru a dezactiva unul:

| Comutator | Produs |
| --- | --- |
| `--debayered` | Demosaic liniar → `Debayered_Images/`: se elimină offsetul de întuneric din pachetul de calibrare, apoi se aplică o demosaicare sensibilă la margini, fără corecție de liniaritate, câmp plat sau separare — același fișier pe care îl salvează o captură la nivelul său `debayered`. Se păstrează orientarea nativă a senzorului (înregistrată în XMP). |
| `--preview` | Previzualizare pe ecran (balans de alb + gamma; extindere în culori false pentru imagini multispectrale) → `Preview_Images/` |
| `--radiance` | radianță float32, W/m²/sr/nm → `Radiance_Images/` (întotdeauna `tiff32/`) |
| `--reflectance` | uint16 reflectanță, compatibilă cu Pix4D → `Reflectance_Calibrated_Images/` |

Camerele principale RGB
emit întotdeauna doar date debayered + previzualizare — radianța/reflectanța pe bandă nu are relevanță pentru un senzor de bandă largă, astfel încât aceste comutatoare nu au niciun efect în cazul lor. Survey3
`.raw` ignoră comutatoarele și urmează calea standard de reflectanță/țintă.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (implicit `auto`) alege referința de reflectanță: `auto` face ca o [țintă de calibrare](calibration-targets.md) din cadru, care a trecut testul de control al calității (QA), să fie referința absolută și revine la raportul de lumină descendentă al senzorului de lumină DAQ (ρ = π·L/E) atunci când nu este prezentă nicio țintă (o țintă care trece controlul de calitate cu un indicator concurează cu DAQ în ceea ce privește scorul măsurat); `target` este strict (fără substituire DAQ); `daq` are autoritate asupra DAQ. Scanările țintelor măsurate pe unitate pot fi furnizate cu `--target-reflectance-dir`.

{% hint style="info" %}
**Citirea pixelilor de reflectanță:**valoarea DN care înseamnă ρ = 1,0 este**pe sursă** — fișierele LATTICE înscriu `Chloros:PixelScale=32768` în XMP; fișierele Survey3
utilizează 65535 și nu conțin eticheta `Chloros:PixelScale` (conțin însă alte etichete de procesare `Chloros:*`). Citiți eticheta și împărțiți la valoarea acesteia, în loc să presupuneți o constantă. Detaliile și singurul caz marginal deliberat fără scală se găsesc în [Referința CLI
](reference/cli-reference.md).
{% endhint %}

**Procesarea începe întotdeauna de la `raw`.** Produsele derivate (exporturi debayered/radianță/reflectanță) nu sunt niciodată reintroduse în fluxul de procesare — reimportarea și procesarea lor ar duce la aplicarea dublă a calculelor de calibrare, așa că Chloros
le omite și indică acest lucru. `--input-level` este o cale de ieșire intenționată atunci când aveți cu adevărat nevoie să forțați un punct de intrare.

***

## Când o execuție eșuează

Începând cu versiunea 1.2.0, `process` eșuează în mod evident, în loc să „reușească” fără a afișa nimic:

* O execuție care **a solicitat produse, dar nu a scris niciunul**— doar `project.json` și `calibration_data.json` — afișează `Processing finished but wrote no image products.` și**iese cu un cod de ieșire diferit de zero**, astfel încât scripturile să o poată detecta. Cauzele obișnuite: folderul de intrare nu a fost recunoscut ca o captură (verificați structura și `--input-level`) sau fiecare produs solicitat era inaplicabil pentru acele camere (de exemplu, solicitarea radianței/reflectanței de la camere care funcționează doar în RGB
).
* O **execuție deliberată numai cu metadate** (toate produsele dezactivate, fără `--indices`) este totuși considerată reușită — o ieșire de imagine goală este rezultatul corect în acest caz.
* Rulați din nou cu `--verbose` și verificați jurnalul backend-ului pentru liniile `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, care explică omisiunile pentru fiecare cameră în parte.
* Pe Windows
, cu opțiunea pentru căi lungi dezactivată, un folder de proiect prea adânc pentru limita de 260 de caractere a căii este respins **înainte** de începerea procesării, folosindu-se cel mai lung folder de proiect care se încadrează (vezi [Unde ajung fișierele de ieșire](#where-the-outputs-land)).
* Comenzile de captură (`lattice capture`, `lattice array-capture` și celelalte familii de captură) menționează fiecare fișier pe care nu l-au putut scrie și ies cu codul `1`; capturile repetate se opresc când unitatea de stocare este plină.

Coduri de ieșire: `0` succes · `1` eroare generică · `2` eroare de argument · `130` întrerupt de Ctrl+C.

***

## Indici de vegetație

Rulați `--indices` cu unul sau mai multe nume prestabilite; fiecare indice este salvat în propriul său folder `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

Cele 22 de nume prestabilite acceptate de `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Un nume care nu figurează încă în proiect este adăugat la acesta. Un nume care nu este o presetare sau pe care benzile camerelor nu îl pot forma este omis, fiind afișat un avertisment care îl menționează. Dacă interfața CLI
nu poate citi lista de indexuri curentă a proiectului, aceasta lasă lista neschimbată și afișează un avertisment în loc să o înlocuiască.

{% hint style="warning" %}
**Există trei liste de indexuri — nu le confundați.**Meniul derulant „Setări proiect” din GUI conține 27 de formule (adaugă `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — aceste cinci sunt disponibile doar în interfața grafică și**nu** sunt valabile pentru `--indices`). Comanda live/offline `lattice index --preset` utilizează propria listă separată cu 22 de presetări. Formulele și calculele de bandă sunt documentate în [Formule pentru indici multispectrali](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Senzori de lumină DAQ: o scurtă prezentare

Familia `daq pool-*` controlează senzorii spectrali DAQ MAPIR
(DAQ-U prin USB, DAQ-M prin BLE, DAQ-E prin Ethernet) prin intermediul pool-ului persistent al backend-ului — GUI, CLI
și SDK
împărtășesc toate un singur handle live. **`pool-*` este calea DAQ acceptată în CLI
livrată**; alte subcomenzi `daq` la care s-ar putea face referire sunt o interfață internă MAPIR
destinată exclusiv sursei și se închid cu o eroare explicită care vă îndrumă către `pool-*`.

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

`pool-record` fără `--duration` rulează până la `pool-record --stop`; directorul de ieșire implicit este `~/Documents/DAQ Live View/` **pe mașina backend-ului**. Profilul de corecție a capacului este ales la momentul conectării (`--cap-id`, implicit pentru backend `sunshine_cosine`) și poate fi schimbat în timp real cu `pool-set-cap` — profilurile de limită și intervalul calibrat al senzorului sunt descrise în capitolele dedicate DAQ din acest manual.

{% hint style="warning" %}
**DAQ-E pe o gazdă cu mai multe plăci de rețea (NIC):** prima detectare automată a profilului `pool-connect --eth` după pornire poate eșua chiar și cu un senzor funcțional. `--eth-host <ip-or-hostname>` este varianta fiabilă — utilizați-o ori de câte ori detectarea nu dă rezultate.
{% endhint %}

***

## Camere LATTICE, PTP și automatizarea proiectelor

Familia `lattice` (peste 45 de subcomenzi) acoperă funcționarea camerelor LATTICE de la un capăt la altul: detectare, capturi individuale, matrice sincronizate persistente cu fluxul de conectare „smart-prep” al GUI-ului, previzualizare live în browser, aliniere, calcule de indexare și diagnosticarea plăcilor de rețea ale gazdei. Un exemplu:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Alături de aceasta: `chloros-cli time-sync` raportează informații despre grandmaster-ul PTP pe care îl rulează gazda Chloros
(camerele LATTICE și senzorii DAQ-E funcționează ca slave pentru acesta, pentru marcarea temporală între dispozitive), iar `chloros-cli project` deschide un proiect Chloros
salvat și controlează camerele, matricile și senzorii acestuia fără interfață grafică — inclusiv rețete de captură YAML scriptate.

Aceste trei familii (`lattice`, `project`, `daq pool-*`) sunt, de asemenea, singurele care acceptă `CHLOROS_BACKEND_URL` pentru controlul unui backend **la distanță**; comenzile de bază vizează întotdeauna mașina locală.

Ghiduri complete sunt disponibile în capitolele dedicate LATTICE din acest manual; fiecare opțiune se regăsește în [Referința CLI
](reference/cli-reference.md).

***

## Depanare: Top 5

| Simptom | Soluție |
| --- | --- |
| `Login required` sau o sarcină programată se blochează la promptul `Email:` | Rulați o dată `chloros-cli login EMAIL 'PASSWORD'` pe această mașină — comenzile fără o sesiune stocată în cache se execută interactiv, în loc să eșueze rapid. |
| `backend unreachable` | Porniți aplicația desktop Chloros
sau porniți manual backend-ul: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` pe Windows
(din folderul `backend` al folderului de instalare; acesta nu se află în `PATH`), `chloros-backend` pe Linux
. `chloros-cli backend status` arată dacă unul dintre ele rulează; `lattice`/`project`/`daq pool-*` nu pornesc niciodată unul. Dacă le direcționați către un backend la distanță, verificați `CHLOROS_BACKEND_URL`. |
| Conectarea la matrice blocată: `FRAMES WILL DROP` / `Reduce ROI to enable` | Inelul de recepție al plăcii de rețea a gazdei a fost resetat la valorile implicite — cauza nr. 1 pentru care un sistem care funcționa anterior refuză să se conecteze, de obicei după o actualizare a driverului plăcii de rețea. Rulați `chloros-cli lattice network --fix` dintr-un terminal **cu drepturi ridicate** (sau setați `ReceiveBufferLen=256`, `PendingReceives=64`); consultați secțiunea *Configurarea și reglarea plăcii de rețea a gazdei* din documentația de referință. |
| Subcomanda `daq` se închide: „necesită pachetul DAQ complet…” | Utilizați comenzile `daq pool-*`, care controlează același senzor prin intermediul backend-ului: conectare, transmisie, înregistrare, selectare cap și, pentru un DAQ-E, setările de rețea, repornire și actualizări de firmware (sau `chloros_sdk.connect_daq_sensor()` din Python
). |
| Jetson afișează un avertisment privind memoria swap înainte de folderele mari | Adăugați memorie swap bazată pe fișiere — interfața CLI
afișează exact comenzile `fallocate`/`swapon` care trebuie executate. |

***

## Obținerea de ajutor

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Fiecare opțiune, fiecare subcomandă:** [Referință CLI
](reference/cli-reference.md)
* **Echivalent Python
:** [Python
SDK
](api-python-sdk.md) și [Referință SDK
](reference/sdk-reference.md)
* **Asistență:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
