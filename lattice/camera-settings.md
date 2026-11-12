# Setări cameră

Fila **Camere** reprezintă panoul de control în timp real al Chloros pentru camerele LATTICE: o zonă principală de afișare care prezintă fiecare cameră conectată sub forma unui panou live și o bară laterală care comută între trei pagini — **lista camerelor**, un**panou de setări**(setări per cameră, pentru grup de camere sau de captură — câte una pe rând) și**Calculatorul de index**. Această pagină documentează fiecare element de control din lista de camere, panoul de setări per cameră și panoul de setări pentru grupul de camere. Modurile de captură, selecția tipului de export și fluxul „Capture All” se găsesc pe pagina asociată [Setări și moduri de captură](capture.md).

Fila „Camere” apare în bara laterală odată ce backend-ul Chloros este pregătit. Toate elementele de control de mai jos comunică cu backend-ul local prin `127.0.0.1:5000`; modificările se aplică imediat camerei live, cu excepția cazurilor în care se specifică altfel.

## Tipuri de camere utilizate pe această pagină

Elementele de control se afișează sau se ascund în funcție de tipul de cameră selectat. Manualul utilizează următorii termeni pe tot parcursul său:

| Termen | Înțeles | Canale de filtrare |
| --- | --- | --- |
| **Cameră RGB** | LATTICE M3C cu filtru FRGB (modelul conține `-FRGB`) | Red / Green / Blue |
| **Multispectrală Bayer** | LATTICE M3C cu FRGN, FOCN sau FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — un filtru de bandă îngustă, o bandă calibrată | Bandă unică |
| **Element al matricei** | O cameră conectată ca parte a unei matrice sincronizate (afișaj combinat sau separat) | Conform filtrului său |

Camerele RGB beneficiază de procesare fotometrică (balans de alb, profiluri de culoare, gamma); camerele multispectrale și mono beneficiază de lanțul radiometric și omit controalele fotometrice. Elementele matricei transmit setările la nivel de flux (format pixel, rezoluție, binning, declanșare, frecvența cadrelor) către matrice — acele rânduri devin doar pentru citire în panoul specific fiecărei camere și se mută în panoul de setări al matricei.

## Zona principală de afișare a fluxului

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Dacă nu sunt conectate camere, zona de afișare a fluxului prezintă un mesaj de bun venit **„Conectați o cameră pentru a începe”**cu două butoane:**Conectare cameră**(verde, deschide fereastra de dialog pentru conectarea unei singure camere) și**Conectare rețea** (albastru, deschide fereastra de dialog pentru conectarea rețelei). Ferestrele de dialog pentru conectare sunt documentate în [Conectarea camerelor](connecting.md); conceptele legate de rețea (sincronizare, niveluri, lățime de bandă) în [Matrice cu mai multe camere](arrays.md). Când deschideți un proiect salvat care conține camere, ecranul de întâmpinare afișează în schimb un indicator de încărcare cu mesajul „Se redeschid N camere salvate…”, în timp ce Chloros restabilește fluxurile din ultima sesiune.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Bara de sus

| Comandă | Ce face |
| --- | --- |
| **Comutator mod de vizualizare**| Comută între**vizualizarea în grilă**(toate casetele ca celule) și**vizualizarea în listă**(rânduri pe toată lățimea: fiecare matrice combinată ca un singur rând compozit, fiecare element al unei**matrice separate** ca rând propriu cu fluxul său și tabelul de ținte de calibrare, apoi O singură cameră independentă — cea activă — mai jos). Sfaturi: „Comută la vizualizarea în grilă” / „Comută la vizualizarea în listă”. |
| **Blocare grilă**(lacăt) | Implicit**blocată** — plăcile sunt fixate în poziție. Deblochează pentru a reordona plăcile prin glisare în orice slot (spațiile libere sunt păstrate). Grila se blochează automat de fiecare dată când se conectează o nouă cameră. Sfaturi: „Deblochează grila (activează glisarea casetelor)” / „Blochează grila (îngheață casetele în poziție)”. |
| Glisor **Zoom flux** | Dimensiunea casetei, de la 60 px până la lățimea completă a containerului. Celulele păstrează un raport de aspect de 4:3. Sub o lățime a celulei de 200 px, suprapunerile cu numele și fps-ul se ascund pentru a menține caseta curată. |

### Casete de flux

Fiecare cameră redă o casetă live compusă; o cameră poate afișa suplimentar trei casete **divizate pe canale** în tonuri de gri (vezi [Divizări pe canale](#display-overlays-drawn-over-the-live-feed)), iar matricile afișează o casetă combinată. Caseta activă are un inel de selecție în culoarea camerei (sau a matricei).

La trecerea cursorului peste o casetă apare un buton de închidere **X**:

* Închiderea unui **tile compozit** în timp ce diviziunile sale pe canale rămân afișate ascunde doar imaginea compozită.
* Închiderea **ultimului tile vizibil al unei camere independente** deconectează camera respectivă.
* **Tile-urile divizate ale membrilor unei matrice combinate nu deconectează niciodată** camera — ele doar o ascund.

Cu grila deblocată, trageți orice casetă în orice slot; dispunerea este salvată odată cu proiectul.

## Bara laterală — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

camerelor Prima pagină a barei laterale afișează toate camerele și matricea conectate:

* **Conectare cameră**(verde) /**Conectare matrice** (albastru, afișează „Detectare...” în timpul scanării). Ambele sunt dezactivate cât timp este deschis un dialog de conectare.
* **Captură totală** (roșu) — capturează toate camerele listate cu tipurile de export alese în Setările de captură. Necesită un proiect deschis. Documentat complet în [Setări și moduri de captură](capture.md).
* **Roata dințată Setări de captură** (lângă Captură totală) — deschide [panoul Setări de captură](capture.md#the-capture-settings-pane). Dezactivat în absența unui proiect sau în timpul capturării.

### Rânduri de camere

Fiecare rând de camere afișează o margine codificată prin culori (culoarea personalizată a camerei), o etichetă „CAM” — cu litera albastră **M**(master) sau**S** verde (slave) pentru membrii matricei — și numele afișat. Numele implicit este `LATTICE-MODEL (serial)`; îl puteți redenumi din panoul de setări al fiecărei camere. Butoane de rând:

| Buton | Efect |
| --- | --- |
| **Ochi**| Comută vizibilitatea. Camerele ascunse ies din grilă și sunt**excluse din „Captură totală”**. |
| **Roată** | Deschide panoul de setări pentru fiecare cameră (secțiunea următoare). |
| **Pauză / Redare**| Îngheață previzualizarea live**doar pe ecran** — capturarea în fundal continuă să ruleze. Camerele puse pe pauză nu pot captura. |
| **X** | Deconectează. Interfața se actualizează imediat (în cel mai bun caz); deconectarea propriu-zisă a backend-ului poate dura între 10 și 30 de secunde. |

### Rânduri ale matricei

Un rând al matricei afișează o insignă „ARRAY” în culoarea matricei, numele matricei (care poate fi redenumit în setările matricei) și un buton **DAQ · activat/dezactivat**—**activat** când senzorul de lumină la nivel de matrice este setat *sau* orice membru are un senzor specific fiecărei camere; infobulele sale indică exact ce senzor alimentează ce. Camerele membre sunt listate indentate dedesubt, cu propriile rânduri. Butoanele rândurilor matricei: **ochi**(ascunde/afișează TOATE elementele împreună),**roată dințată**(panoul de setări al matricei),**X**(deconectează întreaga matrice).

Starea senzorului de lumină (DLS) utilizată în rândurile matricei și în panoul de setări al matricei are patru stări:**oprit**,**în așteptare**(încă nu există spectru),**activ**(un spectru a sosit în ultimele 3 s) și**învechit** — nu există spectru nou în ultimele 3 s, dar ultima citire este *încă utilizată* (citirile DAQ nu expiră niciodată pe calea de captare).

Până când o reordonați, lista afișează mai întâi matricile, apoi camerele independente, cu camerele (și membrii fiecărei matrice) în **ordinea camerelor**: M3C (Bayer) înaintea M3M (mono); printre M3C, FRGB, FRGN, FOCN, apoi FNGB; M3M după lungimea de undă a filtrului, începând cu cea mai mică. Dispunerea implicită a grilei urmează aceeași ordine. Puteți trage camerele independente și grupurile întregi de matrice una peste alta în bara laterală pentru a reordona lista; elementele matricei nu pot fi trase independent.

## Panoul de setări pentru fiecare cameră

Se deschide cu **rotița** de pe un rând de camere. Panoul se glisează peste lista de camere.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Antet**:**mostra de culoare**a camerei (faceți clic pentru a deschide un selector de culori nativ — setează culoarea chenarului barei laterale și a inelului de selecție a plăcilor),**numele**cu un buton**Redenumire**sub formă de creion (dacă salvați un nume gol, se revine la numele implicit `MODEL (serial)`) și**×** pentru a închide.

### Histogramă în timp real

În partea de sus a panoului se află o histogramă de luminanță în timp real, calculată pe baza previzualizării JPEG la ~8 Hz. Media este ponderată după metoda Bayer — (R+2G+B)/4 — pentru a se potrivi cu propria măsurare AE a camerei.

* **Linia punctată Orange**= ținta AE. **Trageți-o pe orizontală pentru a reorienta ținta** — la eliberare se trimite o comandă, iar glisarea comută modul țintei AE pe Manual.
* **Linia continuă Green** = luminozitatea medie actuală (ceea ce oferă AE în prezent).
* **Butonul RGB** (dreapta sus): comută între histogramele suprapuse pe benzi, colorate în funcție de filtrul camerei (de exemplu, pe FRGN: NIR gri, verde, roșu). La camerele mono (M3M), butonul afișează „MONO” și este dezactivat — modul mono afișează întotdeauna histograma de luminanță pe o singură bandă.
* Etichetele axei X respectă adâncimea de biți a senzorului pentru formatul curent al pixelilor: 0..255, 0..1023, 0..4095 sau 0..65535.

### Rânduri

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

cu informații despre cameră

| Rând | Comportament |
| --- | --- |
| **Model** | Doar citire (de ex. `LATT-M3C-L87-FRGN`). |
| **Calibrare radiometrică**| Insignă**„Activă”** de culoare Green, cu o legendă care afișează nivelul de calibrare, hash-ul, data calibrării și lista de benzi, încărcate din pachetul de calibrare al camerei (a se vedea [Calibrarea radiometrică din fabrică](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)). **Ascuns pentru camerele RGB** — acestea dispun de o calibrare fotometrică a balansului de alb, nu de radianță pe bandă. |
| **Raport de calibrare**| Butonul**Descărcare** — deschide în vizualizatorul sistemului de operare fișierul PDF al certificatului de calibrare NIST specific fiecărui număr de serie al camerei. Dacă certificatul nu este încă stocat în cache, Chloros afișează în schimb un mesaj de îndrumare. |
| **Număr de serie** | Doar pentru citire. |
| **Firmware**| Afișează versiunea curentă, apoi identifică versiunea disponibilă pentru acest model (memorată în cache pentru fiecare model — un grup de N camere verifică serverul o singură dată). Stări: „Verificare…” → butonul**„Actualizare la X”**→ „Actualizare în curs…” → „Actualizat de la A la B” / „Eșec: …” / „Omitut: …” / verde**„La zi”**. Informația de ajutor a butonului de actualizare: „Resetare la setările din fabrică + flash + reprogramare UserSet1. ~2–3 minute; nu deconectați.” |
| **IP** | Doar pentru citire. |
| **Temperatură** | Doar pentru citire, actualizată la fiecare 3 s. Devine portocaliu la ≥65 °C și roșu cu un ⚠ la ≥75 °C. |
| **Țintă de calibrare** casetă de selectare | Activează detectarea țintei de reflectanță ArUco cu un tabel de validare NDVI pentru fiecare panou sub fluxul live (vizualizare listă) — vezi [Tabelul țintei de calibrare](#the-calibration-target-table). Disponibilă doar pe durata sesiunii — se deschide întotdeauna dezactivată. |
| Meniul derulant **Senzor de lumină** | Asociază un senzor de lumină DAQ (DAQ-E/M/U, din lista din fila Senzori de lumină) la această cameră pentru corecția iluminării cu lumină descendentă (DLS) și expunere automată predictivă. „„Niciunul” anulează asocierea. Dacă nu sunt conectați senzori, meniul derulant afișează „(nu sunt senzori conectați — deschideți fila DAQ)”. Asocierea este salvată odată cu proiectul. |

#### Tabelul țintelor de calibrare

Tabelul este **doar pentru afișarea datelor de validare** („doar validare — nu modifică calibrarea”). Antetul său indică numărul de serie al camerei, filtrul, numărul țintei și dimensiunea markerului în pixeli, iar fiecare panou are un rând:

* **Panou**, urmat de o coloană pentru fiecare bandă cu valoarea panoului — o cameră Bayer afișează cele trei benzi ale sale după nume și lungime de undă (de ex. FRGN: Red 660 nm / Green 550 nm / NIR 850 nm); o **cameră mono (M3M) afișează o singură bandă** (de exemplu, `F750`) și nu are coloane NDVI.
* **Sat** — procentul de pixeli ai panoului care sunt saturați (chihlimbar de la 1 %, roșu de la 5 %).
* **NDVI Meas**,**NDVI Theo** (NDVI de referință al panoului bazat pe reflectanță) și **Δ**— numai pentru camerele multispectrale Bayer. Un mic simbol după „NDVI Meas” indică sursa din care a fost calculat NDVI-ul măsurat:**ρ**(reflectanță — este disponibilă o citire a senzorului de lumină),**L**(radianță — nu există citire a senzorului de lumină) sau**DN**(valori brute ale camerei). Doar un**NDVI**bazat pe**ρ** este comparabil cu NDVI Theo: pentru L și DN, coloana Δ este estompată, iar o notă precizează acest lucru („Nu există citire a senzorului de lumină: NDVI-ul provine din radianță, deci Δ nu reprezintă o verificare a calibrării” / „NDVI-ul provine din valorile brute ale camerei, deci Δ nu reprezintă o verificare a calibrării”). În cazul valorii ρ, Δ este verde sub 0,02, galben sub 0,05 și roșu în celelalte cazuri.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Expunere și amplificare

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Toate valorile numerice introduse aici utilizează rotițe de reglare cu accelerare la menținere: atingere = ±1, menținere >1,5 s = ±10, menținere >3 s = ±100. Valoarea este trimisă către cameră atunci când eliberați butonul.

| Control | Interval / opțiuni | Implicit | Se aplică la | Ce face |
| --- | --- | --- | --- | --- |
| **Expunere (us)**| Valoarea minimă/maximă în timp real a camerei | Auto | Toate | Timpul de expunere în microsecunde, cu un comutator**Auto/Manual**. Auto = expunere automată continuă din partea camerei. Setarea unei expuneri manuale lasă**Câștigul** pe Auto, dacă acesta era setat astfel, permițând utilizarea întregii game de câștig a camerei, până când editați câștigul sau îl setați pe Manual. |
| **Câștig (dB)**| Valoarea minimă/maximă în timp real a camerei (de ex., până la 48 dB) | Manual (dezactivat) | Toate | Câștig analogic/digital cu propriul comutator**Auto/Manual**. |
| **Luminozitatea țintă AE**| 0–255 | 80, modul**Auto**| Toate (modificabil când AE sau câștigul automat sunt activate) | Luminozitatea vizată de AE. În**Auto**(implicit), un controler bazat pe histogramă alege singur valoarea țintă, menținând expunerea la 60–75 % din valoarea maximă a senzorului. Introducerea unei valori sau glisarea liniei portocalii a histogramei comută modul în**Manual**. |
| **Netezire AE** | 0,5–40, pas de 0,1 | 8,0 | Toate | Amortizarea AE. Informație rapidă: „Cu cât este mai mică, cu atât AE reacționează mai rapid (poate pulsa la fps ridicat). Cu cât este mai mare, cu atât este mai lină / mai lentă.” Valorile mult sub cea implicită pot face ca AE să pulseze și să destabilizeze transmisia la rate de cadre ridicate; 8,0 este valoarea implicită stabilă. |
| **Regiunea de interes AE**| Casetă de selectare „Activat” + buton**Țintire**| Dezactivat | Toate | Când este activată, AE măsoară doar regiunea marcată cu linie verde punctată, în loc de întregul cadru.**„Aim”** activează funcția „click-to-place” pe fluxul live: un clic centrează o regiune la 30 % din cadru; clic-glisare trasează un dreptunghi personalizat (minimum 5 % × 5 %). Funcția „Aim” se dezactivează automat după o singură plasare. Regiunea este mapată înapoi la coordonatele native ale camerei, ținând cont de orice rotație/oglindire pe care ați setat-o, și este salvată odată cu proiectul. |
| **Viteza de reglare AE** | 0,1–5, pas de 0,1 | 1,0 | Doar pentru membrii matricei | Cât de repede ținta AE automată urmărește schimbările de luminozitate ale scenei; la 1,0× se verifică din nou la fiecare 2,5 s. |
| **Protecția zonelor luminoase** | Strictă (1 %) / Normală (5 %) / Relaxată (15 %) | Strictă | Camere care expun această setare | Cât din cadru se poate decupa în alb înainte ca AE să întunece imaginea. |

{% hint style="info" %}
**Cerințe de lumină pentru camerele multispectrale Bayer (RGN / OCN / NGB):** scena trebuie să aibă suficientă lumină în toate cele trei canale, altfel calibrarea nu funcționează corect — o singură expunere a senzorului acoperă toate cele trei spectre. Utilizați un senzor de lumină DAQ pentru a măsura lumina sau treceți în modul complet mono (M3M), astfel încât fiecare bandă să aibă propria expunere. Dacă o captură încalcă această cerință, Chloros o detectează și vă avertizează (notificarea „unmix-clamp”). În timp real, o casetă de separare a canalelor afișează **„Date de bandă cu încredere scăzută · N%”** atunci când funcția „unmix” a limitat acea bandă la 25 % sau mai mult din pixelii săi.
{% endhint %}

{% hint style="info" %}
**Modul în care expunerea automată gestionează expunerea și amplificarea (1.3.0).** Cu câștigul setat pe „Auto”, Chloros menține câștigul la 0 dB până când expunerea atinge cea mai mare valoare permisă (limita superioară a AE, perioada de declanșare a unei matrice sau plafonul de expunere DAQ); abia atunci crește câștigul. Într-o matrice, rata de declanșare are prioritate: expunerea automată umple perioada stabilită de această rată, apoi crește amplificarea, astfel încât o scenă care se întunecă nu mai reduce rata de declanșare a modului „Auto” (în versiunea 1.2.0 se întâmpla acest lucru). O expunere **manuală**pe un element al matricei, cu o durată mai mare decât perioada de declanșare, încetinește rata de declanșare a matricei pentru a se adapta la aceasta; rata automată revine odată ce fiecare element care a încetinit-o revine la expunerea automată. Cu**Expunerea**și**Luminozitatea țintă AE** setate ambele pe „Auto” și cu un senzor de lumină DAQ care furnizează o citire, expunerea fiecărei camere este menținută odată ce expunerea automată s-a stabilizat, atâta timp cât lumina măsurată rămâne în limitele de aproximativ ±25 %; o schimbare de lumină mai mare și susținută, o repornire a setărilor (de exemplu, o modificare a zonei de interes) sau propria dvs. modificare a expunerii o reactivează.
{% endhint %}

{% hint style="info" %}
**Cum funcționează expunerea automată (1.3.0).**

* O expunere manuală menține amplificarea pe „Auto” până când modificați amplificarea.
* Zonele ușor luminoase nu întunecă scena: o reflexie de până la 1 % din cadru este ignorată, iar în cazul panourilor unei ținte de calibrare cunoscute, până la 0,5 % dintr-un panou.
* Limita maximă de expunere DAQ se aplică numai cât timp o țintă de calibrare se află în câmpul vizual; dacă nu se vede niciun panou, camera expune pentru scenă.
* Expunerea rămâne stabilă la lumină constantă și se stabilizează în câteva secunde după o schimbare majoră. O suprasaturare nu mai limitează expunerea definitiv, iar o modificare a zonei de interes (ROI) restabilește expunerea.
* Ținta de luminozitate nu depășește raza de acțiune a camerei, astfel încât o lumină care revine nu provoacă reflexii în cadru.
* O cameră a cărei expunere automată nu mai răspunde în lumina puternică a soarelui este menținută la nivelul corespunzător luminii în care se află, citit de senzorul de lumină sau, în lipsa acestuia, de propriile cadre ale camerei, și este eliberată când lumina se schimbă. Un senzor de lumină întunecat sau acoperit nu perturbă acest proces.
{% endhint %}

### Formatul pixelilor și rezoluția

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Elementele matricei** afișează rândurile „Current” (format + LxH) și „Binning”, numai pentru citire, cu nota „Setat în setările matricei” — o repornire a fluxului pe un element ar întrerupe sincronizarea, așa că acestea sunt gestionate în [panoul de setări al matricei](#panoul-setări-matrice).**Camerele independente** dispun de:

| Control | Opțiuni | Ce face |
| --- | --- | --- |
| **Formatul pixelilor** | 8 biți / 10 biți / 12 biți / 16 biți | Adâncimea de biți a senzorului; familia de formate este cea a senzorului. Valoarea implicită este de 12 biți, iar un proiect salvează setările dvs., inclusiv 8 biți, și se reconectează la acestea. |
| **Rezoluție** | Completă / Jumătate / Sfert | Relativă la binning-ul curent: Complet = 2048/N × 1536/N pentru binning N×N. |
| **Binning** | 1x1 (niciunul) / 2x2 / 4x4 | Binning hardware N×N — valorile mai mari reduc rezoluția, dar îmbunătățesc raportul semnal-zgomot (SNR) și rata de cadre. Modificarea acestuia repornește fluxul și resetează orice ROI la noul câmp vizual complet. |
| **Actual** | numai pentru citire | Lățimea (W) și înălțimea (H) efective, precum și decalajul (x, y) aplicat. |

### Previzualizare live

Tot ce se află în această secțiune se referă **doar la afișare**— modifică ceea ce vedeți în fluxul live, în timp ce capturile salvate rămân liniare și nemodificate — cu o singură excepție:**Vigneta** este radiometrică și afectează și exporturile (prezentată mai jos).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Control | Interval / opțiuni | Implicit | Se aplică la | Ce face |
| --- | --- | --- | --- | --- |
| **Rezoluție de redare** | 360p (cea mai rapidă) / 480p / 720p / 1080p / Rezoluția nativă a senzorului (cea mai lentă) | 720p | Toate | Înălțimea la care backend-ul rulează lanțul de previzualizare radiometrică. O valoare mai mică asigură o rată de cadre mai mare fără a modifica câmpul vizual. |
| **Index**| Casetă de selectare „Activați” + rotiță | Dezactivat | Numai multispectral Bayer,**nu** pentru membrii cu matrice combinată | Previzualizare live a indicelui de vegetație. Roata dințată deschide fereastra partajată [Calculator index](#index-calculator-pane), preîncărcată cu benzile naturale ale filtrului camerei (de ex. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Expresia personalizată plus LUT (pornit/oprit, nivel implicit 7 stopuri, minim implicit 0,2, maxim implicit 1) este calculată pentru fiecare cadru de previzualizare. Membrii matricei combinate ascund acest rând — matricea deține un singur indice partajat. |
| **Balanța de alb** | Oprit / O singură dată / Continuu + un buton de recaptare | Continuu | Numai RGB | Balans de alb în timp real. Butonul de reîmprospătare recapturează balansul de alb din spectrul DLS curent (dezactivat când modul este Oprit). |
| **Gamma** | Activat / Oprit | Activat | Numai RGB | Afișează gamma (γ = 2,2 LUT) în previzualizarea live. Capturile salvate rămân liniare. |
| **Reducere zgomot** | Casetă de selectare + intensitate 0–100 | Dezactivat / 50 | Toate (per cameră, chiar și în cadrul matricilor) | Filtru bilateral în previzualizarea în timp real. Valoare mai mare = detalii mai fine, dar mai estompate. |
| **Claritate** | Casetă de selectare + intensitate 0–100 | Dezactivat / 30 | Toate | Mască de neclaritate în previzualizarea în timp real, aplicată ultima. Poate amplifica zgomotul. Doar în previzualizare. |
| **Vigneta**| Casetă de selectare + intensitate 0–100 | Dezactivat / 0 | Toate | Eliminare manuală a vignetei reziduale (luminează colțurile), suprapusă peste estimarea Smart Vignette a matricei.**Radiometric — afectează vizualizarea în timp real ȘI exporturile**, spre deosebire de Denoise/Sharpness. |
| **Profil de culoare** | Raw / Liniar / Natural / Îmbunătățit / Temperatură personalizată | Natural | Numai RGB | Vezi mai jos. |
| **Temperatura culorii** | 2000–10000 K, pas de 100 | 5500 K | Numai RGB, profil de temperatură personalizat | Fixează balansul de alb la o temperatură de culoare corelată fixă (intrarea DLS este ignorată). Ultima valoare Kelvin selectată este reținută la schimbarea profilurilor. |
| **Zonele luminoase** | de la −100 la 0, cu pas de 1 | 0 (dezactivat) | Numai RGB | Noutate în versiunea 1.3.0. Întunecă zonele cele mai luminoase și mai albe ale imaginii (0 = neschimbat, −100 = intensitate maximă). Se aplică previzualizării în timp real **și capturilor la nivel de afișaj (Previzualizare)**, care înregistrează setarea (`Chloros:DisplayHighlights`); exporturile de radianță și reflectanță nu sunt niciodată afectate. Nu poate recupera datele senzorului care au fost decupate — reduceți expunerea pentru asta. Suprapunerea zebră urmează imaginea întunecată. Dezactivată cu profilul de culoare Raw („Nu se aplică cu profilul de culoare Raw.”). |
| **Saturație** | 0–200 (100 = neutru) | 100 | Numai RGB | Saturația HSV în previzualizarea live. |
| **Contrast** | 0–200 (100 = neutru) | 100 | Numai RGB | Contrast liniar în jurul tonului de gri mediu în previzualizarea live. |
| **Oglindire orizontală / Oglindire verticală** | Casete de selectare | Dezactivat | Toate | Răsuci previzualizarea pe orizontală / pe verticală. |
| **Rotație**| 0° / 90° / 180° / 270° | 0° | Toate | Rotește previzualizarea. Orientarea se aplică la sfârșitul lanțului de previzualizare din backend.**Capturi salvate (1.3.0):**produsele de tip Previzualizare, Radianță, Reflectanță și Indice ale unei camere independente sunt rotite prin Oglinzire/Rotație, în timp ce**fișierele Raw și Debayered rămân în orientarea nativă a camerei**; fiecare fișier de captură înregistrează setarea în XMP-ul său, astfel încât procesarea ulterioară a unui fișier raw o aplică în același mod.**Fișierele membre ale matricei rămân native** (setarea este înregistrată, nu aplicată), iar vizualizările compozite ale matricei o ignoră. |

**Semantica profilurilor de culoare** (camere RGB):

* **Raw** — ocolește complet lanțul de procesare.
* **Liniar** — semnal întunecat + câmp plat + balans de alb; fără matrice de culoare, fără gamma.
* **Natural** *(implicit)* — liniar plus matricea de corecție a culorii măsurată și o curbă de ton adaptabilă scenei.
* **Îmbunătățit**— Natural plus vibranță și contrast local CLAHE. Costul suplimentar se aplică**numai previzualizării live** — capturile salvate beneficiază întotdeauna de finisajul complet, indiferent de profil.
* **Temperatură personalizată** — Natural cu balansul de alb fixat la valoarea în Kelvin aleasă de dvs.

{% hint style="warning" %}
Pentru modurile Natural, Îmbunătățit și Temperatură personalizată, panoul afișează o notă privind tonul: cadrele sunt luminate în funcție de propria scenă, astfel încât imaginile *de afișare* salvate nu sunt comparabile între ele. **Exportați radianța sau reflectanța pentru măsurători.**
{% endhint %}

### Suprapuneri de afișare (desenate peste fluxul live)

Acestea sunt disponibile doar în interfața de utilizator — sunt suprapuse peste videoclip, fără a afecta niciodată fluxul sau capturile.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Suprapunere | Comenzi | Implicit | Ce face |
| --- | --- | --- | --- |
| **Zebra** | Casetă de selectare + prag 200–255 | Dezactivat / 250 | Dungi diagonale magenta pe pixelii decupați. |
| **Reticul** | Casetă de selectare | Dezactivat | Marcaj în centrul cadrului. |
| **Grilă** | Dezactivat / 3 × 3 / 9 × 9 | Dezactivat | Grilă de compoziție. |
| **Histogramă** | Casetă de selectare + lățime 0,10–0,90 din cadru | Dezactivat / 0,25 | O bandă de histogramă afișată pe flux. |
| **Focus Peak** | Casetă de selectare + prag 20–200 + mostră de culoare | Dezactivat / 80 / `#ff5722` | Evidențierea marginilor Sobel pentru focalizare. |
| **Împărțiri pe canale** | „Afișează împărțiri (Red / Green / NIR)” / butonul „Ascunde divizările” | Ascuns | Adaugă trei casete independente în tonuri de gri pentru fiecare canal, alături de imaginea compusă (eticheta butonului respectă canalele de filtru ale camerei). Fiecare casetă divizată poate fi trasă și are aceeași culoare ca marginea camerei. Nu este disponibil pe camerele monocrome. Se salvează odată cu proiectul. |

### Spot Meter

* Caseta de selectare **Click to Sample**: faceți clic pe fluxul live pentru a preleva o probă dintr-un singur pixel (acesta este marcat cu un reticul în formă de cruce) sau faceți clic și trageți pentru a selecta o regiune din care se calculează media pixelilor. Butonul**Clear**șterge proba și reticulul. Se exclude reciproc cu modul**Aim** al AE-ROI.
* **Meniul derulant**Afișare**:**Raw (adâncime de biți)**— valori digitale native la adâncimea de biți a senzorului (de ex. 12 biți → 0..4095) — sau**Afișare (8 biți)** (implicit). Când un indice în timp real este activ, opțiunea „Afișare” arată în schimb valoarea indicelui calculat (de ex. NDVI).
* Panoul de afișare prezintă coordonatele pixelilor, dimensiunea cadrului, formatul pixelilor, adâncimea de biți și un tabel de canale (Chan / Valoare / %) cu etichetele benzilor și lungimile de undă; perechile de verde Bayer sunt mediate; eșantioanele din regiune afișează „N px avg”.

Starea expometrului punctual este valabilă doar pentru sesiunea curentă.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Expunere automată predictivă (bazată pe DLS)

Această secțiune apare numai atunci când **este conectat cel puțin un senzor de lumină DAQ** — solverul are nevoie de un spectru descendent în timp real pentru a funcționa.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Control | Interval | Implicit | Ce face |
| --- | --- | --- | --- |
| **Activare** | Casetă de selectare | Activat (camere autonome) | Un solver de formă închisă utilizează spectrul DLS împreună cu scalarii pachetului de calibrare al camerei pentru a aduce banda cea mai luminoasă aproape de saturație, menținând în același timp banda cea mai întunecată peste pragul SNR — o singură expunere pe rezolvare, fără buclă de stabilizare. Conceput pentru timelapse alimentat cu energie solară, unde fiecare cadru trebuie expus corect. Sistemul revine automat la AE reactiv ori de câte ori citirea DLS este învechită/lipsă sau pachetul de calibrare nu este încărcat. |
| **Netezire (α)** | 0,05–1,0, pas de 0,05 | 0,3 | Netezirea soluțiilor predictive succesive (cu cât este mai mică, cu atât netezirea este mai mare). |
| **Reflectanța scenei**| Butonul**Recalibrare ρ** | — | Reestimează factorul de reflectanță al scenei utilizat de solver. |

{% hint style="info" %}
**Conectarea matricei dezactivează AE predictivă în mod implicit** — pentru matrice, AE inteligentă de la Chloros împreună cu expunerea automată a camerei gestionează expunerea (cu protecție împotriva saturației), iar estimarea reflectanței pentru o singură scenănu este sigură în scenele mixte. Puteți să o reactivați pentru fiecare cameră în parte aici, dacă doriți în mod specific o expunere radiometrică bazată pe DLS.
{% endhint %}

**Plafonul de expunere bazat pe DAQ și AE fixat pe incidență.** Indiferent de caseta de selectare de mai sus, atunci când un senzor de lumină DAQ este alocat unei camere RGB, Chloros calculează — pe baza iradianței absolute descendente măsurate — expunerea maximă × amplificarea la care o suprafață cu reflectanță de 100 % rămâne sub pragul de clipping și o aplică ca **plafon**pentru expunerea automată. Începând cu versiunea 1.3.0, plafonul se aplică numai atât timp cât o țintă de calibrare se află în câmpul vizual al camerei respective (văzută în cadrul acesteia în ultimele 10 secunde); în absența unei ținte în câmpul vizual, se activează în schimb expunerea automată obișnuită a camerei, cu propria protecție împotriva supraexpunerii, astfel încât restul scenei să nu fie menținut mai întunecat decât este necesar. În timp ce limita maximă este activă, camera funcționează**în modul fixat pe lumină incidentă**: funcționează în buclă deschisă la expunerea măsurată în funcție de lumina incidentă, cu amplificarea la 0 dB — expunerea urmează lumina măsurată, nu conținutul scenei. Deoarece limita maximă poate doar să scurteze expunerea, ea nu poate provoca singură decuparea. Începând cu versiunea 1.3.0, pragul maxim este calculat pe baza constantei de expunere proprii a camerei, preluată din pachetul său de calibrare (o valoare medie la nivel de flotă doar atunci când pachetul nu conține una). Pragul maxim se dezactivează automat — iar expunerea automată normală a scenei se reia — ori de câte ori nu se află nicio țintă de calibrare în cadru, citirea DAQ lipsește, este expirată (>30 s), sau este întunecată, sau dacă ≥15 % din cadru se decupează la expunerea fixată (ceea ce înseamnă că senzorul și camera detectează iluminări diferite). Nu există un comutator în interfața grafică; acesta este comportamentul standard ori de câte ori o cameră RGB are o legătură DAQ.

### Achiziție și declanșare

Elementele<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

matricei afișează în plus rândurile **Rol**(doar pentru citire) (Master în albastru / Slave în verde),**Linie de sincronizare**și**Echivalenți**.

| Control | Opțiuni | Implicit | Note |
| --- | --- | --- | --- |
| **Mod declanșare** | Oprit / Pornit | Pornit | Dezactivat pentru elementele matricei (matricea gestionează declanșarea). |
| **Sursă de declanșare** | Software / Linia 0 (M8) / Linia 1 / Linia 2 | Linia 0 | Ascunsă când Modul de declanșare este Dezactivat; dezactivată pentru elementele din matrice. Linia 0 este intrarea de declanșare externă optoizolată M8. |
| **Rata de cadre**| Auto / Manual + valoare | Auto |**Auto**: limita de frecvență a cadrelor camerei este dezactivată — expunerea dictează fps, iar caseta afișează frecvența reală în timp real.**Manual**: limitați numărul de cadre pe secundă (fps) cu ajutorul unui glisor (de la 1 până la maximul limitat de lățimea de bandă), pornind de la rata reală curentă. Membrii matricei văd o valoare de tip „N fps (live)”, doar pentru citire, cu mențiunea „Setat în setările matricei”. |

### Rețea / Transport

| Rând | Comportament |
| --- | --- |
| **Dimensiunea pachetului**| 1500 (Standard) / 9000 (Jumbo) — implicit**Jumbo**. |
| **Debit** | Limită de debit al legăturii, numai pentru citire, în MB/s. Backend-ul reechilibrează această valoare între toate camerele conectate la fiecare conectare/deconectare. |
| **Gestionarea bufferului** | Mod de gestionare a bufferului, numai pentru citire. |

### Captură

Panoul se încheie cu un buton **„Deschide setările de captură…”** care redirecționează către [panoul Setări de captură](capture.md#the-capture-settings-pane) (dezactivat până la deschiderea unui proiect — „Creați sau deschideți un proiect pentru a salva capturile”). Dacă camera este ascunsă sau pusă pe pauză, un mesaj vă reamintește să o afișați/reluați înainte de captură.

## Panoul de setări pentru matrice

Se deschide cu ajutorul **rotiței**dintr-un rând**Matrice**. Antet: numele matricei cu un creion de redenumire și**×** pentru închidere. Secțiunile de mai jos marcate cu *numai combinate* apar doar pentru matricele conectate în modul de afișare combinat.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sincronizare

Rânduri **Master**,**Slaves**și**Linie de sincronizare** numai pentru citire.

### Senzor de lumină ambientală

Afișat atât pentru matrice combinate, cât și pentru cele separate:

* Meniul derulant **Senzor de lumină**— leagă un DAQ de întreaga matrice. Selecția se aplică imediat, se propagă către meniul derulant**Senzor de lumină** al fiecărei camere din matrice (puteți totuși să suprascrieți setările pentru fiecare cameră în parte) și începe redirecționarea spectrelor către matrice.
* Linia **Stare** în timp real: Oprit · „Se așteaptă primul spectru…” · „Activ — toate camerele din matrice au iluminarea corectată” · „Niciun spectru nou în ultimele 3 s — se utilizează în continuare ultima citire (fără expirare a timpului de așteptare)…”.
* Notă în panou: „Corecție radiometrică la nivel de matrice. Setările individuale ale camerelor au prioritate.”

### Țintă de calibrare

Propriul său grup, imediat după Senzorul de lumină ambientală, atât pentru matricele combinate, cât și pentru cele separate:

* Caseta de selectare **Activare** — „Detectează ținta MAPIR ArUco și validează LUT-ul NDVI față de reflectanța panoului”; generează suprapunerea țintei și tabelele de validare ale plăcii combinate. Pe o matrice **separată**, activează [tabelul țintei de calibrare] propriu fiecărui element(#the-calibration-target-table) (câte una pentru fiecare rând de camere în vizualizarea listă).

### Captură — setări uniforme ale senzorilor

Afișat atât pentru matrice combinate, cât și pentru cele separate (antetul grupului este „Captură (uniformă pentru toate camerele)”). Aceste setări se aplică uniform fiecărui element (modificările individuale ar perturba sincronizarea). Modificările sunt stocate temporar și aplicate împreună cu **Aplicare**— cu excepția comutatorului**ROI activat / ROI dezactivat**, care se aplică imediat.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Control | Opțiuni / interval | Ce face |
| --- | --- | --- |
| **Format pixel** | 8 biți / 10 biți / 12 biți / 16 biți | Adâncime de biți uniformă pentru toți membrii; fiecare cameră adoptă formatul corespunzător acelei adâncimi pentru propriul senzor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning hardware — păstrează câmpul vizual complet, îmbunătățind în același timp raportul semnal-zgomot (SNR) și rata de cadre. Modificarea acestuia resetează câmpurile ROI la noul câmp vizual complet. |
| Presetare **rezoluție** | Completă / Jumătate / Sfert | Relativă la binning; completează câmpurile ROI cu o decupare centrată. |
| **Decupare ROI (px)**| Comutator**ROI activat / dezactivat**, butonul**Selectare**, câmpurile L / Î / X / Y, harta senzorului complet | Decupare senzor — consultați [Editorul ROI](#the-roi-editor). Lățimea/înălțimea se aliniază la multipli de 16 (minim 64); decalajele se aliniază la multipli de 4. Un indiciu „max LxH” arată limita maximă, iar**Resetare** readuce câmpurile la câmpul vizual complet. În timpul editării, caseta portocalie de decupare urmărește fiecare modificare a câmpului pe hartă și pe segmentul matricei. |
| **Rata de declanșare**| Comutator Auto / Manual + fps 0,5–10, pas 0,5 |**Auto**(implicit): backend-ul calculează rata de declanșare pe baza rezoluției și a lățimii de bandă — câmpul de introducere este dezactivat și afișează valoarea calculată.**Manual**: fixează valoarea la apăsarea butonului „Aplicare”. O rată fixată pe care o nouă dimensiune a cadrului nu o poate suporta este menținută la cea mai rapidă rată permisă de acea dimensiune și revine la valoarea inițială când dimensiunea cadrului o permite din nou. |

Notă în panou: „Modificările de format/rezoluție repornesc toate camerele pentru scurt timp. Rata de declanșare se aplică în timp real.” Butoanele **Aplicare / Anulare** se află în partea de jos a panoului.

#### Editorul ROI

Noutate în versiunea 1.3.0. Rândul de decupare ROI și harta de sub acesta funcționează împreună:

* **ROI activat / ROI dezactivat**— activează sau dezactivează decuparea senzorului**imediat**(camerele se repornesc pentru scurt timp; caseta matricei afișează „Se aplică setările…”).**Oprit**citește întregul senzor și**memorează decupajul dvs.**— câmpurile continuă să îl afișeze;**Pornit** îl aplică din nou. Activarea ROI fără un decupaj setat aplică un decupaj centrat la jumătate de dimensiune („Nu a fost setat niciun decupaj, așa că s-a aplicat un decupaj centrat la jumătate de dimensiune. Reglați-l folosind câmpurile.”).
* **Selectare**— desenează decupajul pe hartă: trageți de-a lungul acestuia sau faceți clic pe două colțuri opuse (tasta Esc anulează). Dreptunghiul se înregistrează în câmpurile L/H/X/Y, aliniat conform pașilor de mai sus; nimic nu se aplică până nu apăsați**Aplicare**, astfel încât să îl puteți regla mai întâi cu ajutorul săgeților din câmpuri. Un dreptunghi la fiecare apăsare a butonului Select.
* **Harta**afișează decupajul pe întregul senzor, astfel încât îl puteți lărgi din nou: „Senzor complet, în timp real” în timp ce camerele citesc senzorul complet, sau „Instantaneu full-frame de la {ora}. Camerele citesc doar decupajul.” în timp ce sunt decupate. Dacă matricea a transmis întotdeauna doar imagini decupate, harta afișează „Încă nu există vizualizare full-frame: această matrice a transmis doar imagini decupate.” cu un buton**Afișare cadru complet**, care dezactivează ROI-ul. Eticheta decupajului `W×H @ X,Y` se află în afara casetei (deasupra acesteia sau dedesubt, când caseta atinge marginea superioară), atât pe hartă, cât și pe caseta live.
* **Aplicare**validează câmpurile: o decupare activează ROI-ul; cadrul complet (după**Resetare**) îl lasă dezactivat, păstrând în memorie decuparea anterioară; un ROI care este dezactivat și ale cărui câmpuri afișează încă decuparea memorată rămâne dezactivat atunci când aplicați o altă modificare (format, binning, rată).
* **Pe o matrice aliniată, fereastra de decupare a fiecărui element este poziționată în funcție de aliniere**, astfel încât fiecare cameră să încadreze aceeași porțiune a scenei. O matrice decupată înainte de a fi aliniată este aliniată mai întâi la senzorul complet (decuparea este eliminată, alinierea este rezolvată, iar decuparea este restabilită cu ferestre specifice fiecărei camere).
* După o modificare a decupajului, expunerea automată se reconfigurează pentru noul cadru. În cadrul unui decupaj, expunerea unui element poate continua până la sfârșitul perioadei de declanșare.
* Starea de activare/dezactivare a ROI și decupajul memorat sunt salvate odată cu proiectul.

### Aliniere (co-înregistrare)

Afișat atât pentru **matrice combinate, cât și pentru cele separate** (versiunea 1.2.0 îl afișa doar pentru matricele combinate); antetul grupului este „Aliniere”.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Caseta **Rezidual**: „RMS x.xx px” — verde sub 1 px, galben sub 3 px, roșu în celelalte cazuri sau dacă vreo cameră a eșuat; „fără profil” înainte de prima rezolvare.
* Linia de rezumat: „✓ Toate camerele aliniate (N)” / „⚠ p/N camere aliniate —  <serial (filter)="">eșec” / „Decupare activă — Recalibrați pentru aliniere (utilizează senzorul complet)” / „Se așteaptă stabilizarea expunerii…” / „Se aplică setările…” (camerele se repornesc) / „Matricea a încetat să transmită cadre — reconectați matricea”.
* Tabel per cameră: cameră (ultimele 4 cifre ale numărului de serie + filtru), eroare de reproiectare în px cu numărul de potriviri („ref” pentru camera principală) și scorul de corelație încrucișată normalizat al suprapunerii în raport cu pragul minim de trecere de 0,35.
* **Recalibrare aliniere** buton (afișează „Calibrează alinierea” înainte de primul profil) — reexecută co-înregistrarea pe cadre noi.
* **Caseta de selectare „Expunere automată a camerelor pentru aliniere”** (bifată implicit) — luminează temporar camerele întunecate sau fără contrast (mai întâi expunerea, apoi amplificarea), astfel încât acestea să aibă textură care să se potrivească, apoi restabilește expunerea automată.

Chloros aliniază automat fiecare matrice după conectare, în ambele moduri de afișare, odată ce expunerea automată a elementelor s-a stabilizat (continuă să încerce până la 5 minute); previzualizarea combinată se aliniază, de asemenea, la deschidere. O matrice separată nu are o imagine compusă, așa că această secțiune verifică starea alinierii și execută ea însăși funcția „Recalibrare”. Recalibrați dacă s-a modificat focalizarea sau adâncimea scenei. Alinierea este **destinată exclusiv sesiunii respective** — nu este niciodată salvată într-un profil, deoarece depinde de distanța scenei din acel moment. Capturile pot fi totuși exportate cu înregistrare la nivel de pixel (vezi [Exporturi aliniate](capture.md#per-array-controls)).

### Vigneta inteligentă

* Caseta de selectare **Activare corecție**— aplică estimarea vignetei pentru fiecare cameră în parte la lanțul radiometric (în timp real**și** la exporturi).
* **Calibrare din vizualizarea curentă** — îndreptați mai întâi matricea către o țintă uniformă (ecran plat, perete sau cer); fiecare cameră este aplatizată individual, iar starea raportează un câștig de aplatizare de „n/N camere · −x,x %”.**Ștergere** elimină estimarea.
* Stare: „Calibrat”, „Necalibrat” sau „Calibrat parțial — rulați din nou” atunci când doar estimările unor camere se aplică încă la încadrarea lor curentă (de exemplu, după ce o operațiune de recalibrare a mutat fereastra de decupare a unei camere) — rulați din nou „Calibrare”.
* Reglați fin pentru fiecare cameră cu glisorul **Vignette** specific fiecărei camere din [Previzualizare live](#live-preview).

### Previzualizare live

* Caseta de selectare **Histogramă**+ glisorul de lățime (0,10–0,90 din cadru) — banda de histogramă afișată în flux pe fiecare dală a fiecărui membru simultan (inclusiv dalele cu canale separate), [Suprapunere histogramă](#display-overlays-drawn-over-the-live-feed) setată pentru întreaga matrice. Se afișează ca fiind activată atunci când fiecare membru o are deja. Pe o matrice**separată**, acesta este singurul rând din grup.
* *Numai în regim combinat:* **Index**: caseta de selectare „Activați” + rotița de setări — deschide [Calculatorul de index](#index-calculator-pane) partajat, cu benzi desenate din**toate** camerele membre. O linie de previzualizare a expresiei de dedesubt afișează expresia curentă („Nicio expresie setată — deschideți calculatorul pentru a crea una”), actualizată la fiecare secundă.
* *Numai în modul combinat:* Meniul derulant **Rezoluție de randare**(aceleași presetări ca pentru fiecare cameră, implicit 720p): înălțimea fluxului de vizualizare live**și** dimensiunea de export a compoziției salvate. Notă în panou: „Previzualizare + dimensiunea compoziției salvate. Imaginile de la fiecare cameră se exportă întotdeauna la rezoluție maximă.”

### Straturi de afișare *(numai în modul combinat)** Caseta de selectare **Activare** (implicit dezactivată — camera principală este afișată direct; activată = compoziție stratificată).
* Meniurile derulante **Meniurile derulante**Prim-plan**/**Fundal**: fiecare cameră membră (după nume) sau**Index**. Când Prim-planul este setat pe Index, pixelii aflați în afara intervalului minim/maxim al LUT afișează stratul de fundal.

### Vizualizare divizată *(numai combinată)*

**„Afișează camerele membre”**— un buton**Împărțire / Ascundere camere membre** care adaugă fluxul live al fiecărei camere membre sub formă de casete separate în grilă, lângă imaginea compusă. Casetele citesc bufferul de cadre existent al matricei (fără conexiune suplimentară la cameră). Doar în vizualizarea grilă; se salvează pentru fiecare matrice odată cu proiectul.

### Capabilități

Un panou numai pentru citire, reîmprospătat la fiecare 5 s:

* **Etichetă nivel**: „Captură simultană” (verde) · „Captură simultană (emisie eșalonată FTD)” (verde) · „Captură eșalonată (derivă de 100 ms)” (galben) · „Configurație prea mare” (roșu).
* **Starea cadrelor**: „x,xx % incomplete” — verde sub 1 %, galben sub 5 %, roșu la 5 % sau mai mult.
* **Linia de legătură**: „NIC {mbps} Mbps – susținut {MB/s} MB/s”.

Acesta este bugetul de lățime de bandă în timp real al matricei. Pentru detalii privind fps-ul de bază și modelul de rețea — precum și ce trebuie modificat când nivelul devine galben sau roșu — consultați [Matrice cu mai multe camere](arrays.md) și [Referința CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Panoul „Calculator index”

A treia pagină din bara laterală, comună pentru rotița „Index” specifică fiecărei camere și pentru rotița „Index” a matricei combinate (câte una pe rând — titlul afișează „Calculator index — <camera name="">” sau „Calculator index — <array name="">”). Aceasta primește lista de benzi (benzile naturale ale filtrului camerei-benzile naturale ale camerei sau toate benzile de la membrii matricei), expresia curentă și configurația LUT (pornit/oprit, nivel — implicit 7 stopuri, minim — implicit 0,2, maxim — implicit 1), plus un histogramă a indicelui în timp real. **Aplicare** confirmă expresia; modificările LUT se aplică în timp real în previzualizare.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Setări per cameră vs. setări gestionate la nivel de matrice

Referință rapidă pentru a vedea ce se află unde atunci când o cameră face parte dintr-o matrice:

| Gestionate la nivel de matrice (doar pentru citire în panoul camerei) | Încă per cameră în interiorul unei matrice |
| --- | --- |
| Formatul pixelilor, rezoluția, binning | Expunere automată (expunere, amplificare, țintă, netezire, ROI) |
| Mod/sursă de declanșare, rata cadrelor | Reducerea zgomotului, claritate, vignetare |
| | Orientare (oglindire/rotație), suprapuneri de afișare (setări matrice → Previzualizare live setează histograma pe fiecare membru simultan), expunere punctuală |
| | Index (matrice cu afișare separată), asociere senzor de lumină |

Alte comportamente transversale:

* **Afișare combinată vs. separată** se alege la conectarea matricei: combinată = o singură casetă compusă aliniată (membrii transmit doar prin Split View); separată = fiecare membru redă propria casetă sincronizată. O cameră nu afișează niciodată atât un flux independent, cât și o casetă din matrice.
* **Reconectare automată**: deschiderea unui proiect salvat restabilește camerele și matricile acestuia și reaplică toate setările salvate la backend înainte ca fluxurile să fie reluate.
* **Filtrare captură**: camerele ascunse sau puse pe pauză sunt excluse din opțiunea „Capture All”; o matrice este blocată complet numai atunci când TOȚI membrii sunt ascunși/pusi pe pauză. Consultați [Setări și moduri de captură](capture.md).

## Cum se păstrează setările

Starea filei camerei este salvată **împreună cu proiectul**, nu în browser:

* Fiecare modificare reactivă realizează o captură instantanee a camerelor și a grupurilor în fișierul `cameras.json` al proiectului (cu o întârziere de 500 ms). Aceasta include numele și culorile camerelor, setările de expunere/amplificare/AE, formatul pixelilor/rezoluția/binningul, rata de declanșare, setările de previzualizare (rezoluția de randare, reducerea zgomotului, claritatea, vignetarea, profilul de culoare, zonele luminoase, saturația/contrastul), orientarea, suprapunerile, divizarea canalelor, configurația indexului, setările AE predictive, ROI-ul AE, numele matricilor, modul de afișare, setările de captură a matricilor (inclusiv decuparea ROI, dacă ROI este activat și decuparea memorată atunci când este dezactivat), bugetul de cablare al matricei și blocul de grilă (zoom de alimentare, modul de vizualizare, blocarea grilei, ordinea manuală a plăcilor, camere ascunse, plăci închise, cameră activă).
* Asocierile senzorilor de lumină sunt salvate în fișierul `sensors.json` al proiectului.
* Redeschiderea proiectului reconectează hardware-ul și reaplică toate aceste setări.
* **Niciun proiect deschis = doar sesiune**: în absența unui proiect, nimic nu se păstrează la închiderea aplicației Chloros.
* Valabil doar pentru sesiune, indiferent de proiect: starea de pauză, eșantioanele de măsurare punctuală, caseta de selectare „Țintă de calibrare” pentru fiecare cameră (deschisă întotdeauna în mod implicit) și profilul de aliniere a matricei (recalculat pentru fiecare sesiune, conform proiectării).
* O singură excepție: selecțiile de export din **Setări de captură** și modul de captură se păstrează pentru fiecare proiect în spațiul de stocare local al aplicației, nu în `cameras.json` — consultați [Setări și moduri de captură](capture.md).</array></camera></serial>
