# Chloros Python SDK Referință

**Versiune:**

1.2.0**Generat:**29 iulie 2026, ora 19:19 ·**Revizuit:** 30 august 2026**Pachet:** `chloros-sdk` (PyPI)**Public țintă:** Optimizat pentru utilizarea de către LLM; ușor de citit de către oameni.**Domeniu de aplicare:** Fiecare clasă publică, funcție și ajutor expus de `import chloros_sdk`, cu exemple care pot fi copiate și lipite, acoperind procesarea imaginilor, controlul unei singure camere, matrice sincronizate, senzori DAQ și automatizarea proiectelor.

Dacă aveți nevoie doar de informațiile esențiale, accesați:
- [Instalare și ghid de pornire rapidă](#installation)
- [Smart-Connect pentru matrice LATTICE](#smart-connect-for-lattice-cameras)
- [Sesiuni cu senzori DAQ](#daq-sensor-sessions)
- [Automatizarea proiectelor](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arhitectura în 60 de secunde

SDK-ul este un strat subțire de tip „Python” care acoperă backend-ul Chloros (același server Flask pe care îl utilizează interfața grafică pentru desktop și CLI). Pentru automatizare, importați `chloros_sdk` și apelați metode de nivel înalt; în fundal, fiecare apel devine o cerere HTTP către backend-ul local de pe portul 5000 — `http://127.0.0.1:5000/api/...` (în mod deliberat nu `localhost`, care se rezolvă mai întâi ca `::1` pe Windows și durează ~2 s pe cerere în cazul unui backend exclusiv IPv4). Backend-ul deține parcul de echipamente — camere, senzori DAQ, profiluri de aliniere, buffere de cadre — astfel încât scripturile SDK pot coexista cu interfața grafică fără a intra în conflict pentru porturile seriale sau lățimea de bandă a plăcilor de rețea.

Există trei interfețe pe care le veți utiliza:

1. **`ChlorosLocal` + funcții gratuite** (`process_folder`, `process_lattice_capture`) — Flux de procesare a imaginilor. Rulați un folder întreg prin calibrare / debayer / export de index dintr-o singură apelare a funcției „Python”.
2. **Mânerele Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Deschideți o sesiune persistentă pentru hardware-ul activ. Același flux „smart-prep” ca și în interfața grafică: sondă de rețea, selectare automată a nivelului, PTP, inițializare AE, configurare declanșare GPIO.
3. **`ChlorosProject` / `open_project`** — Încarcă un proiect salvat (folder cu `cameras.json` + `sensors.json` + `project.json`), conectează totul simultan și efectuează capturi de date cu identificatori denumiți.

Interfețele 1 și 2 **pornesc automat un backend local** dacă nu există deja unul în ascultare (același binar inclus în pachet pe care GUI-ul/CLI ) — astfel încât un script simplu funcționează dintr-un shell nou, fără a fi necesar să porniți mai întâi un backend. Treceți `auto_start_backend=False` pentru a dezactiva această opțiune (de exemplu, când indicați un backend la distanță, care nu este niciodată pornit). Consultați [Pornirea automată a backend-ului](#backend-auto-start). Surface 3 se comportă diferit: `open_project()` nu acceptă parametrul `auto_start_backend`, iar `connect_all()` nu pornește niciodată un backend — verifică o dată `http://127.0.0.1:5000` și, dacă nu primește răspuns, revine în mod silențios la controlul direct (fără backendfără backend) `lattice_sdk`. Doar `proj.process()` și `stream(..., overlays=True)` construiesc în mod leneș un `ChlorosLocal()` (care pornește automat).

Toate trei sunt protejate prin autentificare: rulați `chloros-cli login` o dată pe mașină sau conectați-vă prin interfața grafică de pe desktop. Apelurile SDK Apelurile fără o sesiune validă generează eroarea `ChlorosAuthenticationError`.

Cerințe:
- Python 3.7+ (conform specificațiilor pachetului; dezvoltat/testat pe versiunea 3.10)
- Chloros Desktop instalat local (fișierul binar backend este inclus în programul de instalare)
- Autentificare activă pe Chloros+. Nivelul minim pentru SDK / CLI este **Copper**sau superior (Copper / Bronze / Silver / Gold); nivelul gratuit**Iron**nu are acces la SDK / CLI. Această regulă este aplicată**pe partea de server**: fiecare cerere marcată cu SDK / CLI trebuie să includă atât o sesiune activă, cât și un plan plătit; în caz contrar, backend-ul returnează `403` împreună cu `error_code: PLAN_UPGRADE_REQUIRED` (afișat ca `ChlorosLicenseError` de către `ChlorosLocal` și ca `ChlorosConnectError` de către helperii `connect_*`). Un solicitant deconectat primește în schimb codul de eroare `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — cele două sunt distincte deoarece rulararea din nou a `chloros-cli login` remediază prima eroare, dar nu o poate remedia pe a doua.
- Utilizarea offline este acceptată în cadrulperioadei de grație a planului: nivelul este citit din cache-ul de validare a serverului (5 min) sau din cache-ul licenței semnate, asociate dispozitivului (30 de zile pentru planurile lunare, până la expirarea abonamentului pentru cele anuale). Când această perioadă de grație expiră, planul revine la varianta gratuită, iar accesul la SDK / CLI se întrerupe până când dispozitivul poate accesa serverul măcar o dată. `chloros-cli status` (`GET /api/license-status`) rămâne accesibil în nivelul gratuit, astfel încât motivul este evident — este singura rută SDK / CLI scutită de restricția de nivel.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS sau o versiune mai recentă**, sau Jetson (JetPack 6). Ubuntu 20.04**nu** este acceptat: dependențele `.deb` derivă din ceea ce leagă backend-ul, inclusiv `libc6 (>= 2.34)`, iar focal include glibc 2.31.

---

## Instalare

Python SDK este un strat subțire Python care acoperă backend-ul Chloros. Pentru orice altceva în afară de câteva fluxuri de lucru dedicate exclusiv DAQ, aveți nevoie de **pachetul desktop Chloros instalat local** (programul de instalare Windows sau Linux `.deb`) — acesta furnizează fișierul binar de backend, mediul de execuție Arena SDK pentru camerele LATTICE și pachetele de calibrare.

Cele mai recente descărcări: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Pasul 1 — Instalați pachetul de platformă „Chloros”

#### „Windows” (.exe)

1. Descărcați `Chloros-Setup-x.y.z.exe` de pe pagina de descărcare.
2. Rulați programul de instalare și urmați instrucțiunile asistentului. Calea de instalare implicită este `C:\Program Files\MAPIR\Chloros\`.
3. Lansați Chloros cel puțin o dată și conectați-vă cu contul dvs. Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Pasul 2 — Instalați Python SDK

**Programul de instalare Chloros include un pachet „wheel” corespunzător SDK.** Fiecare program de instalare Windows și fiecare fișier .deb Linux plasează pe disc un fișier `chloros_sdk-X.Y.Z-py3-none-any.whl` care corespunde exact versiunii GUI / CLI / backend. Nu este necesar să urmăriți PyPI pentru a rămâne sincronizat.

#### Windows

Programul de instalare rulează automat `pip install` cu fișierul wheel inclus, utilizând lansatorul de pe sistemul dvs. Python (se preferă lansatorul `py.exe`, dar se recurge la `python -m pip`). Nu este necesară nicio acțiune — `import chloros_sdk` funcționează în mediul dvs. Python după o instalare reușită. Dacă pe sistem nu există Python, programul de instalare omite în mod silențios acest pas, iar interfața grafică + CLI continuă să funcționeze.

#### Linux (.deb)

Fișierul .deb plasează roata în `/usr/lib/chloros/sdk/`. Fișierul `postinst` afișează comanda exactă — distribuțiile PEP 668 refuză în mod implicit scrierea globală prin pip, așa că nu se realizează instalarea automată:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pentru implementările Jetson izolate (air-gapped), acest proces se desfășoară complet offline — fișierul wheel se află deja pe disc.

#### PyPI public

Pentru gazdele care utilizează doar pip (fără pachetul desktop „Chloros” instalat; fluxuri de lucru cu backend la distanță sau exclusiv DAQ):

```bash
pip install chloros-sdk
```

PyPI este actualizat la versiunile de lansare ale programului de instalare, astfel încât fișierul wheel publicat corespunde celei mai recente versiuni stabile. Versiunile de dezvoltare (de exemplu, `1.1.4.dev1`) sunt distribuite numai prin fișierul wheel inclus în programul de instalare.

#### Verificare

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ este necesar un abonament.** Toate apelurile către SDK necesită o autentificare activă la Chloros+. Rulați `chloros-cli login user@example.com 'YourPassword'` o singură dată pe fiecare mașină; datele de autentificare sunt stocate în cache în `~/.chloros/`.

### Am nevoie de pachetul pentru desktop?

Pachetul pip singur **nu este** suficient pentru majoritatea fluxurilor de lucru. Iată de ce are nevoie fiecare interfață SDK:

| Interfață SDK | Are nevoie de pachetul Desktop? | De ce |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Da** | Pornește automat binarul backend la `/usr/lib/chloros/chloros-backend` (Linux) sau `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Da**(local)**/ Nu**(la distanță) | Clienți Pure HTTP prin backend. Backend local → este necesar pachetul pentru desktop. Backend la distanță → `backend_url=`**prin intermediul unui tunel** (consultați Modul Backend la distanță — backend-urile livrate se leagă doar la loopback). |
| `ChlorosProject` / `open_project` | **Da** | Gestionează proiectele salvate prin backend. |
| Clase LATTICE directe (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Da** | Necesită runtime-ul nativ Arena SDK inclus în pachetul pentru desktop. În caz contrar, `CAMERA_AVAILABLE` este `False` la import. |
| Clase DAQ directe (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nu** | „Python” pur prin pyserial/bleak/zeroconf. Un mediu exclusiv pip poate gestiona DAQ-urile de la un capăt la altul. |

### Modul Remote-Backend (gazdă exclusiv pip, prin tunel)

> **Backend-ul livrat nu este accesibil prin LAN.** Versiunile de producție
> se leagă numai de loopback (ambele familii de loopback) și refuză categoric
> singurul mod non-loopback (`CHLOROS_CLOUD_MODE`), astfel încât
> `backend_url="http://<lan-ip>:5000"` **nu poate funcționa cu un
> Chloros instalat** — acel model a funcționat întotdeauna doar cu un backend de tip source/dev
> . Pentru a controla un backend de pe o altă mașină, redirecționați portul său loopback
> manual și direcționați SDK către tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Gazdele fără monitor / CI / robotice pot păstra o mașină cu instalarea completă a desktopului ca „server Chloros” și `pip install chloros-sdk` peste tot în rest — dar transportul între ele se face prin tunelul configurat de utilizator de mai sus, nu printr-o URL directă în LAN.

> **Limitare cunoscută — `ChlorosLocal` nu suportă exclusiv pip.** `ChlorosLocal(backend_url=BACKEND)` rezolvă în prezent un binar backend local în constructorul său *înainte* de a interoga URL și generează `ChlorosBackendError` („Backend Chloros nu a fost găsit…”) atunci când nu este instalat niciun pachet desktop — chiar și în cazul în care există un backend la distanță accesibil. Doar interfața smart-connect de mai sus (`connect_camera` / `connect_array` / `connect_daq_sensor`, plus `analyze_array_network` și ajutoarele `list_*` / `discover_*`) funcționează de pe o gazdă exclusiv pip.

### Flux de lucru exclusiv DAQ (gazdă-only)

Dacă aveți nevoie doar de senzori DAQ și nu utilizați camerele LATTICE sau procesarea imaginilor, pachetul pip este autonom:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Fără backend, fără fișier .deb, fără Chloros+ nu este necesară autentificarea pentru lucrul direct cu hardware-ul de DAQ.

---

## Ghid de pornire rapidă

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indexul de nivel superior al API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Prelucrarea imaginilor — `ChlorosLocal`

Clasa principală a pipeline-ului. Pornește backend-ul la prima utilizare, creează și configurează proiecte, monitorizează progresul și returnează rezumate după execuție.

### Constructor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metode

| Metodă | Descriere |
| --- | --- |
| `create_project(project_name, camera=None)` | Creează un proiect nou (opțional cu un șablon de cameră, cum ar fi `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importă imagini RAW/TIF/JPG/DNG **și înregistrări ale senzorului de lumină `.daq`**. Returnează `count` (imagini) și `scan_count` (înregistrări). Afișează un avertisment doar dacă folderul nu conține niciuna dintre acestea. |
| `export_light_sensor(daq=True, csv=True)` | Scrie fișierele calibrate `.daq` + `.csv` pentru fiecare înregistrare a senzorului de lumină din proiect, în `<project>/Light Sensor/`. A se vedea [Înregistrări ale senzorului de lumină](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Setează parametrii de procesare. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Rulați pipeline-ul. Returnează `{"status": "complete", "async": False}`, plus o cheie `summary` atunci când backend-ul furnizează una — consultați [Rezumatul post-rulare și sfaturi](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Verifică starea backend-ului. |
| `logout()` | Șterge credențialele din cache. |
| `shutdown_backend()` | Opriți backend-ul (dacă a fost pornit cu opțiunea SDK -started). |
| `discover_cameras()` | Descoperiți camerele LATTICE **prin intermediul backend-ului acestei instanțe** (`/api/camera/discover`). Returnează o listă de dicționare (`serial`, `model`, `ip`, …) — aceeași structură pe care o vede GUI/ CLI. Listă goală dacă nu se găsește niciuna sau dacă backend-ul este inaccesibil. |
| `camera_capture(output_dir, format="tiff", **settings)` | Capturează un singur cadru**prin backend**(pornit automat de acest identificator), astfel încât acesta să beneficieze de aceeași pregătire ca în GUI/ CLI (implicit 12 biți, reutilizare din pool, metadate de calibrare încorporate). Rezolvați ținta cu `serial=` sau `device_index=`; transmiteți `exposure`/`gain`/`pixel_format`/`preset` ca `**settings`. Returnează dicționarul de metadate vechi (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Generează cadre de previzualizare compuse prin suprapunere dintr-o cameră comună — client MJPEG simplu pe ruta `/api/camera/<serial>/stream-annotated` a backend-ului (zebra / grilă / reticul / histogramă / peaking / punct desenat pe partea serverului). `decode=True` generează matrice BGR; `False` generează octeți brute JPEG. De asemenea, accesibil la nivel de proiect ca `ChlorosProject.stream(overlays=True)`. |

Utilizați-l ca manager de context pentru curățare garantată:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Înregistrări cu senzor de lumină — calibrate `.daq` + `.csv`

Un DAQ-U / DAQ-M / DAQ-E poate fi înregistrat **fără** pachetul său de calibrare. Aceasta este
ceea ce înregistratoarele publice [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
înregistratoare (`record_daq.py`) fac în mod implicit: ele scriu valorile brute ale senzorului și marchează
fișierul astfel încât Chloros să preia**după numărul de serie** — mai întâi din cache-ul local,
apoi din cloud-ul MAPIR — și o aplică la import.

Chloros scrie rezultatul înapoi sub forma a două produse pentru fiecare înregistrare, sub
`<project>/Light Sensor/`:

| Produs | Ce reprezintă |
| --- | --- |
| `<name>_calibrated.daq` | Arhiva reprocesabilă — aceeași schemă ca o înregistrare în timp real, declarând acum pachetul care a generat-o. Reimportarea acesteia **nu** o calibrează a doua oară. |
| `<name>_calibrated.csv` | Iradierea spectrală în W/m²/nm pe grila de lungimi de undă proprie a senzorului, un rând per citire, plus coloane fotometrice (putere totală, lux fotopic/scotopic, PPFD și defalcarea sa pe albastru/verde/roșu, lungimea de undă de vârf). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Numai senzori fără pachet (DAQ-A).** Numărul brut de impulsuri spectrale ale senzorului — *nu* iradianța. A se vedea mai jos. |

`process()` efectuează această exportare ca una dintre etapele sale. **Nu** necesită imagini:
un senzor de lumină utilizat independent reprezintă un flux de lucru de primă clasă, iar un astfel de proiect nu conține
imagini prin definiție.

**Înregistrările DAQ-A se exportă sub formă de numărări brute.** Familia DAQ-A este anterioară sistemului de pachete pe serie
și nu are niciun pachet de preluat — este calibrat pe teren în raport cu o
țintă de reflectanță, motiv pentru care nu a avut niciodată nevoie de unul. Aceste înregistrări se exportă
sub prefixul `_raw` în loc de `_calibrated`: un nume de fișier diferit în loc de un indicator
în interiorul fișierului, deoarece informația trebuie să rămână intactă atunci când este trimisă prin e-mail doar ca nume simplu. Antetul
`.csv` indică `raw spectral sensor counts (NOT irradiance)` și avertizează că
valorile sunt comparabile **în cadrul** fișierului — exact scopul pentru care calibrarea bazată pe țintă le utilizează
— și nu între senzori. Coloanele fotometrice dependente de putere (puterea totală,
lux fotopic/scotopic, PPFD) returnează **NULL** în loc să fie integrate din numărări.

Un DAQ-U / DAQ-M / DAQ-E al cărui pachet pur și simplu nu a putut fi preluat este totuși **omisi**,
nu este scris în format brut: în acest caz, pachetul există, iar „reconectarea și reprocesarea” reprezintă un sfat real.

Înregistrările vechi **v1.01 / v1.02** (un DAQ-A-SD le scrie) nu conțin o epocă pentru fiecare citire,
ci doar ora de scriere a fișierului. Modulul de potrivire imagine↔flux descendent încă le respinge — potrivirea unui
cadru cu o oră de scriere ar fi o eroare invizibilă — dar exportatorul le citește, iar
CSV afișează `clock=daq_created_on`, astfel încât produsul precizează pe ce ceas se bazează.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

O înregistrare al cărei pachet de calibrare nu poate fi preluat(offline sau un senzor fără
calibrare în fișier) este raportată sub codul `skipped` **cu motivul**. Nu este niciodată
salvat ca fișier „calibrat” care conține date brute — conectați-vă la internet și
rulați din nou, iar exportul se va finaliza.

### Apeluri de progres

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Rezumat și sfaturi după rulare

La finalizare, `process()` preia `GET /api/processing-summary` și atașează corpul ca `result["summary"]`. Preluarea se face pe baza principiului „best-effort” și nu blochează niciodată o returnare reușită — dacă rezumatul nu este disponibil, `process()` revine la forma simplă `{"status": "complete", "async": False}`. Fiecare intrare din `summary["hints"]` — propoziții complete cu remedierea sugerată, de exemplu, motivul pentru care o execuție a produs zero ieșiri — este, de asemenea, retransmisă ca un `UserWarning` de tip „Python”, astfel încât execuțiile cu ieșire zero se autodiagnostică chiar dacă nu inspectați niciodată dicționarul:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` este jumătatea lizibilă de către mașină:

| Cheie | Ce numără |
| --- | --- |
| `models` | Grupuri de camere în execuție. |
| `images_in_groups` | Imaginile sursă din aceste grupuri. |
| `targets_found` | Ținte de reflectanță detectate. |
| `images_calibrated` | Imaginile calibrate de execuție. |
| `exported_files` | **Fișiere cu produse imagistice generate de execuție.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Înregistrări ale senzorului de lumină, numărate separat în mod intenționat — acestea provin dintr-o etapă diferită etapă și există chiar și pentru rulări fără imagini deloc, așa că includerea lor ar face ca o rulare doar cu DAQ să pară că a exportat imagini. |

Alături de acestea: `summary["output_dirs"]` (fiecare director în care s-a scris),
`summary["light_sensor_export"]`, `summary["stopped"]` (valabile atunci când utilizatorul a întrerupt
rularea, astfel încât numărările parțiale să nu fie interpretate ca o rulare finalizată care a produs prea puțin) și
`summary["groups"]` (defalcarea pe grupuri).

`exported_files` este înregistrat de pipeline **pe măsură ce scrie**, nu este extras ulterior din
obiectele de imagine ale proiectului. Strategiile paralele și GPU își construiesc propriile
(în subprocese de lucru pentru căile GPU), astfel încât vechea scanare raporta
`0 file(s) written` pentru fiecare astfel de execuție și apoi emitea indicația de exporturi zero — la execuțiile
în care totul funcționase. Dacă creați un script pe baza acestui număr, o execuție paralelă fără erori
raportează acum un număr diferit de zero.

Săririle senzorului de lumină raportează motivul pe care cititorul l-a stabilit efectiv pentru fiecare fișier — o
schemă ilizibilă, un pachet lipsă, o eroare de scriere — **deduplicate**, astfel încât douăzeci de fișiere
omise din aceeași cauză sunt interpretate ca o singură cauză, în loc de douăzeci de repetări ale acesteia.

> **`process()` nu se declanșează atunci când o execuție nu produce imagini.** Acesta este singurul aspect în care SDK și
> CLI diferă în mod deliberat: `chloros-cli process` tratează „au fost solicitate produse, dar niciunul nu a fost
> scris” ca pe o eroare și se închide cu un cod de ieșire diferit de zero, în timp ce SDK se închide normal și raportează
> condiția prin intermediul `summary` / hints. Dacă pipeline-ul dvs. se oprește la o execuție goală, verificați-l
> personal — inspectați `summary` (sau numărați fișierele din folderul proiectului) în loc să vă bazați pe
> absența unei excepții. Cauzele obișnuite sunt un folder de intrare care nu a fost recunoscut ca
> captură și produse omise ca fiind inaplicabile pentru camerele prezente (de exemplu, radianța de la camerele RGB - numai
> camerele).

### Funcții de utilitate

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Valori acceptate

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Ieșire radiometrică (fluxul de procesare multispectral LATTICE)

Nivelul de export multispectral LATTICE (M3C/M3M) al fluxului de procesare `process` — `reflectance` (implicit), `radiance`, `sensor-response` sau `all` (fiecare mod aplicabil pentru fiecare imagine) — corespunde setării de procesare **„Radiometric output”** a proiectului. `configure()` are un cuvânt-cheie dedicat:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Soluția de rezervă avansată — scrierea cheii `"Radiometric output"` a proiectului prin `custom_settings` — funcționează în continuare, dar rețineți că înlocuiește întregul bloc de setări (vezi avertismentul de mai jos):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (implicit) împarte radianța camerei la **fluxul descendent DAQ corelat cu timestamp-ul**, determinat automat dintr-un fișier `.daq` (DAQ-U/M/E) înregistrat**sau un `.csv` nativ DAQ-M**găsit alături de imagini; orice pachet de calibrare per cameră sau per DAQ care lipsește local este**preluat automat din AWS** la prima utilizare. CLI expune acest lucru sub formă de comutatoare de produs per tip pe `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **înlocuiește** întregul bloc de setări calculate (ocolește, conform proiectării, celelalte cuvinte-cheie și validarea din `configure()`). Când îl utilizați, includeți fiecare cheie `Project Settings` care vă interesează, așa cum se arată în exemplul de mai sus.

---

## Smart-Connect pentru camerele LATTICE

Sesiuni backend persistente pentru hardware-ul live. Se utilizează aceleași puncte finale ca și în GUI, astfel încât comportamentul este identic pe SDK / CLI / GUI.

### O singură cameră — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Semnătura `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Metode

| Metodă | Descriere |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Citește nodurile GenICam; returnează `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Scrie noduri după nume prietenos (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Capturează un **cadru**. Returnează o listă cu un singur element conținând dicționare cu metadate ale cadrului. (Captura în rafale/cu mai multe cadre a fost eliminată — apelați `capture()` într-o buclă dacă aveți nevoie de o serie.) |
| `disconnect()` | Eliberare din pool. Nu produce nicio operație dacă ne-am atașat la o sesiune deja deschisă. |

`capture()` controale de export (același model ca și matricea + GUI):

- `processing` / `levels` — `processing="all"` salvează toate tipurile de export aplicabile; `levels=["raw","radiance"]` salvează doar acelea (suprascrie `processing`). Omiteți ambele pentru setarea implicită a backend-ului.
- `force_daq=True` — salvează valoarea citită de DAQ/DLS ca fișier sidecar `.daq` chiar și în cazul unei capturi doar în format brut, astfel încât cadrul să poată fi reprocesat ulterior în reflectanță/indice. Nu are efect dacă nu este conectat niciun DAQ.

### Matrice sincronizată — `ArraySession` (Smart-Prep)

`connect_array` este **punctul de intrare recomandat** pentru configurațiile cu mai multe camere. Acesta execută în fundal fluxul complet de pregătire inteligentă (smart-prep) al interfeței grafice:

1. **Analiza rețelei** (`/api/camera/array/recommend`) — identifică cea mai mare dimensiune de cadru care se încadrează în nivelul sim-emit fără a pierde cadre.
2. **Selectare automată a nivelului** — `sim-capture-sim-emit` dacă cablul suportă acest lucru; în caz contrar, `sim-capture-ftd-stagger` sau `slip-emit-and-capture`.
3. **Reducere automată**— reduce în mod silențios dimensiunea cadrului / crește gruparea (binning) atunci când canalul de comunicație nu poate susține rezoluția solicitată.**Această măsură de siguranță nu acoperă suprasubscrierea agregată**: un număr prea mare de camere pentru cablu nu poate fi remediat prin reducerea dimensiunii cadrelor — vezi [Suprasubscriere](#over-subscription-the-per-cam-floor).
4. **PTP activat** în mod implicit — marcajele de timp între camere sunt comparabile cu o precizie de microsecunde.
5. **Selectare automată a formatului de pixeli pentru fiecare cameră** — camere „RGB” → `BayerRG8`, multispec → `BayerRG12`.
6. **Inițializare AE** — înregistrează starea AE curentă a fiecărei camere, astfel încât conectarea să nu reseteze expunerea în timpulzbor.
7. **Configurarea declanșării GPIO** — `connect_array` activează fiecare cameră (`TriggerMode=On`, `TriggerSource=Line2`), astfel încâtsă acționeze camerele slave prin cablul M8. Aceasta este o etapă valabilă doar pentru matrice: o singură cameră pornită cu `LatticeCamera` funcționează în schimb în mod liber.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Semnătură

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Valori `force_tier`:
- `"sim-capture-sim-emit"` — simultan adevărat (toate camerele declanșează la aceeași flanc de ceas).
- `"sim-capture-ftd-stagger"` — eșalonare flexibilă în timp(came-urile emit la momente ușor decalate, astfel încât pachetele se serializează pe magistrală).
- `"slip-emit-and-capture"` — captură secvențială pe cameră (fără sincronizare temporală; singura opțiune când nicio dimensiune de cadru nu se potrivește cu sincronizarea).

`wire_ceiling_mbps` suprascrie **bugetul de lățime de bandă susținută al gazdei** în MB/s — singura
valoare de care depinde întreaga alocare a matricei. Lăsați-o la `None` pentru a utiliza valoarea detectată automat
. Reduceți-o când matricea raportează cadre corupte GVSP: valoarea automată este derivată
din rata de legătură anunțată de placa de rețea, care supraestimează adaptoarele USB, de benzile PCIe subdimensionate și de
structurile partajate supraîncărcate — iar această supraestimare se manifestă sub formă de cadre corupte, mai degrabă decât ca o
conexiune vizibil lentă. Valoarea este stocată în blocul de captură a matricei din proiect, astfel încât o
redeschidere sau un `connect_array` ulterior o restabilește ca orice altă setare a matricei.
Consultați [Starea matricei](#array-health--which-subsystem-is-losing-frames).

#### Supra-abonare (limita minimă per cameră)

Reglarea ritmului de emisie simulată alocă fiecărei camere o parte din bugetul de bandă protejat împotriva coliziunilor, cu o limită minimă de **8 MB/s per cameră**(`per_cam_floor_bps`). Odată ce `N × floor` depășește limita maximă de protecție împotriva coliziunilor, matricea**supra-alocă lățimea de bandă**— modul de eșec este pierderea de pachete GVSP, nu o rată de cadre mai mică — și nu există nicio soluție legată de dimensiunea cadrului:**binning-ul și ROI reduc numărul de octeți pe cadru, nu numărul de octeți pe secundă**pe care îl compară verificarea agregată. Limite practice la rezoluție maximă pe o gazdă 1 GbE:**6 camere la 1500 MTU, 9 cu cadre jumbo** (`max_cams_collision_safe` din răspunsul analizei indică limita maximă pentru conexiunea dvs.). Soluții: mai puține camere, cadre jumbo de la un capăt la altul sau o placă de rețea mai rapidă.

- Răspunsurile `analyze_array_network()` și `/api/camera/array/connect` conțin `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` și `per_cam_floor_bps`. Când `oversubscribed` este adevărat, proiecția **resetează la zero câmpurile fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`), în loc să raporteze o rată înșelătoare, de tipul „lent, dar funcțional”.
- `POST /api/camera/array/connect` acceptă un parametru de corp `pin_resolution` (**numai HTTP — nu un argument kwarg de tip SDK**; `connect_array` nu îl expune). Fixarea elimină plasa de siguranță a reducerii treptate a binning-ului, astfel încât o conexiune suprasubscrisă cu `pin_resolution` setat este**refuzată categoric** cu o eroare care specifică fiecare soluție. Fără fixare, conectarea continuă cu reducerea treptată, dar avertizează că reducerea nu poate elibera agregatul.
- Soluție de rezervă pentru testare: setați `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` în mediul backend-ului pentru a reduce refuzul la un avertisment puternic — vă conectați oricum și acceptați pierderea de pachete.

#### Starea matricei — care subsistem pierde cadre

`GET /api/camera/array/<array_id>/capability` conține un bloc activ `health` pe o
matrice conectată, reevaluat într-o fereastră **de 10 secunde** rulantă. Acesta împarte pierderea de cadre
în două cauze care necesită remedieri opuse, în loc de o singură rată de „incompletitudine” care
nu le identifică pe niciuna dintre ele:

| Câmp | Ce înseamnă | Ce subsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pe serial) | Cadrul **a sosit și era defect din punct de vedere structural**— pierdere de pachete GVSP. |**Rețea**: lățime de bandă, ritm de transmisie, inelul de recepție al plăcii de rețea, MTU |
| `never_arrived_rate_pct` (pe serial) | Cadrul **nu a sosit deloc**— camera nu s-a declanșat sau nu a transmis nimic. |**Declanșare / sincronizare**: cablu M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Cea mai slabă rată a camerei pentru fiecare. | — |
| `per_cam_rate_pct` | Rata combinată de funcționare incompletă pe cameră (ambele cauze împreună). | — |
| `stable_for_seconds` | Cât timp a rămas fiecare cameră sub 0,01 %. | — |

Alături de `health`, același raport indică valoarea totală a alocării rămase neutilizată:

| Câmp | Ce înseamnă |
| --- | --- |
| `wire_ceiling_mbps` | Bugetul de lățime de bandă susținut al gazdei, MB/s. |
| `wire_ceiling_source` | De unde provine acea valoare, în cuvinte — de exemplu, `USB-capped 200 MB/s (was theoretical 1062; …)` sau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` atunci când `wire_ceiling_mbps=` l-a setat. |
| `nic_is_usb` | `true` pentru un adaptor USB Ethernet. |

Nu există un wrapper „SDK” pentru acest punct final — citiți-l direct:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Citire:** o valoare diferită de zero pentru `gvsp_corrupt_rate_pct`, cu `never_arrived_rate_pct` la 0, înseamnă că
că declanșarea și sincronizarea cablului sunt perfecte, iar 100 % din pierderi se află pe traseul rețelei — reduceți
`wire_ceiling_mbps` și reconectați-vă. Modelul invers indică mai degrabă cablul de sincronizare sau
linia de declanșare.

> **`target_fps` nu este factorul determinant pentru cadrele corupte.** Ritmul GevSCPD este setat o singură dată la
> conectare, astfel încât reducerea ratei de declanșare modifică ciclul de lucru și nu
> rata de emisie simultană a rafalei . O reducere măsurată de 5× a cererii nu a produs nicio îmbunătățire, în timp ce
> scăderea limitei maxime a cablului de la 240 la 200 MB/s a dus aceeași configurație de la 10,4 % cadre corupte la
> 0,00 %.

> **Reducerea automată în timpul transferului nu este disponibilă pe firmware-ul TRI032S.** O matrice în funcțiune nu poate
> remedia singură această problemă; deconectați-o și reconectați-o, astfel încât programul de planificare a timpului de conectare să se adapteze la
> noua limită maximă.

Un **adaptor USB Ethernet este limitat la 200 MB/s** de către sondă, indiferent de
specificațiile sale: tabelul de eficiență care transformă rata de legătură într-o valoare susținută este
derivată din PCIe, iar o placă de rețea USB anunță rata de legătură Ethernet, fiind totuși limitată de
busul USB și de driverul său. Limita este absolută, nu o fracțiune — un adaptor USB 1 GbE
produce ~80 MB/s și nu este afectat.

#### Metode `ArraySession`

| Metodă | Descriere |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Un grup de capturi sincronizate. Returnează un `CaptureResult` (listă de dicționare de cadre + `.skipped`). Comenzi de export mai jos. |
| `capture(..., smart=True)` | **Captură inteligentă** — așteaptă stabilizarea AE pe toate camerele, apoi se declanșează. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Captură rapidă: numai date brute + citirea DAQ atribuită (+ indexul combinat liber). Reflectă butonul „Captură rapidă” din interfața grafică. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | O singură captură / Continuă / La intervale într-o buclă limitată. Returnează `list[CaptureResult]`.**Necesită `count` și/sau `duration_s`** pentru a se termina (SDK-ul nu acceptă Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Pornește înregistrarea vizualizării live cu index combinat în format video/GIF → `RecorderHandle`. Un singur înregistrator compozit per matrice. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Pornește o→ `RecorderHandle`. Reprocesare offline cu `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocesare offlineo serie RAW salvată în videoclipuri calibrate. Se blochează până la finalizare (`wait=True`) și returnează `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Interoghează o sarcină de compilare offline: `{running, result, error, burst_dir}`. |
| `disconnect()` | Eliberează întregul array. |

`capture()` comenzi de export (același punct final folosit de GUI/ CLI):

- `processing` / `levels` — `processing="all"` (sau `levels=["raw","radiance",…]`) salvează fiecare tip de export aplicabil pentru fiecare cameră; o singură valoare `processing` salvează doar acel nivel.
- `aligned=True` — aliniază exportul non-raw al fiecărui element la [profilul de aliniere](#array-alignment) al matricei (co-înregistrat); datele brute rămân nealiniate, dar transportă transformarea în metadate. Se revine la nealiniat (cu un avertisment afișat în `alignment` al rezultatului) dacă matricea nu are profil.
- `render_index=False` — omite suprapunerea indicelui de vegetație pentru fiecare cameră; implicit, aceasta este redată acolo unde este configurată.
- `force_daq=True` — salvează citirea DAQ/DLS atribuită ca fișier sidecar `.daq` chiar și atunci când niciun nivel selectat nu are nevoie de aceasta.

**Compresie TIFF (opțiunea HTTP):**`ArraySession.capture()` nu trimite nicio cheie `compression`, astfel încât se aplică setarea implicită a backend-ului — `POST /api/camera/array/capture` citește un parametru de corp `compression`, `"deflate"` în mod implicit (zlib L1 fără pierderi + predictor orizontal, ~4,1 MB pe cadru la rezoluție maximă). `"none"` scrie necomprimat (~6,3 MB/cadru) cu o**~5× mai rapidă** — ambele sunt fără pierderi și se citesc identic la import. Clasa `SDK` nu expune niciun parametru pentru aceasta; soluția de rezervă este ``chloros-cli lattice array-capture --compression none`` sau `HTTP` în format brut. DEFLATE deține, de asemenea, GIL-ul Python, astfel încât scrierile comprimate nu se paralelizează între firele de scriere per cameră — captarea susținută la rezoluție maximă cu 8 camere la rata senzorului necesită `compression: "none"`. Detalii: [CLI Referință → array-capture](cli-reference.md).**Suprascrieri de export pe membru (numai HTTP):**același punct final acceptă și `exclude_serials` (list — eliminarea membrilor din setul salvat; matricea se declanșează în continuare ca un singur grup sincronizat, iar membrii excluși sunt returnați în `excluded`), `serial_levels` (suprascrieri la nivel de cameră `{serial: [level tokens]}`) și `serial_index` (suprascrieri de suprapunere de index per cameră `{serial: bool}`). Aceștia sunt parametri de corp cu paritate GUI și**nu sunt încă argumente kwargs dSDK**; membrii absenți din hărți revin la `levels` / `render_index` la nivel de matrice.

##### Inspectarea camerelor omise — `CaptureResult.skipped`

`ArraySession.capture()` returnează un `CaptureResult`, care este o subclasă a `list`: iterați-l, indexați-l, aplicați-i `len()` — fiecare model existent continuă să funcționeze. Codul nou poate inspecta atributul `.skipped` pentru a vedea care camere au fost excluse și de ce. Cel mai frecvent caz este cel al camerelRGBe dintr-unde filtre atunci când se solicită `processing="radiance"` sau `"reflectance"` — radianța pe pixel Bayer nu are sens pentru un senzor de bandă largă, așa că backend-ul omite acele camere în loc să genereze date fără sens.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokenurile de motiv urmează modelul `<level>-not-applicable-to-rgb-cam` (o intrare pentru fiecare nivel omis, fiecare conținând `level`). Omisiunile specifice reflectanței sunt `reflectance-skipped-no-fresh-dls` (nu este disponibilă nicio citire recentă a radiației descendente), `reflectance-skipped-bound-daq-unavailable (…)` (nu s-a putut accesa DAQ-ul asociat) și `dls-uncalibrated-band-<nm>` — banda se află în mare parte în afara intervalului calibrat radiometric al senzorului de lumină al DAQ-ului (~374–974 nm), astfel încât divizarea absolută a reflectanței bazată pe DAQ este refuzată, iar cadrul revine în mod evident la răspunsul bazat pe senzor. Dintre codurile de produs disponibile, doar F988 o declanșează; calea acceptată de această cameră este fluxul de lucru cu panoul de reflectanță.

Nivelurile `processing`:

| Nivel | Ieșire |
| --- | --- |
| `"raw"` | Bayer monocanal (camere monocrome: banda unică) direct de la senzor. |
| `"debayered"` *(implicit SDK)* | BGR cu 3 canale prin demosaic biliniar (camere monocrome: 1 canal în tonuri de gri). |
| `"radiance"` | float32 W/m²/sr/nm prin lanțul radiometric complet. Numai multispectral — camerele „RGB” sunt omise. |
| `"reflectance"` | uint16 0..32768 (compatibil cu Pix4D-ready); necesită o asociere DAQ activă pentru referință absolută. Numai multispectral. |
| `"display"` | Lanț complet care corespunde previzualizării din GUI (CCM + WB + gamma conform profilului camerei). |
| `"all"` | **Un fișier pentru fiecare nivel aplicabil** pentru fiecare cameră (corespunzător setării implicite „Capture All” / CLI din interfața grafică). Fișierul returnat `CaptureResult` conține apoi un dicționar de cadre pentru fiecare `(cam, level)`, cu nivelul specificat în fiecare dicționar; nivelurile inaplicabile apar în `.skipped`. Valorile de citire DAQ utilizate pentru orice cadru de reflectanță sunt salvate ca fișier auxiliar `.daq`. |

> **Notă — valoarea implicită diferă de cea din CLI.** `ArraySession.capture()` are ca valoare implicită `processing="debayered"`; comanda `chloros-cli lattice array-capture` are ca valoare implicită `processing="all"`. Transmiteți explicit `processing="all"` din SDK pentru a oglindi salvarea pe mai multe niveluri din CLI /GUI.

### Moduri de captură și dispozitive de înregistrare

Interfața matricei reflectă panoul de captură din interfața grafică: modurile de declanșare Single / Continuous / Interval / Fastest, plus două dispozitive de înregistrare (video compozit în timp real și rafală raw → reprocesare offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**reprezintă bucla Continuă/Interval a SDK. Deoarece nu există `Ctrl+C` pentru a o întrerupe dintr-un script,**trebuie** să transmiteți `count` și/sau `duration_s` (se oprește când se atinge oricare dintre ele). `interval_s` se măsoară de la începutul fiecărei treceri (în concordanță cu interfața grafică). Restul parametrilor kwargs sunt transmise direct către `capture()`.
- **`record`** este de *nivel de monitorizare*: captează compozitul cu index combinat în timp real, așa cum este afișat, astfel încât fluxul combinat trebuie să fie deschis pentru ca cadrele să fie înregistrate. Un singur înregistrator de compozit per matrice (generează o eroare dacă unul este deja în execuție).
- **`burst` → `build_video`** este de *nivel de analiză*: `burst` scrie cadre brute + un manifest pentru fiecare cadru + un `.daq` pentru fiecare citire DLS distinctă sub `<output>/bursts/<base>/` la rata maximă a buclei de captareviteză maximă a buclei de captură (fără lanț, fără exiftool, fără vizualizare live). `build_video` sincronizează temporal fiecare cadru cu cel mai apropiat `.daq` și rulează din nou lanțul de radianță/reflectanță/indice al fluxului de import. `products` este o listă de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (implicit: indicele combinat). `burst().stop()` inițiază automat și o generare a indicelui combinat cu efort maxim, returnată ca `build_job` în rezultatul de oprire.

#### `RecorderHandle`

Returnat de `ArraySession.record()` și `ArraySession.burst()`. Utilizați-l ca manager de context pentru a opri automat la ieșirea din domeniul de aplicare sau controlați-l manual.

| Membru | Descriere |
| --- | --- |
| `job_id` | ID-ul sarcinii backend (șir). |
| `kind` | `"composite"` (din `record`) sau `"raw"` (din `burst`). |
| `start_stats` | Dicționarul returnat de apelul `start`. |
| `result` | `None` în timpul rulării; dicționarul cu rezultatul final al opririi odată ce s-a oprit. |
| `stats(timeout=10.0)` | Statistici în timp real ale sarcinii (cadre scrise, fps realizat, timp scurs). |
| `stop(timeout=60.0)` | Oprește înregistratorul; returnează și memorează în cache rezultatul final. Idempotent (o a doua apelare returnează rezultatul memorat în cache). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Conectarea la un array deja conectat — `attach_array`

Dacă matricea este deja activă (a fost deschisă de interfața grafică sau o sesiune anterioară de SDK a apelat `connect_array`), utilizați `attach_array` pentru a obține un identificator al acesteia, în loc să o reconectați-conectare. `connect_array` generează întotdeauna eroarea „Camera  se află<sn> deja în matrice <id>” în acea situație, deoarece trimiterea comenzii POST către `/array/connect` pentru un membru din pool nu este idempotentă; `attach_array` citește `/api/camera/array/list` și face potrivirea fie după array_id, fie după seriale.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Model: scripturile SDK care co-cu interfața grafică a desktopului ar trebui să încerce mai întâi `attach_array` și să recurgă la `connect_array` dacă nu există încă nicio matrice în grup.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Important — ieșirea din context-manager DUCE la deconectare.**`ArraySession.disconnect()` trimite întotdeauna un POST către `/array/disconnect`; nu există o protecție de tip „atașat, dar nu deținut”, așa cum există pentru `CameraSession` / `DAQSensorSession`. Dacă partajați spațiul cu GUI-ul și nu doriți să dezactivați matricea la ieșirea din domeniul de aplicare,**nu utilizați blocul `with`** — păstrați identificatorul într-o variabilă obișnuită și omiteți `disconnect()` explicit:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Asistent pentru analiza rețelei

Util înainte de a deschide matricea — estimează dacă setările propuse se vor potrivi:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` este unul dintre `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (altfel `error`). `auto_capped_fps` înseamnă că rezoluția solicitată se potrivește cu inelul RX doar la o rată de declanșare limitată — păstrați rezoluția și treceți de la `target_fps=result["recommended"]["recommended_target_fps"]` la `connect_array` (vezi [Exemplul 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Cum se interpretează proiecția** (același model ca în panoul „Setări matrice” din interfața grafică):

- **Rafala (`frame_bytes_total`) se însumează pentru fiecare cameră în parte, la formatul real de pixeli al fiecărei camere.**Camerele mono**M3M**transmit Mono12 (2 biți/pixel) indiferent de valoarea `pixel_format` pe care o transmiteți, astfel încât un cadru la rezoluție maximă cu 4 camere are**~25 MB** cu trei camere mono, nu ~12,6 MB, cum ar rezulta din ipoteza că toate sunt pe 8 biți. Backend-ul determină formatul fiecărei camere pe baza modelului acesteia.
- **Admittance (`burst_fits_nic_ring`) ține cont de capacitatea de descărcare**, nu se bazează pe comparația „burst complet vs. inel”: emisia de simulare se potrivește atunci când gazda golește inelul RX mai repede decât îl umplu camerele. O gazdă de 10G + camere de 1 GbE**admite** rezoluția completă chiar și atunci când burst-ul depășește inelul; o gazdă de 1 GbE blochează (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` reprezintă un plafon conservator de recuperare serială** — `max(readout+emit, N×emit)` cu transmisia per cameră limitată la legătura de cameră de 1 GbE, independent de expunere. De exemplu, ~2,8 fps pentru o matrice de 4 camere la rezoluție maximă de 12 biți (corespunde valorilor măsurate în timpul rulării de ~2,7–3,0). Model complet: [Referință CLI → Modelul fps și rafale pentru matrice](cli-reference.md#array-fps--burst-model).
- **Supra-abonarea (`oversubscribed: true`) înseamnă că pragul minim de N × pe cameră depășește** — câmpurile fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) afișează valoarea 0, iar reducerea automată/gruparea nu pot remedia problema (acestea reduc numărul de octeți pe cadru, nu numărul de octeți pe secundă). Soluțiile constau în reducerea numărului de camere, utilizarea cadrelor jumbo sau o placă de rețea mai rapidă; `max_cams_collision_safe` raportează limita maximă (6 camere la rezoluție maximă pe 1 GbE la 1500 MTU, 9 cu cadre jumbo). Răspunsul conține, de asemenea, `aggregate_demand_bps`, `collision_safe_ceiling_bps` și `per_cam_floor_bps` (8 MB/s). Consultați [Supra-abonare](#over-subscription-the-per-cam-floor).

### Descoperire și listare

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Matricile LATTICE rulează AE continuu în fundal imediat ce sunt conectate, dar o scenă nou orientată necesită un moment pentru a converge. **Smart-Capture** este soluția convenabilă integrată: interoghează expunerea fiecărei camere, așteaptă până când matricea este stabilă pe o fereastră, apoi declanșează capturarea. Este echivalentul din interfața grafică: butonul de captură „inteligentă” al aplicației pentru desktop apelează același punct final din backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Când utilizați `ChlorosProject` (secțiunea următoare), aveți la dispoziție mai multe opțiuni de reglare:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Politica smart-AE este conservatoare în mod implicit. Strângeți `exposure_tolerance_pct` pentru lucrări radiometrice exigente; lărgiți-o pentru scene care se schimbă rapid, unde doriți doar „suficient de aproape”.

---

## Sesiuni de senzori DAQ

Pool de backend persistent pentru senzori spectrali (DAQ-U prin USB, DAQ-M prin BLE, DAQ-E prin Ethernet). Reflectă suprafața camerei: detectare inteligentă, reutilizarea grupului, atașare idempotentă.

### Detectare inteligentă (fără configurare)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritate: Ethernet → BLE → USB. Transmiteți orice indiciu explicit pentru a fixa transportul.

### Transport fixat

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Metode `DAQSensorSession`

| Metodă | Descriere |
| --- | --- |
| `status(timeout=10.0)` | Rezumatul intrării din pool (starea de streaming/înregistrare, intervalul de lungimi de undă, SHA de calibrare, timpul de integrare, frame_avg, starea AE). |
| `latest(n=1, timeout=10.0)` | Returnează până la N cadre de spectru cele mai recente. |
| `stream_start()` / `stream_stop()` | Reluare / pauză streaming (mânerul rămâne deschis). |
| `record_start(output_dir=None, device_name=None)` | Pornește înregistrarea unui fișier .daq. Returnează calea fișierului. Refuză pentru DAQ-U/M fără un pachet de calibrare AWS (DAQ-E este exceptat). |
| `record_stop()` | Oprește înregistrarea. Returnează `{path, rows}`. |
| `disconnect()` | Eliberare din pool. Fără efect pentru mânerele atașate, dar care nu sunt deținute. |

> **Profilurile de corecție a capacității (`cap_id`) nu sunt un buton „SDK”.** `connect_daq_sensor()` / `DAQSensorSession` nu expun niciun parametru `cap_id` sau metodă `set_cap`. Selectați un profil deprin intermediul CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) sau prin rutele HTTP ale backend-ului (`/api/daq`, `/api/daq/connect` și `/api/daq/<id>/cap-id` acceptă `cap_id`).

### Descoperire — găsirea unei adrese la care să te conectezi

`discover_daq_sensors()` scanează USB / BLE / ETH în căutarea senzorilor pe care *ați putea* deschide. Este echivalentul DAQ al lui `discover_lattice_cameras()` și singura modalitate de a obține **adresa MAC BLE a unui DAQ-M** — un DAQ-E are un nume de gazdă, iar un DAQ-U un port COM, dar adresa MAC nu este imprimată pe dispozitiv și nici listată de sistemul de operare.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Câmp | Descriere |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / MAC BLE / nume gazdă — se transmite către `connect_daq_sensor` ca `port=` / `mac=` / `eth_host=`. |
| `display` | Etichetă lizibilă pentru om. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, sau `None` pentru un port pe care scanarea nu îl poate identifica (adaptoarele seriale USB nu pot fi distinse fără o sondă, astfel încât elementele necunoscute sunt afișate, nu ascunse). |
| `extra` | Detalii specifice fiecărui tip de transport (numele anunțat BLE, producătorul USB, IP/firmware DAQ-E/…). Valorile goale sunt omise. |

| Parametru | Implicit | Descriere |
| --- | --- | --- |
| `transports` | toate cele trei | Secvență (sau șir CSV) care limitează scanarea. Merită specificată când știi ce vrei — BLE este partea lentă. |
| `scan_timeout` | 5 | Fereastra de scanare pentru fiecare tip de transport, exprimată în secunde; backend-ul limitează valoarea la 1–20. |
| `timeout` | 60,0 | Limita maximă „HTTP” pentru întregul apel (la fel ca în restul „SDK”). |
| `auto_start_backend` | `True` | Pornește un backend local dacă nu rulează niciunul. Nu pornește niciodată pentru un `backend_url` la distanță. |

> **Senzorii deja deschisi în grup nu apar.** Un periferic BLE conectat încetează să mai transmită anunțuri, iar un port COM deschis nu poate fi detectat, astfel încât funcția de descoperire listează ceea ce este *disponibil pentru conectare*. Este de așteptat un rezultat gol imediat după ce ați conectat ceva — utilizați `list_daq_sensors()` pentru ceea ce dețineți deja. Transporturile a căror scanare nu poate fi executată (nu este instalat bleak / zeroconf) sunt omise în loc să genereze o eroare, astfel încât o mașină fără Bluetooth primește totuși răspunsurile USB și ETH.

### Listare

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Co-utilizare cu GUI / CLI

Dacă GUI-ul are deja un senzor deschis, apelarea `connect_daq_sensor(port="COM3")` din Python returnează un identificator marcat cu `already_connected=True`. `disconnect()` al sesiunii este atunci o operație fără-op, astfel încât scriptul dvs. SDK să nu smulgă senzorul din GUI la ieșirea din program.

### Clasele de hardware direct (fără backend)

`daq_sdk` este reexportat de `chloros_sdk`, astfel încât să puteți controla senzorii de la un capăt la altul în cadrul procesului, fără backend:

> **Disponibilitate:**`daq_sdk` este inclus în instalarea desktop a Chloros,**nu** cu pachetul PyPI — `pip install chloros-sdk` vă oferă `lattice_sdk`, dar omite `chloros_sdk.DAQ_AVAILABLE == False`. Verificați acest indicator înainte de a utiliza aceste clase; pe o gazdă care utilizează doar pip, controlați senzorul prin [`connect_daq_sensor()`](#daq-sensor-sessions), care nu necesită biblioteci locale de transport.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Preferați calea smart-connect (`connect_daq_sensor`) atunci când doriți proprietate partajată cu GUI; utilizați clasele directe pentru scripturi fără interfață grafică care dețin senzorul în mod exclusiv.

---

## Automatizarea proiectului — `ChlorosProject`

Un proiect „Chloros” salvat este un folder care conține `cameras.json` + `sensors.json` + `project.json`. `open_project` încarcă manifestul, iar `connect_all` conectează online fiecare dispozitiv salvat cu setările sale salvate — aceeași stare a hardware-ului pe care ar produce-o interfața grafică.

### Exemplu minim

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Sau ca manager de context:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metodele `ChlorosProject`

| Metodă | Descriere |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Descoperă și conectează fiecare dispozitiv salvat. Returnează un raport de conectare pentru fiecare clasă. Utilizează un backend activ atunci când unul ascultă pe `127.0.0.1:5000`; în caz contrar, revine în mod silențios la controlul direct (fără backend) al dispozitivelor `lattice_sdk` — nu generează niciodată un backend. |
| `disconnect_all()` | Închide totul. |
| `capture_all(output_dir=".")` | Un cadru de la fiecare cameră + matrice + spectru de la fiecare senzor. |
| `stream(camera, overlays=False, fps=10.0)` | Generator care produce cadre BGR `numpy` de la o cameră (sau matrice) numită. `overlays=False` este o buclă directă de captură `lattice_sdk` (matricea generează dicționare `{serial: frame}`). `overlays=True` se direcționează prin `ChlorosLocal.camera_stream()` → fluxul M, cu blocul `ui.overlay` salvat de cameră transmis ca parametri de interogare. Necesită modul backend și o **cameră autonomă**: o cameră în modul direct generează `RuntimeError` (backend-ul poatepoate prelua o cameră deținută de acest proces), iar un array generează `NotImplementedError` (suprapune compoziția pentru fiecare cameră — transmite un element după nume). Echivalentul unei operațiuni unice: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Execută alinierea pe fiecare matrice conectată în prezent. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Rulează pipeline-ul de calibrare/indexare pe imaginile proiectului (încorporează `ChlorosLocal.process`; aceste patru sunt **singurele** argumente acceptate — `indices=` etc. generează eroarea `TypeError`; se setează indicii prin `ChlorosLocal.configure()`). Construiește în mod leneș un `ChlorosLocal()`, care pornește automat un backend. |

Atribute:
- `proj.cameras` — `Dict[str, CameraHandle]` indexat după nume ȘI număr de serie.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexat după nume ȘI array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexat după nume ȘI slot_id.
- `proj.config` — dicționarul `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Niveluri de procesare.** `capture()`, `grab()` și `frame_stream()` acceptă toate același token `processing`,
iar lanțul este cumulativ — fiecare nivel execută tot ce se află deasupra lui:

| Nivel | Ieșire | Note |
| --- | --- | --- |
| `raw` | Bayer cu 1 canal, nativ pentru senzor | Fără demosaic. Suprapunerile nu sunt disponibile la acest nivel. |
| `debayered` | BGR cu 3 canale (**implicit**) | Demosaic biliniar. Singurul nivel care funcționează fără modul backend. |
| `radiance` | float32, W/m²/sr/nm | Lanț radiometric complet: demosaic + separare 3×3 (multispectral) + DSNU + câmp plat + scală NIST, cu expunerea × amplificarea eliminate astfel încât valorile să fie absolute. |
| `reflectance` | uint16, 32768 = 1,0 | Radianța împărțită la iradianța descendentă (ρ = π·L/E). Necesită o citire DLS/DAQ — vezi nota de mai jos. |
| `display` | 8 biți, similar sRGB | Redare echivalentă cu interfața grafică: CCM + balans de alb + gamma prin profilul de culoare activ al camerei. |

Orice altceva în afară de `debayered` necesită modul backend; o cameră în modul direct generează
`NotImplementedError`. `reflectance` are nevoie de o citire utilă a fluxului descendent — punctul final al cadrului trage
DAQ-ul grupat în slotul DLS al camerei automat, dar fără un DAQ asociat, lanțul refuză
ieșirea de reflectanță și marchează sincer retrogradarea în metadatele returnate, în loc să
returneze un produs de calitate inferioară.

> **Scala DN de reflectanță — nu o codificați rigid.** Reflectanța LATTICE utilizează `32768` = ρ 1,0 și înregistrează
> XMP `Chloros:PixelScale=32768`; reflectanța Survey3 utilizează `65535` = ρ 1,0 și nu conține
> etichete `Chloros:*`. Citește eticheta și împarte la valoarea acesteia. Este definită în domeniul uint16, deci rămâne
> `32768` pentru fiecare format care redimensionează (TIFF pe 16 biți, PNG /JPG, 32 de biți procent) — normalizează
> mai întâi tipul de date stocat înapoi la uint16 (×257 din 8 biți, ×65535 din float). Singura excepție:
> o captură cu sursă de 8 biți scrisă ca 8 biți TIFF este *tăiată*, nu este redimensionată, deci nicio scală nu o descrie
> — Chloros omite complet `PixelScale` și tuplul MicaSense în acest caz. Tratați o
> etichetă lipsă dintr-un fișier de reflectanță LATTICE ca „fără scală validă”, nu ca o valoare implicită.

> **EXIF este preluat în export.** `process()` copiază blocul GPS al capturii sursă
> **și ExifIFD-ul acestuia** în fiecare produs, astfel încât exporturile conțin `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` și `CameraSerialNumber`, precum și
> georeferențierea. `FocalLength` este elementul pe baza căruia Pix4D calculează distanța dintre eșantioanele la sol — fără acesta,
> reconstrucția revine la o scară extrem de eronată (un caz măsurat a transformat un amplasament de 411 m
> într-unul de 47,8 km). Copia nu este în mod deliberat `-all:all`: etichetele structurale ale IFD0 întrerup
> ieșirea LATTICE, iar `ExifImageWidth`/`Height` sunt excluse deoarece descriu
> capturarea sursei, nu rasterul exportat.

Sub-indicatori ai etapei de captură (se aplică nivelurilor radiometrice — `radiance`, `reflectance`, `display`):

| Indicator | Implicit | Semnificație |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + câmp plat + separare 3x3 + scală radiometrică NIST. |
| `apply_white_balance` | `True` | LUT pentru balansul de alb. Ține cont de DLS atunci când un DAQ este asociat camerei. |
| `apply_index` | `False` | Evaluarea indicelui de vegetație. |
| `index_expression` | `None` | Formulă de suprascriere. Dacă nu este goală → activează automat indicele. |
| `annotated` | `False` | Suprapunere decorații GUI (zebra/grilă/vârfuri). Indisponibil pentru `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Tipul de returnare este `CapturePathMap`, nu `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` este `Dict[str, Union[str, List[str]]]`: o singură-nivel
> `processing` atribuie fiecărui serial o singură cale, în timp ce unul pe mai multe niveluri (`"all"` sau o
> listă explicită `levels`) îi atribuie **lista ordonată** a fiecărui produs salvat pentru acea
> cameră. Un compozit combinat în timp real, dacă ar fi transmis în flux, ajunge sub cheia suplimentară
> `"combined"`, mai degrabă decât sub un serial. Codul care presupune `str` generează o eroare în
> forma de listă fără ca vreun verificator de tip să obiecteze — adnotarea indica `Dict[str, str]`
> pentru o perioadă după lansarea formei de listă, motiv pentru care există aliasul. Normalizați
> atunci când doriți forma plată:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Alinierea matricilor

`ArrayHandle` expune suprafața completă de aliniere. Profilele sunt, în mod implicit, valabile doar pentru sesiune — apelați explicit `export_alignment()` pentru a le păstra.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Aliniere la conectare

`connect_all(align=...)` poate alinia automat fiecare matrice la conectare:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Se revine la `project.json["config"]["auto_align_on_connect"]` dacă nu este specificat altceva.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Hardware direct (fără backend)

Când doriți o dependență zero de backend (CI, roboți fără interfață grafică, sisteme încorporate), importați direct `lattice_sdk` și `daq_sdk` — ambele sunt reexportate de `chloros_sdk`. Verificați `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` se află în pachetul PyPI (dar necesită prezența motorului de execuție Arena SDK), în timp ce `daq_sdk` este livrat doar cu instalarea pentru desktop.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Presetările și declanșatorul

Trei dintre cele patru presetări sunt de tip **free-run**: camera expune continuu, iar
`capture()` returnează următorul cadru. `triggered` este excepția — acesta armează
camera pentru un semnal de margine hardware pe linia 2, astfel încât nu capturează nimic până când nu apare unul.

| Presetare | Declanșator | Se utilizează când |
| --- | --- | --- |
| `default` | funcționare liberă | utilizare generală |
| `high_speed` | funcționare liberă | 8 biți, limită de 60 fps, expunere scurtă |
| `high_quality` | funcționare liberă | 12 biți, fără limită de fps — alegerea obișnuită pentru fotografii |
| `triggered` | **pregătită, Linia 2** | camera este conectată la un și este declanșată de un alt dispozitiv |

Dacă alegeți `triggered` (sau setați manual `trigger_mode="On"`) fără ca nimic
să acționeze Linia 2, fiecare `capture()` va expira — în mod corect, deoarece ați cerut
cameră să aștepte. Mesajul de eroare „SDK” explică acest lucru atunci când se întâmplă; vezi
[SC_ERR_TIMEOUT în timpul capturii](#direct-hardware-backend-free).

> **Notă — Mesajele „GVSP probe” / `SC_ERR_TIMEOUT -1011` la conectare nu sunt erori.**&gt; La conectare, SDK încearcă să negocieze**cadre jumbo** (pachete GVSP de 9000 de octeți) pentru un debit mai mare. Pe o legătură directă-la-punct (de exemplu, o adresă `169.254.x.x` locală de legătură), rețeaua nu poate transporta de obicei cadre jumbo, astfel încât această sondare expiră și înregistrează linii precum:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Aceasta este **soluția de rezervă prevăzută**: SDK revine automat la pachetele standard de 1500 de octeți, iar camera continuă să se conecteze în mod normal (liniile `[chunk-enable …]` care urmează fac parte din secvența normală de conectare). Captura funcționează în continuare.
>
> Puteți sări peste această probă, dar **nu este doar un instrument de reducere a jurnalelor — dezactivează cadrele jumbo.** Camera răspunde la ping-urile „Don&#x27;t-Fragment” doar până la 1500 de octeți, indiferent de cât de bună este rețeaua dvs., așa că testul de ping singur nu poate detecta niciodată cadrele jumbo; această probă este singura care poate face acest lucru. Dezactivați-o și camera va transmite pentru totdeauna pachete standard de 1500 de octeți, pe orice rețea:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Merită să o folosiți doar pe o rețea despre care *știți* că nu poate suporta pachete jumbo, unde economisește aproximativ o secundă din timpul de conectare pentru fiecare cameră. Deoarece este o schimbare reală, nu una doar cosmetică, SDK vă informează acum despre acest lucru când o utilizați:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nu-l modificați decât dacă aveți un motiv întemeiat.** Dacă rămâne activat, la fiecare conectare se reevaluează rețeaua de care dispuneți efectiv: conectați-vă la un switch compatibil cu jumbo, iar următoarea conexiune va detecta automat jumbo, fără a fi nevoie de configurare sau repornire.
>
> Dacă *doriți* lățimea de bandă jumbo, activați jumbo end-to-end (MTU NIC 9000 + un switch care le transmite)sau fixați-l cu `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` când știți că legătura îl suportă — deși este de preferat să folosiți `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` pe bază de comandă decât să-l setați permanent, deoarece o dimensiune fixată omite testarea și oprește adaptarea la rețeaua din fața sa. **Fiecare** dispozitiv din traseu trebuie să transmită pachete jumbo — inclusiv orice splitter sau injector PoE, acesta fiind motivul obișnuit pentru care o configurație care altfel ar fi capabilă să transmită pachete jumbo nu poate face acest lucru.

> **`SC_ERR_TIMEOUT -1011` în timpul `capture()` / `grab*()` este o problemă diferită — aceea este o eroare reală.**&gt; Nota de mai sus se referă doar la `-1011` înregistrată de**sonda de timp de conectare**. Aceeași eroare generată de o**captură** înseamnă că camera s-a conectat corect, dar nu trimite nicio imagine:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Indiciul decisiv este o cameră al cărei canal de *control* funcționează corect — detectarea funcționează, setările și înregistrările `[chunk-enable …]` se efectuează cu succes — în timp ce *fiecare* cadru depășește timpul de așteptare.
>
> **Cauza obișnuită este faptul că camera este setată să funcționeze pe baza unui declanșator hardware.** Cu `trigger_mode="On"` și `trigger_source="Line2"`, camera nu transmite absolut nimic până când nu apare o flanc electric pe cablul de sincronizare M8. Dacă nu aveți niciun cablu care să alimenteze acea linie, fiecare captură așteaptă la nesfârșit. Camera nu este defectă și rețeaua funcționează corect — face exact ceea ce i s-a cerut.
>
> `CameraSettings()` și presetările `default` / `high_speed` / `high_quality` funcționează în mod liber, iar o captură care se declanșează în timp ce este activată oferă o explicație, în loc să afișeze doar un simplu `-1011`. `PRESETS["triggered"]` activează Line2, conform specificațiilor.
>
> Pentru a forța orice cameră să funcționeze în mod liber:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Dacă expiră în continuare cu `trigger_mode="Off"`, camera nu transmite efectiv date — trimiteți-ne jurnalul și `ip link show`.

#### Profiluri de culoare (previzualizare live RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` selectează profilul de culoare al afișajului pentru **previzualizarea live** pe camerele dRGBă (camerele multispectrale ignoră această setare):

| Profil | Semnificație |
| --- | --- |
| `raw` | Ocolește complet lanțul radiometric. |
| `linear` | DSNU + flat + WB, fără CCM, fără gamma. |
| `natural` | Liniar + CCM măsurat + gamma sRGB, doar cu finisajul ieftin (netezirea cromatică + desaturarea zonelor luminoase) — setarea implicită realistă. |
| `enhanced` | `natural` plus finisajul complet cu paritate de hub (eliminarea franjurilor, vibranță, contrast local CLAHE). Aspect mai bogat la aproximativ **dublul costului de finisare pe cadru**, deci o rată de cadre LIVE mai mică. |
| `custom_temp` | `natural`, dar WB fixat la `custom_cct_k` Kelvin (DLS ignorat; limitat la 2000–10000 K la nivelul backend-ului). |

Profilul este un buton de reglare a vitezei/aspectului **destinat exclusiv previzualizării live**: capturile salvate beneficiază întotdeauna de finisajul complet, indiferent de profilul selectat, astfel încât alegerea `natural` pentru a câștiga timp de cadru nu scade calitatea a ceea ce ajunge pe disc. Un profil necunoscut activează `ValueError`; atunci când un backend chloros este accesibil, modificarea este trimisă și către acesta prin POST, astfel încât următorul cadru de previzualizare să o reflecte (utilizatorii direct-SDKi fără un backend primesc totuși modificarea setărilor).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Camere mono (M3M) și `Calibration`

O cameră mono **M3M** (`M3M-<lens>-F<wavelength>`) este monobandă: un singur plan în tonuri de gri, fără mozaic Bayer, fără matrice de interferență spectrală 3×3. `Calibration` o recunoaște și expune un indicator `is_mono`. Reflectanța se aplică în continuare ca obandă (demixarea este matricea identitate), dar calculele multibandă pe o singură cameră generează rezultate valabile, în loc să returneze date fără sens:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Pentru a construi un indice de vegetație folosind echipament monocromatic, combinați mai multe camere M3M la lungimi de undă diferite într-un stack multibandă aliniat (vezi [Alinierea matricei](#array-alignment)) și calculați indicele pe acel stack, în loc să îl calculați pe o singură cameră.

Modul direct DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Chei acceptate pentru `apply_sensor_settings`**— exact `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; învechit în favoarea `cap_id`), `filter_model` (DAQ-M) și `cap_id` (toate tipurile de DAQ; `None`/`""`/`"none"` = senzor simplu, fără corecție de capacitate). Cheile necunoscute sunt**ignorate în mod tacit** — de exemplu, `{"integration_time": 64}` nu face nimic (trebuie să fie `integration_time_ms`). Returnează `{"applied": [...], "errors": {...}}` și nu generează niciodată excepții.

`chloros_sdk` reexportă doar suprafața de bază utilizată mai sus. API-ul public complet `daq_sdk` (22 de nume) adaugă următoarele — importați-le direct din `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Excepții

Interceptați clasa de bază pentru a gestiona „orice problemă apărută în Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` și `ChlorosConfigurationError` sunt exportate la nivel superior alături de restul; ele pot fi, de asemenea, importate din `chloros_sdk.exceptions`, așa cum se arată.

Ierarhie:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exemple end-to-end

### 1. Procesarea unui folder cu o bară de progres personalizată

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matrice LATTICE în timp real → Reflectanță + Referință DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campanie de captură bazată pe proiect

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Flux de cadre multi-cameră → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de captură directă pe hardware fără interfață grafică (fără backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Testarea capacităților înainte de conectarea unui ansamblu de 4 camere

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Echivalentul rețetei de captură (Python pur)

Limbajul DSL al rețetelor din „CLI” are un echivalent direct în „Python”:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Pornire automată a backend-ului

Punctele de intrare smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, și `discover_lattice_cameras` — sunt clienți „thin” HTTP care presupun că un backend ascultă pe `127.0.0.1:5000` (URL-ul implicit al interfeței smart-connect). Când interfața grafică sau CLI rulează deja, unul dintre ele este activ. Dacă se pornește doar dintr-un script simplu, s-ar putea să nu existe — așa că aceste funcții **pornesc automat binarul backend inclus** (fără fereastră, la fel cum o face `ChlorosLocal`) înainte de prima lor apelare, apoi așteaptă până la `backend_startup_timeout` ca acesta să pornească.

Reguli:

- **Se lansează întotdeauna doar un URL local.** Un `backend_url` care indică spre `localhost` / `127.0.0.1` / `[::1]` este eligibil; orice alt host este considerat a fi mașina altcuiva și nu este niciodată generat.
- **Backend-ul rămâne în funcțiune pentru reutilizare** (la fel ca în cazul CLI) — nu are loc o oprire implicită la ieșirea din script. Rularea din nou a scriptului reutilizează backend-ul activ.
- **Dezactivați cu `auto_start_backend=False`** la oricare dintre aceste apeluri (de exemplu, când ați indicat un backend la distanță sau când gestionați singur ciclul de viață al backend-ului).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Dacă binarul inclus nu poate fi localizat sau pornit, apelul ulterior HTTP generează o eroare `ChlorosConnectError` care poate fi rezolvată și **adaptată platformei**, în loc de o simplă urmărire a conexiunii— pe Windows vă îndrumă către aplicația desktop sau către o comandă `chloros-cli`; pe Linux (fără GUI) vă îndrumă către o comandă `chloros-cli` sau către `.deb`.

---

## Mediu și antete

SDKul marchează fiecare apel către backend HTTP cu `X-Chloros-Client: sdk`. Backend-ul aplică regulile de licențiere de la SDK / CLI (este necesară autentificarea **și** un plan plătit de la Chloros+) în locul opțiunii gratuite a interfeței grafice. Această setare se aplică automat la momentul importului — nu este necesar să faceți nimic.

`http://localhost` și `http://127.0.0.1` sunt detectate ca backend local. Apelurile către alte gazde (de exemplu, propriul serviciu de analiză) rămân neschimbate.

Puteți suprascrie backend-ul URL prin transmiterea valorii `backend_url=` (sau `api_url=` pe `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un backend non--loopback ajunge doar la un backend source/dev — backend-urile livrate se leagă doar la loopback; consultați Modul backend la distanță pentru modelul tunelului.)

---

## Versiuni și compatibilitate

- Versiunea SDK este expusă ca `chloros_sdk.__version__`.
- SDK leagă comportamentul de versiunea backend-ului inclus în pachet. Combinarea unui backend mai vechi SDK cu unul mai nou funcționează de obicei (puncte finale compatibile cu versiunile ulterioare), dar combinarea unui backend mai nou SDK cu unul mai vechi poate genera erori `404` pe noile puncte finale — actualizați aplicația desktop pentru a se potrivi.
- Interfața Smart-Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) și terminalul de analiză a rețelei returnează scheme stabile de tip „JSON”; câmpurile noi sunt aditive.

---

## Indicații pentru depanare

- **`ChlorosAuthenticationError: Login required`** → Rulați o dată `chloros-cli login EMAIL PASSWORD` pe acest computer sau conectați-vă prin intermediul aplicației desktop Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Apelurile smart-connect pornesc automat un backend local, așa că acest mesaj apare doar când binarul inclus nu poate fi găsit/pornit (de exemplu, o gazdă care utilizează doar pip, fără pachet desktop). Mesajul ține cont de platformă: pe Windows deschideți aplicația desktop sau rulați orice comandă `chloros-cli`; pe Linux rulați o comandă `chloros-cli` (nu există GUI) sau instalați `.deb`. Pentru un backend la distanță, treceți `backend_url=` (și `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** la import → `lattice_sdk` nu s-a putut încărca (de obicei, DLL-urile de rulare Arena SDK nu sunt instalate). Suprafața fără cameră funcționează în continuare.
- **Conectarea matricei returnează o rezoluție sub-nativă**→ Funcția smart-prep a backend-ului reduce automat dimensiunea cadrului pentru a se potrivi cu cablul. Folosiți `analyze_array_network()` pentru a vedea de ce, apoi fie actualizați legătura, fie acceptați redimensionarea, fie treceți la `force_tier="slip-emit-and-capture"` pentru captură secvențială. Măsura de siguranță a redimensionării**nu** acoperă suprasubscrierea agregată (`oversubscribed: true`, câmpurile fps 0): numărul prea mare de camere pentru canalul de comunicație nu poate fi remediat prin binning/ROI — reduceți numărul de camere, activați cadrele jumbo sau treceți la o placă de rețea mai rapidă (a se vedea [Supra-abonarea](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` raportează că inelul de recepție al plăcii de rețea este foarte mic (~0,26 MB) / porți de conectare cu mesajul „FRAMES WILL DROP”** → Inelul de recepție al plăcii de rețea a gazdei este la valoarea implicită (adesea resetat la 32 după o actualizare a driverului plăcii de rețea). Pe un adaptor Realtek USB 10GbE, setați `ReceiveBufferLen=256` și `PendingReceives=64` (nivel ridicat), apoi reporniți backend-ul pentru a reciti inelul. Procedura completă: [Referință CLI → Configurarea și reglarea plăcii de rețea a gazdei](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Gazda se blochează la repornire/oprire, ulterior apar erori WMI `Invalid class` / placa de rețea nu se activează** → Driverul USB 10GbE învechit provoacă eroarea `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Actualizați driverul adaptorului la o versiune curentă (≥ 2026) și reaplicați setările inelului de recepție. Consultați [Referința CLI → Configurarea și reglarea plăcii de rețea a gazdei](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflectanța a fost refuzată** → Pentru reflectanța pe scară absolută, trebuie să se asocieze un DAQ activ cu camera (sau matricea). Asociați-l fie prin intermediul GUI, fie utilizați `processing="radiance"` (W/m²/sr/nm), care nu necesită un senzor asociat.
- **Captura `smart=True` durează mai mult decât era de așteptat** → Convergența AE depinde de dinamica scenei; măriți valoarea `exposure_tolerance_pct` sau scurtați `stability_window_s` dacă doriți un (mai puțin stabil).

---

## Vezi și

- [Referință CLI](cli-reference.md) — fiecare subcomandă CLI corespunde unei apeluri SDK.
- [Ghidul senzorilor DAQ](../daq/README.md) — reguli specifice senzorilor privind cablarea, calibrarea și înregistrarea.
- Documentație online: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
