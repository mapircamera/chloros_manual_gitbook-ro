# Setări cameră

Fila **Camere** reprezintă panoul de control în timp real al aplicației „Chloros

” pentru camerele LATTICE: o zonă principală de afișare care prezintă fiecare cameră conectată sub forma unei plăci live și o bară laterală care comută între trei pagini — **lista camerelor**, un**panou de setări**(setări pentru fiecare cameră, pentru grup de camere sau pentru captură — câte una pe rând) și**Calculatorul de index**. Această pagină documentează fiecare element de control din lista de camere, panoul de setări pentru fiecare cameră și panoul de setări pentru grupul de camere. Modurile de captură, selecția tipului de export și fluxul „Capture All” se găsesc pe pagina corespunzătoare [Setări și moduri de captură](capture.md).

Fila „Camere” apare în bara laterală odată ce backend-ulChloros

este pregătit. Toate elementele de control de mai jos comunică cu backend-ul local prin `127.0.0.1:5000`; modificările se aplică imediat camerei live, cu excepția cazului în care se specifică altfel.

## Tipuri de camere utilizate pe această pagină

Elementele de control se afișează sau se ascund în funcție de tipul de cameră selectat. Manualul utilizează următorii termeni pe tot parcursul său:

| Termen | Înțeles | Canale de filtru |
| --- | --- | --- |
| **CamerăRGB** | LATTICE M3C cu filtru FRGB (modelul conține `-FRGB`) |Red

/Green

/Blue

|
| **Multispectrală Bayer** | LATTICE M3C cu FRGN, FOCN sau FNGB | FRGN:Red

/Green

/NIR

· FOCN:Orange

/Cyan

/NIR

· FNGB:NIR

/Green

/Blue

|
| **Mono (M3M)** | LATTICE M3M — un filtru de bandă îngustă, o bandă calibrată | Bandă unică |
| **Membru al matricei** | O cameră conectată ca parte a unei matrice sincronizate (afișaj combinat sau separat) | Conform filtrului său |

CamereleRGB

e beneficiază de procesare fotometrică (balans de alb, profiluri de culoare, gamma); camerele multispectrale și mono beneficiază de lanțul radiometric și omit controalele fotometrice. Elementele matricei transmit setările la nivel de flux (format pixel, rezoluție, binning, declanșare, rată de cadre) către matrice — acele rânduri devin numai pentru citire în panoul specific fiecărei camere și sunt mutate în panoul de setări al matricei.

## Zona principală de flux



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

În absența camerelor conectate, zona de flux afișează un mesaj de bun venit **„Conectați o cameră pentru a începe”**cu două butoane:****Conectare cameră**(verde, deschide fereastra de dialog pentru conectarea unei singure camere) și**Conectare matrice** (albastru, deschide fereastra de dialog pentru conectarea matricei). Ferestrele de dialog pentru conectare sunt documentate în [Conectarea camerelor](connecting.md); conceptele legate de matrice (sincronizare, niveluri, lățime de bandă) sunt descrise în [Matrice cu mai multe camere](arrays.md). Când deschideți un proiect salvat care conține camere, ecranul de întâmpinare afișează în schimb un indicator de încărcare cu mesajul „Se redeschid N camere salvate…”, în timp ceChloros

restabilește fluxurile din ultima sesiune.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Bara de sus

| Comandă | Ce face |
| --- | --- |
| **Comutare mod de vizualizare**| Comută între**vizualizarea în grilă**(toate casetele ca celule) și**vizualizarea în listă** (matrice cu lățime maximă în partea de sus, O singură cameră activă dedesubt). Sfaturi: „Comută la vizualizarea în grilă” / „Comută la vizualizarea listă”. |
| **Blocare grilă**(lacăt) | Implicit**blocată** — casetele sunt fixate în poziție. Deblochează pentru a reordona casetele prin glisare în orice slot (spațiile libere sunt păstrate). Grila se blochează automat de fiecare dată când se conectează o nouă cameră. Sfaturi: „Deblochează grila (permite glisarea casetelor)” / „Blochează grila (îngheață casetele în loc)”. |
| Glisorul **Zoom flux** | Dimensiunea casetei, de la 60 px până la lățimea maximă a containerului. Celulele păstrează un raport de aspect de 4:3. Sub o lățime a celulei de 200 px, suprapunerile cu numele și fps-ul se ascund pentru a menține caseta curată. |

### Casete de flux

Fiecare cameră redă o casetă live compusă; o cameră poate afișa suplimentar trei casete **pe canal** (vezi [Împărțiri pe canale](#display-overlays-drawn-over-the-live-feed)), iar matricile redau o casetă combinată. Caseta activă are un inel de selecție în culoarea camerei (sau a matricei).

La trecerea cursorului peste o casetă apare un buton de închidere **X**:

* Închiderea unei casete **compozite** în timp ce diviziunile sale de canal rămân vizibile ascunde doar caseta compozită.
* Închiderea **ultimului panou vizibil al unei camere independente** deconectează camera respectivă.
* **Panourile divizate ale membrilor unei matrice combinate nu deconectează niciodată** camera — ele doar o ascund.

Cu grila deblocată, trage orice casetă în orice slot; dispunerea se salvează odată cu proiectul.

## Bara laterală — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

camerelor Prima pagină a barei laterale listează toate camerele și matricea conectate:

* **Conectare cameră**(verde) /**Conectare matrice** (albastru, afișează „Detectare...” în timpul scanării). Ambele sunt dezactivate cât timp este deschis un dialog de conectare.
* **Captură totală** (roșu) — capturează toate camerele listate cu tipurile de export alese în Setările de captură. Necesită un proiect deschis. Documentat complet în [Setări și moduri de captură](capture.md).
* **Roata dințată Setări de captură** (lângă Captură totală) — deschide [panoul Setări de captură](capture.md#the-capture-settings-pane). Dezactivat în absența unui proiect sau în timpul captării.

### Rânduri de camere

Fiecare rând de camere afișează o margine codificată prin culori (culoarea personalizată a camerei), o etichetă „CAM” — cu litera albastră **M**(master) sau**S** verde (slave) pentru membrii matricei — și numele afișat. Numele implicit este `LATTICE-MODEL (serial)`; îl puteți redenumi din panoul de setări al fiecărei camere. Butoane de rând:

| Buton | Efect |
| --- | --- |
| **Ochi**| Comută vizibilitatea. Camerele ascunse ies din grilă și sunt**excluse din „Captură totală”**. |
| **Roată dințată** | Deschide panoul de setări pentru fiecare cameră (secțiunea următoare). |
| **Pauză / Redare**| Îngheață previzualizarea live**doar pe ecran** — capturarea din backend continuă să ruleze. Camerele puse pe pauză nu pot captura. |
| **X** | Deconectare. Interfața se actualizează imediat (în cel mai bun caz); deconectarea efectivă în fundal poate dura între 10 și 30 de secunde. |

### Rânduri ale matricei

Un rând al matricei afișează o insignă „ARRAY” în culoarea matricei, numele matricei (care poate fi redenumit în setările matricei) și o **DAQ · activat/dezactivat**—**activat** când senzorul de lumină la nivel de matrice este setat *sau* orice membru are un senzor specific pentru cameră; informația de tooltip indică exact ce senzor alimentează ce. Camerele membre sunt listate indentate dedesubt, cu propriile lor rânduri. Butoanele rândului de matrice: **ochi**(ascunde/afişează TOATE componentele împreună),**roată dinţată**(panoul de setări al matricei),**X**(deconectează întreaga matrice).

Starea senzorului de lumină (DLS) utilizată în rândurile matricei şi în panoul de setări al matricei are patru stări:**oprit**,**în așteptare**(încă nu există spectru),**activ**(un spectru a sosit în ultimele 3 s) și**învechit** — nu există spectru nou în ultimele 3 s, dar ultima citire este *încă utilizată* (citirile DAQ nu expiră niciodată pe calea de captare).

Puteți trage camerele independente și grupurile întregi de matrice unele peste altele în bara laterală pentru a reordona lista; elementele matricei nu pot fi trase independent.

## Panoul de setări pentru fiecare cameră

Se deschide cu **rotița** dintr-un rând de camere. Panoul se deplasează peste lista de camere.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Antet**:**mostra de culoare**a camerei (faceți clic pentru a deschide un selector de culori nativ — setează culoarea chenarului barei laterale și a inelului de selecție a plăcilor),**numele**cu un buton**Redenumire**în formă de creion (salvarea unui nume gol revine la valoarea implicită `MODEL (serial)`) și**×** pentru a închide.

### Histogramă în timp real

În partea de sus a panoului se află o histogramă de luminozitate în timp real, calculată pe baza previzualizării „JPEG

” la ~8 Hz. Media este ponderată după metoda Bayer — (R+2G+B)/4 — pentru a se potrivi cu măsurarea AE specifică camerei.

* **Linia punctatăOrange**= ținta AE. **Trageți-o pe orizontală pentru a redefini ținta** — la eliberare se trimite o comandă, iar glisarea comută modul țintei AE pe Manual.
* **Linia continuăGreen** = media reală a luminozității (ceea ce oferă AE în prezent).
* **ButoaneleRGB** (dreapta sus): comută histogramele suprapuse pe benzi, colorate în funcție de filtrul camerei (de exemplu, pe FRGN: griNIR

, verde, roșu). La camerele mono (M3M), butonul afișează „MONO” și este dezactivat — modul mono afișează întotdeauna histograma de luminozitate pe o singură bandă.
* Etichetele axei X respectă adâncimea de biți a senzorului pentru formatul curent de pixeli: 0..255, 0..1023, 0..4095 sau 0..65535.

### Rânduri

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

cu informații despre cameră

| Rând | Comportament |
| --- | --- |
| **Model** | Doar citire (de ex. `LATT-M3C-L87-FRGN`). |
| **Calibrare radiometrică** |Green

Insignă **„Activ”**cu o legendă care afișează nivelul de calibrare, hash-ul, data calibrării și lista benzilor, încărcate din pachetul de calibrare al camerei (vezi [Calibrare radiometrică din fabrică](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Ascuns pentru camerele dRGB

** — acestea dispun de o calibrare fotometrică a balansului de alb, nu de radianță pe bandă. |
| **Raport de calibrare**| Butonul**Descărcare** — deschide fișierul PDF al certificatului de calibrare NIST specific fiecărui număr de serie al camerei în vizualizatorul sistemului de operare. Dacă certificatul nu este încă stocat în cache, se afișează în schimb un mesaj de îndrumare „Chloros

”. |
| **Număr de serie** | Doar pentru citire. |
| **Firmware**| Afișează versiunea curentă, apoi identifică versiunea disponibilă pentru acest model (memorată în cache pentru fiecare model — o rețea de N camere verifică serverul o singură dată). Stări: „Se verifică…” → butonul**„Actualizare la X”**→ „Se actualizează…” → „Actualizat de la A la B” / „Eșuat: …” / „Omit: …” /**„La zi”** (verde). Sfatul afișat la trecerea cursorului peste butonul de actualizare: „Resetare la setările din fabrică + actualizare firmware + reprogramare UserSet1. ~2–3 minute; nu deconectați.” |
| **IP** | Doar citire. |
| **Temperatură** | Doar pentru citire, actualizată la fiecare 3 s. Devine portocaliu la ≥65 °C și roșu cu un ⚠ la ≥75 °C. |
| **Caseta de selectare**Țintă de calibrare** | Activează detectarea țintei de reflectanță ArUco cu un tabel de validare „NDVI

” pentru fiecare panou, situat sub fluxul live (vizualizare listă). Doar pentru sesiune — se deschide întotdeauna dezactivată. |
| Meniul derulant **Senzor de lumină** | Asociază un senzor de lumină DAQ (DAQ-E/M/U, din lista din fila Senzori de lumină) cu această cameră pentru corecția iluminării cu lumină descendentă (DLS) și expunerea automată predictivă. „Niciunul” anulează asocierea. Dacă nu sunt conectați senzori, meniul derulant afișează „(nu sunt senzori conectați — deschide fila DAQ)”. Asocierea este salvată împreună cu proiectul. |

### Expunere și amplificare

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Toate câmpurile numerice de aici utilizează rotițe de reglare cu accelerare la menținere: atingere = ±1, menținere &gt;1,5 s = ±10, menținere &gt;3 s = ±100. Valoarea este trimisă către cameră atunci când eliberați butonul.

| Control | Interval / opțiuni | Implicit | Se aplică la | Ce face |
| --- | --- | --- | --- | --- |
| **Expunere (us)**| Valoarea minimă/maximă în timp real a camerei | Auto | Toate | Timpul de expunere în microsecunde, cu un comutator**Auto/Manual**. Auto = expunere automată continuă din partea camerei. |
| **Câștig (dB)**| Valoarea minimă/maximă în timp real a camerei (de ex. până la 48 dB) | Manual (dezactivat) | Toate | Câștig analogic/digital cu propriul comutator**Auto/Manual**. |
| **Luminozitate țintă AE**| 0–255 | 80, modul**Auto**| Toate (modificabil când AE sau câștigul automat sunt activate) | Luminozitatea vizată de AE. În modul**Auto**(implicit), un controler de fundal bazat pe histogramă alege singur valoarea țintă, menținând expunerea la 60–75 % din valoarea maximă a senzorului. Introducerea unei valori sau glisarea liniei portocalii a histogramei comută modul în**Manual**. |
| **Netezire AE** | 0,5–40, pas 0,1 | 8,0 | Toate | Amortizarea AE. Informație rapidă: „Cu cât valoarea este mai mică, cu atât AE reacționează mai rapid (poate pulsa la fps ridicat). Cu cât valoarea este mai mare, cu atât este mai lin/mai lent.” Valorile mult sub cele implicite pot face ca AE să pulseze și să destabilizeze transmisia în direct la rate de cadre ridicate; 8,0 este valoarea implicită stabilă. |
| **Zona de interes AE**| Casetă de selectare „Activat” + buton**Țintire**| Dezactivat | Toate | Când este activată, AE măsoară doar regiunea marcată cu linie verde punctată, în loc de întregul cadru.**Țintire** activează funcția „clic pentru plasare” pe fluxul live: un clic centrează o regiune la 30 % din cadru; un clic și glisare trasează un dreptunghi personalizat (minimum 5 % × 5 %). Funcția „Aim” se dezactivează automat după o singură plasare. Regiunea este remapată în coordonatele native ale camerei, ținând cont de orice rotație/oglindire pe care ați setat-o, și este salvată odată cu proiectul. |
| **Viteza de reglare AE** | 0,1–5, pas de 0,1 | 1,0 | Numai pentru membrii matricei | Cât de repede ținta AE automată urmărește schimbările de luminozitate ale scenei; la 1,0× se verifică din nou la fiecare 2,5 s. |
| **Protecție împotriva supraexpunerii** | Strictă (1 %) / Normală (5 %) / Relaxată (15 %) | Strict | Camere care expun setarea | Cât din cadru se poate tăia în alb înainte ca AE să întunece imaginea. |

{% hint style="info" %}
**Cerințe de iluminare pentru camerele multispectrale Bayer (RGN

/OCN

/NGB

):** scena trebuie să aibă suficientă lumină în toate cele trei canale, altfel calibrarea nu funcționează corect — o singură expunere a senzorului acoperă toate cele trei spectre. Folosiți un senzor de lumină DAQ pentru a măsura lumina sau treceți la modul complet monocrom (M3M), astfel încât fiecare bandă să aibă propria expunere. Dacă o captură încalcă această regulă,Chloros

o detectează și vă avertizează (notificarea „unmix-clamp”).
{% endhint %}

### Formatul pixelilor și rezoluția

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Elementele matricei** afișează rânduri de tip „Current” (format + LxH) și „Binning” numai pentru citire, cu nota „Setat în setările matricei” — o repornire a fluxului pe un element ar întrerupe sincronizarea, așa că acestea sunt gestionate în [panoul de setări al matricei](#array-settings-pane).**Camerele autonome** dispun de:

| Control | Opțiuni | Ce face |
| --- | --- | --- |
| **Formatul pixelilor** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formatul pixelilor senzorului (adâncime de biți). |
| **Rezoluție** | Completă / Jumătate / Sfert | Relativă la binning-ul curent: Completă = 2048/N × 1536/N pentru binning N×N. |
| **Binning** | 1x1 (niciunul) / 2x2 / 4x4 | Binning hardware N×N — valorile mai mari reduc rezoluția, dar îmbunătățesc raportul semnal-zgomot (SNR) și rata de cadre. Modificarea acestuia repornește fluxul și resetează orice zonă de interes (ROI) la noul câmp vizual complet. |
| **Curenți** | numai pentru citire | Lățimea (W) și înălțimea (H) efective, precum și offsetul (x, y) aplicat. |

### Previzualizare live

Tot ce se află în această secțiune se referă **doar la afișare**— modifică ceea ce vedeți în fluxul live, în timp ce capturile salvate rămân liniare și nemodificate — cu o singură excepție:**Vigneta** este radiometrică și afectează și exporturile (prezentată mai jos).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Control | Interval / opțiuni | Implicit | Se aplică la | Ce face |
| --- | --- | --- | --- | --- |
| **Rezoluție de redare** | 360p (cea mai rapidă) / 480p / 720p / 1080p / Rezoluția nativă a senzorului (cea mai lentă) | 720p | Toate | Înălțimea la care backend-ul rulează lanțul de previzualizare radiometrică. O valoare mai mică asigură o rată de cadre mai mare fără a modifica câmpul vizual. |
| **Index**| Casetă de selectare „Activat” + rotiță | Dezactivat | Numai multispectral Bayer,**nu** pentru elementele cu matrice combinată | Previzualizare live a indicelui de vegetație. Rotița deschide panoul partajat [Calculator de index](#index-calculator) preîncărcat cu benzile naturale ale filtrului camerei (de ex. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Expresia personalizată plus LUT (pornit/oprit, nivel implicit 3, minim implicit 0,2, maxim implicit 1) este calculată la fiecare cadru de previzualizare. Membrii matricei combinate ascund acest rând — matricea deține un singur indice comun. |
| **Balanța de alb** | Oprit / O singură dată / Continuu + un buton de recaptare | Continuu | NumaiRGB

| Balanță de alb în timp real. Butonul de reîmprospătare recapturează balansul de alb din spectrul DLS curent (dezactivat când modul este Oprit). |
| **Gamma** | Activat / Oprit | Activat | NumaiRGB

| Afișează gamma (γ = 2,2 LUT) în previzualizarea live. Capturile salvate rămân liniare. |
| **Reducere zgomot** | Casetă de selectare + intensitate 0–100 | Dezactivat / 50 | Toate (per cameră, chiar și în cadrul matricilor) | Filtru bilateral în previzualizarea în timp real. Valori mai mari = detalii mai fine, dar mai estompate. |
| **Claritate** | Casetă de selectare + intensitate 0–100 | Dezactivat / 30 | Toate | Mască de neclaritate în previzualizarea live, aplicată ultima. Poate amplifica zgomotul. Doar în previzualizare. |
| **Vignetă**| Casetă de selectare + intensitate 0–100 | Dezactivat / 0 | Toate | Eliminare manuală a vignetei reziduale (luminează colțurile), suprapusă peste estimarea Smart Vignette a matricei.**Radiometric — afectează vizualizarea în timp real ȘI exporturile**, spre deosebire de Denoise/Sharpness. |
| **Profil de culoare** | Raw / Liniar / Natural / Îmbunătățit / Temperatură personalizată | Natural | Numai pentru „RGB

” | Vezi mai jos. |
| **Temperatură de culoare** | 2000–10000 K, pas de 100 | 5500 K | Numai profilulRGB

și Temperatură personalizată | Fixează balansul de alb la o temperatură de culoare corelată fixă (intrarea DLS este ignorată). Ultima valoare Kelvin selectată este memorată la schimbarea profilurilor. |
| **Saturație** | 0–200 (100 = neutru) | 100 | NumaiRGB

| Saturație HSV în previzualizarea live. |
| **Contrast** | 0–200 (100 = neutru) | 100 | NumaiRGB

| Contrast liniar în jurul griului mediu în previzualizarea live. |
| **Oglindire H / Oglindire V** | Casete de selectare | Dezactivat | Toate | Răsucește previzualizarea pe orizontală / pe verticală. |
| **Rotație**| 0° / 90° / 180° / 270° | 0° | Toate | Rotește previzualizarea. Orientarea se aplică la sfârșitul lanțului de previzualizare din backend —**capturile salvate rămân în orientarea nativă a camerei**, iar vizualizările compozite de tip matrice o ignoră. |**Semantica profilului de culoare** (camereleRGB

):

* **Raw** — ocolește complet lanțul de procesare.
* **Liniar** — semnal întunecat + câmp plat + balans de alb; fără matrice de culoare, fără gamma.
* **Natural** *(implicit)* — liniar plus matricea de corecție a culorii măsurată și o curbă de ton adaptabilă la scenă.
* **Îmbunătățit**— Natural plus vibranță și contrast local CLAHE. Costul suplimentar se aplică**numai previzualizării live** — capturile salvate beneficiază întotdeauna de finisajul complet, indiferent de profil.
* **Temperatură personalizată** — Natural cu balansul de alb fixat la valoarea în Kelvin aleasă de dvs.

{% hint style="warning" %}
Pentru opțiunile Natural, Îmbunătățit și Temperatură personalizată, panoul afișează o notă privind tonul: cadrele sunt luminate în funcție de propria scenă, astfel încât imaginile *afișate* salvate nu sunt comparabile între ele. **Exportați radianța sau reflectanța pentru măsurători.**
{% endhint %}

### Suprapuneri de afișare (desenate peste fluxul live)

Acestea sunt disponibile doar în interfața de utilizator — sunt suprapuse peste videoclip, fără a afecta niciodată fluxul sau capturile.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Suprapunere | Comenzi | Implicit | Ce face |
| --- | --- | --- | --- |
| **Zebra** | Casetă de selectare + prag 200–255 | Dezactivat / 250 | Dungi diagonale magenta pe pixelii decupați. |
| **Reticul** | Casetă de selectare | Dezactivat | Marcaj de centrare a cadrului. |
| **Grilă** | Dezactivat / 3 × 3 / 9 × 9 | Dezactivat | Grilă de compoziție. |
| **Histogramă** | Casetă de selectare + lățime 0,10–0,90 din cadru | Dezactivat / 0,25 | O bandă de histogramă afișată în timp real. |
| **Focus Peak** | Casetă de selectare + prag 20–200 + mostră de culoare | Dezactivat / 80 / `#ff5722` | Evidențierea marginilor Sobel pentru focalizare. |
| **Împărțirea canalelor** | „Afișează divizări (Red

/Green

/NIR

)” / butonul „Ascunde divizări” | Ascuns | Adaugă trei casete independente în tonuri de gri pentru fiecare canal, alături de imaginea compusă (eticheta butonului respectă canalele de filtru ale camerei). Fiecare casetă divizată poate fi trasă și are aceeași culoare ca marginea camerei. Nu este disponibil pe camerele monocrome. Se salvează odată cu proiectul. |

### Spot Meter

* Caseta de selectare **Click to Sample**: faceți clic pe fluxul live pentru a preleva o probă dintr-un singur pixel (acesta este marcat cu un reticul în formă de cruce) sau faceți clic și trageți pentru a selecta o regiune din care se calculează media pixelilor.**Clear**șterge proba și reticulul. Se exclude reciproc cu modul**Aim** al AE-ROI.
* Meniul derulant **Afișare**:**Raw (adâncime de biți)**— valori digitale native la adâncimea de biți a senzorului (de ex. 12 biți → 0..4095) — sau**Afișare (8 biți)** (implicit). Când un indice în timp real este activ, opțiunea „Afișare” arată în schimb valoarea indicelui calculat (de ex.,NDVI

).
* Panoul de citire listează coordonatele pixelilor, dimensiunea cadrului, formatul pixelilor, adâncimea de biți și un tabel de canale (Chan / Valoare / %) cu etichetele benzilor și lungimile de undă; perechile de verde Bayer sunt mediatizate; eșantioanele din regiune afișează „N px avg”.

Starea expozimetrului punctual este valabilă doar pentru sesiunea curentă.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Expunere automată predictivă (bazată pe DLS)

Această secțiune apare doar atunci când **este conectat cel puțin un senzor de lumină DAQ** — solverul are nevoie de un spectru descendent în timp real pentru a funcționa.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Control | Interval | Implicit | Ce face |
| --- | --- | --- | --- |
| **Activare** | Casetă de selectare | Activat (camere autonome) | Un solver de formă închisă utilizează spectrul DLS împreună cu scalarii din pachetul de calibrare al camerei pentru a aduce banda cea mai luminoasă aproape de saturație, menținând în același timp banda cea mai slabă deasupra pragului SNR — o singură expunere pe rezolvare, fără buclă de stabilizare. Conceput pentru timelapse-uri alimentate cu energie solară, unde fiecare cadru trebuie expus corect. Backend-ul revine automat la AE reactiv ori de câte ori citirea DLS este învechită/lipsă sau pachetul de calibrare nu este încărcat. |
| **Netezire (α)** | 0,05–1,0, pas de 0,05 | 0,3 | Netezirea soluțiilor predictive succesive (cu cât valoarea este mai mică, cu atât netezirea este mai mare). |
| **Reflectanța scenei**| Butonul**Recalibrare ρ** | — | Reestimează factorul de reflectanță a scenei utilizat de solver. |

{% hint style="info" %}
**Funcția „Array connect” dezactivează implicit AE predictiv** — pentru matrice, AE inteligent alChloros

plus expunerea automată din partea camerei gestionează expunerea (cu protecție împotriva saturației), iar estimarea unică a reflectanței scenei de către AE predictiv nu este sigură în cazul scenelor mixte. Puteți să o reactivați pentru fiecare cameră în parte aici, dacă doriți în mod specific o expunere radiometrică bazată pe DLS.
{% endhint %}

**Limita maximă de expunere bazată pe DAQ și AE fixată pe incidență.** Independent de caseta de selectare de mai sus, atunci când un senzor de lumină DAQ este atribuit unei camereRGB

,Chloros

calculează — pe baza iradianței absolute descendente măsurate — expunerea maximă × câștig la care o suprafață cu reflectanță de 100 % rămâne sub pragul de clipping și o aplică ca **limită maximă**pentru expunerea automată. Cât timp limita maximă este activă, camera funcționează în**modul fixat pe lumina incidentă**: funcționează în buclă deschisă la expunerea măsurată în funcție de lumina incidentă, cu amplificarea la 0 dB — expunerea urmează lumina măsurată, nu conținutul scenei. Deoarece limita superioară poate doar să scurteze expunerea, ea nu poate provoca singură clipping. Limita superioară se dezactivează automat — iar expunerea automată normală a scenei se reia — ori de câte ori citirea DAQ lipsește, este învechită (&gt;30 s), sau întunecată, sau dacă ≥15 % din cadru se clipează la expunerea fixată (ceea ce înseamnă că senzorul și camera detectează iluminări diferite). Nu există un comutator în interfața grafică; acesta este comportamentul standard ori de câte ori o camerăRGB

are o legătură DAQ.

### Achiziție și declanșare

Elementele<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

matricei afișează în plus rândurile **Rol**(doar pentru citire) (Master în albastru / Slave în verde),**Linie de sincronizare**și**Echivalenți**.

| Control | Opțiuni | Implicit | Note |
| --- | --- | --- | --- |
| **Mod declanșare** | Oprit / Pornit | Pornit | Dezactivat pentru elementele matricei (matricea gestionează declanșarea). |
| **Sursă de declanșare** | Software / Linia 0 (M8) / Linia 1 / Linia 2 | Linia 0 | Ascunsă când Modul de declanșare este Dezactivat; dezactivată pentru elementele matricei. Linia 0 este intrarea de declanșare externă optoizolată a modelului M8. |
| **Rata de cadre**| Auto / Manual + valoare | Auto |**Auto**: limita de frecvență a cadrelor camerei este dezactivată — expunerea dictează fps, iar caseta afișează frecvența reală în timp real.**Manual**: limitați fps cu un glisor (de la 1 până la maximul limitat de lățimea de bandă), pornind de la frecvența reală curentă. Membrii matricei văd o valoare de tip „N fps (live)”, numai pentru citire, cu mențiunea „Setat în setările matricei”. |

### Rețea / Transport

| Rând | Comportament |
| --- | --- |
| **Dimensiune pachet**| 1500 (Standard) / 9000 (Jumbo) — implicit**Jumbo**. |
| **Debit** | Limită a debitului de legătură, numai pentru citire, în MB/s. Backend-ul reechilibrează această valoare între toate camerele conectate la fiecare conectare/deconectare. |
| **Gestionarea bufferului** | Mod de gestionare a bufferului, doar pentru citire. |

### Captură

Panoul se încheie cu un buton **„Deschide setările de captură…”** care duce la [panoul Setări de captură](capture.md#the-capture-settings-pane) (dezactivat până la deschiderea unui proiect — „Creați sau deschideți un proiect pentru a salva capturile”). Dacă camera este ascunsă sau pusă în pauză, un mesaj vă reamintește să o afișați/reluați înainte de captură.

## Panoul de setări pentru matrice

Deschideți-l folosind **rotița**dintr-un rând ARRAY. Antet: numele matricei cu un creion de redenumire și**×** pentru închidere. Secțiunile de mai jos marcate cu *numai combinate* apar doar pentru matricele conectate în modul de afișare combinat.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sincronizare

Rânduri **Master**,**Slaves**și**Sync Line** numai pentru citire.

### Senzor de lumină ambientală

Afișat atât pentru matrice combinate, cât și pentru cele separate:

* Caseta de selectare **Calibration Target** — „Detectează o țintă ArUco dMAPIR

și valideazăNDVI

față de LUT-ul de reflectanță al panoului”; controlează suprapunerea țintei și tabelul de validare ale plăcii combinate.
* Meniul derulant **Light Sensor**— asociază un DAQ întregii matrice. Selecția se aplică imediat, se propagă către meniul derulant**Senzor de lumină** al fiecărei camere din matrice (puteți în continuare să suprascrieți setările pentru fiecare cameră în parte) și începe să transmită spectrele către matrice.
* Linia **Stare** în timp real: Oprit · „Se așteaptă primul spectru…” · „Activ — toate camerele din matrice sunt corectate din punct de vedere al iluminării” · „Niciun spectru nou în ultimele 3 s — se utilizează în continuare ultima citire (fără expirare a timpului de așteptare)…”.
* Notă în panou: „Corecție radiometrică la nivel de matrice. Setările individuale ale camerelor au prioritate asupra acesteia.”

### Captură — setări uniforme ale senzorului *(numai combinate)*

Aceste setări se aplică uniform tuturor camerelor din matrice (modificările individuale ar perturba sincronizarea). Modificările sunt stocate temporar și aplicate simultan.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Control | Opțiuni / interval | Ce face |
| --- | --- | --- |
| **Format pixel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Format uniform al senzorului pentru toate elementele. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning hardware — păstrează câmpul vizual complet, îmbunătățind în același timp raportul semnal-zgomot (SNR) și rata de cadre. Modificarea acestei setări resetează câmpurile ROI la noul câmp vizual complet. |
| **Rezoluție** prestabilită | Completă / Jumătate / Sfert | Relativă la binning; completează câmpurile ROI cu o decupare centrată. |
| **Decupare ROI (px)**| Câmpuri numerice L / Î / X / Y | Decupare senzor. Lățimea/înălțimea se aliniază la multipli de 16 (minimum 64); decalajele se aliniază la multipli de 4. Un indiciu „max LxH” arată limita maximă, iar**Resetare** revine la câmpul vizual complet. În timpul editării, pe caseta matricei se afișează o casetă portocalie de previzualizare a decupajului în timp real (inclusiv o schemă a senzorului complet atunci când decupajul este extins spre exterior). |
| **Rata de declanșare**| Comutator Auto / Manual + fps 0,5–10, pas 0,5 |**Auto**(implicit): backend-ul calculează rata de declanșare pe baza rezoluției și a lățimii de bandă — câmpul de introducere este dezactivat și afișează valoarea calculată.****Manual**: fixează valoarea la apăsarea butonului**Apply**. |

Notă în panou: „Modificările de format/rezoluție repornesc scurt toate camerele. Rata de declanșare se aplică în timp real.” Butoanele **Apply**/**Cancel** se află în partea de jos a panoului.

### Aliniere (co-înregistrare) *(doar combinată)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Caseta **Rezidual**: „RMS x,xx px” — verde sub 1 px, galben sub 3 px, roșu în celelalte cazuri sau dacă vreo cameră a eșuat; „fără profil” înainte de prima rezolvare.
* Linia de rezumat: „✓ Toate camerele aliniate (N)” / „⚠ p/N camere aliniate —  <serial (filter)="">eșec” / „Decupare activă — Recalibrați pentru aliniere (utilizează senzorul complet)” / „Așteptare pentru stabilizarea expunerii…”.
* Tabel per cameră: cameră (ultimele 4 cifre ale numărului de serie + filtru), eroare de reproiectare în px cu numărul de potriviri („ref” pentru camera principală) și scorul de corelație încrucișată normalizat al suprapunerii în raport cu pragul minim de trecere de 0,35.
* Butonul **Recalibrare aliniere** (afișează „Calibrare aliniere” înainte de primul profil) — reexecută co-înregistrarea pe cadre noi.
* Caseta de selectare **„Expunere automată a camerelor pentru aliniere”** (bifată implicit) — luminează temporar camerele întunecate sau cu imagine plată (mai întâi expunerea, apoi amplificarea), astfel încât acestea să aibă textură care să se potrivească, apoi restabilește expunerea automată.

Previzualizarea combinată se aliniază automat la deschidere; recalibrați dacă s-au modificat focalizarea sau adâncimea scenei. Alinierea este **destinată exclusiv sesiunii respective** — nu este niciodată salvată într-un profil, deoarece depinde de distanța scenei din momentul respectiv. Capturile pot fi totuși exportate cu înregistrare la nivel de pixel (consultați [Exporturi aliniate](capture.md#per-array-controls)).

### Vigneta inteligentă

* Caseta de selectare **Activare corecție**— aplică estimarea vignetei pentru fiecare cameră în parte la lanțul radiometric (în timp real**și** la exporturi).
* **Calibrare din vizualizarea curentă**— îndreptați mai întâi matricea către o țintă uniformă (ecran plat, perete sau cer); fiecare cameră este aplanată individual, iar starea raportează un câștig de aplanare de „n/N camere · −x,x %”. Opțiunea**Ștergere** elimină estimarea.
* Reglați fin pentru fiecare cameră cu glisorul **Vignette** specific fiecărei camere din [Previzualizare live](#live-preview).

### Previzualizare live *(numai combinată)** **Index**: bifați caseta de selectare + rotița — se deschide [Calculatorul de index](#index-calculator-pane) partajat, cu benzi desenate din**toate** camerele membre. O linie de previzualizare a expresiei de dedesubt afișează expresia curentă („Nicio expresie setată — deschideți calculatorul pentru a crea una”), actualizată la fiecare secundă.
* **Meniul derulant**Rezoluție de randare**(aceleași presetări ca pentru fiecare cameră, implicit 720p): înălțimea fluxului de vizualizare în direct**și** dimensiunea de export a compoziției salvate. Notă în panou: „Previzualizare + dimensiunea compoziției salvate. Imaginile pentru fiecare cameră se exportă întotdeauna la rezoluție maximă.”

### Straturi de afișare *(numai combinate)** Caseta de selectare **Activare** (implicit dezactivată — camera principală este afișată direct; activată = compoziție stratificată).
* Meniuri derulante **Prim-plan**/**Fundal**: fiecare cameră membră (după nume) sau**Index**. Când opțiunea Prim-plan este setată pe Index, pixelii aflați în afara intervalului Min/Max al LUT afișează stratul Fundal.

### Vizualizare divizată *(numai în regim combinat)*

**„Afișează camerele membre”**— un buton**Împărțire / Ascundere camere membre** care adaugă fluxul live al fiecărei camere membre sub formă de casete separate în grilă, lângă compoziția combinată. Casetele citesc bufferul de cadre existent al matricei (fără conexiune suplimentară la cameră). Doar vizualizare în grilă; se salvează pentru fiecare matrice împreună cu proiectul.

### Capacități

Un panou numai pentru citire, reîmprospătat la fiecare 5 s:

* **Etichetă nivel**: „Captură simultană” (verde) · „Captură simultană (emisie eșalonată FTD)” (verde) · „Captură eșalonată (deviere de 100 ms)” (portocaliu) · „Configurație prea mare” (roșu).
* **Starea cadrelor**: „x,xx % incomplete” — verde sub 1 %, galben sub 5 %, roșu la 5 % sau mai mult.
* **Linie de legătură**: „NIC {mbps} Mbps - susținut {MB/s} MB/s”.

Acesta este bugetul de lățime de bandă în timp real al matricei. Pentru modelul de rețea și fps-ul de bază — precum și ce trebuie modificat când nivelul devine galben sau roșu — consultați [Matrice cu mai multe camere](arrays.md) și [Referința CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panoul „Calculator index”

A treia pagină din bara laterală, comună pentru butonul de configurare „Index” per cameră și pentru butonul de configurare „Index” al matricei combinate (câte una pe rând — titlul afișează „Calculator index — <camera name="">” sau „Calculator index — <array name="">”). Aceasta primește lista benzilor (benzile naturale ale filtrului camerei-naturale ale camerei sau toate benzile de la membrii matricei), expresia curentă și configurația LUT (pornit/oprit, nivel — implicit 3, minim — implicit 0,2, maxim — implicit 1), plus un histogramă de index în timp real. Butonul **Aplicare** confirmă expresia; modificările LUT se aplică în timp real în previzualizare.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Setări per cameră vs. setări gestionate la nivel de matrice

Referință rapidă pentru a vedea ce se află unde atunci când o cameră este membru al unei matrice:

| Gestionate la nivel de matrice (doar citire în panoul camerei) | Încă per cameră în interiorul unei matrice |
| --- | --- |
| Format pixel, rezoluție, binning | Expunere automată (expunere, amplificare, țintă, netezire, ROI) |
| Mod/sursă de declanșare, frecvență de cadre | Reducere zgomot, claritate, vigneta |
| | Orientare (oglindire/rotație), suprapuneri de afișare, expunere punctuală |
| | Index (matrice cu afișare separată), asocierea senzorului de lumină |

Alte comportamente comune:

* **Afișare combinată vs. separată** se alege la conectarea la matrice: combinată = o singură secțiune compozită aliniată (fluxurile membrilor doar prin Split View); separat = fiecare membru redă propria sa casetă sincronizată. O cameră nu afișează niciodată atât un flux independent, cât și o casetă din matrice.
* **Reconectare automată**: deschiderea unui proiect salvat restabilește camerele și matricea acestuia și reaplică toate setările salvate la backend înainte ca fluxurile să fie reluate.
* **Controlul capturii**: camerele ascunse sau puse pe pauză sunt excluse din „Capture All”; o matrice este blocată complet numai atunci când TOȚI membrii sunt ascunși/în pauză. Consultați [Setări și moduri de captură](capture.md).

## Cum se păstrează setările

Starea filei camerei este salvată **împreună cu proiectul**, nu în browser:

* Fiecare modificare reactivă salvează instantanee ale camerelor și matrice-lor în fișierul `cameras.json` al proiectului (cu o întârziere de 500 ms). Aceasta include numele și culorile camerelor, setările de expunere/amplificare/AE, formatul pixelilor/rezoluția/binningul, rata de declanșare, setările de previzualizare (rezoluția de randare, reducerea zgomotului, claritatea, vignetarea, profilul de culoare, saturația/contrastul), orientarea, suprapunerile, divizările de canale, configurația indexului, setările AE predictive, ROI AE, numele matricelor, modul de afișare, setările de captură ale matricelor (inclusiv poziția de decupare a ROI) și blocul de grilă (zoom feed, modul de vizualizare, blocarea grilei, ordinea manuală a plăcilor, camerele ascunse, plăcile închise, camera activă).
* Asocierile senzorilor de lumină sunt salvate în fișierul `sensors.json` al proiectului.
* Redeschiderea proiectului reconectează hardware-ul și reaplică toate aceste setări.
* **Niciun proiect deschis = numai sesiune**: în absența unui proiect, nimic nu se păstrează după închiderea aplicației „Chloros”.
* Numai sesiune, indiferent de proiect: starea de pauză, eșantioanele de măsurare punctuală, caseta de selectare „Țintă de calibrare” pentru fiecare cameră (deschisă întotdeauna ca dezactivată) și profilul de aliniere a matricei (recalculat pentru fiecare sesiune conform proiectării).
* O singură excepție: selecțiile de export din **Setări de captură** și modul de captură sunt păstrate pentru fiecare proiect în spațiul de stocare local al aplicației, nu în `cameras.json` — consultați [Setări și moduri de captură](capture.md).</array></camera></serial>
