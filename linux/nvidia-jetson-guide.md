# Ghid NVIDIA Jetson

Chloros pe NVIDIA Jetson permite procesarea imaginilor multispectrale la marginea rețelei — pe teren, pe drone și în instalații la distanță. Chloros detectează automat modelul dvs. Jetson și optimizează strategia de procesare pentru hardware-ul dvs.

***

## Modele Jetson acceptate

| Model                | RAM            | Strategie de procesare                                   | Utilizare recomandată                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB partajat | `GPU_PARALLEL` (4 procesori)                            | Performanță maximă, seturi de date mari                      |
| **Jetson Orin NX**   | 8-16 GB partajat  | `GPU_PARALLEL` (3 procesori, 16 GB) / `GPU_SINGLE` (8 GB) | Recomandare principală pentru implementare aeriană și pe teren |
| **Jetson Orin Nano** | 8 GB partajat     | `GPU_SINGLE` (1 procesor)                               | Calcul la margine de nivel de bază                                 |
| **Jetson Nano**      | 4-8 GB partajat   | `GPU_SINGLE` (1 procesor)                               | Nivel de bază, cu memorie limitată                          |

{% hint style="info" %}
**Modelele Jetson vechi** (TX2, TX1, Xavier NX) s-ar putea să nu fie acceptate. Performanța va varia în funcție de memoria GPU disponibilă și de capacitățile CUDA.
{% endhint %}

***

## Cerințe

* **JetPack 6.x** (se recomandă cea mai recentă versiune)
* **NVIDIA CUDA** (inclusă în JetPack)
* **Licență Chloros+** (necesară pentru accesul la CLI/SDK)

## Instalare

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Pentru detalii generale privind instalarea Linux, consultați [Instalarea Linux](linux-installation.md).

***

## Adaptare dinamică a calculului pe Jetson

Chloros detectează automat modelul dvs. Jetson și selectează strategia optimă de procesare. **Nu este necesară nicio reglare manuală.**

### Cum funcționează

La pornire, Chloros creează un profil al sistemului dvs.:

1. **Detectează modelul Jetson** prin `/proc/device-tree/model`
2. **Citește memoria GPU/memoria partajată disponibilă**

3.**Selectează o strategie de procesare** (`GPU_PARALLEL`, `GPU_SINGLE` sau `CPU_PARALLEL`)
4. **Setează automat numărul de lucrători, tipul de pipeline și alocarea memoriei**

### Comportament pe model

| Model Jetson                | Strategie       | Processori | Pipeline                       | Concurentă |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (eficient din punct de vedere al memoriei) | Serializat  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serializat  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializat  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (cale GPU completă)    | Concurent  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Concurent  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** este soluția ideală pentru implementarea la margine — beneficiază de strategia `GPU_PARALLEL` cu 3 lucrători simultani, oferind procesare GPU paralelă reală într-un format compact.
{% endhint %}

Diferența cheie între platforme este **memoria**. Un Jetson Nano cu 8 GB de memorie partajată trebuie să proceseze imaginile una câte una folosind o abordare în blocuri eficientă din punct de vedere al memoriei, în timp ce un Orin NX cu 16 GB poate rula simultan 3 imagini prin GPU folosind pipeline-ul fuzionat cu debit mai mare.

Pentru referința completă privind adaptarea de calcul, consultați [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md).

***

## Gestionarea termică

Dispozitivele Jetson au o marjă termică limitată, în special în implementările închise sau aeriene. Chloros include monitorizarea termică automată și limitarea:

| Temperatură         | Acțiune                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funcționare normală — viteză maximă de procesare          |
| **70 °C** (Avertisment)  | Reducerea automată a dimensiunii lotului                   |
| **80 °C** (Critic) | Limitarea agresivă — concurență redusă         |
| **90°C** (Oprire) | Oprirea completă a procesării GPU — este necesară răcirea |

{% hint style="warning" %}
**Asigurați o ventilație adecvată și disiparea căldurii** pentru o procesare susținută, în special în carcase de teren închise sau sisteme aeriene. Limitarea termică va reduce randamentul de procesare pentru a proteja hardware-ul.
{% endhint %}

***

## Gestionarea memoriei

Dispozitivele Jetson utilizează **memorie unificată** — GPU-ul și CPU-ul împart aceeași memorie RAM fizică. Aceasta înseamnă că VRAM-ul raportat (de exemplu, 15,3 GB pe Orin NX 16 GB) nu este memorie dedicată GPU-ului; este împărțită cu sistemul de operare și alte procese.

### Recomandări privind spațiul de swap

Pentru seturi de date mari sau procesarea debayer Texture Aware, Chloros poate recomanda crearea unui spațiu de swap:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Estimări de memorie per imagine:**

* Debayer standard: ~10 MB per imagine
* Debayer Texture Aware: ~15 MB per imagine

Chloros calculează automat memoria necesară pe baza dimensiunii setului de date și vă avertizează dacă se recomandă swap.

### Fallback OOM (Out of Memory)

Dacă se detectează o condiție de memorie insuficientă în timpul procesării:

1. Chloros reduce automat numărul de procesori GPU
2. Trece de la pipeline-ul `fused_gpu` la cel `tiled_gpu` (mai eficient din punct de vedere al memoriei)
3. Continuă procesarea la un debit redus, în loc să se blocheze

***

## Implementare pe teren

### Considerații privind alimentarea

| Model Jetson     | Consum tipic de energie | Note                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C sau mufă cilindrică    |
| Jetson Orin Nano | 7-15 W              | Mufă cilindrică DC          |
| Jetson Orin NX   | 10-25 W             | Mufă cilindrică DC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD sau mufă cilindrică |

Planificați-vă bugetul de alimentare pentru procesare susținută — consumul maxim de energie are loc în timpul Thread 3 (Procesare), care solicită intens GPU-ul.

### Recomandări privind stocarea

* **SSD NVMe** recomandat cu tărie pentru implementările arm64
* Cardurile SD sunt prea lente pentru procesare — utilizați-le doar ca suport de boot
* Planificați un spațiu de 2-3 ori mai mare decât dimensiunea datelor brute ale imaginii pentru rezultatul procesat

### Funcționare fără monitor prin SSH

Chloros CLI este ideal pentru implementările Jetson fără monitor:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Procesare automatizată cu systemd

Creați un serviciu systemd pentru procesare automatizată:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Asociați-l cu un cronometru systemd pentru procesare programată:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Fluxuri de lucru exemplificative

### Procesare Jetson de bază

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK pe Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Procesare în lot a mai multor zboruri

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sisteme Jetson recomandate pentru utilizare pe teren

Pentru implementări pe teren și aeriene, luați în considerare aceste opțiuni de plăci de bază Jetson Orin NX de 16 GB:

* **Aeriene/drone**: Sisteme cu rezistență la vibrații (MIL-STD), ușoare (sub 300 g), răcire pasivă
* **Teren dificil**: Carcase impermeabile IP67/IP69K cu conectivitate pentru cameră PoE GigE
* **Minim/buget**: kituri pentru dezvoltatori cu carcase suplimentare

Contactați [Asistența MAPIR](https://www.mapir.camera/community/contact) pentru recomandări specifice de hardware pentru scenariul dvs. de implementare.

***

## Pași următori

* [Linux Instalare](linux-installation.md) — Detalii generale privind instalarea Linux
* [Adaptare dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md) — Referință completă privind strategia de calcul
* [Conductă de procesare](../processing-architecture/processing-pipeline.md) — Înțelegerea conductei cu 4 fire
* [CLI : Linia de comandă](../CLI.md) — Referință completă pentru CLI
* [API : Python SDK](../api-python-sdk.md) — Referință completă pentru SDK
