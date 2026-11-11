---
metaLinks: {}
---

# Noțiuni introductive

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros este o aplicație software de la [MAPIR](https://www.mapir.camera) destinată procesării imaginilor multispectrale, controlului în timp real al echipamentelor MAPIR și înregistrării datelor de la senzori. Chloros 1.3.0 suportă întreaga gamă de produse MAPIR:

* **Camerele Survey3** — prelucrează capturile RAW+JPG transformându-le în hărți calibrate de reflectanță și indici de vegetație. Consultați [Camerele compatibile](supported-cameras.md).
* **Camere LATTICE** — conectează în timp real modulele de camere multispectrale GigE, individual sau ca matrice sincronizate de camere multiple: previzualizează, captează și prelucrează datele în produse calibrate de radianță și reflectanță. Vezi [secțiunea LATTICE](lattice/README.md).
* **Senzori de lumină DAQ** — senzori spectrali DAQ-U (USB), DAQ-M (Bluetooth) și DAQ-E și DAQ-E-S (Ethernet): spectre calibrate în timp real, înregistrări `.daq` și iluminare descendentă pentru procesarea reflectanței. Consultați [secțiunea DAQ](daq/README.md).

{% hint style="success" %}
**Noutăți în Chloros 1.3.0**: Asistenții AI pot controla Chloros prin `chloros-mcp`, iar CLI, SDK și `chloros-mcp` oferă aceleași funcționalități. Produsele cu indice LATTICE au fost îmbunătățite, matricile complete de senzori sunt mai rapide și funcționează pe orice rețea, iar backend-ul continuă să funcționeze între comenzi. Consultați lista de noutăți de mai jos și [Descărcați](download.md) programele de instalare.
{% endhint %}

{% hint style="info" %}
**Folosiți Chloros cu un asistent AI?** Acest manual este conceput special pentru asta. Indicați-i asistentului:

* `https://mapir.gitbook.io/chloros/llms.txt` — index lizibil de către mașini pentru fiecare pagină.
* Orice pagină în format Markdown brut — adăugați `.md` la URL-ul acesteia (de ex. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referința CLI](reference/cli-reference.md) și [Referința SDK](reference/sdk-reference.md) — pagini de referință complete, cu valori exacte, scrise pentru utilizarea de către modelele LLM.

Exemplu de prompt: *„Citește https://mapir.gitbook.io/chloros/reference/cli-reference.md,, apoi scrie un script care se autentifică și prelucrează folderul ~/flights/flight_001 în fișiere GeoTIFF de reflectanță + NDVI.”*

Sau lăsați-l să ruleze direct Chloros: `chloros-mcp`, un server MCP instalat împreună cu SDK, funcționează cu Claude Desktop, Claude Code, Cursor și alți clienți MCP.

Ghid complet: [Utilizarea Chloros cu asistenții AI](ai-assistants.md).
{% endhint %}

***

## Noutăți în Chloros 1.3.0

* **Asistenții AI pot controla Chloros.** `chloros-mcp`, un server MCP instalat împreună cu Python SDK, permite aplicațiilor Claude Desktop, Claude Code, Cursor și altor clienți MCP să proceseze imagini și să controleze camerele LATTICE, matricile și senzorii de lumină DAQ. Consultați [Utilizarea Chloros cu asistenți AI](ai-assistants.md).
* **CLI-ul, SDK-ul și `chloros-mcp` oferă aceleași funcționalități.** Din motive de securitate, doar autentificarea, configurarea DAQ-E și serverele DAQ directe rămân în cadrul CLI-ului. Unele scripturi din versiunea 1.2.0 necesită modificări: consultați secțiunea **Modificări în versiunea 1.3.0** din partea de sus a [Referinței CLI](reference/cli-reference.md) și a [Referinței SDK](reference/sdk-reference.md).
* **Produse indexate LATTICE îmbunătățite.** Indicele NDVI și celelalte produse indexate exportate din capturile LATTICE au fost îmbunătățite, în toate modurile de procesare.
* **Opțiuni ale programului de instalare pentru Windows.**În secțiunea**Setări de sistem**, programul de instalare oferă opțiunile**Activare căi lungi Windows**și**Excluderea backend-ului Chloros din scanarea Microsoft Defender**, ambele bifate implicit. Prima opțiune permite ca folderele de proiect să aibă nume mai lungi de 260 de caractere; a doua opțiune împiedică scanarea în timp real să încetinească captarea cu camera (aproximativ 30% din rata de captare la viteze ridicate). De asemenea, programul de instalare adaugă acum folderul CLI în `PATH` o singură dată și elimină intrările duplicate lăsate de versiunile anterioare ale programului de instalare. Vezi [Descărcare](download.md).
* **Dosarele de proiect cu structură prea adâncă sunt respinse din start.** Cu opțiunea „Windows Long Paths” dezactivată (setarea implicită a Windows), un dosar de proiect prea adânc pentru limita de 260 de caractere a căii este respins înainte ca programul să se execute, fiind afișat cel mai lung dosar de proiect care se încadrează în limită și instrucțiuni privind activarea opțiunii „Windows Long Paths”. Anterior, o astfel de execuție nu genera niciun produs. Cu căile lungi activate, produsele din dosare foarte adânci își păstrează acum metadatele (etichete GPS și XMP).
* **Măști de limitare.** Când separarea benzilor unui produs de radianță sau reflectanță LATTICE produce o valoare negativă a benzii, acea valoare este limitată la zero, iar o mască per pixel dintr-un folder `clamp_masks/` situat lângă folderele produselor înregistrează care pixeli și benzi au fost afectați. Produsul își denumește masca în eticheta `Chloros:UnmixClampMask`. Vizualizările în timp real se decupează acum exact la fel ca în cazul exportului. Consultați [Formate de imagine de ieșire](output-image-formats.md).
* **Produsele se descriu singure.** Produsele indexate poartă propriile etichete de procesare (`Chloros:ProcessingLevel=index`, expresia indexului și dacă au fost calculate pe baza radianței sau a reflectanței) în locul celor ale fișierului brut sursă. Produsele Survey3 poartă etichete de procesare Chloros în fiecare mod de procesare. Fișierele brute și cele debayered LATTICE rămân în orientarea nativă a senzorului și își înregistrează orientarea în XMP; Image Viewer le rotește doar pentru afișare.
* **Pachetele pentru Linux sunt livrate nearsivate.** Backend-ul și interfața CLI sunt instalate ca foldere sub `/usr/lib/chloros/`, astfel încât nu se mai dezarhivează la fiecare lansare. Pe x86_64, backend-ul nu mai are nevoie de aproximativ 100 de secunde pentru pornire, iar `.deb` are aproximativ jumătate din dimensiunea anterioară. ExifTool-ul inclus în pachet rulează acum pe o instalare curată de Ubuntu. `.deb` instalează acum și Python SDK pentru sistemul `python3` (instalați-l cu `sudo apt install ./chloros_1.3.0_<arch>.deb`), mărește bufferele de recepție ale socket-urilor din kernel de care au nevoie fluxurile camerei și adaugă un serviciu backend care rulează sub drepturile utilizatorului dvs. Pachetul Jetson, `chloros_1.3.0_arm64_jp6.deb`, este compilat pentru JetPack 6 și procesează pe GPU-ul Jetson cu versiunea CUDA a PyTorch pentru JetPack. Pe Linux, fiecare imagine exportată ajunge acum pe disc o singură dată (este scrisă și etichetată în memorie, apoi copiată la destinație): pe un Jetson, unde discul dictează ritmul, o rulare LATTICE cu 288 de imagini durează 14 minute în loc de 24. Consultați [Instalarea pe Linux](linux/linux-installation.md).
* **Camerele LATTICE OCN citesc mai precis în lumina soarelui.** Radianța, reflectanța și indicii beneficiază de o corecție rapidă după separarea benzilor, astfel încât o țintă gri este citită aproape de un NDVI neutru în soare. Camerele RGN rămân neschimbate.
* **Rapoarte de erori din aplicație.** O pictogramă de eroare din partea de sus a filei „Debug Log” trimite către MAPIR o descriere, până la 10 imagini, jurnalele și detaliile sistemului. A se vedea [GUI: Navigare](navigation.md).
* **Matrice LATTICE.**Matricea păstrează senzorul complet în mod implicit, iar transportul comprimat pe 12 biți (`--packed`, implicit `auto`) crește rata de cadre a senzorului complet cu aproximativ o treime; o matrice funcționează pe orice rețea la rata de cadre pe care aceasta o permite (o conexiune lentă reduce rata de cadre în loc să refuze matricea sau să-i reducă cadrele); o matrice ale cărei camere au fost repornite se recuperează singură, cu PTP reactivat; expunerea automată nu mai reduce rata respectivă în condiții de lumină slabă (umple perioada, apoi crește amplificarea); Captura continuă ține pasul cu rețelele rapide; un control**Highlights** pentru camerele RGB; iar `ArraySession` din SDK capătă `grab()`, `frame_stream()` și `continuous()`. Consultați [secțiunea LATTICE](lattice/README.md).
* **Expunere automată mai stabilă.** Expunerea se menține constantă la lumină constantă și se stabilizează în câteva secunde după o schimbare; zonele foarte luminoase nu mai întunecă scena; o expunere manuală lasă amplificarea pe modul „Auto”, iar o cameră care nu mai răspunde în lumina soarelui puternică este menținută la nivelul corespunzător luminii respective, citit de senzorul de lumină sau, în lipsa acestuia, de propriile cadre ale camerei. Consultați [Setările camerei](lattice/camera-settings.md).
* **Senzori de lumină.** **Suport pentru DAQ-E-S:** difuzorul său fix nu necesită corecție a capacului, iar fiecare citire include înclinația, rotația și unghiul de înclinare ale unității. O înregistrare indică dacă capacul a fost ales sau presupus, iar procesarea avertizează în cazul unui capac presupus: DAQ-U, DAQ-M și DAQ-E sunt livrate fără capac, așa că denumiți-l pe cel montat. Înregistrările live `.daq` supraviețuiesc unei întreruperi de curent (păstrați un fișier `.daq-journal` împreună cu `.daq`), iar un DAQ-E care încetează să transmită date este reconectat cu intervalul marcat în înregistrarea sa. Un proiect care conține doar înregistrări de la senzori de lumină le listează și le procesează atât din aplicație, cât și din CLI, iar o înregistrare realizată cu `chloros_scripts` este listată ca aparținând DAQ-ului care a realizat-o, cu un număr de caractere maxim pe care îl puteți seta. Fiecare captură salvează valoarea citită de DAQ-ul folosit. Consultați [secțiunea DAQ](daq/README.md).
* **Procesare.**Cadrele LATTICE încep procesarea în timpul detectării țintei. Referințele țintei și ale DAQ-ului sunt stabilite pentru fiecare bandă. Un proiect mutat își reconectează fișierele. Exporturile unei capturi pe 8 biți sunt pe 8 biți, într-un nou format**TIFF (8 biți)**. O execuție Survey3 fără țintă de calibrare este salvată ca produs de rezervă (`Vignette_Corrected_Images` sau `Sensor_Response_Images`), nu ca imagine de reflectanță. Etichetele DLS pe bandă ale unei imagini de reflectanță sunt întotdeauna cele ale propriului cadru. Opțiunea „Texture Aware” este refuzată pentru capturile LATTICE, care beneficiază întotdeauna de procesarea standard de debayer.
* **CLI și SDK.** Căile relative de ieșire se află acum în folderul din care rulați comanda sau scriptul (un backend instalat le rezolva anterior în propriul folder de instalare și dădea eroare). Backend-ul continuă să ruleze între comenzi și scripturi și se oprește automat după 30 de minute de inactivitate; `chloros-cli backend status` și `backend stop` gestionează acest lucru. Comenzile de captură menționează fiecare fișier pe care nu l-au putut scrie și se închid cu codul 1, iar capturile repetate se opresc atunci când unitatea de stocare este plină. O execuție reușită a `process` raportează `Image products written: N (plus M clamp masks)`. Pe Windows, `ChlorosLocal` / `process_folder` găsesc backend-ul instalat (și utilizează unul care rulează deja), iar `read_image_audit_tags` funcționează dintr-un fișier simplu `pip install`.

***

## Noutăți în Chloros 1.2.0

* **Controlul camerei în timp real — noua filă „Camere”.** Conectați camerele LATTICE una câte una sau ca matrice sincronizate de mai multe camere (sincronizare temporală PTP, captură declanșată hardware), cu suprapuneri de previzualizare în timp real, histograme pe benzi, expunere automată inteligentă, un calculator de index în timp real și actualizări de firmware pentru camere direct din aplicație.
* **Senzori de lumină — noua filă „Senzori de lumină”.** Conectați senzori DAQ-U (USB), DAQ-M (Bluetooth) și DAQ-E (Ethernet); vizualizați spectre calibrate în timp real (W/m²/nm), înregistrați fișiere `.daq` în proiectul dvs., alegeți profiluri de corecție a capacității și actualizați firmware-ul DAQ-E prin rețea.
* **Moduri de captură și înregistratoare.** Captură unică / Continuă / la intervale, plus un mod de captură „Fastest Capture” exclusiv pentru date brute; selectarea, pentru fiecare proiect, a camerelor și a tipurilor de export generate de opțiunea „Capture All”; înregistratoare matriciale pentru videoclipuri indexate de calitate de monitorizare și rafale de date brute de calitate de analiză, cu generarea offline a videoclipurilor.
* **Fluxul de procesare LATTICE.** Importați folderele de captură LATTICE și transformați fiecare cadru brut în produse debayered, de previzualizare, radianță float32 (W/m²/sr/nm) și reflectanță, cu comutatoare pentru fiecare produs. Reflectanța poate proveni de la o țintă de calibrare din cadru sau de la fluxul descendent DAQ; alinierea matricei se aplică exporturilor; calibrarea din fabrică lipsă este descărcată automat pe baza numărului de serie al camerei.
* **Proiectele rețin configurația hardware.** Camerele și senzorii de lumină conectați sunt salvați împreună cu proiectul (`cameras.json` / `sensors.json`) și se reconectează cu setările salvate atunci când redeschideți proiectul. Consultați [GUI: Proiecte](projects.md).
* **Îmbunătățiri ale vizualizatorului de imagini.** Afișarea pixelilor/indicelui cursorului cu scalarea corectă a reflectanței pentru fiecare fișier, histograme ale straturilor, un glisor de grupare GSD, moduri de grilă „Per Trigger” / „Per Camera”, vizualizări ale produsului LATTICE și exporturi ale sandbox-ului Index/LUT pe disc.
* **CLI și SDK, extinse semnificativ.** Noi familii de comenzi `lattice`, `daq pool-*`, `project` și `time-sync`; noi opțiuni `process` (`--input-level`, comutatoare per produs, `--reflectance-source`, indicatori de aliniere a matricei); mânere de conectare inteligentă SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`) care pornesc automat backend-ul; Automatizarea `open_project()`; pachetul SDK este inclus în programele de instalare și publicat pe PyPI sub numele de `chloros-sdk`.
* **Semantică onestă a eșecurilor.** O execuție `chloros-cli process` care a solicitat produse, dar nu a generat niciunul, eșuează acum în mod evident și se închide cu un cod de ieșire diferit de zero; execuțiile reușite raportează câte produse de imagine au generat.
* **Nou format de ieșire.** Produsele sunt salvate în dosare `<project>/<camera>/<format>/<Product>_Images/` și păstrează numele fișierului sursă — dosarul, nu sufixul numelui fișierului, identifică produsul. Consultați [Formate de imagine de ieșire](output-image-formats.md).
* **Mai multe intrări, planuri și limbi.** Suport pentru intrări `.dng`; toate cele 38 de limbi ale interfeței sunt complet disponibile; limite hardware per plan, cu utilizare gratuită (fără autentificare) de până la 4 camere și 2 senzori de lumină.
* **Fiabilitate.** Opțiunea „Stop Processing” se închide corect, cu un rezumat sincer al execuției, proiectele cu mai multe camere exportă datele de la fiecare cameră, iar actualizările programului de instalare nu vă mai deconectează.***

Chloros este disponibil în 3 interfețe de utilizare:

## Chloros: Aplicație GUI pentru desktop

Fereastră separată, autonomă, cu toate funcționalitățile, inclusiv filele „Camere live” și „Senzori de lumină”. _Numai pentru Windows._

## [Chloros CLI: Interfață de linie de comandă](CLI.md)

Prelucrare în lot prin linia de comandă, plus comenzi în timp real `lattice`, `daq pool-*`, `project` și `time-sync`. Perfect pentru automatizare, scripturi și funcționare fără interfață grafică. Disponibil pe **Windows, Linux amd64 și Linux arm64 (NVIDIA Jetson)**. _Pentru a accesa CLI este necesar un abonament Chloros+ cu plată._

## [Chloros API: Python SDK](api-python-sdk.md)

Interfață programatică Python pentru automatizare și fluxuri de lucru personalizate: procesare completă a fluxului de lucru, sesiuni live cu camere/matrice, sesiuni cu senzori DAQ și automatizarea proiectelor salvate. Se instalează odată cu pachetul desktop/CLI și este publicat și sub denumirea `pip install chloros-sdk`. _Pentru a accesa API-ul este necesar un abonament Chloros+ plătit._

***

## Platforme acceptate

| Platformă | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Da | Da | Da |
| **Linux amd64 (x86_64)** | Nu | Da | Da |
| **Linux arm64 (NVIDIA Jetson)** | Nu | Da | Da |

Pentru instrucțiuni de instalare pe Linux, consultați secțiunea [Linux și Edge Computing](linux/linux-overview.md).

***

## Începeți în trei pași

1. **Instalați** — descărcați și rulați programul de instalare pentru platforma dvs. Consultați [Descărcare](download.md).
2. **Autentificați-vă (opțional pentru interfața grafică)** — interfața grafică procesează imagini gratuit, fără a fi necesar un cont. O [autentificare Chloros+](chloros+-login.md) deblochează procesarea paralelă, accelerarea GPU, limite mai mari pentru dispozitive și acces la CLI/SDK.
3. **Creați-vă primul proiect** — deschideți Chloros, creați un [Proiect nou](projects.md), [adăugați imaginile](processing-images-gui/adding-files-to-a-project.md) și [începeți procesarea](processing-images-gui/starting-the-processing.md). Pentru a controla hardware-ul în timp real, deschideți fila „Camere” sau „Senzori de lumină” — consultați [GUI: Navigare](navigation.md).

***

## Chloros+

Deși Chloros este gratuit pentru majoritatea sarcinilor, s-ar putea să doriți mai multe funcționalități. În acest caz, o licență plătită pentru Chloros+ vă poate fi de folos. Cu o licență Chloros+ puteți debloca funcții noi, precum:

* **Procesare multi-thread**: accelerați considerabil procesarea imaginilor pentru proiecte mai mari prin procesarea simultană a imaginilor în cadrul fluxului de lucru.
* **Accelerare GPU (CUDA)**: profitați de opțiunile actuale de memorie GPU mai mare pentru a accelera și mai mult fluxul de procesare a imaginilor. Vă recomandăm 4 GB sau mai mult de VRAM pentru rezultate optime.
* **Chloros+**[**CLI**](CLI.md) **Acces**: rulați Chloros+ din linia de comandă pentru a automatiza și integra programul în propriul software. Disponibil pentru orice nivel de abonament plătit; aplicat la nivel de server.
* **Chloros+**[**API**](api-python-sdk.md) **Acces:** rulați Chloros+ din Python pentru control programatic, permițând integrarea perfectă cu fluxurile dvs. de cercetare, fluxurile de lucru de analiză a datelor și aplicațiile personalizate. Disponibil pe orice nivel de abonament plătit; aplicat pe partea de server.
* **Limite hardware mai mari**: conectați mai multe camere și senzori de lumină simultan. Fără autentificare, interfața grafică (GUI) conectează până la 4 camere și 2 senzori de lumină DAQ; planurile cu plată măresc ambele limite:

| Plan | Camere | Senzori de lumină DAQ |
| --- | --- | --- |
| Iron (gratuit, fără autentificare) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Utilizarea mai multor dispozitive**: fiecare licență Chloros+ permite înregistrarea a 2 sau mai multe dispozitive. Folosiți contul dvs. MAPIR Cloud pentru a gestiona dispozitivele înregistrate. Adăugați suport pentru mai multe dispozitive prin actualizarea licenței dvs. Chloros+.
* **Metodă avansată de debayerizare sensibilă la textură:** o metodă de debayerizare de înaltă calitate, sensibilă la margini, combinată cu un model de eliminare a zgomotului bazat pe AI/ML, care elimină aproape tot zgomotul generat de debayerizare.
* **Formule personalizate pentru indici multispectrali:** introduceți indici multispectrali personalizați în calculatoarele raster Chloros, atât pentru procesare, cât și pentru mediul de testare de vizualizare a imaginilor.
* **Linux și Edge Computing:** rulați Chloros pe platformele Linux x86_64 și ARM64, inclusiv NVIDIA Jetson, pentru procesare pe teren și la marginea rețelei. Consultați [Prezentarea generală Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Prețuri și înscriere la Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
