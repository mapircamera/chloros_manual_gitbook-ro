# Prezentare generală Linux

Chloros 1.2.0 oferă suport nativ Linux pentru **CLI**și**Python SDK** — procesarea imaginilor multispectrale fără monitor, precum și controlul în timp real al camerelor LATTICE și al senzorilor de lumină DAQ — pe stațiile de lucru, serverele și dispozitivele edge NVIDIA Jetson.

{% hint style="info" %}
**Nu există interfață grafică de desktop pe Linux.**Interfața grafică de desktop Chloros este disponibilă exclusiv pe Windows. Utilizatorii modelului Linux interacționează cu Chloros prin intermediul [CLI](../CLI.md) și [Python SDK](../api-python-sdk.md). `.deb` adaugă o**Chloros CLI** în meniul aplicației — pur și simplu deschide un emulator de terminal care rulează `chloros-cli`.
{% endhint %}

***

## Matrice de compatibilitate cu platformele

| Caracteristică | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfață grafică pentru desktop** | Da | N/A | Nu | Nu |
| **CLI** (`chloros-cli`) | Da | Da | Da | Da |
| **Python SDK** (`chloros-sdk`) | Da | Da | Da | Da |
| **Flux de procesare a imaginilor** | Da | Da | Da | Da |
| **Control cameră LATTICE (în timp real)** | Da (fila „Camere”) | Da (`chloros-cli lattice`, SDK) | Da | Da |
| **Senzori de lumină DAQ (în timp real)** | Da (fila Senzori de lumină) | Da (`chloros-cli daq pool-*`, SDK) | Da | Da |
| **Sincronizare temporală PTP (gazda este grandmaster)** | Da | Da (`chloros-cli time-sync`) | Da | Da |
| **Accelerare GPU (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Debayer cu recunoaștere a texturii** | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Adaptare dinamică a calculului** | Da | Da | Da | Da |
| **Backend ca serviciu de sistem** (`chloros-backend.service`) | Nu | Nu | Da (opțional) | Da (opțional) |
| **Program de actualizare la fața locului** (`chloros-cli update`) | Nu (rulați programul de instalare) | Nu (rulați programul de instalare) | Da | Da |***

## Arhitecturi acceptate

| Arhitectură | Descriere | Pachet |
| --- | --- | --- |
| **amd64 (x86_64)** | Procesoare standard pentru desktop/server (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesoare ARM — familia NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (versiune JetPack 6) |

## Distribuții Linux acceptate

* **Ubuntu 22.04 LTS sau o versiune mai recentă** (amd64)
* **Debian 12 sau o versiune mai recentă** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson Orin)***

## Ce beneficiază utilizatorii Linux

* **Chloros CLI** — interfața completă de linie de comandă pentru procesare în lot, automatizare și scriptare
* **Chloros Python SDK** — interfață programabilă Python pentru fluxuri de lucru de cercetare și instrumente personalizate (instalabilă din PyPI și inclusă, de asemenea, în pachetul `.deb` ca fișier wheel compatibil cu versiunea)
* **Controlul camerelor LATTICE** — descoperiți, conectați, configurați și capturați imagini de la camerele LATTICE și de la rețele sincronizate de camere multiple prin intermediul `chloros-cli lattice` și SDK; `.deb` include runtime-ul Arena SDK necesar camerelor
* ****Controlul senzorilor de lumină DAQ** — conectați senzori DAQ-U/M/E, transmiteți în flux spectre calibrate și înregistrați fișiere `.daq` prin intermediul `chloros-cli daq pool-*` și al SDK
* **Sincronizare temporală PTP** — backend-ul Chloros rulează grandmaster-ul PTP la care camerele LATTICE și senzorii DAQ-E se sincronizează; verificați-l cu `chloros-cli time-sync`, și mențineți-l în funcțiune fără interfață grafică cu ajutorul unității systemd `chloros-backend.service` (consultați [Instalarea Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatizarea proiectelor** — rulați proiectele salvate fără interfață grafică cu ajutorul `chloros-cli project` și al `open_project` din SDK
* **Accelerare GPU** — procesare accelerată prin CUDA pe GPU-urile NVIDIA (desktop și Jetson)
* **Adaptare dinamică a calculului** — detectarea automată a hardware-ului și selectarea strategiei de procesare, cu opțiunea de suprascriere `CHLOROS_STRATEGY` ca soluție de rezervă pentru experți
* **Toate funcțiile de procesare** — același flux de procesare ca la Windows: calibrare, corecție de vignetare, indici de vegetație și toate formatele de export
* **Funcționalități Chloros+** — procesare multithread (în pipeline), debayer cu recunoaștere a texturii și indici personalizați, cu un plan Chloros+ plătit

## Ce nu primesc utilizatorii Linux

* **Interfață grafică pentru desktop** — nu există interfață grafică; toată interacțiunea se realizează prin CLI sau Python SDK
* **Vizualizator de imagini** — nu există vizualizator interactiv de imagini, vizualizare în grilă sau marcaje pe hartă
* **Gestionarea vizuală a proiectelor** — proiectele sunt create și gestionate prin comenzi CLI și apeluri SDK (hardware-ul în sine — camere, senzori, captură — rămâne pe deplin controlabil de la terminal)***

## Cerințe privind licența

Accesul la CLI și SDK necesită un **nivel Chloros+ plătit — Copper sau superior**(Copper, Bronze, Silver, Gold). Nivelul gratuit**Iron** nu oferă acces la CLI/SDK. Limita minimă este impusă de backend, nu doar de CLI:

| Situație | Răspunsul backend-ului |
| --- | --- |
| Nu sunteți autentificat | `401` cu `error_code: AUTH_REQUIRED` |
| Autentificat pe nivelul gratuit Iron | `403` cu `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funcționează pe orice nivel — este singura rută scutită de filtru — astfel încât motivul refuzului este întotdeauna vizibil.

***

## Noțiuni introductive despre Linux

1. **Instalați Chloros** — consultați [Instalarea Linux](linux-installation.md) pentru instalarea `.deb`
2. **Verificați** — `chloros-cli --version` afișează `Chloros CLI 1.2.0`; `chloros-cli selftest` rulează diagnosticul în 7 pași
3. **Instalați Python și SDK** (opțional) — `pip install chloros-sdk`
4. **Autentificați-vă** — `chloros-cli login your@email.com 'your-password'` (o singură dată pe fiecare dispozitiv și din nou după fiecare actualizare a pachetului)
5. **Procesați primul set de date** — `chloros-cli process ~/datasets/flight001`

Pentru NVIDIA Jetson, consultați [Ghidul dedicat NVIDIA Jetson](nvidia-jetson-guide.md) pentru configurarea specifică platformei, comportamentul termic și implementarea în teren.

***

## Pași următori

* [Instalarea Linux](linux-installation.md) — instrucțiuni detaliate de instalare, locațiile fișierelor și depanare pentru amd64 și arm64
* [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md) — configurare specifică pentru Jetson, comportamentul memoriei și termic, implementare în teren
* [CLI : Linia de comandă](../CLI.md) — ghidul CLI
* [API : Python SDK](../api-python-sdk.md) — ghidul SDK
* [Referința CLI](../reference/cli-reference.md) și [Referința SDK](../reference/sdk-reference.md) — liste exhaustive de comenzi/API pentru versiunea 1.2.0
* [Adaptare dinamică la puterea de calcul](../processing-architecture/dynamic-compute-adaptation.md) — modul în care Chloros se adaptează la hardware-ul dvs.

{% hint style="info" %}
**Citirea acestui manual prin programare.** Fiecare pagină este disponibilă și în format Markdown brut la adresa proprie URL plus `.md` (de exemplu, `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), iar un index al întregului manual este publicat la [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
