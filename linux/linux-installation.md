# Instalarea Linux

Chloros este distribuit pentru Linux sub formă de pachete `.deb` care instalează CLI și serverul backend. Python SDK este un pachet pip separat (inclus și în `.deb` ca un wheel cu versiune compatibilă).

Numele fișierelor pachetelor conțin versiunea și arhitectura: `chloros_1.2.0_amd64.deb` pentru x86_64 și `chloros_1.2.0_arm64_jp6.deb` pentru versiunile JetPack 6 Jetson. Înlocuiți fișierul pe care l-ați descărcat efectiv în comenzile de mai jos.

***

## Linux amd64 (x86_64)

### Cerințe de sistem

| Cerință | Minimă | Recomandată |
| --- | --- | --- |
| **Distribuție** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 sau superior |
| **Memorie (RAM)** | 8 GB | 16 GB sau mai mult |
| **Placă grafică** | Niciuna (procesare CPU) | Placă grafică NVIDIA cu 4 GB+ VRAM (12 GB+ deblochează `GPU_PARALLEL`, 7 GB+ menține Texture Aware în afara căii de imagine unică) |
| **Spațiu de stocare** | 2 GB spațiu liber | SSD cu 10 GB+ spațiu liber |
| **Python** | Python 3.7+ (pentru SDK) | Python 3.10+ |

> **Ubuntu 20.04 și Debian 11 nu sunt suportate.** Lista de dependențe a `.deb` este
> derivată din ceea ce backend-ul Chloros leagă efectiv, iar aceasta include
> `libc6 (>= 2.34)`. Atât Focal, cât și bullseye sunt livrate cu glibc 2.31, așa că `apt` refuză
> instalarea din start, în loc să permită eșuarea acesteia mai târziu, în timpul rulării.

### Instalare

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` nu rezolvă dependențele. Dacă semnalează pachete lipsă, `sudo apt-get install -f` (sau `sudo apt --fix-broken install`) finalizează instalarea — aceasta este calea normală, nu o eroare.
{% endhint %}

Verificați instalarea:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Cerințe de sistem

| Cerință | Minimă | Recomandată |
| --- | --- | --- |
| **Platformă** | NVIDIA Jetson cu JetPack 6 | Jetson Orin NX 16 GB sau AGX Orin |
| **JetPack** | JetPack 6.x | Cea mai recentă versiune JetPack 6 |
| **Memorie (RAM)** | 8 GB (partajată între GPU și CPU) | 16 GB+ partajată (12 GB+ este pragul minim pentru procesele paralele pe GPU) |
| **Spațiu de stocare** | 2 GB spațiu liber | SSD NVMe cu 10 GB+ spațiu liber |
| **Python** | Python 3.7+ (pentru SDK) | Python 3.10+ |

### Instalare

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Aceeași structură ca și versiunea amd64 `.deb`, cu o versiune CUDA optimizată pentru Jetson Orin / Orin NX / Orin Nano. Pentru informații privind memoria, temperaturile și comportamentul în condiții reale de utilizare al dispozitivelor Jetson, consultați [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalarea Python și SDK (toate versiunile Linux)

SDK este un client pur Python HTTP pentru backend, astfel încât același pachet funcționează atât pe amd64, cât și pe arm64. Două surse:**De pe PyPI** — versiunea stabilă publicată:

```bash
pip install chloros-sdk
```

**Din fișierul wheel inclus** — garantat compatibil cu CLI/backend-ul pe care tocmai l-ați instalat (utilizați aceasta atunci când versiunea dvs. `.deb` este mai recentă decât cea de pe PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Distribuțiile PEP 668** (Ubuntu 23.10+, Debian 12+) refuză instalările pip la nivel de sistem. Folosiți `pip install --user …`, un mediu virtual sau `sudo pip install --break-system-packages …`. Programul de instalare a pachetelor nu instalează niciodată automat SDK în Python-ul sistemului dvs. — această alegere vă revine.
{% endhint %}

Opțiuni suplimentare:

| Opțiune | Comandă | Adaugă |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pentru transmisiunea în direct a progresului |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pentru transport BLE (DAQ-M) |

Verificați SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` instalează Chloros, CLI și backend-ul. Python, SDK comunică cu acel backend printr-o rețea locală HTTP API (`http://127.0.0.1:5000`) și îl pornește automat când este necesar. Utilizați întotdeauna adresa IPv4 literală în locul `localhost` — `localhost` se poate rezolva la `::1` și poate costa aproximativ două secunde pe cerere.
{% endhint %}

***

## Configurare inițială

### 1. Autentificare

Accesul la CLI și SDK necesită un plan plătit Chloros+ (**Copper** sau superior), aplicat la nivel de server: un apelant deconectat primește `401 AUTH_REQUIRED`, iar un apelant din planul gratuit (Iron) primește `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Datele de autentificare sunt stocate în cache în `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Trebuie să vă autentificați din nou după fiecare instalare sau actualizare.** Scriptul `prerm` al pachetului șterge în mod deliberat `~/.chloros/user_session.json` și licența stocată în cache pentru fiecare utilizator de pe computer, astfel încât o nouă versiune să revalideze întotdeauna licența, în loc să se bazeze pe un cache învechit.
{% endhint %}

### 2. Verificați starea licenței

```bash
chloros-cli status
```

`chloros-cli status` funcționează pe orice nivel (inclusiv cel gratuit), astfel încât puteți vedea întotdeauna de ce accesul este sau nu este disponibil.

### 3. Rulați diagnosticarea sistemului

```bash
chloros-cli selftest
```

Cele șapte verificări se execută în ordine, iar comanda returnează un cod de ieșire diferit de zero dacă vreuna dintre ele eșuează:

| # | Verificare | Ce demonstrează |
| --- | --- | --- |
| 1 | **Versiune** | CLI raportează versiunea sa (`v1.2.0`). |
| 2 | **Port disponibil** | Portul 5000 este liber *sau* a primit deja un răspuns de la un backend Chloros funcțional (ceea ce se consideră o verificare reușită). |
| 3 | **Pornirea backend-ului** | Fișierul binar al backend-ului se lansează. |
| 4 | **Test API (`/api/test`)** | Backend-ul răspunde cu `status: ok`. |
| 5 | **Informații despre sistem** | Afișează `GPU: <name>, CUDA: <bool>, PyTorch: <version>` din `/api/system-info`. |
| 6 | **Modele de denoizare** | Găsește modelele `*.pth.enc` (pe Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| Texture Aware este de fapt utilizabil — necesită CUDA**și** cel puțin un fișier de model. |

Execuția se încheie cu `N/7 checks passed`, listând eventualele eșecuri după nume.

### 4. Prelucrați primul set de date

```bash
chloros-cli process ~/datasets/flight001
```

***

## Fișiere și directoare

### Per utilizator

Chloros își păstrează datele de autentificare și configurația CLI într-un singur director compatibil cu toate platformele, **`~/.chloros/`** (pe Windows, `%USERPROFILE%\.chloros\`). Două cache-uri specifice Linux respectă în schimb convențiile XDG — acestea țin cont de `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` atunci când sunt setate.

| Cale | Scop |
| --- | --- |
| `~/.chloros/user_session.json` | Cache-ul sesiunii de autentificare scris de `chloros-cli login` (șters la fiecare instalare/actualizare de pachet) |
| `~/.chloros/working_directory.txt` | Suprascrierea folderului implicit al proiectului (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferința de limbă CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Setare de limbă partajată cu interfața grafică Windows — un `language` de aici are prioritate față de `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache de o oră pentru verificarea actualizărilor la pornire pentru Linux/Jetson |
| `~/.chloros/backend.log` | Jurnalul backend-ului atunci când acesta a fost lansat de CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pachete de calibrare LATTICE stocate în cache pentru fiecare cameră, indexate după numărul de serie și hash-ul pachetului |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Setări opționale ale utilizatorului pentru profilurile de corecție a capacității DAQ |
| `~/.config/chloros/system_config.json` | Profil hardware stocat în cache din Dynamic Compute Adaptation — ștergeți-l pentru a forța o nouă detectare a hardware-ului |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Jurnale ale serverului backend, câte un fișier pentru fiecare lansare |
| `~/Chloros Projects/` | Dosarul implicit al proiectului atunci când nu este setată nicio suprascriere |

### La nivel de sistem

| Cale | Scop |
| --- | --- |
| `/usr/bin/chloros-cli` | Script de înveliș — setează `LD_LIBRARY_PATH` pentru bibliotecile native incluse în pachet, apoi execută fișierul binar propriu-zis |
| `/usr/bin/chloros-backend` | Script de înveliș — la fel, plus `CHLOROS_PRODUCTION=1`, astfel încât poarta de autentificare a backend-ului să nu se poată dezactiva niciodată în mod silențios |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Fișierele binare compilate |
| `/usr/lib/chloros/arena_runtime/` | Runtime-ul Arena SDK necesar pentru camerele LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modele de denoizare criptate utilizate de debayerul Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK roată compatibilă exact cu această versiune |
| `/usr/lib/chloros/exiftool` | Exiftool inclus (cu legătură simbolică către `/usr/local/bin/exiftool` numai dacă nu există un exiftool de sistem) |
| `/etc/chloros/update.conf` | Configurația canalului de actualizare citită de `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Configurează `net.ipv4.ip_unprivileged_port_start = 319` astfel încât backend-ul să poată lega porturile PTP fără drepturi de root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Direcționează încărcătorul dinamic către `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Acordă utilizatorului conectat acces la puntea serială USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Activare opțională a serviciului backend permanent activ (instalat, **neactivat**) |
| `/usr/share/applications/chloros-cli.desktop` | Intrare în meniul aplicației „Chloros CLI” care deschide un terminal |

## Locația executabilului backend

CLI și SDK detectează automat backend-ul:

| Componentă | Cale |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Puteți suprascrie calea backend-ului cu indicatorul `--backend-exe` CLI sau cu parametrul constructorului `backend_exe` SDK, iar portul cu `--port` (implicit `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` indică familiile de comenzi **`lattice`**,**`project`**și**`daq pool-*`** către un backend la distanță. Familiile de comenzi principale (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) o ignoră în mod deliberat și vizează întotdeauna `http://127.0.0.1:<port>`.
{% endhint %}

***

## Camerele LATTICE și senzorii de lumină DAQ pe Linux

Toate familiile de comenzi live-hardware funcționează pe Linux (amd64 și Jetson):

* **`chloros-cli lattice`** — detectează, conectează, configurează și captează date de la camerele LATTICE și de la matrice sincronizate. `.deb` include mediul de execuție Arena SDK necesar și îl înregistrează în încărcătorul dinamic.
* **`chloros-cli daq pool-*`** — conectează senzorii de lumină DAQ-U/M/E prin intermediul grupului de backend, transmite în flux spectre calibrate și înregistrează fișiere `.daq`. Versiunea compilată CLI include doar familia `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — rulează un proiect salvat (camerele, senzorii și setările de procesare ale acestuia) fără interfață grafică.
* **`chloros-cli time-sync`** — inspectează grandmaster-ul PTP pe care backend-ul Chloros îl rulează pentru camerele LATTICE și senzorii DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` este necesar pentru `pool-latest`, `pool-stream`, `pool-record` și `pool-set-cap`; `pool-list` afișează ID-urile aflate în prezent în pool.

{% hint style="info" %}
**Preferați `--eth-host` pentru prima conexiune DAQ-E pe un computer cu mai multe adrese de rețea.** Funcția de descoperire automată scanează mDNS și poate omite interfața senzorului din cauza unui cache ARP gol, astfel încât prima încercare `pool-connect --eth` după pornire poate eșua chiar și atunci când senzorul funcționează perfect. Specificarea adresei IP sau a numelui de gazdă al senzorului omite complet procesul de descoperire.
{% endhint %}

**Permisiunile seriale DAQ-U** sunt gestionate de regula udev instalată (`uaccess` + grupul `dialout`). Dacă un senzor care era deja conectat rămâne inaccesibil, reîncărcați regulile sau reconectați-l:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consultați [referința CLI](../CLI.md) pentru setul complet de comenzi.

### PTP permanent activ pentru gazde fără interfață grafică

La prima instalare, unitatea systemd `chloros-backend.service` este generată, dar **nu este activată**. Pe un Jetson fără monitor sau pe un server care trebuie să mențină sincronizarea temporală PTP în funcțiune continuă pentru senzorii DAQ-E și camerele LATTICE, activați-o:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Fără aceasta, PTP rulează doar cât timp backend-ul Chloros este activ — adică, în timpul unei sesiuni active CLI/SDK.

Unitatea leagă backend-ul de setările de mediu `127.0.0.1:5000` (setările de mediu `CHLOROS_HOST` / `CHLOROS_PORT` din interiorul unității; se poate suprascrie cu `sudo systemctl edit chloros-backend.service`) și îl repornește în caz de eșec după 5 secunde.

**Cum obține PTP porturile sale.** PTP utilizează UDP 319/320, ambele sub pragul normal de 1024 pentru porturile privilegiate. Pachetul `postinst` scrie `/etc/sysctl.d/60-chloros-ptp.conf` cu `net.ipv4.ip_unprivileged_port_start = 319`, ceea ce permite backend-ului să se conecteze la acestea în timp ce rulează sub contul dvs. de utilizator. De asemenea, aplică `setcap cap_net_bind_service,cap_net_raw=+ep` fișierului binar al backend-ului ca măsură de siguranță suplimentară — de aceea `libcap2-bin` este o dependență declarată a pachetului.***

## Exemple de scripturi Bash

{% hint style="info" %}
**Coduri de ieșire compatibile cu scripturile.**`chloros-cli process` returnează `0` în caz de succes și**o valoare diferită de zero în caz de eșec — inclusiv o execuție care a solicitat produse imagistice, dar nu a generat niciunul** (afișează `Processing finished but wrote no image products.` și menționează numele folderului proiectului și cauzele obișnuite). Execuțiile reușite raportează câte produse imagistice au fost scrise (`Image products written: N`). Coduri de ieșire: `0` succes, `1` eșec, `2` eroare de argument, `130` întrerupt.
{% endhint %}

### Procesarea mai multor seturi de date

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Procesare cu setări personalizate

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Valorile valide pentru `--format` sunt exact patru și conțin spații — puneți-le întotdeauna între ghilimele:

| Valoare `--format` | Folder de ieșire |
| --- | --- |
| `TIFF (16-bit)` *(implicit)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` acceptă `standard` (implicit) sau `texture-aware` (Chloros+).

### Procesare automată cu Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemplu Python SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Rezolvarea problemelor

### CLI nu a fost găsit după instalare

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Permisiune refuzată

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap a eșuat” în timpul instalării

`.deb` aplică `cap_net_bind_service` la `/usr/lib/chloros/chloros-backend`, astfel încât să poată lega porturile PTP 319/320 fără drepturi de root. Dacă `libcap2-bin` lipsea în momentul instalării, apelul este omis. Instalați-l și reinstalați pachetul:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP nu pornește / nu se poate lega la portul 319

Confirmați că pragul porturilor fără privilegii a fost redus și, dacă nu a fost, reaplicați-l pentru pornirea curentă:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Apoi verificați grandmaster-ul:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „Drivere cameră LATTICE neidentificate”

Mediul de execuție Arena SDK nu este rezolvat. Verificați dacă configurația încărcătorului scrisă de pachet este prezentă și actualizată:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Eșec la pornirea backend-ului

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Jurnalele backend-ului pentru lansarea eșuată se află în `~/.cache/chloros/logs/`.

### CUDA nedetectat

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` raportează același lucru într-o singură linie: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Biblioteci partajate lipsă

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Pornire lentă pe sistemele cu card SD

Fișierele binare compilate se extrag automat într-un director temporar la fiecare lansare. Dacă există `/mnt/ssd/tmp`, Chloros îl utilizează automat; în caz contrar, setați `TMPDIR` pe un sistem de fișiere rapid:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Actualizarea Chloros pe Linux

Comanda `update` este disponibilă numai pe Linux/Jetson. Aceasta verifică versiunea publicată în canalul de actualizare configurat la `/etc/chloros/update.conf` și oferă posibilitatea de a descărca și instala versiunea corespunzătoare `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Pe Linux/Jetson, CLI efectuează, de asemenea, o verificare a actualizărilor fără blocare la fiecare pornire (rezultatul fiind stocat în cache timp de o oră în `~/.chloros/update_cache.json`) și afișează `Update available: vX.Y.Z` atunci când există o versiune mai nouă. Setările și proiectele dvs. rămân intacte după o actualizare; va trebui să vă autentificați din nou după aceea.

## Dezinstalare

```bash
sudo apt remove chloros
```

Dezinstalarea oprește `chloros-backend.service`, restabilește limita minimă implicită pentru porturile fără privilegii (1024), elimină legătura simbolică către exiftool inclus în pachet și configurația încărcătorului Arena, precum și șterge datele de autentificare stocate în cache. Proiectele și fișierele de date `~/.chloros/` rămân intacte.

***

## Pași următori

* [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md) — optimizare și implementare specifice pentru Jetson
* [CLI : Linia de comandă](../CLI.md) — ghidul CLI
* [API : Python SDK](../api-python-sdk.md) — ghidul SDK
* [Referința CLI](../reference/cli-reference.md) și [Referința SDK](../reference/sdk-reference.md) — liste exhaustive de comenzi/API pentru versiunea 1.2.0
* [Adaptarea dinamică a capacității de calcul](../processing-architecture/dynamic-compute-adaptation.md) — modul în care Chloros se adaptează la hardware-ul dumneavoastră
