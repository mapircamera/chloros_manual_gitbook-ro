---
metaLinks: {}
---

# Noțiuni introductive

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros este o aplicație software de la [MAPIR](https://www.mapir.camera) pentru procesarea imaginilor și a altor date de la senzori.

***{% hint style="success" %}**Noutăți în Chloros 1.1.0**: Suport nativ pentru Linux (amd64 și arm64), NVIDIA Jetson edge computing, Dynamic Compute Adaptation, pipeline de procesare cu 4 thread-uri, comenzi și opțiuni noi pentru CLI. Consultați [Descărcare](download.md) pentru jurnalul complet al modificărilor.
{% endhint %}

Chloros este disponibil în 3 moduri de aplicație:

## Chloros: Aplicație GUI pentru desktop

Fereastră separată independentă cu toate funcționalitățile. _Numai pentru Windows._

## [Chloros CLI: Interfață de linie de comandă](CLI.md)

Procesare în serie prin linia de comandă. Perfectă pentru automatizare, scripturi și funcționare fără interfață grafică. Disponibil pe **Windows, Linux amd64 și Linux arm64 (NVIDIA Jetson)**. _CLI necesită o licență Chloros+ pentru acces._

## [Chloros API: Python SDK](api-python-sdk.md)

Interfață programatică Python pentru automatizare și fluxuri de lucru personalizate. Perfectă pentru fluxuri de cercetare, integrare cu aplicații Python existente și crearea de instrumente personalizate. Disponibilă pe **toate platformele** prin `pip install chloros-sdk`. _API-ul necesită o licență Chloros+ pentru acces._***

## Platforme acceptate

| Platformă | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Da | Da | Da |
| **Linux amd64 (x86_64)** | Nu | Da | Da |
| **Linux arm64 (NVIDIA Jetson)** | Nu | Da | Da |

Pentru instrucțiuni de instalare Linux, consultați secțiunea [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Deși Chloros este gratuit pentru majoritatea sarcinilor, s-ar putea să doriți mai mult. În acest caz, o licență plătită pentru Chloros+ vă poate fi de folos. Cu o licență Chloros+ puteți debloca noi funcții, cum ar fi:

* **Procesare multi-threaded**: accelerați considerabil procesarea imaginilor pentru proiecte mai mari prin procesarea simultană a imaginilor prin pipeline.
* **Accelerare GPU (CUDA)**: profitați de opțiunile actuale de memorie GPU mai mare pentru a accelera și mai mult fluxul de procesare a imaginilor. Vă recomandăm 4 GB sau mai mult de VRAM pentru rezultate optime.
* **Chloros+**[**CLI**](CLI.md)**Acces**: rulați Chloros+ din linia de comandă pentru a automatiza și integra în propriul software.
* **Chloros+**[**API**](api-python-sdk.md)**Acces:** rulați Chloros+ din Python pentru control programatic, permițând integrarea perfectă cu fluxurile dvs. de cercetare, fluxurile de lucru de analiză a datelor și aplicațiile personalizate.
* **Utilizare pe mai multe dispozitive**: fiecare licență Chloros+ permite înregistrarea a 2+ dispozitive. Utilizați contul dvs. MAPIR Cloud pentru a gestiona dispozitivele înregistrate. Adăugați suport pentru mai multe dispozitive prin actualizarea licenței dvs. Chloros+.
* **Metodă avansată de debayerizare sensibilă la textură:** un debayer de înaltă calitate, sensibil la margini, combinat cu un model de denoizare AI/ML care elimină aproape tot zgomotul de debayerizare. 
* **Formule personalizate de indici multispectrali:** introduceți indici multispectrali personalizați în calculatoarele raster Chloros, atât pentru procesare, cât și pentru mediul de testare de vizualizare a imaginilor.
* **Linux și Edge Computing:** rulați Chloros pe platformele Linux x86\_64 și ARM64, inclusiv NVIDIA Jetson, pentru procesare pe teren și la margine. Consultați [Prezentarea generală a Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Prețuri și înscriere</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
