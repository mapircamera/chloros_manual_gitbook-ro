# Ghidul NVIDIA Jetson

Chloros
pe NVIDIA Jetson permite procesarea imaginilor multispectrale la marginea rețelei — pe teren, pe UAV-uri și în instalații îndepărtate. Chloros
1.3.0 detectează modelul dvs. de Jetson la pornire și optimizează strategia de procesare în funcție de hardware-ul identificat. **Nu este necesară nicio reglare manuală.**
{% hint style="info" %}
**Pachetul Jetson 1.3.0 rulează pe GPU-ul dispozitivului Jetson.** `chloros_1.3.0_arm64_jp6.deb` este conceput pentru JetPack 6 și utilizează versiunea CUDA a PyTorch din JetPack 6, astfel încât procesarea, inclusiv algoritmul de debayer Texture Aware, se execută pe GPU-ul Orin. Pachetul nu conține CUDA în sine: utilizează bibliotecile CUDA și cuDNN instalate de JetPack 6 (`sudo apt install nvidia-jetpack` pe o imagine care nu le conține). În cazul în care nu este disponibil un GPU CUDA, procesarea se execută pe CPU, iar Texture Aware utilizează debayerul standard.
{% endhint %}

***

## Modele Jetson acceptate

| Model                | RAM            | Strategie de procesare                                     | Utilizare recomandată                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB partajat | `GPU_PARALLEL` (2 procesori)                              | Performanță maximă, seturi mari de date                      |
| **Jetson Orin NX**   | 8–16 GB partajată  | `GPU_PARALLEL` (2 procesori, 16 GB) / `GPU_SINGLE` (8 GB)   | Recomandare principală pentru implementare pe teren              |
| **Jetson Orin Nano** | 8 GB partajat     | `GPU_SINGLE` (1 procesor de lucru, secvențial)                     | Calcul la marginea rețelei de nivel de bază                                 |

{% hint style="info" %}
Pachetul Linux
arm64 necesită **JetPack 6**, care este disponibil pe familia Jetson Orin. Modelele mai vechi (Nano, TX2, Xavier NX) nu pot rula JetPack 6 și nu sunt acceptate de pachetul actual.
{% endhint %}

***

## Cerințe

* **JetPack 6.x** (se recomandă cea mai recentă versiune)
* **NVIDIA CUDA și cuDNN** din JetPack 6 (`sudo apt install nvidia-jetpack` dacă imaginea dvs. nu le conține)
* **Plan Chloros
+ plătit** — nivelul Copper sau superior (necesar pentru accesul la toate CLI
/SDK
; aplicat la nivel de server)

## Instalare

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` instalează SDK
-ul Python
pentru sistemul `python3`, corespunzător acestei versiuni. Doar un virtualenv necesită instalarea pachetului wheel inclus: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (complet offline). Pentru detalii generale privind instalarea pe Linux
, locațiile fișierelor și depanarea, consultați [Instalarea pe Linux
](linux-installation.md).

{% hint style="info" %}
**Nu mai este necesară dezarhivarea la pornire.** Începând cu versiunea 1.3.0, backend-ul și interfața CLI
sunt instalate deja dezarhivate în `/usr/lib/chloros/`, astfel încât nu se mai extrag automat într-un director temporar de fiecare dată când sunt pornite. Fișierele temporare sunt în continuare salvate automat în `/mnt/ssd/tmp` atunci când acest folder există; în caz contrar, puteți seta `TMPDIR` la o cale de pe NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptarea dinamică a puterii de calcul pe Jetson

### Cum funcționează

La pornire, Chloros
creează un profil al sistemului dvs.:

1. **Detectează modelul Jetson** prin intermediul `/proc/device-tree/model`
2. **Citește memoria GPU/CPU partajată disponibilă** (Jetson utilizează memorie unificată)
3. **Întreabă PyTorch dacă poate utiliza CUDA** — cu CUDA și cuDNN din JetPack 6 instalate, poate
4. **Selectează automat o strategie de procesare** (`GPU_PARALLEL`, `GPU_SINGLE` sau `CPU_PARALLEL`) și numărul de procesori

Decizia este determinată de **memoria RAM totală partajată**, nu de numele modelului:

* **Sub 12 GB memorie RAM totală**(toate dispozitivele Jetson de 8 GB):**1 procesor de lucru — procesare secvențială deliberată**. Memoria este prea limitată pentru procesoare de lucru concurente, astfel încât imaginile sunt procesate pe rând. Pe modelele Jetson cu**8 GB sau mai puțin**, Thread 3 ocolește complet grupul de procesori de lucru și își execută operațiunile pentru fiecare imagine în cadrul procesului. Jurnalul backend denumește această strategie `GPU_SINGLE`.
* **12 GB sau mai mult**(Orin NX 16 GB, AGX Orin): memoria unificată îndeplinește condițiile pentru `GPU_PARALLEL`, dar numărul de procesori de lucru este**limitat la 2 pe Jetson** — GPU-ul, memoria RAM a proceselor de lucru și contextele CUDA specifice fiecărui proces de lucru utilizează toate același pool partajat, astfel încât un număr mai mare de procese de lucru prezintă riscul apariției erorilor de memorie insuficientă.
* **Fără CUDA utilizabil** (lipsesc bibliotecile CUDA ale JetPack): `CPU_PARALLEL` la 12 GB sau mai mult, câte o imagine pe rând mai jos.

Puteți suprascrie alegerea automată cu variabila de mediu `CHLOROS_STRATEGY` — consultați [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportament specific fiecărui model

| Model Jetson                | Strategie       | Workers | Execuție                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Buclă secvențială în cadrul procesului (`tiled_gpu` în condiții de presiune asupra memoriei) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Buclă secvențială în cadrul procesului                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Procese de lucru concurente, calea `fused_gpu`  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Procese de lucru concurente, cale `fused_gpu`  |

Diferența cheie între platforme este **memoria**. Un Jetson de 8 GB trebuie să proceseze imaginile pe rând, folosind o abordare segmentată eficientă din punct de vedere al memoriei atunci când presiunea este mare, în timp ce un Orin de 16 GB sau mai mult poate rula simultan 2 imagini prin GPU, folosind pipeline-ul fuzionat cu debit mai mare.

### Bugetul GPU pe model

Fiecare model Jetson are, de asemenea, un profil hardware care limitează cât din fondul comun de memorie poate fi alocat procesării și scalează dimensiunile loturilor:

| Model | Plafonul bugetului GPU | Multiplicatorul dimensiunii lotului | Rezervat pentru sistem/afișaj |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Memoria RAM detectată ajustează profilul: pentru un dispozitiv Jetson care raportează **16 GB sau mai mult**, multiplicatorul lotului este mărit cu ×1,2. Dimensiunea de bază a lotului înainte de aplicarea multiplicatorilor este de 8 imagini.

Pentru referința completă privind adaptarea de calcul, consultați [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md).

***

## Limita de frecvență a GPU-ului pentru Texture Aware pe Nano și Orin Nano

Procesul de debayer Texture Aware rulează inferența rețelei neuronale pe GPU, ceea ce poate declanșa **avertismente de supracurent**pe modelele Jetson cu consum redus de energie (clasa 10-15 W) la viteza maximă a ceasului GPU. Când transmiteți `--debayer texture-aware` pe un**Jetson Nano sau Orin Nano**, CLI
-ul verifică frecvența maximă a GPU-ului și o limitează la **510 MHz** (510000000) dacă aceasta este în prezent mai mare:

* Dacă CLI
-ul poate scrie în nodul sysfs al frecvenței GPU-ului, limitarea este **aplicată automat** și se afișează o confirmare.
* Dacă nu (este necesar dreptul de root), CLI
-ul afișează comanda exactă `sudo` pentru a aplica limitarea manual, așteaptă un moment pentru a vă permite să o citiți, apoi continuă — procesarea se desfășoară în continuare, dar pot apărea avertismente de supracurent.

Pentru a aplica limita manual înainte de procesare:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modelele cu putere mai mare (Orin NX 25W, AGX Orin 60W) funcționează la viteza maximă a GPU-ului; nu se aplică nicio limită. Modulul Standard debayer nu declanșează niciodată limita pe niciun model.

{% hint style="info" %}
**Modulul „Texture Aware” pe Jetson procesează întotdeauna o singură imagine la un moment dat.** Fiecare worker ar avea nevoie de propriul context CUDA (~1 GB), plus propria copie a modelului de denoizare, ceea ce memoria unificată nu poate suporta — astfel, pe Jetson, calea Texture Aware este fixată la un singur worker, cu accesul la GPU serializat. Este de așteptat ca Texture Aware să fie semnificativ mai lent decât Standard pe orice dispozitiv Jetson. Texture Aware se aplică imaginilor Survey3
: capturile LATTICE beneficiază întotdeauna de debayer-ul Standard. În absența unui GPU CUDA utilizabil, procesul folosește Standard, iar backend-ul înregistrează o linie `[DEBAYER] … needs a CUDA GPU …` care explică motivul (prezentată alături de `chloros-cli --verbose`).
{% endhint %}

***

## Gestionarea termică

Dispozitivele Jetson au o marjă termică limitată, în special în cazul implementărilor în spații închise sau aeriene. Chloros
monitorizează temperatura SoC-ului și limitează automat dimensiunile loturilor:

| Temperatură         | Acțiune                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funcționare normală — viteză maximă de procesare          |
| **70 °C** (Avertisment) | Dimensiunea lotului se reduce progresiv (de la 100% la 50% între 70 °C și 80 °C) |
| **80 °C** (Critic) | Reducere agresivă a vitezei (50 % → 0 % între 80 °C și 90 °C) |
| **90 °C** (Oprire) | Oprire completă a procesării GPU — este necesară răcirea |

{% hint style="warning" %}
**Asigurați o ventilație și o disipare a căldurii adecvate** pentru procesare susținută, în special în carcase de teren închise sau în sisteme aeriene. Limitarea termică va reduce randamentul de procesare pentru a proteja hardware-ul.
{% endhint %}

***

## Gestionarea memoriei

Dispozitivele Jetson utilizează **memorie unificată** — GPU-ul și CPU-ul împart aceeași memorie RAM fizică. VRAM-ul raportat (de exemplu, ~15,3 GB pe un Orin NX de 16 GB) nu este memorie dedicată GPU-ului; este aceeași memorie RAM pe care o utilizează sistemul de operare și toate celelalte procese.

### Avertisment și recomandări privind swap-ul

Înainte de procesare pe Jetson, CLI
numără imaginile RAW din folderul dvs. de intrare (`.tif`, `.tiff`, `.raw`, `.dng` — previzualizările JPG nu sunt luate în calcul), estimează memoria maximă necesară pentru execuție și **vă avertizează înainte de a începe** dacă este probabil ca memoria RAM + swap să fie insuficientă. Avertismentul poartă titlul `LOW MEMORY WARNING - Jetson Detected`, afișează numărul de imagini, memoria RAM, spațiul de swap curent și valoarea maximă estimată, apoi oferă comenzile exacte `fallocate` / `chmod` / `mkswap` / `swapon` dimensionate pentru proiectul dumneavoastră (niciodată mai mici de 8 GB). Se face o pauză de câteva secunde pentru ca mesajul să nu se piardă în istoricul de derulare, apoi procesarea continuă.**Estimări de memorie utilizate de avertisment:**

| Mod Debayer | Bază | Pe imagine |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (model + runtime Python) | ~15 MB |

Avertismentul se declanșează când valoarea maximă estimată depășește RAM + swap minus o marjă de siguranță de 1 GB, și ia în calcul doar swap-ul **bazat pe fișiere** — o configurație bazată exclusiv pe zram va fi totuși semnalată.

Pentru a adăuga spațiu de swap manual (exemplu: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Gestionarea OOM (Out of Memory)

În timpul procesării, Chloros
monitorizează memoria GPU și își reduce gradul de performanță în mod controlat, în loc să se blocheze:

1. Când utilizarea memoriei GPU depășește **85%**, dimensiunile loturilor sunt reduse preventiv
2. Dacă totuși apare un eveniment de lipsă de memorie, dimensiunea lotului este **redusă la jumătate** și redusă din nou la jumătate la fiecare OOM consecutiv; fiecare lot ulterior procesat cu succes reduce această penalizare cu un pas
3. Sub presiune susținută, pipeline-ul trece de la `fused_gpu` la calea `tiled_gpu`, mai eficientă din punct de vedere al memoriei, și, în ultimă instanță, la procesarea pe CPU

Deoarece GPU-ul împarte memoria RAM a dispozitivului Jetson, memoria RAM plus spațiul de swap limitează în continuare durata de funcționare — avertismentul privind spațiul de swap de mai sus este semnalul de alarmă care trebuie luat în considerare.

***

## Implementare pe teren

### Considerații privind consumul de energie

| Model Jetson     | Consum tipic de energie | Note                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | Conector cilindric DC          |
| Jetson Orin NX   | 10–25 W             | Conector cilindric DC          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD sau conector cilindric |

Planificați-vă bugetul de energie pentru procesare susținută — consumul maxim de energie are loc în timpul Thread 3 (Procesare), care solicită intens GPU-ul.

### Recomandări privind stocarea

* **SSD NVMe** recomandat cu tărie pentru implementările arm64
* Cardurile SD sunt prea lente pentru procesare — utilizați-le doar ca suport de boot
* Prevedeți un spațiu de stocare de 2-3 ori mai mare decât dimensiunea datelor brute ale imaginii pentru rezultatul procesat

### Funcționare fără monitor prin SSH Interfața

Chloros
a SSH
este ideală pentru implementările Jetson fără monitor:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend permanent activ pentru sincronizarea temporală LATTICE / DAQ-E

Dacă Jetson-ul dvs. controlează fără monitor camerele LATTICE sau senzorii de lumină DAQ-E, activați serviciul systemd de backend **ca utilizator al dvs.**, astfel încât grandmaster-ul PTP să funcționeze continuu și scripturile SDK
să găsească un backend în funcțiune (unitatea este instalată, dar nu este activată implicit):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend-ul rulează apoi ca și cum ați fi dumneavoastră: capturile pe care le înregistrează sunt ale dumneavoastră și utilizează fișierul dumneavoastră `chloros-cli login`. Consultați [Instalarea pe Linux
](linux-installation.md#always-on-ptp-for-headless-hosts) pentru detalii, inclusiv contul root `chloros-backend.service` și modul în care pachetul face ca porturile PTP 319/320 să poată fi asociate fără drepturi de root.

### Procesare automatizată cu systemd

Creați un serviciu systemd pentru procesarea automată:

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

`chloros-cli process` se închide cu un cod de ieșire diferit de zero atunci când o execuție care a solicitat produse nu scrie nicio imagine, astfel încât starea de eșec a systemd este semnificativă pentru monitorizare.

Combinați-l cu un timer systemd pentru procesare programată:

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

## Exemple de fluxuri de lucru

### Procesare de bază pe Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### SDK
-Python
pe Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Procesarea în lot a mai multor zboruri

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Sisteme Jetson recomandate pentru utilizare pe teren

Pentru implementări pe teren și aeriene, luați în considerare aceste opțiuni de plăci de bază Jetson Orin NX de 16 GB:

* **Aeriene/drone**: Sisteme cu rezistență la vibrații (MIL-STD), ușoare (sub 300 g), răcire pasivă
* **Utilizare în teren în condiții extreme**: carcase impermeabile IP67/IP69K cu conectivitate pentru cameră GigE PoE
* **Configurație minimă/economică**: kituri pentru dezvoltatori cu carcase suplimentare

Contactați [Asistența MAPIR
](https://www.mapir.camera/community/contact) pentru recomandări specifice privind hardware-ul, adaptate scenariului dvs. de implementare.

***

## Pași următori

* [Instalarea Linux
](linux-installation.md) — Detalii generale privind instalarea Linux

* [Adaptarea dinamică a puterii de calcul](../processing-architecture/dynamic-compute-adaptation.md) — Referință completă privind strategia de calcul
* [Fluxul de procesare](../processing-architecture/processing-pipeline.md) — Înțelegerea fluxului cu 4 fire
* [CLI
: linia de comandă](../CLI.md) — Ghidul CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Ghidul SDK

* [Referință CLI
](../reference/cli-reference.md) și [Referință SDK
](../reference/sdk-reference.md) — Liste exhaustive de comenzi/API
pentru versiunea 1.3.0
