# Prezentare generală Linux

Chloros 1.3.0 oferă suport nativ pentru Linux pentru **CLI**și**Python SDK** — procesarea imaginilor multispectrale fără interfață grafică, precum și controlul în timp real al camerelor LATTICE și al senzorilor de lumină DAQ — pe stații de lucru Linux, servere și dispozitive edge NVIDIA Jetson.

{% hint style="info" %}
**Nu există interfață grafică de desktop pe Linux.**Interfața grafică de desktop Chloros este disponibilă doar pentru Windows. Utilizatorii Linux interacționează cu Chloros prin intermediul [CLI](../CLI.md) și [Python SDK](../api-python-sdk.md). `.deb` adaugă o intrare**Chloros CLI** în meniul aplicației — aceasta deschide pur și simplu un emulator de terminal care rulează `chloros-cli`.
{% endhint %}

***

## Matrice de compatibilitate cu platformele

| Caracteristică | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfață grafică pentru desktop** | Da | N/A | Nu | Nu |
| **Interfață CLI** (`chloros-cli`) | Da | Da | Da | Da |
| **SDK Python** (`chloros-sdk`) | Da | Da | Da | Da |
| **Flux de procesare a imaginilor** | Da | Da | Da | Da |
| **Controlul camerei LATTICE (în timp real)** | Da (fila „Camere”) | Da (`chloros-cli lattice`, SDK) | Da | Da |
| **Senzori de lumină DAQ (în timp real)** | Da (fila Senzori de lumină) | Da (`chloros-cli daq pool-*`, SDK) | Da | Da |
| **Sincronizare temporală PTP (gazda este grandmaster)** | Da | Da (`chloros-cli time-sync`) | Da | Da |
| **Accelerare GPU (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Debayer cu recunoaștere a texturilor** (necesită un GPU CUDA) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Adaptare dinamică a calculului** | Da | Da | Da | Da |
| **Backend ca serviciu de sistem** (`chloros-backend@<user>.service`) | Nu | Nu | Da (opțional) | Da (opțional) |
| **Program de actualizare directă** (`chloros-cli update`) | Nu (rulați programul de instalare) | Nu (rulați programul de instalare) | Da | Da |

***

## Arhitecturi acceptate

| Arhitectură | Descriere | Pachet |
| --- | --- | --- |
| **amd64 (x86_64)** | Procesoare standard pentru desktop/server (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesoare ARM — familia NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Distribuții Linux acceptate

* **Ubuntu 22.04 LTS sau o versiune mai recentă** (amd64)
* **Debian 12 sau o versiune mai recentă** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson Orin)

***

## Ce beneficiază utilizatorii Linux

* **Chloros CLI** — interfața completă de linie de comandă pentru procesare în lot, automatizare și scripturi
* **Chloros Python SDK** — interfață programabilă Python pentru fluxuri de lucru de cercetare și instrumente personalizate, instalată de `.deb` pentru sistemul `python3` (un wheel compatibil cu versiunea pentru virtualenvs se găsește în `/usr/lib/chloros/sdk/`; PyPI este destinat mașinilor fără pachet Chloros)
* **Controlul camerelor LATTICE** — descoperiți, conectați, configurați și capturați imagini de la camerele LATTICE și de la rețele sincronizate de camere multiple prin intermediul `chloros-cli lattice` și al SDK-ului; `.deb` include mediul de execuție Arena SDK necesar camerelor
* **Controlul senzorilor de lumină DAQ** — conectați senzori DAQ-U/M/E, transmiteți în flux spectre calibrate și înregistrați fișiere `.daq` prin intermediul `chloros-cli daq pool-*` și al SDK-ului
* **Sincronizare temporală PTP** — backend-ul Chloros rulează grandmasterul PTP la care se sincronizează camerele LATTICE și senzorii DAQ-E; verificați-l cu `chloros-cli time-sync` și mențineți-l în funcțiune fără interfață grafică cu unitatea systemd `chloros-backend@<user>.service` (consultați [Instalarea pe Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatizarea proiectelor** — rulați proiectele salvate fără interfață grafică cu `chloros-cli project` și `open_project` din SDK
* **Accelerare GPU** — procesare accelerată prin CUDA pe GPU-urile NVIDIA (GPU-uri de desktop pe amd64 și GPU-ul Jetson cu JetPack 6)
* **Adaptare dinamică a calculului** — detectarea automată a hardware-ului și selectarea strategiei de procesare, cu opțiunea de suprascriere `CHLOROS_STRATEGY` ca soluție de rezervă pentru experți
* **Toate funcțiile de procesare** — același flux de lucru ca în Windows: calibrare, corectarea vignetării, indici de vegetație și toate formatele de export
* **Funcționalități Chloros+** — procesare multithread (în pipeline), debayer Texture Aware (pe un GPU CUDA) și indici personalizați, cu un abonament Chloros+ plătit

## Ce nu au la dispoziție utilizatorii de Linux

* **Interfață grafică pentru desktop** — nu există interfață grafică; toată interacțiunea se realizează prin CLI sau Python SDK
* **Vizualizator de imagini** — nu există vizualizator interactiv de imagini, vizualizare în grilă sau marcaje pe hartă
* **Gestionarea vizuală a proiectelor** — proiectele sunt create și gestionate prin comenzi CLI și apeluri SDK (hardware-ul în sine — camere, senzori, captură — rămâne pe deplin controlabil din terminal)

***

## Cerințe de licență

Accesul la CLI și SDK necesită un **nivel Chloros+ plătit — Copper sau superior**(Copper, Bronze, Silver, Gold). Nivelul gratuit**Iron** nu are acces la CLI/SDK. Limita minimă este impusă de backend, nu doar de CLI:

| Situație | Răspunsul backend-ului |
| --- | --- |
| Nu sunteți autentificat | `401` cu `error_code: AUTH_REQUIRED` |
| Autentificat la nivelul gratuit Iron | `403` cu `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funcționează pe orice nivel — este singura rută scutită de filtrare — astfel încât motivul refuzului este întotdeauna vizibil.

***

## Noțiuni introductive pentru Linux

1. **Instalați Chloros** — consultați [Instalarea pe Linux](linux-installation.md) pentru instalarea `.deb`
2. **Verificați** — `chloros-cli --version` afișează `Chloros CLI 1.3.0`; `chloros-cli selftest` rulează diagnosticul în 7 pași
3. **Verificați SDK Python** — `.deb` îl instalează: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (într-un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Autentificați-vă** — `chloros-cli login your@email.com 'your-password'` (o singură dată pe mașină; actualizările pachetului păstrează datele de autentificare, iar dezinstalarea pachetului le șterge)
5. **Procesați primul set de date** — `chloros-cli process ~/datasets/flight001`

Pentru NVIDIA Jetson, consultați [Ghidul dedicat NVIDIA Jetson](nvidia-jetson-guide.md) pentru configurarea specifică platformei, comportamentul termic și implementarea în teren.

***

## Pași următori

* [Instalare Linux](linux-installation.md) — instalare detaliată, locațiile fișierelor și depanare pentru amd64 și arm64
* [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md) — configurare specifică pentru Jetson, comportamentul memoriei și termic, implementare în teren
* [CLI: Linia de comandă](../CLI.md) — ghidul CLI
* [API: Python SDK](../api-python-sdk.md) — ghidul SDK
* [Referință CLI](../reference/cli-reference.md) și [Referință SDK](../reference/sdk-reference.md) — liste exhaustive de comenzi/API pentru versiunea 1.3.0
* [Adaptare dinamică la puterea de calcul](../processing-architecture/dynamic-compute-adaptation.md) — modul în care Chloros se adaptează la hardware-ul dvs.

{% hint style="info" %}
**Citirea acestui manual prin intermediul programării.** Fiecare pagină este, de asemenea, disponibilă în format Markdown brut la propria sa adresă URL, plus `.md` (de exemplu, `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), iar un index al întregului manual este publicat la [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
