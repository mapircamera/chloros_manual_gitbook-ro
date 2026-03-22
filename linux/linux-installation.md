# Instalarea Linux

Chloros este distribuit pentru Linux sub forma unor pachete `.deb` care instalează CLI și backend-ul. Python SDK se instalează separat prin pip.

***

## Linux amd64 (x86_64)

### Cerințe de sistem

| Cerință | Minimă | Recomandată |
| --- | --- | --- |
| **Distribuție** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 sau superior |
| **Memorie (RAM)** | 8 GB | 16 GB sau mai mult |
| **Placă grafică** | Niciuna (procesare CPU) | GPU NVIDIA cu 4 GB+ VRAM |
| **Spațiu de stocare** | 2 GB spațiu liber | SSD cu 10 GB+ spațiu liber |
| **Python** | Python 3.7+ (pentru SDK) | Python 3.10+ |

### Instalare

Descărcați pachetul `.deb` și instalați-l:

```bash
sudo dpkg -i chloros-amd64.deb
```

Verificați instalarea:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Cerințe de sistem

| Cerință | Minim | Recomandat |
| --- | --- | --- |
| **Platformă** | NVIDIA Jetson cu JetPack 6 | Jetson Orin NX 16 GB sau AGX Orin |
| **JetPack** | JetPack 6.x | Ultima versiune JetPack 6 |
| **Memorie (RAM)** | 8 GB (GPU/CPU partajată) | 16 GB+ partajată |
| **Spațiu de stocare** | 2 GB spațiu liber | SSD NVMe cu 10 GB+ spațiu liber |
| **Python** | Python 3.7+ (pentru SDK) | Python 3.10+ |

### Instalare

Descărcați pachetul JetPack 6 `.deb` și instalați-l:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Verificați instalarea:

```bash
chloros-cli --version
```

Pentru configurarea detaliată a Jetson, inclusiv gestionarea termică și implementarea pe teren, consultați [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Instalare (Toate Linux)

Python SDK se instalează separat prin pip și funcționează atât pe amd64, cât și pe arm64:

```bash
pip install chloros-sdk
```

Pentru a include suportul opțional pentru streaming de progres:

```bash
pip install chloros-sdk[progress]
```

Verificați SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pachetul `.deb` instalează Chloros, CLI și backend-ul. Python SDK este un pachet pip separat care comunică cu backend-ul prin intermediul unui HTTP API local.
{% endhint %}

***

## Directoare de configurare

Chloros de pe Linux respectă [Specificația directorului de bază XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Scop | Linux Cale | Windows Echivalent |
| --- | --- | --- |
| **Configurare** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Date / Proiecte** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Credențiale** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Locații executabile backend

Pachetul `.deb` instalează backend-ul într-o locație standard. Pachetele CLI și SDK detectează automat calea backend-ului:

| Metoda de instalare | Calea backend-ului |
| --- | --- |
| Pachetul `.deb` | `/usr/lib/chloros/chloros-backend` |
| Manual / personalizat | `/opt/mapir/chloros/backend/chloros-backend` |

Puteți suprascrie calea backend-ului cu indicatorul `--backend-exe` CLI sau cu parametrul constructorului `backend_exe` SDK.

***

## Configurare inițială

### 1. Activați licența

Este necesară o licență Chloros+ pentru accesul la CLI și SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Verificați starea licenței

```bash
chloros-cli status
```

### 3. Prelucrați primul set de date

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Rulați diagnosticarea sistemului

Verificați dacă sistemul dvs. este configurat corect:

```bash
chloros-cli selftest
```

Aceasta execută 7 verificări de diagnosticare, inclusiv versiunea, pornirea backend-ului, API conectivitatea și disponibilitatea CUDA/GPU.

***

## Exemple de scripturi Bash

### Procesați mai multe seturi de date

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Procesați cu setări personalizate

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Procesare automată cu Cron

Adăugați la crontab (`crontab -e`) pentru a procesa automat seturi de date noi:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Exemplu

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

## Depanare

### CLI Nu a fost găsit după instalare

Dacă `chloros-cli` nu este găsit după instalarea pachetului `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Permisiune refuzată

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### CUDA nedetectat

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Biblioteci partajate lipsă

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Actualizarea Chloros pe Linux

Utilizați comanda de actualizare încorporată pentru a verifica și a instala actualizările:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Pași următori

* [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md) — Optimizare și implementare specifică pentru Jetson
* [CLI : Linia de comandă](../CLI.md) — Referință completă pentru comanda CLI
* [API : Python SDK](../api-python-sdk.md) — Referință completă pentru SDK
* [Adaptare dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md) — Cum se adaptează Chloros la hardware-ul dvs.
