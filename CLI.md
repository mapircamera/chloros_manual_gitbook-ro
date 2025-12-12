# CLI : Linie de comandă

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** oferă acces puternic prin linie de comandă la motorul de procesare a imaginilor Chloros, permițând automatizarea, scriptarea și operarea fără monitor pentru fluxurile de lucru de imagistică.

### Caracteristici cheie

* 🚀 **Automatizare** - Procesare în lot a mai multor seturi de date
* 🔗 **Integrare** - Încorporare în fluxurile de lucru și conductele existente
* 💻 **Funcționare fără interfață grafică** - Funcționare fără GUI
* 🌍 **Multilingv** - Suport pentru 38 de limbi
* ⚡ **Procesare paralelă** - Se adaptează dinamic la CPU-ul dvs. (până la 16 procesoare paralele)

### Cerințe

| Cerință          | Detalii                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistem de operare** | Windows 10/11 (64 biți)                                              |
| **Licență**          | Chloros+ ([plan plătit necesar](https://cloud.mapir.camera/pricing)) |
| **Memorie**           | Minim 8 GB RAM (recomandat 16 GB)                                  |
| **Internet**         | Necesar pentru activarea licenței                                     |
| **Spațiu pe disc**       | Variază în funcție de dimensiunea proiectului                                              |

{% hint style=&quot;warning&quot; %}
**Cerințe de licență**: CLI necesită un abonament plătit Chloros+. Planurile standard (gratuite) nu au acces la CLI. Vizitați [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) pentru a face upgrade.
{% endhint %}

## Începere rapidă

### Instalare

CLI este inclus automat în programul de instalare Chloros:

1. Descărcați și rulați **Chloros Installer.exe**
2. Finalizați asistentul de instalare
3. CLI instalat în: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Programul de instalare adaugă automat `chloros-cli` în calea PATH a sistemului. Reporniți terminalul după instalare.
{% endhint %}

### Configurare inițială

Înainte de a utiliza CLI, activați licența Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Utilizare de bază

Procesați un folder cu setările implicite:

```powershell
chloros-cli process "C:\Images\Dataset001"
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

Procesează imaginile dintr-un folder cu calibrare.

**Sintaxă:**

```bash
chloros-cli process <input-folder> [options]
```

**Exemplu:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opțiuni de comandă de procesare

| Opțiune                | Tip    | Implicit        | Descriere                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Cale    | _Necesar_     | Folder care conține imagini multispectrale RAW/JPG                                         |
| `-o, --output`        | Cale    | La fel ca intrarea  | Folder de ieșire pentru imaginile procesate                                                     |
| `-n, --project-name`  | Șir  | Generat automat | Nume proiect personalizat                                                                    |
| `--vignette`          | Indicator    | Activat        | Activează corectarea vignetării                                                             |
| `--no-vignette`       | Indicator    | -              | Dezactivează corectarea vignetării                                                            |
| `--reflectance`       | Indicator    | Activat        | Activare calibrare reflectanță                                                         |
| `--no-reflectance`    | Indicator    | -              | Dezactivare calibrare reflectanță                                                        |
| `--ppk`               | Indicator    | Dezactivat       | Aplică corecții PPK din datele senzorului de lumină .daq                                      |
| `--format`            | Opțiune  | TIFF (16 biți)  | Format de ieșire: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Număr întreg | Auto           | Dimensiunea minimă țintă în pixeli pentru detectarea panoului de calibrare                          |
| `--target-clustering` | Număr întreg | Auto           | Pragul de grupare a țintelor (0-100)                                                    |
| `--exposure-pin-1`    | Șir  | Niciunul           | Blocare expunere pentru modelul camerei (Pin 1)                                                 |
| `--exposure-pin-2`    | Șir  | Niciunul           | Blocare expunere pentru modelul camerei (Pin 2)                                                 |
| `--recal-interval`    | Număr întreg | Auto           | Interval de recalibrare în secunde                                                      |
| `--timezone-offset`   | Număr întreg | 0              | Decalaj fus orar în ore                                                               |

***

### `login` - Autentificare cont

Conectați-vă cu datele de autentificare Chloros+ pentru a activa procesarea CLI.

**Sintaxă:**

```bash
chloros-cli login <email> <password>
```

**Exemplu:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caractere speciale**: Utilizați ghilimele simple în jurul parolelor care conțin caractere precum `$`, `!` sau spații.
{% endhint %}

**Rezultat:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Ștergeți datele de autentificare

Ștergeți datele de autentificare stocate și deconectați-vă de la contul dvs.

**Sintaxă:**

```bash
chloros-cli logout
```

**Exemplu:**

```powershell
chloros-cli logout
```

**Rezultat:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Verificarea stării licenței

Afișează starea actuală a licenței și a autentificării.

**Sintaxă:**

```bash
chloros-cli status
```

**Exemplu:**

```powershell
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

### `export-status` - Verificarea progresului exportului

Monitorizează progresul exportului Thread 4 în timpul sau după procesare.

**Sintaxă:**

```bash
chloros-cli export-status
```

**Exemplu:**

```powershell
chloros-cli export-status
```

**Caz de utilizare:** Apelați această comandă în timpul procesării pentru a verifica progresul exportului.

***

### `language` - Gestionarea limbii interfeței

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

```powershell
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

| Cod    | Limbă              | Denumire nativă      |
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
| `nl`    | Olandeză                 | Nederlands       |
| `ar`    | Arabă                | العربية          |
| `pl`    | Poloneză                | Polski           |
| `tr`    | Turcă               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indoneziană            | Bahasa Indonesia |
| `vi`    | Vietnameză            | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
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

{% hint style=&quot;success&quot; %}
**Persistență automată**: Preferința dvs. lingvistică este salvată în `~/.chloros/cli_language.json` și persistă în toate sesiunile.
{% endhint %}

***

### `set-project-folder` - Setare folder proiect implicit

Modificați locația folderului proiect implicit (partajat cu GUI).

**Sintaxă:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exemplu:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Afișează folderul proiectului

Afișează locația curentă a folderului proiectului implicit.

**Sintaxă:**

```bash
chloros-cli get-project-folder
```

**Exemplu:**

```powershell
chloros-cli get-project-folder
```

**Rezultat:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Resetare la implicit

Resetează folderul proiectului la locația implicită.

**Sintaxă:**

```bash
chloros-cli reset-project-folder
```

***

## Opțiuni globale

Aceste opțiuni se aplică tuturor comenzilor:

| Opțiune          | Tip    | Implicit       | Descriere                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Cale    | Detectată automat | Calea către executabilul backend                       |
| `--port`        | Număr întreg | 5000          | Numărul portului backend API                          |
| `--restart`     | Indicator    | -             | Forțează repornirea backend-ului (oprește procesele existente) |
| `--version`     | Indicator    | -             | Afișează informații despre versiune și iese                |
| `--help`        | Indicator    | -             | Afișează informații de ajutor și iese                   |

**Exemplu cu opțiuni globale:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Ghid de setări de procesare

### Procesare paralelă

Chloros+ CLI **scalează automat** procesarea paralelă pentru a se potrivi cu capacitățile computerului dvs.:

**Cum funcționează:**

* Detectează nucleele CPU și memoria RAM
* Alocă lucrători: **2× nuclee CPU** (utilizează hyperthreading)
* **Maxim: 16 lucrători paraleli** (pentru stabilitate)

**Niveluri de sistem:**

| Tip sistem   | CPU        | RAM      | Lucrători  | Performanță     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ nuclee  | 32+ GB   | Până la 16 | Viteză maximă   |
| **Gama medie** | 8-15 nuclee | 16-31 GB | 8-16     | Viteză excelentă |
| **Gama inferioară**   | 4-7 nuclee  | 8-15 GB  | 4-8      | Viteză bună      |

{% hint style=&quot;success&quot; %}
**Optimizare automată**: CLI detectează automat specificațiile sistemului dvs. și configurează procesarea paralelă optimă. Nu este necesară configurarea manuală!
{% endhint %}

### Metode Debayer

CLI utilizează **High Quality (Faster)** ca algoritm debayer implicit și recomandat:

| Metodă                      | Calitate | Viteză | Descriere                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Calitate înaltă (mai rapid)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritm sensibil la margini (implicit, recomandat) |

### Corecție vignette

**Ce face:** Corectează scăderea luminii la marginile imaginii (colțurile mai întunecate frecvente în imaginile capturate de cameră).

* **Activat implicit** - Majoritatea utilizatorilor ar trebui să păstreze această opțiune activată
* Utilizați `--no-vignette` pentru a dezactiva

{% hint style=&quot;success&quot; %}
**Recomandare**: Activați întotdeauna corectarea vignetării pentru a asigura o luminozitate uniformă în cadrul imaginii.
{% endhint %}

### Calibrarea reflectanței

Convertește valorile brute ale senzorului în procente standardizate de reflectanță utilizând panouri de calibrare.

* **Activat în mod implicit** - Esențial pentru analiza vegetației.
* Necesită panouri țintă de calibrare în imagini.
* Utilizați `--no-reflectance` pentru a dezactiva.

{% hint style=&quot;info&quot; %}
**Cerințe**: Asigurați-vă că panourile de calibrare sunt expuse corespunzător și vizibile în imaginile dvs. pentru o conversie precisă a reflectanței.
{% endhint %}

### Corecții PPK

**Ce face:** Aplică corecții cinematice post-procesate utilizând date jurnal DAQ-A-SD pentru o precizie GPS îmbunătățită.

* **Dezactivat în mod implicit**
* Utilizați `--ppk` pentru a activa
* Necesită fișiere .daq în folderul proiectului din senzorul de lumină MAPIR DAQ-A-SD.

### Formate de ieșire

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Adâncime de biți</th><th width="116.5999755859375">Dimensiune fișier</th><th>Cel mai potrivit pentru</th></tr></thead><tbody><tr><td><strong>TIFF (16 biți)</strong> ⭐</td><td>Număr întreg pe 16 biți</td><td>Mare</td><td>Analiză GIS, fotogrammetrie (recomandat)</td></tr><tr><td><strong>TIFF (32 biți, procent)</strong></td><td>Float pe 32 de biți</td><td>Foarte mare</td><td>Analiză științifică, cercetare</td></tr><tr><td><strong>PNG (8 biți)</strong></td><td>Întreg pe 8 biți</td><td>Mediu</td><td>Inspecție vizuală, partajare web</td></tr><tr><td><strong>JPG (8 biți)</strong></td><td>Număr întreg pe 8 biți</td><td>Mic</td><td>Previzualizare rapidă, ieșire comprimată</td></tr></tbody></table>***

## Automatizare și scripturi

### Procesare în lot PowerShell

Procesați automat mai multe foldere cu seturi de date:

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

### Windows Script în lot

Buclă simplă pentru procesarea în lot:

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

### Python Script de automatizare

Automatizare avansată cu gestionarea erorilor:

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

## Flux de lucru de procesare

### Flux de lucru standard

1. **Intrare**: folder care conține perechi de imagini RAW/JPG
2. **Descoperire**: CLI scanează automat fișierele imagine acceptate
3. **Prelucrare**: Modul paralel se adaptează la nucleele procesorului (Chloros+)
4. **Ieșire**: Creează subfoldere pentru modelul camerei cu imaginile prelucrate

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

Timpii de procesare tipici pentru 100 de imagini (12 MP fiecare):

| Mod              | Timp      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Mod paralel** | 5-10 min  | i7/Ryzen 7, 16 GB RAM, SSD (până la 16 lucrători) |
| **Mod paralel** | 10-15 min | i5/Ryzen 5, 8 GB RAM, HDD (până la 8 procesoare)   |

{% hint style=&quot;info&quot; %}
**Sfat de performanță**: Timpul de procesare variază în funcție de numărul de imagini, rezoluție și specificațiile computerului.
{% endhint %}

***

## Depanare

### CLI nu a fost găsit

**Eroare:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Soluții:**

1. Verificați locația instalării:

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

***

### Eșec la pornirea backend-ului

**Eroare:**

```
Backend failed to start within 30 seconds
```

**Soluții:**

1. Verificați dacă backend-ul rulează deja (închideți-l mai întâi)
2. Verificați dacă Windows Firewall nu blochează
3. Încercați un alt port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Forțați repornirea backend-ului:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Probleme cu licența/autentificarea

**Eroare:**

```
Chloros+ license required for CLI access
```

**Soluții:**

1. Verificați dacă aveți un abonament Chloros+ activ.
2. Conectați-vă cu datele de autentificare:

```powershell
chloros-cli login user@example.com 'password'
```

3. Verificați starea licenței:

```powershell
chloros-cli status
```

4. Contactați serviciul de asistență: info@mapir.camera

***

### Nu s-au găsit imagini

**Eroare:**

```
No images found in the specified folder
```

**Soluții:**

1. Verificați dacă folderul conține formate acceptate (.RAW, .TIF, .JPG)
2. Verificați dacă calea folderului este corectă (utilizați ghilimele pentru căile cu spații)
3. Asigurați-vă că aveți permisiuni de citire pentru folder.
4. Verificați dacă extensiile fișierelor sunt corecte.

***

### Procesarea se blochează sau se întrerupe.

**Soluții:**

1. Verificați spațiul disponibil pe disc (asigurați-vă că este suficient pentru ieșire).
2. Închideți alte aplicații pentru a elibera memoria.
3. Reduceți numărul de imagini (procesați în loturi).

***

### Portul este deja utilizat.

**Eroare:**

```
Port 5000 is already in use
```

**Soluție:**

Specificați un port diferit:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Întrebări frecvente

### Î: Am nevoie de o licență pentru CLI?

**R:** Da! CLI necesită o **licență Chloros+** plătită.

* ❌ Plan standard (gratuit): CLI dezactivat
* ✅ Planuri Chloros+ (cu plată): CLI complet activat

Abonați-vă la: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Î: Pot utiliza CLI pe un server fără GUI?

**R:** Da! CLI funcționează complet fără interfață grafică. Cerințe:

* Windows Server 2016 sau o versiune ulterioară
* Visual C++ Redistributable instalat
* Memorie RAM suficientă (minimum 8 GB, recomandat 16 GB)
* Activare unică a licenței GUI pe orice mașină

***

### Î: Unde sunt salvate imaginile procesate?

**R:** În mod implicit, imaginile procesate sunt salvate în **același folder ca și cele de intrare**, în subfoldere ale modelului de cameră (de exemplu, `Survey3N_RGN/`).

Utilizați opțiunea `-o` pentru a specifica un folder de ieșire diferit:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Î: Pot procesa mai multe foldere simultan?

**R:** Nu direct într-o singură comandă, dar puteți utiliza scripturi pentru a procesa folderele secvențial. Consultați secțiunea [Automatizare și scripturi](CLI.md#automation--scripting).

***

### Î: Cum salvez ieșirea CLI într-un fișier jurnal?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Î: Ce se întâmplă dacă apăs Ctrl+C în timpul procesării?

**R:** CLI va:

1. Opri procesarea în mod corespunzător
2. Oprirea backend-ului
3. Ieșirea cu codul 130

Imaginile parțial procesate pot rămâne în folderul de ieșire.

***

### Î: Pot automatiza procesarea CLI?

**R:** Desigur! CLI este conceput pentru automatizare. Consultați [Automatizare și scripturi](CLI.md#automation--scripting) pentru exemple de PowerShell, Batch și Python.

***

### Î: Cum pot verifica versiunea CLI?

**R:**

```powershell
chloros-cli --version
```

**Rezultat:**

```
Chloros CLI 1.0.2
```

***

## Obținerea de ajutor

### Ajutor pentru linia de comandă

Vizualizați informațiile de ajutor direct în CLI:

```powershell
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
* **Prețuri**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Exemple complete

### Exemplul 1: Procesare de bază

Procesare cu setări implicite (vignetă, reflectanță):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Exemplul 2: Rezultate științifice de înaltă calitate

32 biți flotant TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Exemplul 3: Procesare rapidă de previzualizare

8 biți PNG fără calibrare pentru revizuire rapidă:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Exemplul 4: Procesare corectată PPK

Aplicați corecții PPK cu reflectanță:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Exemplul 5: Locație de ieșire personalizată

Procesați pe o unitate diferită cu format specific:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Exemplul 6: Flux de lucru pentru autentificare

Finalizați fluxul de autentificare:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exemplul 7: Utilizare multilingvă

Schimbați limba interfeței:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
