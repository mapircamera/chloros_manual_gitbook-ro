# CLI : Linia de comandă

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** oferă acces puternic prin linia de comandă la motorul de procesare a imaginilor Chloros, permițând automatizarea, scriptarea și funcționarea fără interfață grafică pentru fluxurile dvs. de lucru de imagistică.

### Caracteristici cheie

* 🚀 **Automatizare** - Procesare scriptată în lot a mai multor seturi de date
* 🔗 **Integrare** - Încorporare în fluxurile de lucru și pipeline-urile existente
* 💻 **Funcționare fără interfață grafică** - Rulare fără GUI
* 🌍 **Multilingv** - Suport pentru 38 de limbi
* ⚡ **Procesare paralelă** - [Adaptarea dinamică a calculului](processing-architecture/dynamic-compute-adaptation.md) se optimizează automat pentru hardware-ul dvs.

### Cerințe

| Cerință          | Detalii                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistem de operare** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licență**          | Chloros+ ([este necesar un plan plătit](https://cloud.mapir.camera/pricing)) |
| **Memorie**           | Minim 8 GB RAM (se recomandă 16 GB)                                  |
| **Internet**         | Necesar pentru activarea licenței                                     |
| **Spațiu pe disc**       | Variază în funcție de dimensiunea proiectului                                              |

{% hint style="warning" %}
**Cerințe de licență**: CLI necesită un abonament Chloros+ plătit. Planurile standard (gratuite) nu au acces la CLI. Accesați [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) pentru a face upgrade.
{% endhint %}

## Ghid de pornire rapidă

### Instalare

#### Windows

CLI este inclus automat în programul de instalare Chloros:

1. Descărcați și rulați **Chloros Installer.exe**

2. Finalizați expertul de instalare
3. CLI instalat în: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Programul de instalare adaugă automat `chloros-cli` la PATH-ul sistemului. Reporniți terminalul după instalare.
{% endhint %}

#### Linux

Instalați pachetul `.deb` pentru arhitectura dvs.:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Pentru configurarea detaliată a Linux, consultați [Instalarea Linux](linux/linux-installation.md).

### Configurarea inițială

Înainte de a utiliza CLI, activați licența Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Utilizare de bază

Procesați un folder cu setările implicite:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referință comenzi

### Sintaxă generală

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comenzi

### `process` - Procesare imagini

Procesarea imaginilor dintr-un folder cu calibrare.

**Sintaxă:**

```bash
chloros-cli process <input-folder> [options]
```

**Exemple:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Opțiuni ale comenzii de procesare

| Opțiune                | Tip    | Implicit        | Descriere                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Cale    | _Obligatoriu_     | Dosar care conține imagini multispectrale RAW/JPG                                         |
| `-o, --output`        | Cale    | La fel ca intrarea  | Dosar de ieșire pentru imaginile procesate                                                     |
| `-n, --project-name`  | Șir  | Generat automat | Nume personalizat al proiectului                                                                    |
| `--vignette`          | Indicator    | Activat        | Activează corectarea vignetării                                                             |
| `--no-vignette`       | Indicator    | -              | Dezactivează corectarea vignetării                                                            |
| `--reflectance`       | Indicator    | Activat        | Activează calibrarea reflectanței                                                         |
| `--no-reflectance`    | Indicator    | -              | Dezactivează calibrarea reflectanței                                                        |
| `--ppk`               | Indicator    | Dezactivat       | Aplică corecții PPK din datele senzorului de lumină .daq                                      |
| `--format`            | Opțiune  | TIFF (16 biți)  | Format de ieșire: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Număr întreg | Auto           | Dimensiunea minimă a țintei în pixeli pentru detectarea panoului de calibrare                          |
| `--target-clustering` | Număr întreg | Auto           | Pragul de grupare a țintelor (0-100)                                                    |
| `--debayer`           | Opțiune  | `standard`     | Metoda de debayer: `standard` sau `texture-aware` (numai Chloros+)                          |
| `--target`, `--targets` | Indicator  | Dezactivat       | Căutați ținte de calibrare numai într-un subfolder „target” sau „targets” (accelerează procesarea) |
| `--indices`           | Listă    | Niciuna           | Indici de vegetație de calculat (de ex., `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Șir  | Niciuna           | Blochează expunerea pentru modelul de cameră (Pin 1)                                                 |
| `--exposure-pin-2`    | Șir  | Niciunul           | Blocare expunere pentru modelul camerei (Pin 2)                                                 |
| `--recal-interval`    | Număr întreg | Auto           | Interval de recalibrare în secunde                                                      |
| `--timezone-offset`   | Număr întreg | 0              | Decalaj de fus orar în ore                                                               |

***

### `login` - Autentificare cont

Conectați-vă cu datele de autentificare Chloros+ pentru a activa procesarea CLI.

**Sintaxă:**

```bash
chloros-cli login <email> <password>
```

**Exemplu:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caractere speciale**: Folosiți ghilimele simple în jurul parolelor care conțin caractere precum `$`, `!` sau spații.
{% endhint %}

**Rezultat:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Ștergeți datele de autentificare

Ștergeți datele de autentificare stocate și deconectați-vă de la contul dvs.

**Sintaxă:**

```bash
chloros-cli logout
```

**Exemplu:**

```bash
chloros-cli logout
```

**Rezultat:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Utilizatori SDK**: Python SDK oferă, de asemenea, o metodă programatică `logout()` pentru ștergerea datelor de autentificare în cadrul scripturilor Python. Consultați [documentația Python SDK](api-python-sdk.md#logout) pentru detalii.
{% endhint %}

***

### `status` - Verificarea stării licenței

Afișează starea actuală a licenței și a autentificării.

**Sintaxă:**

```bash
chloros-cli status
```

**Exemplu:**

```bash
chloros-cli status
```

**Rezultat:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Verificare progres export

Monitorizează progresul exportului Thread 4 în timpul sau după procesare.

**Sintaxă:**

```bash
chloros-cli export-status
```

**Exemplu:**

```bash
chloros-cli export-status
```

**Caz de utilizare:** Apelați această comandă în timp ce procesarea este în curs de desfășurare pentru a verifica progresul exportului.***

### `language` - Gestionați limba interfeței

Vizualizați sau modificați limba interfeței CLI.

**Sintaxă:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Exemple:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Limbi acceptate (38 în total)

| Cod    | Limbă              | Nume nativ      |
| ------- | --------------------- | ---------------- |
| `en`    | Engleză               | English          |
| `es`    | Spaniolă               | Español          |
| `pt`    | Portugheză            | Português        |
| `fr`    | Franceză                | Français         |
| `de`    | Germană                | Deutsch          |
| `it`    | Italiană               | Italiano         |
| `ja`    | Japoneză              | 日本語              |
| `ko`    | Coreeană                | 한국어              |
| `zh`    | Chineză (simplificată)  | 简体中文             |
| `zh-TW` | Chineză (tradițională) | 繁體中文             |
| `ru`    | Rusă               | Русский          |
| `nl`    | Olandeză                | Nederlands       |
| `ar`    | Arabă                | العربية          |
| `pl`    | Poloneză                | Polski           |
| `tr`    | Turcă               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indoneziană            | Bahasa Indonesia |
| `vi`    | Vietnameză            | Tiếng Việt       |
| `th`    | Thailandeză                  | ไทย              |
| `sv`    | Suedeză               | Svenska          |
| `da`    | Daneză                | Dansk            |
| `no`    | Norvegiană             | Norsk            |
| `fi`    | Finlandeză               | Suomi            |
| `el`    | Greacă                 | Ελληνικά         |
| `cs`    | Cehă                 | Čeština          |
| `hu`    | Maghiară             | Magyar           |
| `ro`    | Română              | Română           |
| `uk`    | Ucraineană             | Українська       |
| `pt-BR` | Portugheză braziliană  | Português Brasileiro |
| `zh-HK` | Cantoneză             | 粵語             |
| `ms`    | Malay                 | Bahasa Melayu    |
| `sk`    | Slovacă                | Slovenčina       |
| `bg`    | Bulgară             | Български        |
| `hr`    | Croată              | Hrvatski         |
| `lt`    | Lituaniană            | Lietuvių         |
| `lv`    | Letonă               | Latviešu         |
| `et`    | Estonă              | Eesti            |
| `sl`    | Slovenă             | Slovenščina      |

{% hint style="success" %}
**Persistență automată**: Preferința dvs. de limbă este salvată în `~/.chloros/cli_language.json` și se păstrează pe durata tuturor sesiunilor.
{% endhint %}

***

### `set-project-folder` - Setare folder proiect implicit

Modificați locația folderului de proiect implicit (partajat cu GUI în Windows).

**Sintaxă:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exemple:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Afișează folderul proiectului

Afișează locația curentă a folderului implicit al proiectului.

**Sintaxă:**

```bash
chloros-cli get-project-folder
```

**Exemplu:**

```bash
chloros-cli get-project-folder
```

**Rezultat:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Resetare la valorile implicite

Resetează folderul proiectului la locația implicită.

**Sintaxă:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Executare diagnosticare sistem

Execută 7 verificări de diagnosticare pentru a verifica configurația sistemului.

**Sintaxă:**

```bash
chloros-cli selftest
```

**Diagnostice efectuate:**

1. Verificare versiune
2. Disponibilitate port (5000)
3. Pornire backend
4. Test de conectivitate API
5. Informații sistem și detectare GPU
6. Verificare modele de denoizare
7. Verificare disponibilitate CUDA

{% hint style="info" %}
**Util pentru depanare**: Rulați `selftest` după instalare pentru a verifica dacă sistemul dvs. este configurat corect, în special pe Linux/Jetson, unde configurarea GPU și CUDA poate necesita verificare.
{% endhint %}

***

### `update` - Verificare actualizări (numai pentru Linux)

Verificați și instalați actualizările CLI pe sistemele Linux.

**Sintaxă:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opțiune    | Descriere                        |
| --------- | ---------------------------------- |
| `--check` | Verifică doar actualizările, nu le instalează |

{% hint style="info" %}
Această comandă este disponibilă numai pe Linux. Pe Windows, actualizările sunt livrate prin intermediul programului de instalare.
{% endhint %}

***

## Opțiuni globale

Aceste opțiuni se aplică tuturor comenzilor:

| Opțiune            | Tip    | Implicit       | Descriere                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Cale    | Detectat automat | Calea către executabilul backend                       |
| `--port`          | Număr întreg | 5000          | Numărul portului backend API                          |
| `--restart`       | Indicator    | -             | Forțează repornirea backend-ului (închide procesele existente) |
| `--version`       | Indicator    | -             | Afișează informații despre versiune și iese                |
| `--help`          | Indicator    | -             | Afișează informații de ajutor și iese                   |

{% hint style="info" %}
**Detectarea automată a backend-ului**: Calea `--backend-exe` este detectată automat pentru fiecare platformă:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manual)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Exemplu cu opțiuni globale:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Ghid de setări de procesare

### Procesare paralelă și adaptare dinamică a calculului

Chloros 1.1.0 include [Adaptarea dinamică a calculului](processing-architecture/dynamic-compute-adaptation.md) — motorul de procesare **detectează automat hardware-ul** și selectează strategia optimă:

| Platformă | Strategie | Lucrători | Pipeline | Note |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Eficient din punct de vedere al memoriei, serializat |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Procesare GPU simultană |
| **Desktop cu GPU de 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Performanță bună pentru desktop |
| **Desktop cu GPU de 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Performanță optimă a desktopului |
| **Sistem doar cu CPU** | `CPU_PARALLEL` | nuclee - 1 | `cpu_fallback` | Nu este necesar GPU |

{% hint style="success" %}
**Nu este necesară configurarea manuală!** Chloros detectează automat CPU-ul, GPU-ul, memoria RAM și (pe Jetson) senzorii termici, apoi configurează automat fluxul de procesare optim.
{% endhint %}

### Metode de debayer

| Metodă | CLI Indicator | Calitate | Viteză | Licență |
| --- | --- | --- | --- | --- |
| **Standard (Rapid, Calitate medie)** | `--debayer standard` | Bună | Rapid | Gratuit / Chloros+ |
| **Sensibil la textură (lent, calitate maximă)** | `--debayer texture-aware` | Maximă | Lent | Numai Chloros+ |

Metoda implicită de debayer este **Standard**. Metoda**Sensibilă la textură** utilizează un model de denoizare AI/ML pentru o ieșire de cea mai înaltă calitate, dar necesită o licență Chloros+ și un GPU NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Corecția vignetării

**Ce face:** Corectează scăderea intensității luminoase la marginile imaginii (colțurile mai întunecate, frecvente în imaginile capturate de cameră).

* **Activată implicit** - Majoritatea utilizatorilor ar trebui să păstreze această opțiune activată
* Utilizați `--no-vignette` pentru a o dezactiva

{% hint style="success" %}
**Recomandare**: Activați întotdeauna corectarea vignetării pentru a asigura o luminozitate uniformă pe întregul cadru.
{% endhint %}

### Calibrarea reflectanței

Convertește valorile brute ale senzorului în procente de reflectanță standardizate folosind panouri de calibrare.

* **Activată implicit** - Esențială pentru analiza vegetației
* Necesită panouri țintă de calibrare în imagini
* Utilizați `--no-reflectance` pentru a dezactiva

{% hint style="info" %}
**Cerințe**: Asigurați-vă că panourile de calibrare sunt expuse corespunzător și vizibile în imaginile dvs. pentru o conversie precisă a reflectanței.
{% endhint %}

### Corecții PPK

**Ce face:** Aplică corecții cinematice post-procesate utilizând datele de jurnal DAQ-A-SD pentru o precizie GPS îmbunătățită.

* **Dezactivat implicit**
* Utilizați `--ppk` pentru a activa
* Necesită fișiere .daq în folderul proiectului de la senzorul de lumină DAQ-A-SD MAPIR.

### Formate de ieșire

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Adâncime de biți</th><th width="116.5999755859375">Dimensiune fișier</th><th>Ideal pentru</th></tr></thead><tbody><tr><td><strong>TIFF (16 biți)</strong> ⭐</td><td>Număr întreg pe 16 biți</td><td>Mare</td><td>Analiză GIS, fotogrammetrie (recomandat)</td></tr><tr><td><strong>TIFF (32 de biți, procent)</strong></td><td>Număr real pe 32 de biți</td><td>Foarte mare</td><td>Analiză științifică, cercetare</td></tr><tr><td><strong>PNG (8 biți)</strong></td><td>Număr întreg pe 8 biți</td><td>Mediu</td><td>Inspecție vizuală, partajare web</td></tr><tr><td><strong>JPG (8 biți)</strong></td><td>Număr întreg pe 8 biți</td><td>Mic</td><td>Previzualizare rapidă, ieșire comprimată</td></tr></tbody></table>***

## Automatizare și scripturi

### Procesare în lot PowerShell (Windows)

Procesați automat mai multe foldere cu seturi de date pe Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Script de lot Windows (Windows)

Buclă simplă pentru procesarea în lot pe Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Procesare batch Bash (Linux)

Procesarea mai multor foldere cu seturi de date pe Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Script de automatizare Python (multiplatformă)

Automatizare avansată cu gestionarea erorilor (funcționează pe Windows și Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Fluxul de lucru al procesării

### Flux de lucru standard

1. **Intrare**: Dosar care conține perechi de imagini RAW/JPG
2. **Descoperire**: CLI scanează automat fișierele imagine acceptate
3. **Prelucrare**: Modul paralel se adaptează la nucleele procesorului (Chloros+)
4. **Ieșire**: Creează subfoldere pentru fiecare model de cameră cu imaginile procesate

### Exemplu de structură de ieșire

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Estimări privind timpul de procesare

Timpuri tipice de procesare pentru 100 de imagini (12 MP fiecare):

| Platformă | Mod | Timp estimat | Note |
| --- | --- | --- | --- |
| **Desktop 12 GB+ GPU** | `GPU_PARALLEL` | 5-10 min | Cea mai rapidă opțiune |
| **Desktop 8 GB GPU** | `GPU_SINGLE` | 10-15 min | Performanță bună |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15-25 min | Calcul la margine |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30-60 min | Memorie limitată |
| **Numai CPU** | `CPU_PARALLEL` | 20-40 min | Nu este necesar GPU |

{% hint style="info" %}
**Sfat privind performanța**: Timpul de procesare variază în funcție de numărul de imagini, rezoluție, metoda de debayer și hardware. Metoda de debayer Texture Aware durează semnificativ mai mult decât cea Standard. Consultați [Adaptarea dinamică a calculului](processing-architecture/dynamic-compute-adaptation.md) pentru detalii.
{% endhint %}

***

## Depanare

### CLI Nu a fost găsit

**Eroare Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Soluții:**

1. Verificați locația de instalare:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utilizați calea completă dacă nu se află în PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Adăugați manual la PATH:
   * Deschideți Proprietăți sistem → Variabile de mediu
   * Editați variabila PATH
   * Adăugați: `C:\Program Files\Chloros\resources\cli`
   * Reporniți terminalul

**Eroare Linux:**

```
chloros-cli: command not found
```

**Linux Soluții:**

1. Verificați instalarea:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Reîncărcați shell-ul:

```bash
source ~/.bashrc
```

3. Verificați permisiunile:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend-ul nu a putut fi pornit**Eroare:**

```

Backend failed to start within 30 seconds
```

**Soluții:**

1. Verificați dacă backend-ul rulează deja (închideți-l mai întâi)
2. Verificați dacă firewall-ul nu blochează (Windows) sau verificați disponibilitatea portului (Linux: `lsof -i :5000`)
3. Încercați un alt port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Forțați repornirea backend-ului:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Pe Linux, verificați dacă executabilul backend-ului există:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Probleme legate de licență / autentificare**Eroare:**

```

Chloros+ license required for CLI access
```

**Soluții:**

1. Verificați dacă aveți un abonament Chloros+ activ
2. Autentificați-vă cu datele de autentificare:

```bash
chloros-cli login user@example.com 'password'
```

3. Verificați starea licenței:

```bash
chloros-cli status
```

4. Contactați serviciul de asistență: info@mapir.camera

***

### Nu s-au găsit imagini**Eroare:**

```

No images found in the specified folder
```

**Soluții:**

1. Verificați dacă folderul conține formate acceptate (.RAW, .TIF, .JPG)
2. Verificați dacă calea către folder este corectă (utilizați ghilimele pentru căile care conțin spații)
3. Asigurați-vă că aveți permisiuni de citire pentru folder
4. Verificați dacă extensiile fișierelor sunt corecte

***

### Procesarea se blochează sau se oprește**Soluții:**

1. Verificați spațiul disponibil pe disc (asigurați-vă că este suficient pentru ieșire)
2. Închideți alte aplicații pentru a elibera memorie
3. Reduceți numărul de imagini (procesați în loturi)

***

### Portul este deja utilizat**Eroare:**

```

Port 5000 is already in use
```

**Soluții:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Întrebări frecvente

### Î: Am nevoie de o licență pentru CLI?

**R:**Da! CLI necesită o**licență Chloros+** plătită.

* ❌ Plan standard (gratuit): CLI dezactivat
* ✅ Planuri Chloros+ (cu plată): CLI complet activat

Abonați-vă la: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Î: Pot folosi CLI pe un server fără GUI?**R:** Da! CLI rulează complet fără interfață grafică. Acesta este principalul caz de utilizare pe Linux.**Server Windows:**
* Server Windows 2016 sau o versiune ulterioară
* Visual C++ Redistributable instalat

**Server Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) sau JetPack 6 (arm64)
* Instalare prin pachetul `.deb`

**Ambele platforme:**
* Minim 8 GB RAM (se recomandă 16 GB)
* Activare licență unică: `chloros-cli login user@example.com 'password'`

***

### Î: Unde sunt salvate imaginile procesate?**R:**În mod implicit, imaginile procesate sunt salvate în**același folder ca și cele de intrare**, în subfoldere denumite după modelul camerei (de ex., `Survey3N_RGN/`).

Utilizați opțiunea `-o` pentru a specifica un alt folder de ieșire:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Î: Pot procesa mai multe foldere simultan?**R:** Nu direct într-o singură comandă, dar puteți utiliza scripturi pentru a procesa folderele secvențial. Consultați secțiunea [Automatizare și scripturi](CLI.md#automation--scripting).***

### Î: Cum pot salva ieșirea CLI într-un fișier jurnal?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Î: Ce se întâmplă dacă apăs Ctrl+C în timpul procesării?**R:** CLI va:

1. Opri procesarea în mod corespunzător
2. Opri backend-ul
3. Ieși cu codul 130

Imaginile procesate parțial pot rămâne în folderul de ieșire.

***

### Î: Pot automatiza procesarea CLI?**R:** Desigur! CLI este conceput pentru automatizare. Consultați [Automatizare și scripturi](CLI.md#automation--scripting) pentru PowerShell (Windows), Batch (Windows), Bash (Linux) și Python (multiplatformă).***

### Î: Cum verific versiunea CLI?**R:**

```bash
chloros-cli --version
```

**Rezultat:**

```

Chloros CLI 1.1.0
```

***

## Obținerea de ajutor

### Ajutor pentru linia de comandă

Vizualizați informațiile de ajutor direct în CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canale de asistență

* **E-mail**: info@mapir.camera
* **Site web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Prețuri**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Exemple complete

### Exemplul 1: Procesare de bază

Procesare cu setările implicite (vignette, reflectanță):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Exemplul 2: Rezultate științifice de înaltă calitate

32-bit float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Exemplul 3: Procesare rapidă a previzualizării

8 biți PNG fără calibrare pentru revizuire rapidă:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Exemplul 4: Procesare cu corecție PPK

Aplicați corecții PPK cu reflectanță:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Exemplul 5: Locație de ieșire personalizată

Procesați într-o locație diferită cu un format specific:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Exemplul 6: Fluxul de autentificare

Flux complet de autentificare (identic pe toate platformele):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exemplul 7: Utilizarea mai multor limbi

Schimbarea limbii interfeței (identică pe toate platformele):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
