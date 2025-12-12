# API : Python SDK

**Chloros Python SDK** oferă acces programatic la motorul de procesare a imaginilor Chloros, permițând automatizarea, fluxurile de lucru personalizate și integrarea perfectă cu aplicațiile și canalele de cercetare Python.

### Caracteristici cheie

* 🐍 **Python nativ** - API curat, Pythonic pentru procesarea imaginilor
* 🔧 **Acces complet la API** - Control complet asupra procesării Chloros
* 🚀 **Automatizare** - Creați fluxuri de lucru personalizate pentru procesarea în lot
* 🔗 **Integrare** - Încorporați Chloros în aplicațiile Python existente
* 📊 **Gata pentru cercetare** - Perfect pentru fluxurile de analiză științifică
* ⚡ **Procesare paralelă** - Se adaptează la nucleele CPU (Chloros+)

### Cerințe

| Cerință          | Detalii                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Trebuie instalat local                                           |
| **Licență**          | Chloros+ ([plan plătit necesar](https://cloud.mapir.camera/pricing)) |
| **Sistem de operare** | Windows 10/11 (64 biți)                                              |
| **Python**           | Python 3.7 sau o versiune superioară                                                |
| **Memorie**           | Minim 8 GB RAM (recomandat 16 GB)                                  |
| **Internet**         | Necesar pentru activarea licenței                                     |

{% hint style=&quot;warning&quot; %}
**Cerințe de licență**: Python SDK necesită un abonament plătit Chloros+ pentru accesul la API. Planurile standard (gratuite) nu au acces la API/SDK. Vizitați [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) pentru a face upgrade.
{% endhint %}

## Începere rapidă

### Instalare

Instalați prin pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Configurare inițială**: Înainte de a utiliza SDK, activați licența Chloros+ deschizând Chloros, Chloros (browser) sau Chloros CLI și conectându-vă cu datele dvs. de autentificare. Acest lucru trebuie făcut o singură dată.
{% endhint %}

### Utilizare de bază

Procesați un folder cu doar câteva linii:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Control complet

Pentru fluxuri de lucru avansate:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Ghid de instalare

### Cerințe preliminare

Înainte de a instala SDK, asigurați-vă că aveți:

1. **Chloros Desktop** instalat ([descărcare](download.md))
2. **Python 3.7+** instalat ([python.org](https://www.python.org))
3. **Licență Chloros+ activă** ([upgrade](https://cloud.mapir.camera/pricing))

### Instalare prin pip

**Instalare standard:**

```bash
pip install chloros-sdk
```

**Cu suport pentru monitorizarea progresului:**

```bash
pip install chloros-sdk[progress]
```

**Instalare pentru dezvoltare:**

```bash
pip install chloros-sdk[dev]
```

### Verificarea instalării

Testați dacă SDK este instalat corect:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Configurare inițială

### Activarea licenței

SDK utilizează aceeași licență ca Chloros, Chloros (Browser) și Chloros CLI. Activați o singură dată prin GUI sau CLI:

1. Deschideți **Chloros sau Chloros (Browser)** și conectați-vă la fila Utilizator <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Sau deschideți **CLI**.
2. Introduceți datele de autentificare Chloros+ și conectați-vă
3. Licența este stocată în memoria cache locală (persistă după repornire)

{% hint style=&quot;success&quot; %}
**Configurare unică**: După conectarea prin GUI sau CLI, SDK utilizează automat licența stocată în cache. Nu este necesară nicio autentificare suplimentară!
{% endhint %}

### Testați conexiunea

Verificați dacă SDK se poate conecta la Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Referință API

### Clasa ChlorosLocal

Clasă principală pentru procesarea locală a imaginilor Chloros.

#### Constructor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametri:**

| Parametru                 | Tip | Implicit                   | Descriere                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL al backend-ului local Chloros          |
| `auto_start_backend`      | bool | `True`                    | Pornește automat backend-ul dacă este necesar |
| `backend_exe`             | str  | `None` (detectare automată)      | Calea către executabilul backend            |
| `timeout`                 | int  | `30`                      | Timp de expirare a cererii în secunde            |
| `backend_startup_timeout` | int  | `60`                      | Timp de expirare pentru pornirea backend-ului(secunde) |

**Exemple:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metode

#### `create_project(project_name, camera=None)`

Creați un nou proiect Chloros.

**Parametri:**

| Parametru      | Tip | Obligatoriu | Descriere                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Da      | Numele proiectului                                     |
| `camera`       | str  | Nu       | Șablon cameră (de exemplu, „Survey3N\_RGN”, „Survey3W\_OCN”) |

**Returnează:** `dict` - Răspuns la crearea proiectului

**Exemplu:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importă imagini dintr-un folder.

**Parametri:**

| Parametru     | Tip     | Obligatoriu | Descriere                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Da      | Calea către folderul cu imagini         |
| `recursive`   | bool     | Nu       | Căutare subfoldere (implicit: False) |

**Returnează:** `dict` - Rezultatele importului cu numărul de fișiere

**Exemplu:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configurează setările de procesare.

**Parametri:**

| Parametru                 | Tip | Implicit                 | Descriere                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | „Calitate înaltă (mai rapidă)” | Metoda Debayer                  |
| `vignette_correction`     | bool | `True`                  | Activare corecție vignetă      |
| `reflectance_calibration` | bool | `True`                  | Activare calibrare reflectanță  |
| `indices`                 | list | `None`                  | Indici de vegetație pentru calcul |
| `export_format`           | str  | „TIFF (16 biți)”         | Format de ieșire                   |
| `ppk`                     | bool | `False`                 | Activare corecții PPK          |
| `custom_settings`         | dict | `None`                  | Setări personalizate avansate        |

**Formate de export:**

* `"TIFF (16-bit)"` - Recomandat pentru GIS/fotogrammetrie
* `"TIFF (32-bit, Percent)"` - Analiză științifică
* `"PNG (8-bit)"` - Inspecție vizuală
* `"JPG (8-bit)"` - Ieșire comprimată

**Indici disponibili:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 și altele.

**Exemplu:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Procesați imaginile proiectului.

**Parametri:**

| Parametru           | Tip     | Implicit      | Descriere                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Mod de procesare: „paralel” sau „serial”   |
| `wait`              | bool     | `True`       | Așteptare finalizare                       |
| `progress_callback` | callable | `None`       | Funcție de apel invers pentru progres (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interval de interogare pentru progres (secunde)   |

**Returnează:** `dict` - Rezultate procesare

{% hint style=&quot;warning&quot; %}
**Mod paralel**: Necesită licență Chloros+. Se adaptează automat la nucleele procesorului (până la 16 procesoare).
{% endhint %}

**Exemplu:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Obține configurația curentă a proiectului.

**Returnează:** `dict` - Configurația curentă a proiectului

**Exemplu:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Obține informații despre starea backend-ului.

**Returnează:** `dict` - Starea backend-ului

**Exemplu:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Oprește backend-ul (dacă a fost pornit de SDK).

**Exemplu:**

```python
chloros.shutdown_backend()
```

***

### Funcții utile

#### `process_folder(folder_path, **options)`

Funcție utilă dintr-o singură linie pentru procesarea unui folder.

**Parametri:**

| Parametru                 | Tip     | Implicit         | Descriere                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obligatoriu        | Calea către folderul cu imagini     |
| `project_name`            | str      | Generat automat  | Numele proiectului                   |
| `camera`                  | str      | `None`          | Șablon cameră                |
| `indices`                 | list     | `["NDVI"]`      | Indici de calculat           |
| `vignette_correction`     | bool     | `True`          | Activare corecție vignetă     |
| `reflectance_calibration` | bool     | `True`          | Activare calibrare reflectanță |
| `export_format`           | str      | „TIFF (16 biți)” | Format de ieșire                  |
| `mode`                    | str      | `"parallel"`    | Mod de procesare                |
| `progress_callback`       | callable | `None`          | Apel invers de progres              |

**Returnează:** `dict` - Rezultate de procesare

**Exemplu:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Suport pentru managerul de context

SDK acceptă manageri de context pentru curățarea automată:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Exemple complete

### Exemplul 1: Prelucrare de bază

Prelucrați un folder cu setările implicite:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Exemplul 2: Flux de lucru personalizat

Control complet asupra fluxului de prelucrare:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Exemplul 3: Procesare în lot a mai multor foldere

Procesarea mai multor seturi de date de zbor:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Exemplul 4: Integrarea fluxului de cercetare

Integrarea Chloros cu analiza datelor:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Exemplul 5: Monitorizarea personalizată a progresului

Urmărirea avansată a progresului cu înregistrare:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Exemplul 6: Gestionarea erorilor

Gestionarea robustă a erorilor pentru utilizare în producție:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Exemplul 7: Instrument de linie de comandă

Creați un instrument CLI personalizat cu SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Utilizare:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Gestionarea excepțiilor

SDK oferă clase de excepții specifice pentru diferite tipuri de erori:

### Ierarhia excepțiilor

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Exemple de excepții

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Subiecte avansate

### Configurarea personalizată a backend-ului

Utilizați o locație sau o configurație personalizată a backend-ului:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Procesare fără blocare

Începeți procesarea și continuați cu alte sarcini:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Gestionarea memoriei

Pentru seturi de date mari, procesați în loturi:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Depanare

### Backend-ul nu pornește

**Problemă:** SDK nu reușește să pornească backend-ul.

**Soluții:**

1. Verificați dacă Chloros Desktop este instalat:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Verificați dacă Windows Firewall nu blochează
3. Încercați calea manuală a backend-ului:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licența nu este detectată

**Problemă:** SDK avertizează că licența lipsește

**Soluții:**

1. Deschideți Chloros, Chloros (browser) sau Chloros CLI și conectați-vă.
2. Verificați dacă licența este stocată în cache:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Contactați asistența tehnică: info@mapir.camera

***

### Erori de import

**Problemă:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Soluții:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Timp de procesare expirat

**Problemă:** Timpul de procesare a expirat

**Soluții:**

1. Măriți timpul de expirare:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Procesați loturi mai mici
3. Verificați spațiul disponibil pe disc
4. Monitorizați resursele sistemului

***

### Port deja utilizat

**Problemă:** Portul backend 5000 ocupat

**Soluții:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Sau găsiți și închideți procesul conflictual:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Sfaturi de performanță

### Optimizați viteza de procesare

1. **Utilizați modul paralel** (necesită Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Reduceți rezoluția de ieșire** (dacă este acceptabil)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Dezactivați indicii inutili**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Procesați pe SSD** (nu pe HDD)

***

### Optimizarea memoriei

Pentru seturi de date mari:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Procesare în fundal

Eliberați Python pentru alte sarcini:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Exemple de integrare

### Integrare Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Întrebări frecvente

### Î: SDK necesită o conexiune la internet?

**R:** Doar pentru activarea inițială a licenței. După conectarea prin Chloros, Chloros (browser) sau Chloros CLI, licența este stocată local în memoria cache și funcționează offline timp de 30 de zile.

***

### Î: Pot utiliza SDK pe un server fără GUI?

**R:** Da! Cerințe:

* Windows Server 2016 sau o versiune ulterioară
* Chloros instalat (o singură dată)
* Licență activată pe orice computer (licență stocată în cache copiată pe server)

***

### Î: Care este diferența între Desktop, CLI și SDK?

| Caracteristică         | GUI Desktop | CLI Linie de comandă | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interfață**   | Point-click | Comandă          | Python API  |
| **Cel mai potrivit pentru**    | Lucru vizual | Scripting        | Integrare |
| **Automatizare**  | Limitată     | Bună             | Excelentă   |
| **Flexibilitate** | De bază       | Bună             | Maximă     |
| **Licență**     | Chloros+    | Chloros+         | Chloros+    |

***

### Î: Pot distribui aplicații create cu SDK?

**R:** Codul SDK poate fi integrat în aplicațiile dvs., dar:

* Utilizatorii finali trebuie să aibă instalat Chloros
* Utilizatorii finali trebuie să aibă licențe active Chloros+
* Distribuția comercială necesită licențe OEM.

Contactați info@mapir.camera pentru întrebări legate de OEM.

***

### Î: Cum actualizez SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Î: Unde sunt salvate imaginile procesate?

În mod implicit, în calea proiectului:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Î: Pot procesa imagini din scripturile Python care rulează conform programului?

**R:** Da! Utilizați Windows Task Scheduler cu scripturile Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Programați prin Task Scheduler să ruleze zilnic.

***

### Î: SDK acceptă async/await?

**R:** Versiunea actuală este sincronă. Pentru comportament asincron, utilizați `wait=False` sau rulați într-un thread separat:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Obținerea de ajutor

### Documentație

* **Referință API**: această pagină

### Canale de asistență

* **E-mail**: info@mapir.camera
* **Site web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Prețuri**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Cod de exemplu

Toate exemplele enumerate aici sunt testate și gata de producție. Copiați-le și adaptați-le pentru cazul dvs. de utilizare.

***

## Licență

**Software proprietar** - Copyright (c) 2025 MAPIR Inc.

SDK necesită un abonament activ Chloros+. Utilizarea, distribuirea sau modificarea neautorizată este interzisă.
