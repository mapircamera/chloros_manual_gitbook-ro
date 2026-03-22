# Prezentare generală Linux

Chloros 1.1.0 oferă suport nativ Linux pentru **CLI**și**Python SDK**, permițând procesarea imaginilor multispectrale fără monitor pe stațiile de lucru, serverele și dispozitivele edge NVIDIA Jetson.

{% hint style="info" %}
**Nu există GUI pe Linux.** GUI-ul desktop Chloros este disponibil numai pe Windows. Utilizatorii Linux interacționează cu Chloros prin intermediul [CLI](../CLI.md) și [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matrice de suport pentru platforme

| Caracteristică | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfață grafică desktop** | Da | N/A | Nu | Nu |
| **CLI** | Da | Da | Da | Da |
| **Python SDK** | Da | Da | Da | Da |
| **Accelerare GPU (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Debayer sensibil la textură** | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Adaptare dinamică a calculului** | Da | Da | Da | Da |***

## Arhitecturi acceptate

| Arhitectură | Descriere | Metodă de instalare |
| --- | --- | --- |
| **amd64 (x86_64)** | Procesoare standard pentru desktop/server (Intel, AMD) | Pachet `.deb` |
| **arm64 (aarch64)** | Procesoare bazate pe ARM, în principal NVIDIA Jetson | Pachet `.deb` (JetPack 6) |

## Distribuții Linux acceptate

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson)***

## Ce primesc utilizatorii Linux

* **Chloros CLI** — Interfață completă de linie de comandă pentru procesare în lot, automatizare și scripting
* **Chloros Python SDK** — Interfață programabilă Python (`pip install chloros-sdk`) pentru integrare în fluxuri de cercetare și instrumente personalizate
* **Accelerare GPU** — Procesare accelerată CUDA pe GPU-uri NVIDIA (desktop și Jetson)
* **Adaptare dinamică a calculului** — Detectare automată a hardware-ului și optimizare a strategiei de procesare
* **Toate funcțiile de procesare** — Același flux de procesare multispectral ca Windows (calibrare, corecție vignetă, indici de vegetație, toate formatele de export)
* **Funcționalități Chloros+** — Procesare multi-threaded, debayer Texture Aware, indici personalizați (cu licența Chloros+)

## Ce nu primesc utilizatorii Linux

* **Interfață grafică desktop** — Fără interfață grafică; toată interacțiunea se face prin CLI sau Python SDK
* **Vizualizator de imagini** — Fără vizualizator de imagini interactiv, vizualizare în grilă sau marcaje pe hartă
* **Gestionarea vizuală a proiectelor** — Proiectele sunt gestionate prin comenzi CLI și apeluri SDK***

## Noțiuni introductive despre Linux

1. **Instalați Chloros** — Consultați [Instalarea Linux](linux-installation.md) pentru instalarea pachetului `.deb`
2. **Instalați Python SDK** (opțional) — `pip install chloros-sdk`
3. **Activați licența** — `chloros-cli login your@email.com 'password'`
4. **Procesați primul set de date** — `chloros-cli process ~/datasets/flight001`

Pentru utilizatorii NVIDIA Jetson, consultați [Ghidul dedicat NVIDIA Jetson](nvidia-jetson-guide.md) pentru configurarea și optimizarea specifică platformei.

***

## Pași următori

* [Instalarea Linux](linux-installation.md) — Instrucțiuni detaliate de instalare pentru amd64 și arm64
* [Ghidul NVIDIA Jetson](nvidia-jetson-guide.md) — Configurare specifică pentru Jetson, gestionarea termică și implementarea în teren
* [CLI : Linia de comandă](../CLI.md) — Referință completă pentru CLI
* [API : Python SDK](../api-python-sdk.md) — Referință completă pentru SDK
* [Adaptare dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md) — Cum se adaptează Chloros la hardware-ul dvs.
