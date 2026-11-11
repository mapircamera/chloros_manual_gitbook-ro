# Instalare pe Linux

Chloros este distribuit pentru Linux sub formă de pachete `.deb` care instalează interfața CLI, serverul backend și SDK-ul Python (pentru sistemul `python3`, plus un fișier wheel compatibil cu versiunea respectivă pentru virtualenvs).

Numele fișierelor pachetelor conțin versiunea și arhitectura: `chloros_1.3.0_amd64.deb` pentru x86_64, iar `chloros_1.3.0_arm64_jp6.deb` pentru arm64 (NVIDIA Jetson, JetPack 6). Înlocuiți fișierul pe care l-ați descărcat efectiv în comenzile de mai jos.

{% hint style="info" %}
**Noutate în versiunea 1.3.0: programele sunt livrate nearsivate.** Backend-ul și interfața CLI sunt instalate ca foldere, `/usr/lib/chloros/backend/` și `/usr/lib/chloros/cli/`, în loc de fișiere auto-extractibile individuale. Acestea nu se mai dezarhivează automat într-un folder temporar la fiecare lansare, astfel încât pornesc mult mai repede (pe x86_64, backend-ul obișnuia să petreacă aproximativ 100 de secunde pentru dezarhivare la fiecare pornire) și nu mai necesită câțiva GB de spațiu liber în `/tmp`. De asemenea, versiunea amd64 `.deb` are aproximativ jumătate din dimensiunea anterioară.
{% endhint %}

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

> **Ubuntu 20.04 și Debian 11 nu sunt acceptate.** Lista de dependențe a `.deb` este
> derivată din ceea ce backend-ul Chloros leagă efectiv, iar aceasta include o versiune
> `libc6` mai nou decât glibc 2.31 livrat de focal și bullseye, așa că `apt` refuză
> instalarea din start, în loc să permită eșuarea acesteia mai târziu, în timpul rulării.

### Instalare

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Utilizați `apt install ./…`, nu `dpkg -i`.** `apt` rezolvă dependențele pachetului și instalează, de asemenea, ceea ce recomandă: cerințele Python SDK, `python3-numpy` și `python3-requests`. `dpkg -i` nu instalează niciuna dintre acestea. CLI-ul și backend-ul sunt autonome și funcționează fără ele, dar `import chloros_sdk` are nevoie de ele; ieșirea instalării menționează orice elemente lipsă. Păstrați `./`, astfel încât `apt` să instaleze fișierul în loc să caute un pachet cu acel nume. Pachetul declară, de asemenea, `libgl1`, pe care OpenCV-ul inclus îl încarcă și care poate lipsi dintr-o imagine minimă de server.
{% endhint %}

Verificați instalarea:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Cerințe de sistem

| Cerință | Minimă | Recomandată |
| --- | --- | --- |
| **Platformă** | NVIDIA Jetson cu JetPack 6 | Jetson Orin NX 16 GB sau AGX Orin |
| **JetPack** | JetPack 6.x | Cea mai recentă versiune de JetPack 6 |
| **Memorie (RAM)** | 8 GB (partajată între GPU și CPU) | 16 GB+ partajată (12 GB+ este pragul pentru procesarea paralelă) |
| **Spațiu de stocare** | 2 GB spațiu liber | SSD NVMe cu 10 GB+ spațiu liber |
| **Python** | Python 3.7+ (pentru SDK) | Python 3.10+ |

### Instalare

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Aceeași structură ca și amd64 `.deb` (Python SDK, tampoane de recepție și servicii backend incluse), pentru Jetson Orin / Orin NX / Orin Nano pe **JetPack 6** (L4T R36). PyTorch-ul său este versiunea CUDA a JetPack 6 pentru Orin, astfel încât procesarea GPU, inclusiv debayerul Texture Aware, rulează pe GPU-ul Jetson. Pachetul nu conține CUDA în sine: utilizează bibliotecile CUDA și cuDNN instalate de JetPack 6 (`sudo apt install nvidia-jetpack` pe o imagine care nu le conține). În cazul în care nu se poate utiliza un GPU CUDA, procesarea se execută pe CPU, iar `--debayer texture-aware` utilizează debayer-ul Standard (backend-ul înregistrează o linie `[DEBAYER] … needs a CUDA GPU …` care explică motivul). Pentru informații privind memoria, temperatura și comportamentul în condiții de utilizare pe teren al Jetson, consultați [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md).***

## Instalarea Python SDK (toate versiunile de Linux)**`.deb` instalează automat SDK-ul.** Ambele pachete (amd64 și Jetson) îl instalează pentru sistemul `python3`, astfel încât `import chloros_sdk` funcționează imediat după instalare, fără pip și fără conexiune la rețea. Pachetele sale (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` și încărcătoarele de proiecte) sunt incluse în `/usr/lib/python3/dist-packages`, care aparține pachetului: o actualizare le înlocuiește, iar dezinstalarea pachetului le elimină. Versiunea corespunde întotdeauna cu interfața CLI și cu backend-ul pe care le-ați instalat.

Cerințele proprii ale SDK-ului, `python3-numpy` și `python3-requests`, sunt **recomandate, nu obligatorii** de către pachet: `sudo apt install ./chloros_<version>_<arch>.deb` le instalează, iar `dpkg -i` nu. Rezultatul instalării raportează versiunea SDK-ului pe care `python3` o importă, orice cerință care lipsește, precum și orice `chloros_sdk` din altă parte a căii de acces a `python3` care suprascrie copia pachetului (cum ar fi una instalată anterior cu `pip install --user`).**Într-un virtualenv**, care nu vede pachetele sistemului, instalați roata inclusă în acesta (complet offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**De pe PyPI** — numai pentru o mașină pe care nu este instalat pachetul Chloros, al cărei SDK controlează un backend pe o altă mașină (vezi [Modul Remote-Backend](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Opțiuni suplimentare, pentru o instalare prin pip (într-un virtualenv sau de pe PyPI):

| Opțiune | Comandă | Adaugă |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pentru transmiterea în timp real a progresului |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pentru transport BLE (DAQ-M) |

Verificați SDK-ul:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
SDK-ul Python comunică cu backend-ul pe care `.deb` îl instalează printr-un HTTP API local (`http://127.0.0.1:5000`). `ChlorosLocal` și utilitarele de conectare îl pornesc automat când este necesar; `connect_all()` al unui proiect salvat nu face niciodată acest lucru, așa că, pe o mașină fără interfață grafică, rulați backend-ul ca [serviciu](#always-on-ptp-for-headless-hosts). Folosiți întotdeauna adresa IPv4 literală în locul lui `localhost` — `localhost` se poate rezolva în `::1` și poate costa aproximativ două secunde pe cerere.
{% endhint %}

***

## Configurare inițială

### 1. Autentificare

Accesul la CLI și SDK necesită un plan Chloros+ plătit (**Copper** sau superior), impus la nivel de server: un apelant deconectat primește `401 AUTH_REQUIRED`, iar un apelant cu plan gratuit (Iron) primește `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Datele de autentificare sunt stocate în cache în `~/.chloros/user_session.json`.

{% hint style="info" %}
**Actualizările păstrează datele de autentificare; dezinstalarea le șterge.** Instalarea unei versiuni mai noi de `.deb` peste o instalare existentă (inclusiv prin `chloros-cli update`) păstrează `~/.chloros/user_session.json` și licența stocată în cache, astfel încât sarcinile CLI/SDK fără interfață grafică continuă să funcționeze. Deletarea pachetului (`sudo apt remove chloros`) șterge datele de autentificare salvate și cache-ul licenței pentru fiecare utilizator de pe mașină, astfel încât o instalare nouă necesită din nou `chloros-cli login`.
{% endhint %}

### 2. Verificați starea licenței

```bash
chloros-cli status
```

`chloros-cli status` funcționează pe orice nivel (inclusiv cel gratuit), astfel încât puteți vedea întotdeauna de ce accesul este sau nu disponibil.

### 3. Rulați diagnosticarea sistemului

```bash
chloros-cli selftest
```

Cele șapte verificări se execută în ordine, iar comanda returnează un cod de ieșire diferit de zero dacă oricare dintre ele eșuează:

| # | Verificare | Ce demonstrează |
| --- | --- | --- |
| 1 | **Versiune** | Interfața CLI-ului raportează versiunea sa (`v1.3.0`). |
| 2 | **Port disponibil** | Portul 5000 este liber *sau* a primit deja un răspuns de la un backend Chloros funcțional (ceea ce se consideră o reușită). |
| 3 | **Pornirea backend-ului** | Fișierul binar al backend-ului se lansează. |
| 4 | **Test API (`/api/test`)** | Backend-ul răspunde cu `status: ok`. |
| 5 | **Informații despre sistem** | Afișează `GPU: <name>, CUDA: <bool>, PyTorch: <version>` din `/api/system-info`. |
| 6 | **Modele de denoizare** | Găsește modele `*.pth.enc` (pe Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| Texture Aware este de fapt utilizabil — necesită CUDA**și** cel puțin un fișier de model. |

Execuția se încheie cu `N/7 checks passed`, listând toate eșecurile după nume.

### 4. Procesează primul tău set de date

```bash
chloros-cli process ~/datasets/flight001
```

***

## Fișiere și directoare

### Per utilizator

Chloros păstrează datele de autentificare și configurația CLI într-un singur director compatibil cu toate platformele, **`~/.chloros/`** (pe Windows, `%USERPROFILE%\.chloros\`). Două cache-uri specifice sistemului Linux respectă în schimb convențiile XDG — acestea utilizează `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` atunci când sunt configurate.

| Cale | Scop |
| --- | --- |
| `~/.chloros/user_session.json` | Cache-ul sesiunii de autentificare scris de `chloros-cli login` (păstrat la actualizările pachetului; șters la dezinstalarea pachetului) |
| `~/.chloros/working_directory.txt` | Suprascriere implicită a folderului de proiect (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferința de limbă pentru CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Setarea limbii partajată cu interfața grafică Windows — o valoare `language` are prioritate față de `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache de o oră pentru verificarea actualizărilor la pornire pe Linux/Jetson |
| `~/.chloros/backend.log` | Jurnalul backend-ului atunci când acesta a fost lansat prin CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pachete de calibrare LATTICE stocate în cache pentru fiecare cameră, indexate după numărul de serie și hash-ul pachetului |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Modificări opționale ale utilizatorului pentru profilurile de corecție a capacității DAQ |
| `~/.config/chloros/system_config.json` | Profil hardware stocat în cache din Dynamic Compute Adaptation — ștergeți-l pentru a forța o nouă detectare a hardware-ului |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Jurnale ale serverului backend, câte un fișier pentru fiecare lansare |
| `~/Documents/MAPIR/Chloros Projects/` | Dosarul implicit al proiectului atunci când nu este setată nicio suprascriere |

### La nivel de sistem

| Cale | Scop |
| --- | --- |
| `/usr/bin/chloros-cli` | Script de înveliș — setează `LD_LIBRARY_PATH` pentru bibliotecile native incluse în pachet (și `TMPDIR` la `/mnt/ssd/tmp` atunci când acel folder există), apoi execută binarul propriu-zis |
| `/usr/bin/chloros-backend` | Script de înveliș — același lucru pentru backend, pe care îl marchează ca o versiune de producție instalată |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Interfața CLI și backend-ul compilate, instalate în format nearsivat (nou în versiunea 1.3.0); programele sunt `cli/chloros-cli` și `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Scripturi de lansare de două linii care pornesc programele propriu-zise de mai sus — păstrate în aceste căi deoarece SDK-ul, CLI-ul și scripturile mai vechi le caută aici |
| `/usr/lib/chloros/arena_runtime/` | Runtime-ul SDK-ului Arena necesar pentru camerele LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modele de denoizare criptate utilizate de debayer-ul Texture Aware |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (și `lattice_sdk`, `lattice_cam`, `arena_api`, încărcătoarele de proiect) | SDK-ul Python, instalat pentru sistemul `python3` și aparținând pachetului |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Roată SDK SDK corespunzătoare exact acestei versiuni, pentru un virtualenv |
| `/usr/lib/chloros/exiftool` | Lansatorul ExifTool inclus în pachet (cu legătură simbolică către `/usr/local/bin/exiftool` numai dacă nu există un exiftool de sistem). Începând cu versiunea 1.3.0, scriptul ExifTool este livrat cu propria sa bibliotecă Perl în `/usr/lib/chloros/exiftool-dist/`, astfel încât rulează fără pachetul de sistem `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Configurația canalului de actualizare citită de `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Setează `net.ipv4.ip_unprivileged_port_start = 319` astfel încât backend-ul să poată lega porturile PTP fără drepturi de root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Mărește bufferele de recepție ale socket-urilor din kernel (`net.core.rmem_max`, `net.core.rmem_default`) la 32 MB pentru fluxurile camerelor LATTICE și le aplică la instalare (o valoare deja setată la un nivel mai mare rămâne neschimbată). La setarea implicită a kernelului, pachetele de la cameră sunt pierdute în kernel, iar o matrice pierde cadre și rata de cadre; `chloros-cli lattice network` raportează starea bufferelor atunci când acestea sunt pe terminate. A se vedea [Gazde Linux — tampoane de recepție pentru socketuri](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Direcționează încărcătorul dinamic către `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Acordă utilizatorului autentificat acces la puntea serială USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Serviciu backend „always-on” opțional care rulează backend-ul ca utilizatorul pe care îl numiți (instalat, **neactivat**) — acesta este cel care trebuie activat; consultați [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Același serviciu care rulează backend-ul ca **root**(instalat,**neactivat**), păstrat astfel încât o mașină care l-a activat să continue să funcționeze |
| `/var/log/chloros/` | Jurnalul detaliat al serviciului root, accesibil grupului `adm` (șters la eliminarea pachetului) |
| `/usr/share/applications/chloros-cli.desktop` | Intrare în meniul aplicației „Chloros CLI” care deschide un terminal |

**Firewall (nou în versiunea 1.3.0).** Dacă `ufw` sau `firewalld` este activ la momentul instalării, pachetul deschide porturile UDP `5002` (date DAQ-E), `5353` (descoperire mDNS), `319` și `320` (PTP) și elimină aceste reguli atunci când pachetul este dezinstalat. Dacă un firewall este instalat, dar nu este activ, acesta nu adaugă nimic și afișează porturile care trebuie permise în cazul în care îl activați ulterior. Nu deschide niciun port TCP: o actualizare de firmware DAQ-E, în cadrul căreia senzorul se reconectează la acest computer prin TCP pentru a colecta imaginea, eșuează cu codul de eroare `… device never connected back on TCP <port> …`, în timp ce un firewall activ blochează acea conexiune.

## Locația executabilului backend

CLI-ul și SDK-ul detectează automat backend-ul:

| Componentă | Cale |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (un program de lansare pentru `/usr/lib/chloros/backend/chloros-backend`) |

Suprascrieți calea backend-ului cu indicatorul CLI-ului `--backend-exe` sau cu parametrul constructorului SDK-ului `backend_exe`, iar portul cu `--port` (implicit `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` direcționează familiile de comenzi **`lattice`**,**`project`**și**`daq pool-*`** către un backend la distanță. Comenzile de bază (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) o ignoră în mod deliberat și vizează întotdeauna `http://127.0.0.1:<port>`.
{% endhint %}

***

## Camerele LATTICE și senzorii de lumină DAQ pe Linux

Toate familiile de comenzi live-hardware funcționează pe Linux (amd64 și Jetson):

* **`chloros-cli lattice`** — detectează, conectează, configurează și captează date de la camerele LATTICE și de la matrice sincronizate. `.deb` include mediul de execuție Arena SDK necesar și îl înregistrează în încărcătorul dinamic.
* **`chloros-cli daq pool-*`** — conectează senzorii de lumină DAQ-U/M/E prin intermediul grupului de backend, transmite în flux spectre calibrate, înregistrează fișiere `.daq` și schimbă profilurile de capacitate, iar pentru un DAQ-E modifică setările de rețea, îl repornește și îi actualizează firmware-ul. `chloros-cli daq discover` caută senzorii la care v-ați putea conecta, iar `chloros-cli daq --help` afișează lista subcomenzilor `pool-*`.
* **`chloros-cli project`** — rulează un proiect salvat (camerele, senzorii și setările de procesare ale acestuia) fără interfață grafică.
* **`chloros-cli time-sync`** — inspectează grandmasterul PTP pe care backend-ul Chloros îl rulează pentru camerele LATTICE și senzorii DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` este necesară pentru `pool-latest`, `pool-stream`, `pool-record` și `pool-set-cap`; `pool-list` afișează ID-urile aflate în prezent în pool.

{% hint style="info" %}
**`--eth-host` omite etapa de descoperire.** `pool-connect --eth` găsește un DAQ-E prin mDNS, cu o soluție de rezervă ARP care funcționează dintr-un cache ARP gol; transmiterea adresei IP sau a numelui de gazdă al senzorului cu `--eth-host` permite conectarea directă la acesta.
{% endhint %}

**Permisiunile seriale DAQ-U** sunt gestionate de regula udev instalată (`uaccess` + grupul `dialout`). Dacă un senzor care era deja conectat rămâne inaccesibil, reîncărcați regulile sau reconectați-l:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consultați [referința CLI](../CLI.md) pentru setul complet de comenzi.

### PTP permanent activ pentru gazde fără interfață grafică

Pachetul instalează două servicii systemd pentru backend, niciunul dintre ele nefiind activat. Pe un Jetson fără monitor sau pe un server care trebuie să mențină sincronizarea temporală PTP în funcțiune continuă pentru senzorii DAQ-E și camerele LATTICE, sau în cazul în care scripturile SDK conectează proiecte salvate (`connect_all()` nu pornește niciodată un backend), activați serviciul **ca utilizator**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` rulează backend-ul ca acel utilizator: capturile pe care le scrie sunt ale acestuia, jurnalul său detaliat se află în `~/.cache/chloros/logs` al acestuia și utilizează `chloros-cli login` (`~/.chloros`), așa că autentificați-vă mai întâi o dată ca acel utilizator. Instalarea pachetului afișează această comandă cu numele dvs. de utilizator completat.

Fără un serviciu, backend-ul (și, odată cu acesta, grandmasterul PTP) rulează doar atâta timp cât este pornit de ceva — o comandă `chloros-cli` sau un script SDK.

Unitatea leagă backend-ul de setările de mediu `127.0.0.1:5000` (setările de mediu `CHLOROS_HOST` / `CHLOROS_PORT` din interiorul unității; se pot suprascrie cu `sudo systemctl edit chloros-backend@.service`) și îl repornește în caz de eșec după 5 secunde.

**Serviciul root.**`chloros-backend.service` rulează backend-ul ca**root**. Acesta rămâne instalat astfel încât o mașină care l-a activat să continue să funcționeze, dar tot ce scrie un backend root aparține utilizatorului root, astfel încât un utilizator nu își poate șterge propriile capturi; jurnalul său se află în `/var/log/chloros`, accesibil grupului `adm`. Pentru a trece la contul dvs. de utilizator:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backend-ul unui serviciu este partajat și nu este oprit niciodată de un client.** Scripturile `chloros-cli` și SDK îl utilizează așa cum îl găsesc: `process` rulează pe acesta, încărcându-și propriul proiect în locul celui anterior. În cazul în care CLI-ul ar reporni altfel backend-ul (`--restart` sau un `process` al cărui import nu a găsit nimic), acesta numește unitatea, afișează `sudo systemctl restart <unit>` și iese din `1`, deoarece un backend oprit în acest mod ar rămâne inactiv. Reporniți-l manual cu `sudo systemctl restart chloros-backend@$USER.service`. Un backend al unui serviciu nu se oprește niciodată singur când este inactiv, iar `chloros-cli backend stop` îl refuză, indicând comanda `systemctl stop`.**Cum obține PTP porturile sale.** PTP utilizează porturile UDP 319/320, ambele situate sub pragul normal de 1024 pentru porturile privilegiate. Pachetul `postinst` scrie `/etc/sysctl.d/60-chloros-ptp.conf` cu `net.ipv4.ip_unprivileged_port_start = 319`, ceea ce permite backend-ului să le lege în timp ce rulează ca utilizatorul dvs. Începând cu versiunea 1.3.0, pachetul nu mai setează nicio capacitate de fișier (`setcap`) pe backend, iar `libcap2-bin` nu mai este o dependență: pe backend-ul despachetat, o capacitate de fișier l-ar împiedica să pornească, așa că nu adăugați una singuri.***

## Exemple de scripturi Bash

{% hint style="info" %}
**Coduri de ieșire compatibile cu scripturile.**`chloros-cli process` încheie `0` în caz de succes și**cu o valoare diferită de zero în caz de eșec — inclusiv o execuție care a solicitat produse imagistice, dar nu a generat niciunul** (afișează `Processing finished but wrote no image products.` și menționează numele folderului proiectului și cauzele obișnuite). Execuțiile reușite raportează câte produse imagistice au fost scrise (`Image products written: N`, plus `(plus M clamp masks)` când au fost scrise măști). Coduri de ieșire: `0` succes, `1` eșec, `2` eroare de argument, `130` întrerupt.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Valorile valide pentru `--format` sunt exact patru și conțin spații — puneți-le întotdeauna între ghilimele:

| Valoare `--format` | Folder de ieșire |
| --- | --- |
| `TIFF (16-bit)` *(implicit)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` acceptă `standard` (implicit) sau `texture-aware` (Chloros+).

### Procesare automată cu Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemplu de SDK-Python pentru Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Depanare

### CLI-ul nu a fost găsit după instalare

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend-ul nu pornește după adăugarea capacităților de fișiere

Nu rulați `setcap` pe backend. Începând cu versiunea 1.3.0, pachetul nu setează nicio capacitate de fișier, iar porturile 319/320 ale PTP provin din setarea `sysctl` descrisă în secțiunea [Cum obține PTP porturile sale](#always-on-ptp-for-headless-hosts). O capacitate de fișier plasează backend-ul despachetat într-un mod de execuție securizată în care acesta nu își poate găsi propriile biblioteci, astfel încât nu pornește. Dacă ați adăugat una, eliminați-o:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
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

### Comenzile `lattice` eșuează cu mesajul „Drivere cameră LATTICE neidentificate”

Backend-ul nu a putut încărca runtime-ul camerei LATTICE cu care este instalat. Fiecare comandă `lattice` rulează pe backend; interfața CLI-ului în sine nu conține drivere de cameră. Reinstalați Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` instalează mediul de execuție alături de backend, în `/usr/lib/chloros/arena_runtime`, și îl înregistrează în încărcătorul dinamic. Pentru a verifica această înregistrare:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Backend-ul nu a putut fi pornit

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Jurnalele backend-ului pentru lansarea eșuată se găsesc în `~/.cache/chloros/logs/`. Dacă portul 5000 aparține unui serviciu backend (`systemctl status chloros-backend@$USER.service`), reporniți serviciul în loc să opriți procesul acestuia.

### CUDA nedetectat

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` raportează același lucru într-o singură linie: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Biblioteci partajate lipsă

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Pornire lentă pe sistemele cu card SD

Începând cu versiunea 1.3.0, backend-ul și interfața CLI sunt instalate în format nearsivat, astfel încât nu se mai extrag automat într-un director temporar la fiecare lansare. Fișierele temporare sunt mutate automat în `/mnt/ssd/tmp` dacă acel folder există; în caz contrar, le puteți direcționa către un sistem de fișiere rapid:

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

Pe Linux/Jetson, CLI-ul efectuează, de asemenea, o verificare de actualizare fără blocare la fiecare pornire (rezultatul fiind stocat în cache timp de o oră în `~/.chloros/update_cache.json`) și afișează `Update available: vX.Y.Z` atunci când există o versiune mai nouă. Setările, proiectele și datele de autentificare rămân intacte după o actualizare.

Actualizarea pachetului oprește toate serviciile backend pe durata instalării, astfel încât un script care rulează își pierde camerele și senzorii, iar apoi repornește fiecare dintre cele care sunt activate (unul pornit manual, dar care nu a fost niciodată activat, rămâne oprit, la fel ca după o repornire a sistemului).

## Dezinstalare

```bash
sudo apt remove chloros
```

Dezinstalarea oprește și dezactivează toate serviciile backend, restabilește limita implicită a porturilor fără privilegii (1024), elimină fișierul de configurare al bufferului de recepție (valorile curente rămân până la repornire), legătura simbolică către exiftool inclusă în pachet, configurația încărcătorului Arena, regulile de firewall adăugate de pachet și SDK-ul Python instalat de acesta, și șterge datele de autentificare stocate în cache. Proiectele dvs. și fișierele de date `~/.chloros/` rămân intacte. `sudo apt purge chloros` șterge, de asemenea, `/var/log/chloros`.

***

## Pași următori

* [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md) — Optimizare și implementare specifice pentru Jetson
* [CLI: Linia de comandă](../CLI.md) — ghidul CLI
* [API: Python SDK](../api-python-sdk.md) — ghidul SDK
* [Referință CLI](../reference/cli-reference.md) și [Referință SDK](../reference/sdk-reference.md) — liste exhaustive de comenzi/API pentru versiunea 1.3.0
* [Adaptare dinamică a puterii de calcul](../processing-architecture/dynamic-compute-adaptation.md) — modul în care Chloros se adaptează la hardware-ul dvs.
