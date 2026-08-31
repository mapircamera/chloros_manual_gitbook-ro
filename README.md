---
metaLinks: {}
---

# Noțiuni introductive

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

este o aplicație software de la [MAPIR

](https://www.mapir.camera) destinată procesării imaginilor multispectrale, controlului în timp real al hardware-ului dMAPIR

și înregistrării datelor de la senzori.Chloros

1.2.0 suportă întreaga gamă de produse dMAPIR

:

* **CamereleSurvey3** — prelucrează capturile RAW+JPG în hărți calibrate de reflectanță și indici de vegetație. Consultați [Camerele compatibile](supported-cameras.md).
* **Camerele LATTICE** — conectați în timp real modulele de camere multispectrale GigE, individual sau ca matrice sincronizate de camere multiple: previzualizați, capturați și prelucrați imaginile pentru a obține produse calibrate de radianță și reflectanță. Consultați [secțiunea LATTICE](lattice/README.md).
* **Senzori de lumină DAQ** — senzori spectrali DAQ-U (USB), DAQ-M (Bluetooth) și DAQ-E (Ethernet): spectre calibrate în timp real, înregistrări `.daq` și iluminare descendentă pentru procesarea reflectanței. Consultați [secțiunea DAQ](daq/README.md).

{% hint style="success" %}
**Noutăți în versiunea 1.2.0 aChloros**: control în timp real al camerelor și matricelor LATTICE, integrarea senzorilor de lumină DAQ, moduri de captură și înregistratoare, un flux complet de procesare radiometrică LATTICE, automatizarea proiectelor prinCLI

/SDK

și multe altele. Consultați lista „Noutăți” de mai jos și [Descărcați](download.md) pentru jurnalul de modificări.
{% endhint %}

{% hint style="info" %}
**FolosițiChloros

cu un asistent AI?** Acest manual este conceput special pentru asta. Indicați-i asistentului dvs.:

* `https://mapir.gitbook.io/chloros/llms.txt` — indexul citibil de mașină al fiecărei pagini.
* Orice pagină în format Markdown brut — adăugați `.md` la adresaURL

a paginii respective (de exemplu, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [ReferințaCLI

](reference/cli-reference.md) și [ReferințaSDK

](reference/sdk-reference.md) — pagini de referință complete, cu valori exacte, scrise pentru utilizarea de către modelele LLM.

Exemplu de prompt: *„Citește https://mapir.gitbook.io/chloros/reference/cli-reference.md,, apoi scrie un script care să se autentifice și să proceseze folderul ~/flights/flight_001 în fișiere GeoTIFF de tip reflectanță +NDVI

.”*

Ghid complet: [UtilizareaChloros

cu asistenții AI](ai-assistants.md).
{% endhint %}

***

## Noutăți înChloros

1.2.0

* **Controlul camerei în timp real — noua filă „Camere”.** Conectați camerele LATTICE una câte una sau ca matrice sincronizate de mai multe camere (sincronizare temporală PTP, captură declanșată hardware), cu suprapuneri de previzualizare în timp real, histograme pe benzi, expunere automată inteligentă, un calculator de index în timp real și actualizări de firmware pentru camere direct din aplicație.
* **Senzori de lumină — noua filă „Senzori de lumină”.** Conectați senzori DAQ-U (USB), DAQ-M (Bluetooth) și DAQ-E (Ethernet); vizualizați spectre calibrate în timp real (W/m²/nm), înregistrați fișiere `.daq` în proiectul dvs., alegeți profiluri de corecție a capacității și actualizați firmware-ul DAQ-E prin rețea.
* **Moduri de captură și înregistratoare.** Captură unică / Continuă / la intervale, plus un mod de captură „Fastest Capture” exclusiv pentru date brute; selectarea, pentru fiecare proiect, a camerelor și a tipurilor de export generate de opțiunea „Capture All”; înregistratoare matriciale pentru videoclipuri indexate de calitate de monitorizare și rafale de date brute de calitate de analiză, cu generarea offline a videoclipurilor.
* **Fluxul de procesare LATTICE.** Importați folderele de captură LATTICE și transformați fiecare cadru brut în produse debayered, de previzualizare, radianță float32 (W/m²/sr/nm) și reflectanță, cu comutatoare pentru fiecare produs. Reflectanța poate proveni de la o țintă de calibrare din cadru sau de la lumina descendentă DAQ; alinierea matricei se aplică exporturilor; calibrarea din fabrică lipsă se descarcă automat pe baza numărului de serie al camerei.
* **Proiectele rețin configurația hardware.** Camerele și senzorii de lumină conectați sunt salvați odată cu proiectul (`cameras.json` / `sensors.json`) și se reconectează cu setările salvate atunci când redeschideți proiectul. Consultați [GUI: Proiecte](projects.md).
* **Îmbunătățiri ale vizualizatorului de imagini.** Afișarea pixelilor/indicelui cursorului cu scalarea corectă a reflectanței pentru fiecare fișier, histograme ale straturilor, un glisor de grupare GSD, moduri de grilă „Per Trigger” / „Per Camera”, vizualizări ale produsului LATTICE și exporturi ale sandbox-ului Index/LUT pe disc.
* **Funcțiile „CLI

” și „SDK

”, extinse considerabil.** Noi familii de comenzi `lattice`, `daq pool-*`, `project` și `time-sync`; noi opțiuni `process` (`--input-level`, comutatoare per produs, `--reflectance-source`, indicatori de aliniere a matricei);SDK

mânere smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) care pornesc automat backend-ul; automatizarea `open_project()`; roata „SDK

” este inclusă în pachetele de instalare și publicată pe PyPI sub numele `chloros-sdk`.
* **Semantică onestă a eșecurilor.** O execuție `chloros-cli process` care a solicitat produse, dar nu a scris niciunul, eșuează acum în mod evident și se închide cu un cod de ieșire diferit de zero; execuțiile reușite raportează câte produse imagine au fost scrise.
* **Nou format de ieșire.** Produsele sunt stocate în dosare `<project>/<camera>/<format>/<Product>_Images/` și păstrează numele fișierului sursă — produsul este identificat de dosar, nu de un sufix al numelui fișierului. Consultați [Formate de imagine de ieșire](output-image-formats.md).
* **Mai multe intrări, planuri și limbi.** Suport pentru intrări `.dng`; toate cele 38 de limbi ale interfeței sunt complet disponibile; limite hardware per plan, cu utilizare gratuită (fără autentificare) de până la 4 camere și 2 senzori de lumină.
* **Fiabilitate.** Opțiunea „Stop Processing” se închide corect, cu un rezumat sincer al rulării, proiectele cu mai multe camere exportă datele de la fiecare cameră, iar actualizările programului de instalare nu vă mai deconectează.***

Chloros

este disponibil în 3 interfețe de utilizare:

##Chloros

: Aplicație GUI pentru desktop

Fereastră separată, autonomă, cu toate funcționalitățile, inclusiv filele „Camere live” și „Senzori de lumină”. _Numai pentru Windows._

## [Chloros

CLI

: Interfață de linie de comandă](CLI.md)

Prelucrare în lot prin linia de comandă, plus comenzi în timp real `lattice`, `daq pool-*`, `project` și `time-sync`. Perfect pentru automatizare, scripturi și funcționare fără interfață grafică. Disponibil pe **Windows

,Linux

amd64 șiLinux

arm64 (NVIDIA Jetson)**. _Pentru a accesa CLI este necesar un abonament plătit de tip „Chloros

+”._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Interfață programaticăPython

pentru automatizare și fluxuri de lucru personalizate: procesare completă a fluxului de lucru, sesiuni live cu camere/matrice, sesiuni cu senzori DAQ și automatizarea proiectelor salvate. Se instalează odată cu pachetul desktop/CLI

și este publicat și sub denumirea `pip install chloros-sdk`. _Pentru a accesa API-ul, este necesar un abonament plătitChloros

+._

***

## Platforme acceptate

| Platformă | Interfață grafică |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Da | Da | Da |
| **Linux

amd64 (x86_64)** | Nu | Da | Da |
| **Linux

arm64 (NVIDIA Jetson)** | Nu | Da | Da |

Pentru instrucțiuni de instalare peLinux

, consultați secțiunea [Linux

&amp; Edge Computing](linux/linux-overview.md).

***

## Începeți în trei pași

1. **Instalare** — descărcați și rulați programul de instalare pentru platforma dvs. Consultați [Descărcare](download.md).
2. **Autentificați-vă (opțional pentru interfața grafică)** — interfața grafică procesează imagini gratuit, fără a fi necesar un cont. O [Chloros

+ autentificare](chloros+-login.md) deblochează procesarea paralelă, accelerarea prin GPU, limite mai mari pentru dispozitive și acces laCLI

/SDK

.
3. **Creați-vă primul proiect** — deschidețiChloros

, creați un [Proiect nou](projects.md), [adăugați imaginile](processing-images-gui/adding-files-to-a-project.md) și [începeți procesarea](processing-images-gui/starting-the-processing.md). Pentru a controla hardware-ul în timp real, deschideți fila „Cameras” (Camere) sau „Light Sensors” (Senzori de lumină) — consultați [GUI: Navigare](navigation.md).

***

##Chloros

+

DeșiChloros

este gratuit pentru majoritatea sarcinilor, s-ar putea să doriți mai mult. În acest caz, o licență plătită pentruChloros

+ vă poate fi de folos. Cu o licențăChloros

+ puteți debloca funcții noi, cum ar fi:

* **Procesare multithread**: accelerați considerabil procesarea imaginilor pentru proiecte mai mari prin procesarea simultană a imaginilor prin pipeline.
* **Accelerare GPU (CUDA)**: profitați de opțiunile actuale de memorie GPU mai mare pentru a accelera și mai mult fluxul de procesare a imaginilor. Vă recomandăm 4 GB sau mai mult de VRAM pentru rezultate optime.
* **Acces laChloros

+**[**CLI**](CLI.md): rulațiChloros

+ din linia de comandă pentru a automatiza și integra programul în propriul software. Disponibil pentru orice nivel de abonament plătit; aplicat la nivel de server.
* **Chloros

+**[**API**](api-python-sdk.md) **Acces:** rulațiChloros

+ de pePython

pentru control programatic, permițând integrarea perfectă cu fluxurile dvs. de cercetare, fluxurile de lucru de analiză a datelor și aplicațiile personalizate. Disponibil pe orice nivel de abonament plătit; aplicat pe partea de server.
* **Limite hardware mai mari**: conectați mai multe camere și senzori de lumină simultan. Fără autentificare, interfața grafică (GUI) conectează până la 4 camere și 2 senzori de lumină DAQ; planurile cu plată măresc ambele limite:

| Plan | Camere | Senzori de lumină DAQ |
| --- | --- | --- |
| Iron (gratuit, fără autentificare) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Utilizarea mai multor dispozitive**: fiecare licențăChloros

+ permite înregistrarea a 2 sau mai multe dispozitive. Utilizați contul dvs.MAPIR

Cloud pentru a gestiona dispozitivele înregistrate. Adăugați suport pentru mai multe dispozitive prin actualizarea licenței dvs.Chloros

+.
* **Metodă avansată de debayerizare sensibilă la textură:** o metodă de debayerizare de înaltă calitate, sensibilă la margini, combinată cu un model de eliminare a zgomotului bazat pe AI/ML, care elimină aproape tot zgomotul generat de debayerizare.
* **Formule personalizate pentru indici multispectrali:** introduceți indici multispectrali personalizați în calculatoarele raster de laChloros

, atât pentru procesare, cât și pentru mediul de testare pentru vizualizarea imaginilor.
* **Procesare la margini (Linux

) și Edge Computing:** rulațiChloros

pe platformeleLinux

x86\_64 și ARM64, inclusiv NVIDIA Jetson, pentru procesare pe teren și la margini. Consultați [Prezentarea generală aLinux

](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Prețuri și înscriere</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
