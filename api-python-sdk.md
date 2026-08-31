# API : Python SDK

{% hint style="info" %}
**Căutați versiunea completă a API?** Această pagină este un tutorial practic. Fiecare clasă publică, metodă, semnătură exactă și exemplu care poate fi copiat și lipit se găsește în [Referința SDK](reference/sdk-reference.md), care este optimizată pentru asistenții AI.**Lucrezi cu un asistent AI?** Lipeste acest URL în chat pentru ca acesta să aibă versiunea completă și actualizată a Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Fiecare pagină a acestui manual este disponibilă în format Markdown brut la adresa cu slug-ul scris cu litere mici + `.md`, iar întregul manual este indexat la `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` pe PyPI) controlează toate funcționalitățile aplicației desktop începând cu Python: procesarea în serie a imaginilor, controlul în timp real al camerei LATTICE și al matricei, sesiunile DAQ cu senzori de lumină și automatizarea proiectelor salvate. Este un strat subțire peste același backend local pe care îl utilizează atât interfața grafică, cât și CLI (HTTP pe `127.0.0.1:5000`), astfel încât comportamentul este identic pe toate cele trei interfețe.

## Instalare

Instalarea se realizează în doi pași: mai întâi pachetul pentru desktop Chloros (care oferă backend-ul de procesare și mediile de execuție pentru hardware), apoi pachetul Python.

**Pasul 1 — Instalați Chloros.** Windows: rulați programul de instalare pentru desktop (calea implicită `C:\Program Files\MAPIR\Chloros\`) de pe pagina [Descărcare](download.md). Linux: instalați pachetul `.deb` ([Instalarea Linux](linux/linux-installation.md)).**Pasul 2 — Instalați SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Este posibil să nici nu aveți nevoie de pip: fiecare program de instalare include un wheel SDK corespunzător. Programul de instalare Windows îl instalează automat în sistemul dvs. Python; programul de instalare Linux `.deb` îl plasează în `/usr/lib/chloros/sdk/` și afișează comanda exactă `pip install --user`. PyPI este actualizat la fiecare versiune lansată, astfel încât `pip install chloros-sdk` corespunde celei mai recente versiuni stabile.

**Pasul 3 — Autentificați-vă o singură dată pe fiecare mașină:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Datele de autentificare sunt stocate în cache în `~/.chloros/` (pe ambele platforme). Pe Windows vă puteți autentifica în mod echivalent prin fila „<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">” () a aplicației desktop. SDK necesită un plan plătit Chloros+ — consultați [Cerințele privind licența](#license-requirement) de mai jos.

| Cerință | Detalii |
| --- | --- |
| **Chloros instalat** | Windows: program de instalare pentru desktop; Linux: pachetul `.deb` (furnizează fișierul binar backend) |
| **Python** | 3.7 sau o versiune ulterioară (dezvoltat/testat pe 3.10) |
| **Sistem de operare** | Windows 10/11 pe 64 de biți, Ubuntu 22.04 LTS sau o versiune mai recentă, sau NVIDIA Jetson (JetPack 6) |
| **Licență** | Autentificare activă Chloros+, orice nivel plătit (Copper sau superior) |

## Succesul în 60 de secunde

O singură comandă creează un proiect, importă un folder, configurează procesarea și rulează pipeline-ul — pornind automat backend-ul dacă acesta nu rulează deja:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Pe Linux, utilizați căile Linux: `/home/user/drone_images/flight001`. SDK funcționează identic pe ambele platforme.)

Procesați un folder de capturi LATTICE? Folosiți wrapper-ul compatibil cu LATTICE — acesta aplică setările implicite corecte (fără detectarea țintei panoului, debayer standard):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — control complet al fluxului de lucru

Pentru orice operațiune mai complexă decât o singură linie de comandă, folosiți `ChlorosLocal`. Acesta pornește backend-ul la prima utilizare (`auto_start_backend=True`), creează și configurează proiecte, monitorizează progresul și returnează un rezumat după finalizarea procesării.

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

{% hint style="info" %}
Păstrați valoarea implicită `http://127.0.0.1:5000` în loc să o înlocuiți cu `localhost` — în cazul lui Windows, `localhost` se rezolvă mai întâi în `::1` și durează aproximativ 2 secunde pe cerere în cazul backend-ului exclusiv IPv4.
{% endhint %}

Utilizați-l ca manager de context pentru o curățare garantată:

```python
import chloros_sdk

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

`configure()` acceptă următoarele cuvinte-cheie: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` și `custom_settings`. Valorile principale:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Butoanele specifice LATTICE (`input_level`, `radiometric_output`, familia `array_alignment*`) sunt documentate împreună cu tabelele complete de valori în [Referința SDK](reference/sdk-reference.md#supported-values).

### Urmărirea progresului

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Citirea rezumatului post-execuție — și detectarea execuțiilor goale

La finalizare, `process()` atașează rezumatul de procesare al backend-ului sub forma `result["summary"]`. Fiecare intrare din `summary["hints"]` este o propoziție completă care explică orice aspect demn de menționat — de exemplu, de ce o execuție a produs zero rezultate — iar fiecare indiciu este, de asemenea, retransmis ca un Python `UserWarning`, astfel încât execuțiile goale se autodiagnostică chiar dacă nu inspectați niciodată dicționarul:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` nu generează o eroare atunci când o execuție nu produce imagini.** Acesta este singurul aspect în care SDK și CLI diferă în mod deliberat: `chloros-cli process` tratează „au fost solicitate produse, dar nu a fost scris niciunul” ca pe o eroare și se închide cu un cod de ieșire diferit de zero, în timp ce SDK se închide normal și raportează această condiție prin `summary` / sugestii. Dacă pipeline-ul dvs. ar trebui să se oprească în cazul unei rulări goale, verificați acest lucru personal — inspectați `summary` (sau numărați fișierele din folderul proiectului) în loc să vă bazați pe o excepție.
{% endhint %}

## Smart Connect — hardware activ

Trei asistenți deschid sesiuni persistente în grupul de hardware al backend-ului — același grup pe care îl folosește interfața grafică, astfel încât scripturile SDK coexistă cu aplicația desktop fără a intra în conflict pentru porturile seriale sau lățimea de bandă a rețelei. Toate cele trei pornesc automat un backend local dacă nu rulează niciunul.

### O singură cameră LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Matrice sincronizată — `connect_array`

`connect_array` este punctul de intrare recomandat pentru configurațiile cu mai multe camere. Acesta rulează același flux de pregătire inteligentă ca și interfața grafică: analiza rețelei, selectarea automată a nivelului de sincronizare, sincronizarea temporală PTP, selectarea formatului de pixeli pentru fiecare cameră, inițializarea AE și activarea declanșării GPIO. **Prima cameră din șir este master** (aceasta generează impulsul de declanșare hardware); restul sunt slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Adăugați `smart=True` la orice captură de matrice pentru a aștepta stabilizarea expunerii automate pe toate camerele înainte de declanșare. Pentru modurile de captură (Single / Continuous / Interval / Fastest), înregistratoare, burst-to-video și alinierea matricei, consultați [Referința SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Senzor de lumină DAQ — `connect_daq_sensor`

Fără argumente, `connect_daq_sensor()` detectează automat protocolul de transport (ordine de prioritate: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Fiecare cadru conține valoarea de 135 de puncte `spectrum` (W/m²/nm după calibrare), un indicator `is_saturated` și CIE `x`, `y`, `z`. Pentru a specifica un senzor sau un protocol de transport anume — alegerea fiabilă pe gazdele cu mai multe interfețe de rețea, unde detectarea automată prin Ethernet poate omite un DAQ-E funcțional la prima încercare — transmiteți un indiciu explicit:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Rețineți că profilurile de corecție a limitelor (`cap_id`) **nu** sunt un parametru de tip SDK — selectați-le în schimb prin `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Proiecte salvate — `open_project`

Un proiect Chloros salvat păstrează hardware-ul conectat (`cameras.json` + `sensors.json` alături de `project.json`), iar `chloros_sdk.open_project(path)` poate reconecta totul simultan și poate efectua capturi în funcție de numele dispozitivului. Consultați [Automatizarea proiectelor](reference/sdk-reference.md#project-automation--chlorosproject) în documentația de referință.

## Ce obține o instalare exclusiv prin pip

Verificați indicatorii de disponibilitate la nivel de modul înainte de a utiliza suprafețele hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Pe un gazdă cu **doar** `pip install chloros-sdk` și fără pachetul desktop Chloros:

* `ChlorosLocal`, `process_folder` și `process_lattice_capture` **nu** funcționează — acestea au nevoie de binarul backend inclus în programul de instalare pentru desktop.
* Asistenții smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) sunt clienți puri HTTP, așa că funcționează cu un backend de pe o altă mașină — însă backend-urile livrate se leagă doar la loopback, așa că trebuie să redirecționați portul singuri (de ex. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) și să transmiteți `backend_url="http://127.0.0.1:5000"` împreună cu `auto_start_backend=False`. Consultați [Modul backend la distanță](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Clasele LATTICE cu acces direct la hardware (`LatticeCamera`, `CameraPool`, …) se pot importa, dar necesită mediul de execuție Arena SDK din pachetul pentru desktop — fără acesta, `CAMERA_AVAILABLE` este `False`.
* `daq_sdk` (clasele DAQ directe) este inclus în instalarea pentru desktop, nu în pachetul PyPI, așa că `DAQ_AVAILABLE` este `False` pe o mașină care folosește doar pip — controlați senzorii DAQ prin `connect_daq_sensor()` către un backend (tunelat) în schimb.

## Cerințe de licență

Accesul la SDK necesită o autentificare activă Chloros+ pe orice nivel plătit — **Copper sau superior**(Copper / Bronze / Silver / Gold); nivelul gratuit Iron nu are acces la SDK/CLI. Aplicarea se face**pe partea de server**: fiecare cerere SDK trebuie să includă atât o sesiune activă, cât și un plan plătit; în caz contrar, backend-ul returnează `403` / `PLAN_UPGRADE_REQUIRED` (generat ca `ChlorosLicenseError` de către `ChlorosLocal` și ca `ChlorosConnectError` de către helper-ii `connect_*`). Un apelant deconectat primește în schimb `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — rulară din nou a `chloros-cli login` rezolvă primul caz, dar nu și pe al doilea.

Utilizarea offline funcționează în perioada de grație a planului: nivelul este citit din cache-ul de validare a serverului (5 minute) sau din cache-ul licenței semnate, legate de mașină (30 de zile pentru planurile lunare; până la expirarea abonamentului pentru cele anuale). Când perioada de grație expiră, planul trece la nivelul gratuit, iar accesul prin SDK se oprește până când dispozitivul se conectează o dată la server. `chloros-cli status` rămâne accesibil la nivelul gratuit, astfel încât motivul este întotdeauna vizibil. Consultați [Chloros+ Autentificare](chloros+-login.md).

## Excepții

Interceptați clasa de bază pentru a gestiona „orice problemă Chloros”:

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

Toate excepțiile din pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) derivă din `ChlorosError`. O atenție: `ChlorosConnectError` — generată doar de `connect_camera` / `connect_array` / `connect_daq_sensor` — derivă din `Exception` simplu, **nu** din `ChlorosError`, deci `except ChlorosError` nu o va detecta. Ierarhia completă se găsește în [Referința SDK](reference/sdk-reference.md#exceptions).

## Vezi și

* [Referința SDK](reference/sdk-reference.md) — suprafața completă API, optimizată pentru asistenții AI.
* [Referința CLI](reference/cli-reference.md) — fiecare subcomandă CLI corespunde unui apel SDK.
* [Descărcare](download.md) — programe de instalare pentru Windows și Linux.
