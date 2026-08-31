# Fila DAQ din Chloros

Fila DAQ — denumită **Senzori de lumină** în bara laterală a Chloros — reprezintă panoul de control în timp real pentru [senzorii de lumină DAQ-U, DAQ-M și DAQ-E](README.md): conectați senzorii prin orice protocol de transport, vizualizați spectrele calibrate în timp real, calculați reflectanța în timp real dintr-o pereche de senzori și înregistrați fișiere `.daq` direct în proiectul dvs.

Fila devine disponibilă odată ce backend-ul Chloros a terminat de pornit. Graficele din filă sunt alimentate de serviciul DAQ al Chloros printr-o conexiune în timp real care se reconectează automat (interval de așteptare de 2–10 s) în cazul unei întreruperi; cât timp serviciul este inaccesibil, rândul „Stare” al unui senzor afișează **Niciun server**.

Aspectul este format dintr-o **bară laterală a senzorilor**(un rând pentru fiecare senzor conectat) plus o**zonă de grafice** (o casetă de grafic pentru fiecare senzor sau grup).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Conectarea unui senzor

Faceți clic pe **Conectare senzor** în partea de sus a barei laterale. Fereastra de dialog de conectare se deschide în zona principală (sau ca o fereastră suprapusă atunci când se adaugă un alt senzor — în acest caz apare un buton Anulare).

| Control | Comportament |
| --- | --- |
| **Tipul dispozitivului** | `DAQ-U (USB)` (implicit), `DAQ-M (Bluetooth)` sau `DAQ-E (Ethernet)`. Comutarea repornește scanarea pentru transportul nou selectat. |
| **Port / Dispozitiv BLE / Nume gazdă / IP** | Afișează lista dispozitivelor detectate sub forma `device - description`; prima intrare recunoscută ca senzor este selectată automat. În timpul scanării, se afișează `Scanning...` (USB), `Scanning (N)...` cu o numărătoare inversă de 8 secunde (BLE) sau `Discovering ethernet sensors (N)...` cu o numărătoare inversă de 5 secunde (Ethernet). Rezultatele goale sunt afișate ca `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Reîmprospătare** | Reanalizează imediat transportul selectat (dezactivat în timpul scanării BLE/Ethernet). |
| **Conectare** | Se activează odată ce este selectat un dispozitiv; redenumește la `Connecting...` în timp ce se realizează conexiunea. |

Descoperirea se execută numai **cât timp fereastra de conectare este afișată pe ecran** și se repetă la fiecare 15 secunde numai pentru transportul selectat — simpla deschidere a filei nu declanșează scanarea. În caz de eșec, fereastra afișează: *„Conectarea a eșuat. Încercați să deconectați și să reconectați senzorul, apoi faceți clic din nou pe Conectare.”*

Bara laterală se deschide automat când se conectează primul senzor.

{% hint style="info" %}
**DAQ-E nu apare?** DAQ-E nu are LED de stare — verificați indicatorul PoE/link de pe comutatorul sau portul injectorului la care este conectat și așteptați câteva secunde după pornire pentru ca acesta să se inițializeze. Dispozitivul Chloros trebuie să se afle în același domeniu de difuzare (mDNS nu traversează routerele). Pe Windows, acceptați solicitarea firewall-ului Defender la prima legare a socket-urilor multicast de către Chloros (mDNS UDP 5353, date DAQ-E UDP 5002, PTP UDP 319/320). Două unități DAQ-E dintr-o rețea LAN sunt detectate separat, fiecare sub propriul nume de gazdă `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Tipul dispozitivului oferă DAQ-U (USB), DAQ-M (Bluetooth) și DAQ-E (Ethernet)</figcaption></figure>***

## Bara laterală a senzorilor

Fiecare senzor conectat primește un rând (plus un rând pentru fiecare grup „Ambient+Object”). Rândurile pot fi reordonate prin glisare, iar ordinea lor reordonează și casetele graficului. Faceți clic pe un rând pentru a face ca senzorul/grupul respectiv să devină graficul activ în vizualizarea listă.

| Element | Semnificație |
| --- | --- |
| Marginea stângă colorată | Culoarea graficului senzorului. |
| Insignă de transport | `DAQ-U` / `DAQ-M` / `DAQ-E`, sau o insignă verde `REF` pentru un grup de reflectanță Ambient+Obiect. |
| Numele dispozitivului | Implicit este numărul de serie al senzorului (identitatea sa stabilă pentru calibrare, numele fișierelor `.daq` și potrivirea la import); numele personalizate se păstrează pentru fiecare proiect. |
| Pilula **Calibrat** (verde) | Se afișează când este încărcat pachetul de calibrare din fabrică al senzorului, adică spectrele sunt exprimate în W/m²/nm. |
| Pilula **Actualizare disponibilă** (chihlimbar, numai DAQ-E) | Firmware-ul curent este mai vechi decât cel inclus în această versiune Chloros. În timpul unei actualizări, aceasta afișează progresul în timp real (`Flashing… N%`, `Restarting sensor…`, apoi `Updated X → Y` sau `Failed`). |
| Ochi | Comută vizibilitatea acestui senzor pe graficul său. |
| Roată dințată | Deschide fereastra modală cu setările pentru fiecare senzor (mai jos). |
| ✕ (roșu) | Deconectează senzorul sau elimină un grup „Ambient+Object”. |

Deasupra rândurilor se află două butoane:

* **Conectează senzorul** — deschide fereastra de dialog de conectare (se redenumește în `Connecting...` cât timp este ocupat).
* **Înregistrează tot / Oprește tot**— pornește sau oprește o înregistrare `.daq` pe**fiecare**senzor conectat. Necesită cel puțin un senzor**și un proiect deschis** (sfat: „Deschide un proiect pentru a înregistra”); devine roșu cât timp se derulează o înregistrare.

În stare goală, afișează „Niciun senzor conectat”.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Setări pentru fiecare senzor (fereastra cu rotița)

Se deschide cu pictograma rotiței dintr-un rând de senzori. Conținut în ordine:

* **Rânduri de informații** — Tipul dispozitivului (DAQ-U/M/E), Conexiune (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (port COM, adresă BLE sau gazdă) și serial.
* **Raport de calibrare: Descărcare** — preia certificatul de calibrare al acestei unități, trasabil conform NIST (PDF), și îl deschide în vizualizatorul dvs. de PDF. Disponibil odată ce numărul de serie este cunoscut; certificatul este stocat în cache la prima conectare.
* **Numele dispozitivului** — faceți clic pe creion pentru a redenumi; se păstrează pentru fiecare proiect.
* **Culoarea liniei graficului** — eșantion de culoare; se păstrează pentru fiecare proiect.
* **Timp de integrare (ms)**— glisor + număr,**1–500 ms**, implicit**32 ms**. Dezactivat când AE este activat.
* **Media cadrelor**— glisor + număr,**1–50 cadre**, implicit**20**.
* **AE: ACTIVAT/DEZACTIVAT**— comutator pentru expunere automată;**implicit ACTIVAT** la conectare. Dezactivați-l pentru a seta manual timpul de integrare.
* **Oprire streaming / Pornire streaming** — întrerupeți sau reluați transmisia live.
* **Înregistrare / Oprire înregistrare** — înregistrare `.daq` per senzor (necesită un proiect deschis).
* **Cap** — profilul de corecție a capacului (secțiunea următoare).
* **Rânduri cu informații în timp real** — Timp de integrare (ms), FPS, Eșantioane, Înregistrare (roșu `REC` sau `Off`) și Stare (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Numai pentru DAQ-E: rânduri de rețea, firmware și PTP

* **Nume gazdă / IP** — adresa curentă a unității.
* **Firmware** — versiunea actuală de firmware, plus o celulă de acțiune: un<version\>

buton</version\>

**Actualizare la \<version\>** apare atunci când această versiune Chloros include o imagine de firmware DAQ-E mai recentă. Actualizarea se descarcă prin rețea în aproximativ 30 de secunde; senzorul se repornește și se reconectează automat, iar o întrerupere a transferului lasă firmware-ul curent intact. Progresul este afișat în timp real (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), iar celula afișează `Up to date` atunci când este actualizată.
* **Sincronizare PTP** — starea PTP în timp real (revine la `unknown`). Firmware-ul DAQ-E v1.2.0+ participă la standardul IEEE 1588 PTPv2 ca ceas exclusiv de tip slave; backend-ul gazdei Chloros este grandmasterul PTP, iar fiecare cameră DAQ-E și LATTICE din rețeaua LAN îi este subordonată în domeniul 0, menținând marcajele de timp cu o abatere de aproximativ 1 ms.

Pentru un grup „Ambient+Object”, fereastra modală „Gear” afișează doar senzorii sursă ai grupului, numele dispozitivului și culoarea liniei graficului.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Selectarea capacului

Meniul derulant **Cap** indică Chloros ce capac fizic este montat peste difuzorul senzorului și aplică profilul de corecție măsurat din fabrică al acelui capac fiecărui spectru. Opțiunile depind de model:

| Model | Opțiuni de capac |
| --- | --- |
| DAQ-U | Niciunul (senzor neacoperit), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (corector cosinus) |
| DAQ-M | Niciunul (senzor neacoperit), Sunshine (corector cosinus) |
| DAQ-E | Niciunul (senzor neacoperit), FOV 15°, FOV 45°, FOV 90°, Sunshine (corector cosinus) |

**Setarea implicită pentru fiecare model este „Sunshine” (corector cosinus)** — MAPIR livrează fiecare DAQ cu capacul „Sunshine” instalat, aceasta fiind configurația standard pentru utilizare în exterior: o vedere emisferică de 180° cu eroare de cosinus ≤ ±4 % până la 60° și ≤ ±4,5 % până la 70° (nerecomandată sub o elevație solară de ~15°), cu atenuare prin design (~12×). Selecția dvs. se păstrează în proiect.

{% hint style="warning" %}
**Selecția capacului trebuie să corespundă cu capacul fizic.**Nici senzorul, nici software-ul nu pot detecta ce capac este montat. Selecția determină atât corecția în timp real, cât și ștampila înscrisă în fiecare fișier `.daq` — având în vedere atenuarea de ~12× a capacului Sunshine, o schimbare nedeclarată a capacului corectează eronat spectrele cu aproximativ acest factor. (Scoaterea și remontarea aceluiași capac determină o abatere de aproximativ 1,5 %.) Alegeți**Niciunul (senzor gol)** numai atunci când capacul este îndepărtat fizic; pe un DAQ-E, opțiunea „Niciunul” aplică în continuare un profil geometric din fabrică pentru difuzorul său de sticlă încastrat — nu este o operație inactivă — iar un DAQ-E gol reprezintă o configurație de banc de lucru, nu o configurație de teren acceptată.
{% endhint %}

{% hint style="info" %}
Actualizare față de un manual anterior: comutatorul „Sunshine Diffuser Installed” din browser, prezent în versiunea 1.1.0, a dispărut. Gestionarea capacelor se face acum prin acest profil de capac pentru fiecare senzor, aplicat pe partea de server.
{% endhint %}

***

## Zona graficului

O bară fixă în partea de sus conține un **comutator între vizualizarea listă ⇄ grilă**și un glisor**Zoom grafic** (dimensiunea casetei 200–2000 px). Vizualizarea comută automat la grilă atunci când există mai mult de un grup de grafice și revine la listă atunci când există unul sau mai puține. Modul de vizualizare și dimensiunea graficului se păstrează pentru fiecare proiect.**Graficul spectrului** pentru fiecare senzor afișează:

* **Axa X** — Lungimea de undă (nm). Grila senzorului este de 340–1010 nm la 5 nm (135 de puncte), interpolată la 1 nm pentru afișare.
* **Axa Y** — Putere (W/m²), cu un prefix SI automat (m/µ/n) ales pe baza valorii de vârf. Spectrele reprezintă iradianța spectrală (W/m²/nm) calibrată radiometric pe toate cele trei tipuri de transport.
* O umplere spectrală în culorile curcubeului sub o singură curbă; mai mulți senzori pe același grafic se suprapun sub formă de linii colorate cu umplere estompată.
* **Trecere cu mouse-ul**— un cursor vertical cu lungimea de undă și valoarea pe senzor;**trageți** pentru a mări (apare un buton de micșorare în timpul măririi).
* Un buton **+** (doar în vizualizarea cu grilă) pentru a adăuga un senzor la acest grafic sau pentru a crea un grup (mai jos).
* Numele dispozitivului centrat în partea de sus și un indicator de încărcare până la sosirea primului cadru.

**Saturația** nu este marcată pe grafic: un senzor saturat afișează textul de stare roșu `SATURATED` și un rând roșu `Saturated: Yes` în tabelul cu date în timp real. Reduceți timpul de integrare sau reactivați AE pentru a o șterge.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabelul cu date în timp real (vizualizare listă)

Sub grafic, în vizualizarea listă, reîmprospătat la fiecare 500 ms:

* **Toate modelele**: Eșantion de culoare a luminii (sRGB din CIE XYZ), Saturat (Da/Nu), CIE 1931 X/Y/Z, Cromaticitate x/y, CIE u′/v′, CCT (K), CRI (Ra), lungimea de undă dominantă (nm), lungimea de undă de vârf (nm), puritatea de excitație, Duv, CIE L\*/a\*/b\* și Munsell H/V/C.
* **Numai senzori calibrați**(oricare dintre DAQ-U / DAQ-M / DAQ-E odată ce pachetul de calibrare din fabrică este încărcat — insigna verde**Calibrat** din rândul senzorului indică acest lucru): Putere totală (W/m²), lux fotopic (lx), lux scotopic (lx), raportul S/P, PPFD și PPFD Red/Green/Blue (µmol/m²/s), și iradianțele opice — con S, melanopic, rodopic, con M, con L (toate în W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Grupuri de reflectanță (Ambianță + Obiect)

Doi senzori conectați pot fi combinați într-un afișaj de reflectanță în timp real — fără a fi necesară o cameră:

1. În vizualizarea tip grilă, faceți clic pe **+**pe o casetă a graficului și alegeți**Combinare ambientală + obiect**.
2. Alegeți un senzor **Sursă de lumină ambientală**și un senzor**Scaner de obiect**(doi senzori distincti), apoi**Creați**.

Chloros calculează R(λ) = obiect(λ) / ambiental(λ) pentru fiecare lungime de undă din cele două fluxuri în timp real (0 când ambiental ≤ 0). Eticheta grupului urmează clasa de calibrare a senzorilor:

* Ambii senzori calibrați (pachet încărcat) → **„Reflectanță aparentă”**.
* Unul dintre senzori necalibrat → **„Reflectanță relativă”**.

Grupul apare ca un rând verde `REF` în bara laterală și are propriul grafic (umplere în culorile curcubeului, valori afișate la trecerea cursorului cu 4 zecimale, zoom prin glisare).

Meniul **+**oferă, de asemenea, opțiunea**Adăugare senzor nou** cu trei opțiuni de plasare: *Combinare senzor nou* (adăugare la acest grafic), *Mutare senzor existent aici* sau *Vizualizare senzor nou* (grafic propriu).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabelul indicilor de vegetație

În vizualizarea listă, un tabel al indicilor de vegetație se află sub graficul unui grup de reflectanță, calculat pe baza reflectanței în timp real la centrele benzilor **albastru 450 / verde 550 / roșu 670 / NIR 800 nm** (valori cu 4 zecimale, `---` când nu se poate calcula; treceți cu mouse-ul peste numele unui indice pentru a vedea denumirea completă):

* **Afișate întotdeauna** (invariabile la scară, orice combinație de senzori): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Numai atunci când ambii senzori sunt calibrați la putere** (ambele pachete încărcate): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Înregistrarea fișierelor `.daq`

* Înregistrarea necesită un **proiect deschis** — în caz contrar, atât opțiunea „Înregistrează tot” (bara laterală), cât și butonul de înregistrare pentru fiecare senzor sunt dezactivate.
* Fișierele sunt salvate în **`<project folder>/light_sensor/`**; numele fișierelor conțin ID-ul senzorului și o marcă temporală, iar numele dispozitivului este stocat împreună cu înregistrarea.
* Când o înregistrare se oprește (Oprire, Oprire totală, sau o deconectare în timpul înregistrării), fișierul `.daq` finalizat este **adăugat automat la proiectul deschis** — acesta apare în lista de fișiere a proiectului fără a fi necesară adăugarea manuală, fiind gata să servească drept date de radiație descendentă pentru [procesarea reflectanței](README.md).
* Un indicator roșu `REC` apare în rândurile în timp real ale ferestrei modale de setări în timpul înregistrării.

Pentru valorile cantitative ale iradianței, calculați media pentru cel puțin 15 secunde de date — aceasta este o caracteristică a instrumentului, nu un defect.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Configurații cu senzori multipli și persistența proiectului

* Combinați mai mulți senzori pe un singur grafic (axe comune), păstrați grafice separate (configurare automată a grilei), mutați senzorii între grafice, reordonați rândurile/casetele prin glisare și ascundeți senzori individuali cu butonul sub formă de ochi.
* Pentru fiecare proiect, Chloros păstrează: numele dispozitivelor, culorile graficelor, dimensiunea graficelor, modul de vizualizare și setările fiecărui senzor (timp de integrare, medierea cadrelor, starea AE, selecția capacului).
* **Redeschiderea unui proiect reconectează automat senzorii acestuia** după adresă — port COM pentru DAQ-U, dispozitiv BLE pentru DAQ-M, numele de gazdă mDNS pentru DAQ-E (rezolvat chiar dacă IP-ul unității s-a schimbat) — și reaplică profilul de cap salvat al fiecărui senzor, medierea cadrelor, starea AE și timpul de integrare manual.***

## Asocierea camerei (DLS)

Nu este necesară nicio asociere. Spre deosebire de fluxurile de lucru DLS pentru drone, care asociază un senzor de lumină cu o cameră încă de la început, Chloros asociază datele DAQ cu imaginile în etapa ulterioară: la momentul importului/procesării, valorile `.daq` sunt interpolate la marcajul de timp al expunerii fiecărei capturi. Înregistrați cu orice senzor conectat (`.daq` este adăugat automat în proiect), iar procesarea reflectanței găsește citirile corecte în funcție de timp — consultați [Senzori de lumină DAQ](README.md) pentru a afla cum sunt utilizate datele de lumină descendentă.</version\>
