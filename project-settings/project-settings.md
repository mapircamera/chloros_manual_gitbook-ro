# Setări proiect

Bara laterală „Setări proiect” (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) din Chloros vă permite să configurați toate aspectele legate de procesarea imaginilor, detectarea țintelor de calibrare, calcularea indicilor multispectrali și opțiunile de export pentru proiectul dumneavoastră. Aceste setări sunt salvate odată cu proiectul și pot fi salvate ca șabloane pentru a fi reutilizate în mai multe proiecte.

## Accesarea setărilor proiectului

Pentru a accesa setările proiectului:

1. Deschideți un proiect în Chloros
2. Faceți clic pe fila **Setări proiect**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

din bara laterală din stânga
3. Panoul de setări va afișa toate opțiunile de configurare disponibile, organizate pe categorii

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Setările care depind de alte setări sunt estompate.** Atunci când un comutator părinte face imposibilă o setare (de exemplu, debifarea *Calibrare reflectanță / balans de alb* face imposibilă opțiunea *Export reflectanță*), comanda dependentă este dezactivată, iar informația rapidă indică comutatorul care trebuie modificat.
{% endhint %}

***

## Afișare

### Rezoluția miniaturilor imaginilor

* **Tip**: Meniu derulant
* **Opțiuni**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Implicit**: Implicit (512 px)
* **Descriere**: Rezoluția (latura cea mai lungă, în pixeli) la care sunt redate miniaturile din grila de imagini. Valorile mai mari oferă o imagine mai clară la mărire, dar se încarcă mai lent și consumă mai multă memorie. Rezoluția completă corespunde dimensiunii originale a imaginii.
* **Notă**: Doar pentru afișare — această setare nu afectează niciodată procesarea sau fișierele exportate.***

## Detectarea țintelor

Aceste setări controlează modul în care Chloros detectează și procesează țintele de calibrare din imaginile dvs. Ambele sunt active doar atunci când este activată opțiunea **Calibrare prin reflectanță / balans de alb** (în caz contrar, acestea sunt estompate, deoarece detectarea țintelor este omisă în totalitate).

### Suprafața minimă a eșantionului de calibrare (px)

* **Tip**: Număr
* **Interval**: de la 0 la 10.000 de pixeli
* **Implicit**: 25 de pixeli
* **Descriere**: Stabilește suprafața minimă (în pixeli) necesară pentru ca o regiune detectată să fie considerată o probă validă de țintă de calibrare. Valorile mai mici vor detecta ținte mai mici, dar pot crește numărul de rezultate fals pozitive. Valorile mai mari necesită regiuni țintă mai mari și mai clare pentru detectare.
* **Când să reglați**:
  * Măriți valoarea dacă obțineți detectări eronate pe artefacte mici din imagine
  * Reduceți valoarea dacă țintele de calibrare apar mici în imagini și nu sunt detectate

### Gruparea minimă a țintelor (0-100)

* **Tip**: Număr
* **Interval**: 0 până la 100
* **Implicit**: 60
* **Descriere**: Controlează pragul de grupare pentru regiunile cu culori similare la detectarea țintelor de calibrare. Valorile mai mari necesită gruparea mai multor culori similare, ceea ce duce la o detectare mai conservatoare a țintelor. Valorile mai mici permit o variație mai mare a culorilor în cadrul unui grup de ținte.
* **Când să reglați**:
  * Măriți valoarea dacă țintele de calibrare sunt împărțite în mai multe detectări
  * Micșorați valoarea dacă țintele de calibrare cu variații de culoare nu sunt detectate în totalitate

***

## Procesare

Aceste setări controlează modul în care Chloros procesează și calibrează imaginile dvs.

### Corecția vignetării

* **Tip**: Casetă de selectare
* **Implicit**: Activat (bifat)
* **Descriere**: Aplică corecția vignetării pentru a compensa întunecarea produsă de obiectiv la marginile imaginilor. Vignetarea este un fenomen optic comun în care colțurile și marginile unei imagini apar mai întunecate decât centrul, din cauza caracteristicilor obiectivului.
* **Efect secundar**: Această opțiune selectează, de asemenea, ce *produs de rezervă necalibrat* va scrie o execuție (a se vedea mai jos).

### Calibrarea reflectanței / balansul de alb

* **Tip**: Casetă de selectare
* **Implicit**: Activat (bifat)
* **Descriere**: Activează calibrarea reflectanței — pe baza țintelor de calibrare detectate în cadru și/sau a datelor de lumină descendentă de la senzorul de lumină DAQ, în funcție de cameră și de ceea ce este disponibil. Aceasta normalizează valorile de reflectanță în întregul set de date și asigură măsurători consistente, indiferent de condițiile de iluminare.
* **Când este dezactivată**: Detectarea țintelor este omisă în totalitate, iar**niciun produs de reflectanță nu poate fi generat de nicio cameră** — fie că este vorba de Survey3 bazat pe ținte, fie de LATTICE bazat pe DAQ. Setările dependente (*Export reflectanță*, *Interval minim de recalibrare* și pragurile de detectare a țintelor) sunt estompate.

### Produse de rezervă necalibrate: Exportare răspuns senzor / Exportare corectată pentru vignetare

* **Tip**: Două casete de selectare
* **Implicit**: Ambele activate (bifate)
* **Descriere**: Când un cadru nu poate fi calibrat în funcție de reflectanță (nu s-a găsit nicio țintă de calibrare sau calibrarea în funcție de reflectanță este dezactivată), acesta este înregistrat ca un *produs de rezervă necalibrat*. **Există exact unul dintre cele două produse de rezervă pe fiecare execuție, pentru fiecare model de cameră**, ales prin comutatorul *Corecție vignetă*:
  * Corecția vignetării **activată**→ `Vignette_Corrected_Images/` (determinată de**Export cu vignetare corectată**)
  * Corecția vignetării **dezactivată**→ `Sensor_Response_Images/` (reglementată de**Exportare cu răspunsul senzorului**)
* Produsul de rezervă care nu este activat este afișat în gri. Debifarea celui activat împiedică complet scrierea acelui fișier.

### Produse de export LATTICE

Pentru proiectele care conțin capturi LATTICE, fiecare cadru LATTICE importat se ramifică către fiecare produs activat **și aplicabil**într-o singură etapă de procesare. Patru casete de selectare controlează această ramificare (toate sunt**activate** implicit):

| Setare | Folder de ieșire | Ce se exportă |
| --- | --- | --- |
| **Export debayered** | `Debayered_Images/` | Imaginea debayered liniar. Se aplică pentru RGB și camerele multispectrale. |
| **Exportare previzualizare** | `Preview_Images/` | Previzualizarea afișată. RGB = balans de alb (iluminant DAQ dacă este disponibil, altfel „gray-world”) + gamma; multispectral = extindere în culori false. |
| **Radianță de export** | `Radiance_Images/` | Radianță spectrală de tip Float32 în W/m²/sr/nm. Numai pentru multispectral (M3C/M3M) — nu se aplică master-urilor RGB. Se scrie întotdeauna ca TIFF pe 32 de biți, indiferent de setarea *Format imagine calibrată*. |
| **Reflectanță de export**| `Reflectance_Calibrated_Images/` | Reflectanță Uint16, scalată astfel încât**32768 = reflectanță 1,0** (marcată ca XMP `Chloros:PixelScale`). Numai pentru imagini multispectrale, înregistrat atunci când o înregistrare corespunzătoare de tip `.daq` de tip „downwelling” (sau o țintă din cadru care a trecut testul de control al calității) acoperă cadrul. |

* Camerele principale RGB emit date debayered + previzualizare; radianța/reflectanța sunt omise pentru acestea, deoarece nu sunt aplicabile.
* Adâncimea de biți a datelor debayered/previzualizare respectă setarea *Format imagine calibrată*; radianța este întotdeauna float32.
* Procesarea Survey3 nu este afectată de aceste patru comutatoare.

Aceleași patru comutatoare există fără prefix ca `chloros-cli process --debayered / --preview / --radiance / --reflectance` și ca parametrii corespunzători ai SDK. Acestea au înlocuit vechiul indicator `--radiometric-output`, care nu mai există.

{% hint style="warning" %}
**Dezactivarea tuturor produselor aplicabile duce la eșuarea rulării.** Începând cu versiunea 1.2.0, o execuție de procesare care a solicitat produse, dar nu a scris nicio imagine, raportează o eroare, iar CLI se închide cu un cod de ieșire diferit de zero, în loc să raporteze un succes tacit. Jurnalul menționează produsul pe care nu l-a putut scrie și motivul. O execuție deliberată numai cu metadate (fără nicio solicitare) este în continuare considerată un succes.
{% endhint %}

### Sursa de reflectanță (setare de proiect, configurată prin CLI/SDK)

Proiectul stochează, de asemenea, **referința de reflectanță** utilizată de produsul de reflectanță LATTICE. Nu există un control dedicat în panoul de setări; valoarea este stocată în configurația proiectului ca `Processing → "Target reflectance source"` și se setează cu `chloros-cli process --reflectance-source {auto,target,daq}` sau cu parametrul `reflectance_source` al SDK:

* **`auto`** (implicit): o țintă de calibrare în cadru care trece testul de control al calității (QA) devine referința absolută, revenindu-se la raportul de divizare a radiației descendente din DAQ (ρ = πL/E) atunci când nu este prezentă nicio țintă sau când testul de control al calității eșuează.
* **`target`**: reflectanță strict determinată de țintă — fără substituție DAQ.
* **`daq`**: reflectanță determinată de DAQ; țintele din cadru nu sunt utilizate ca referință.

Valoarea stocată este comparată fără a se ține cont de majuscule și minuscule, iar câteva variante ortografice sunt acceptate ca aliasuri: `target`, `target_image`, `empirical` și `empirical_line` înseamnă toate **țintă**; `daq`, `dls`, `light_sensor` și `sensor` înseamnă toate**daq**. Orice altceva — inclusiv o cheie absentă — se interpretează ca**auto**.

Scanările țintă **măsurate** pe unitate sunt căutate după numărul de serie/QR al unității țintă, ca `<serial>.csv`, în trei locuri: directorul indicat cu `--target-reflectance-dir` (stocat ca `Processing → "Target reflectance dir"`), în propriul folder al proiectului `target_reflectance/` și în calea din variabila de mediu `CHLOROS_TARGET_REFLECTANCE_DIR`. Când nu există nicio scanare măsurată pentru unitatea respectivă, se utilizează în schimb curba nominală publicată pentru modelul țintă.

### Metoda de demosaicare

* **Tip**: Selecție din meniu derulant
* **Opțiuni**:
  * Standard (Rapid, Calitate medie)
  * Texture Aware (Lent, Calitate maximă) \[Chloros+]
* **Implicit**: Standard (Rapid, calitate medie)
* **Descriere**: Selectează algoritmul de demosaic utilizat pentru a converti datele brute ale senzorului cu model Bayer în imagini color. Metoda „Standard (Rapid, calitate medie)” oferă un echilibru optim între viteza de procesare și calitatea imaginii. Metoda „Sensibilă la textură (lent, calitate maximă)” \[Chloros+] utilizează un algoritm de demosaicare de înaltă calitate, sensibil la contururi, combinat cu un model de reducere a zgomotului bazat pe AI/ML, care elimină aproape tot zgomotul generat de procesul de demosaicare. Modelul „Texture Aware” necesită memorie GPU (VRAM) pentru a rula. Vă recomandăm să îl utilizați atunci când aveți la dispoziție &gt;4 GB VRAM pentru o procesare mai rapidă.
* **Atunci când rândul este un meniu derulant**: meniul derulant cu două opțiuni apare numai când**ambele**condiții sunt îndeplinite — sunteți conectat cu un abonament Chloros+ eligibil,**și** proiectul nu conține capturi LATTICE. În caz contrar, rândul se afișează ca text simplu cu textul „`Standard (Fast, Medium Quality)`”, fără opțiuni de selectare.
* **Notă privind LATTICE**: Nu există un model Texture Aware antrenat pentru LATTICE, iar fluxul de lucru impune demosaicarea standard pentru cadrele LATTICE, indiferent de valoarea stocată. Dacă adăugați un folder LATTICE într-un proiect în care opțiunea Texture Aware era deja selectată, Chloros rescrie setarea la Standard, în loc să păstreze o valoare învechită în `project.json`.

### Interval minim de recalibrare

* **Tip**: Număr
* **Interval**: de la 0 la 3.600 de secunde
* **Implicit**: 0 secunde
* **Descriere**: Stabilește intervalul minim de timp (în secunde) între utilizarea țintelor de calibrare. Când este setat la 0, Chloros va utiliza fiecare țintă de calibrare detectată. Când este setat la o valoare mai mare, Chloros va utiliza doar țintele de calibrare care sunt separate de cel puțin acest număr de secunde, reducând timpul de procesare pentru seturile de date cu capturi frecvente ale țintelor de calibrare.
* **Când se ajustează**:
  * Setați la 0 pentru o precizie maximă a calibrării atunci când condițiile de iluminare variază
  * Măriți valoarea (de exemplu, la 60–300 de secunde) pentru o procesare mai rapidă atunci când iluminarea este constantă și aveți imagini frecvente ale țintelor de calibrare

### Decalajul fusului orar al senzorului de lumină

* **Tip**: Număr
* **Interval**: de la -12 la +12 ore
* **Implicit**: 0 ore
* **Descriere**: Specifică decalajul de fus orar (în ore față de UTC) pentru marcajele de timp ale datelor senzorului de lumină, utilizat la corelarea jurnalelor senzorului de lumină cu orele de captare a imaginilor. Înregistrările `.daq` mai recente conțin propriul fus orar, astfel încât această opțiune este necesară în principal pentru jurnalele mai vechi înregistrate în ora locală.

### Aplicare corecții PPK

* **Tip**: Casetă de selectare
* **Implicit**: Dezactivat (nebifat)
* **Descriere**: Activează utilizarea corecțiilor cinematice post-procesate (PPK) de la înregistratoarele DAQ MAPIR care conțin un GPS (GNSS). Când este activată, Chloros va utiliza orice fișiere jurnal .daq care conțin date privind pinul de expunere din directorul proiectului dvs. și va aplica corecții precise de geolocalizare imaginilor dvs.
* **Cerință**: În directorul proiectului trebuie să existe un fișier jurnal .daq cu intrări privind pinul de expunere
* **Când să activați**: Se recomandă activarea permanentă a corecției PPK dacă aveți intrări de feedback privind expunerea în fișierul jurnal .daq.

### Pinul de expunere 1

* **Tip**: Selecție din meniu derulant
* **Vizibilitate**: Vizibilă numai când opțiunea „Aplică corecții PPK” este activată ȘI sunt disponibile date de expunere pentru Pinul 1
* **Opțiuni**:
  * Numele modelelor de camere detectate în proiect
  * „Nu utiliza” – Ignoră acest pin de expunere
* **Implicit**: Selectat automat în funcție de configurația proiectului
* **Descriere**: Alocă o cameră specifică pinului de expunere 1 pentru sincronizarea temporală PPK. Pinul de expunere înregistrează momentul exact în care se declanșează obturatorul camerei, ceea ce este esențial pentru o geolocalizare PPK precisă.
* **Comportamentul selecției automate**:
  * O singură cameră + un singur pin: Selectează automat camera
  * O singură cameră + două pini: Pinul 1 este atribuit automat camerei
  * Mai multe camere: Este necesară selectarea manuală

### Pinul de expunere 2

* **Tip**: Selecție din meniu derulant
* **Vizibilitate**: Vizibilă numai când opțiunea „Aplică corecții PPK” este activată ȘI sunt disponibile date de expunere pentru pinul 2
* **Opțiuni**:
  * Denumirile modelelor de camere detectate în proiect
  * „Nu utiliza” – Ignoră acest pin de expunere
* **Implicit**: Selectat automat în funcție de configurația proiectului
* **Descriere**: Alocă o cameră specifică pinului de expunere 2 pentru sincronizarea temporală PPK atunci când se utilizează o configurație cu două camere.
* **Comportament de selectare automată**:
  * O singură cameră + un singur pin: Pinul 2 este setat automat la „Nu utilizați”
  * O singură cameră + doi pini: Pinul 2 este setat automat la „Nu utilizați”
  * Mai multe camere: Este necesară selectarea manuală
* **Notă**: Aceeași cameră nu poate fi atribuită simultan atât pinului 1, cât și pinului 2.***

## Senzor de lumină DAQ

Această secțiune apare în Setările proiectului și listează fiecare fișier DAQ de lumină descendentă din proiect — înregistrările `.daq` și jurnalele de lumină descendentă DAQ-M `.csv`. Înregistrările realizate în fila „Senzori de lumină” sunt adăugate automat la proiectul deschis.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Fiecare rând afișează fișierul, modelul senzorului și corecția capacului difuzorului aplicabilă efectiv pentru acel fișier. Deasupra rândurilor se află un singur element de control valabil pentru întregul proiect:

### Suprascrierea capacului (toate fișierele)

* **Tip**: Selecție din meniu derulant
* **Opțiuni**: `Auto` plus profilurile de corecție a capacului valabile pentru tipurile de senzori prezenți în proiect
* **Implicit**: Auto
* **Salvat ca**: `Processing → "DAQ cap id"` (implicit `auto`)
* **Descriere**: `Auto` utilizează capacitatea înregistrată pentru fiecare fișier (se presupune capacitatea de soare acolo unde nu s-a înregistrat nimic — toate dispozitivele de achiziție de date MAPIR sunt livrate cu corectorul de soare). Alegerea unui capac specific are prioritate asupra**tuturor** fișierelor de radiație descendentă din proiect: înregistrările brute sunt corectate cu acesta, iar înregistrările care au deja o limită sunt re-referențiate (corecția înregistrată este anulată și se aplică cea selectată).
* **Important**: Limita selectată trebuie să corespundă cu cea montată fizic în timpul înregistrării. Nici senzorul, nici software-ul nu pot detecta limita fizică — un ID de limită necorespunzător corectează greșit spectrele.

Există în mod deliberat **un singur** control la nivel de proiect, în loc de meniuri derulante pentru fiecare fișier: setarea se aplică fiecărei surse de radiație descendentă din proiect.***

## Alinierea matricei

Această secțiune apare **numai** atunci când cel puțin o imagine din proiect conține transformarea de aliniere modul-la-modul pe care matricile LATTICE o aplică în momentul captării (etichete XMP `Chloros:Alignment*`). Aceasta arată câte imagini conțin etichete de aliniere, care cameră este de referință (insignă `REF` ), precum și un tabel cu numărul de imagini pentru fiecare cameră.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Aplicare aliniere matrice

* **Tip**: Casetă de selectare
* **Implicit**: Activat (bifat)
* **Stocat ca**: `Processing → "Array alignment"`
* **Descriere**: Deformează fiecare produs procesat (debayered / previzualizare / radianță / reflectanță / indice) în geometria de referință comună a matricei, utilizând transformarea înregistrată la captură. Dezactivat = export în geometria nativă specifică fiecărui senzor.

### Decupare la suprapunerea comună

* **Tip**: Casetă de selectare (activă numai când opțiunea *Aplicare aliniere matrice* este activată)
* **Implicit**: Activată (bifată)
* **Stocată ca**: `Processing → "Array alignment crop"`
* **Descriere**: Decupează exporturile aliniate la regiunea comună tuturor modulelor camerei, astfel încât fiecare bandă să aibă aceeași amprentă. Dezactivat păstrează întreaga suprafață a senzorului (umplere neagră în afara sursei).

### Reeșantionare

* **Tip**: Meniu derulant (activ numai când opțiunea *Aplică alinierea matricei* este activată)
* **Opțiuni**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Implicit**: Biliniar
* **Stocat ca**: `Processing → "Array alignment interpolation"`
* **Descriere**: Interpolarea utilizată de transformarea de aliniere. Opțiunea „Cel mai apropiat” păstrează valorile exacte ale sursei (fără amestecare între pixeli) pentru o analiză radiometrică strictă; opțiunea „Bilinear” este cea mai potrivită pentru cartografiere și utilizare vizuală.

Aceleași trei opțiuni există și fără prefix, sub formele `chloros-cli process --array-alignment`, `--array-alignment-crop` și `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Aceste setări vă permit să configurați indici multispectrali pentru analiză și vizualizare.

### Adăugare indice

* **Tip**: Panou special de configurare a indicilor
* **Descriere**: Deschide un panou interactiv în care puteți selecta și configura indici multispectrali de vegetație (NDVI, NDRE, EVI, etc.) care să fie calculați în timpul procesării imaginii. Puteți adăuga mai mulți indici, fiecare cu propriile setări de vizualizare.
* **Indici disponibili**: Meniul derulant al interfeței grafice include**27** de formule de indici multispectrali predefinite (consultați [Formule de indici multispectrali](multispectral-index-formulas.md) pentru lista completă, inclusiv denumirile acceptate de opțiunea `--indices`).
* **Funcționalități**:
  * Selectați dintre formulele de indici predefinite
  * Trageți canalele de filtru ale camerei dvs. în sloturile de bandă ale formulei
  * Configurați gradientele de culoare pentru vizualizare (LUT – tabele de consultare)
  * Setați valorile pragului și modurile de decupare
  * Creați formule de indice personalizate
* **Notă**: Indicii nu sunt calculați pentru camerele monocrome LATTICE M3M cu o singură bandă — indicii multibandă sunt nedefiniti pe o singură bandă. Survey3 și LATTICE M3C nu sunt afectate.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Fiecare indice pe care îl adăugați este redat sub formă de expresie matematică, cu un cerc colorat pentru fiecare slot de bandă: roșu = Red, verde = Green, albastru = Blue, portocaliu = Orange, cyan = Cyan, violet = NIR, magenta = RE. Trageți un cerc din rândul de deasupra formulei pe un slot pentru a-l lega; faceți dublu-pe un slot legat pentru a-l șterge. Indicele se calculează o singură dată după ce fiecare slot utilizat de formulă are un canal.

### Formule personalizate (Funcționalitate Chloros+)

* **Tip**: Matrice de definiții de formule personalizate
* **Disponibilitate**: Necesită autentificarea cu un abonament Chloros+ eligibil.
* **Descriere**: Vă permite să creați și să salvați formule personalizate de indici multispectrali folosind operații matematice între benzi. Formulele personalizate sunt salvate împreună cu setările proiectului și pot fi utilizate la fel ca indicii încorporați.
* **Cum se creează**:
  1. În panoul de configurare a indicilor, deschideți calculatorul de formule personalizate
  2. Scrieți formula folosind **simbolurile de bandă-slot**, nu numele benzilor
  3. Salvați formula cu un nume descriptiv — aceasta va apărea apoi în partea de jos a meniului derulant pentru formule, iar dvs. trageți cercurile canalelor camerei dvs. pe sloturile respective, exact ca în cazul unei presetări încorporate
* **Sintaxa formulei**:
  * Sloturi de bandă: `x`, `y`, `z`, `a`, `b`, `c` — șase poziții pe care le asociați canalelor reale prin glisare
  * Operatori: `+`, `-`, `*`, `/`, `^` și `()` pentru grupare
  * Funcții: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **De ce simboluri, nu nume de formații**: o formulă scrisă ca `(y-x)/(y+x)` funcționează pe orice cameră, deoarece maparea prin glisare și plasare decide dacă `y` este NIR de 850 nm al unui filtru RGN sau de 808 nm NIR al unui filtru OCN. Presetările încorporate sunt stocate în același mod — consultați [Formule index multispectrale](multispectral-index-formulas.md) pentru forma exactă a simbolurilor tuturor celor 27.
* **Unde funcționează**: formulele personalizate sunt salvate odată cu setările proiectului și pot fi utilizate atât în [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md), cât și în procesare. Acestea**nu** sunt acceptate de lista de nume CLI/SDK `--indices`, care doar extinde cele 22 de nume de presetări încorporate.***

## Export

Aceste setări controlează formatul și calitatea imaginilor procesate exportate.

### Format imagine calibrată

* **Tip**: Selecție din meniu derulant
* **Opțiuni**:
  * **TIFF (16 biți)** – Format TIFF necomprimat de 16 biți
  * **TIFF (32 de biți, procent)** – TIFF de 32 de biți cu virgulă mobilă, cu valori de reflectanță exprimate în procente
  * **PNG (8 biți)** - Format PNG comprimat pe 8 biți
  * **JPG (8 biți)** - Format JPEG comprimat pe 8 biți
* **Implicit**: TIFF (16 biți)
* **Descriere**: Selectează formatul de fișier pentru salvarea imaginilor procesate și calibrate. Fișierele exportate sunt stocate într-un subfolder specific fiecărui format, în interiorul folderului fiecărei camere (`tiff16`, `tiff32`, `png8`, `jpg8`), cu câte un dosar `<Product>_Images/` pentru fiecare produs. Fișierele exportate păstrează numele fișierului sursă — produsul este identificat de dosar, nu de sufixul numelui fișierului.
* **Recomandări privind formatul**:
  * **TIFF (16 biți)**: Recomandat pentru analize științifice și fluxuri de lucru profesionale. Păstrează calitatea maximă a datelor, fără artefacte de compresie. Ideal pentru analiza multispectrală și prelucrarea ulterioară în software GIS.
  * **TIFF (32 de biți, procent)**: Ideal pentru fluxuri de lucru care necesită valori de reflectanță exprimate în procente (0-100%). Oferă precizie maximă pentru măsurătorile radiometrice.
  * **PNG (8 biți)**: Potrivit pentru vizualizare pe web și vizualizare generală. Fișiere de dimensiuni mai mici cu compresie fără pierderi, dar cu o gamă dinamică redusă.
  * **JPG (8 biți)**: Dimensiuni de fișier minime, ideal doar pentru previzualizări și afișare pe web. Utilizează compresie cu pierderi, care nu este potrivită pentru analize științifice.
* **Notă**: Radianța LATTICE este întotdeauna exportată ca TIFF cu 32 de biți în format float, indiferent de această setare.***

## Salvare șablon de proiect

Această funcție vă permite să salvați setările curente ale proiectului ca șablon reutilizabil.

* **Tip**: Câmp de introducere text + buton Salvare
* **Descriere**: Introduceți un nume descriptiv pentru șablonul de setări și faceți clic pe pictograma de salvare. Șablonul va stoca toate setările curente ale proiectului (detectarea țintelor, opțiunile de procesare, indicii și formatul de export) pentru a fi reutilizate cu ușurință în proiectele viitoare. Șabloanele sunt stocate în folderul `Project Templates/` din folderul de salvare al proiectului dvs. și pot fi, de asemenea, selectate sau exportate din meniul principal (*Selectare șablon* / *Salvare șablon* / *Exportare șablon*).
* **Cazuri de utilizare**:
  * Creați șabloane pentru diferite sisteme de camere (RGB, multispectrale, NIR)
  * Salvați configurații standard pentru tipuri specifice de culturi sau fluxuri de lucru de analiză
  * Partajați setări uniforme în cadrul echipei
* **Mod de utilizare**:
  1. Configurați toate setările dorite pentru proiect
  2. Introduceți un nume pentru șablon (de exemplu, „RedEdge Survey3 NDVI Standard”)
  3. Faceți clic pe pictograma de salvare
  4. Șablonul poate fi acum încărcat la crearea de proiecte noi

***

## Salvare folder proiect

Această setare specifică locația în care sunt salvate în mod implicit proiectele noi.

* **Tip**: Afișarea căii către director + butonul „Editare”
* **Implicit (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Implicit (Linux)**: `~/Chloros Projects`
* **Descriere**: Afișează directorul implicit curent în care sunt create proiectele Chloros noi. Faceți clic pe pictograma de editare pentru a selecta un alt director. Modificarea este stocată sub forma unei singure linii de text în `~/.chloros/working_directory.txt` — în Windows, aceasta este `C:\Users\<Username>\.chloros\working_directory.txt`. Dacă acel fișier lipsește sau indică o cale care nu mai există, Chloros revine la setarea implicită de mai sus. Fișierul CLI citește și scrie același fișier, astfel încât `chloros-cli` și interfața grafică (GUI) sunt întotdeauna în concordanță cu privire la locația proiectelor.
* **Șabloanele de proiect** se află într-un subfolder `Project Templates/` din acest director.
* **Când să modificați**:
  * Setați o unitate de rețea pentru colaborarea în echipă
  * Schimbați pe o unitate cu mai mult spațiu de stocare pentru seturi de date mari
  * Organizați proiectele pe ani, clienți sau tipuri de proiecte în dosare diferite
* **Notă**: Modificarea acestei setări afectează doar proiectele NOI. Proiectele existente rămân în locațiile lor inițiale.***

## Persistența setărilor

Un proiect Chloros este un **dosar**. Toate setările proiectului sunt salvate în `project.json` din interiorul acestuia; hardware-ul conectat este memorat împreună cu acesta în `cameras.json` și `sensors.json`, astfel încât redeschiderea unui proiect reconectează și camerele și senzorii de lumină ai acestuia. Când redeschideți un proiect, toate setările sunt restabilite exact așa cum le-ați lăsat. Proiectele salvate pot fi, de asemenea, controlate fără interfață grafică cu ajutorul fișierului `chloros-cli project` sau al fișierului SDK`open_project`.

### Ierarhia setărilor

Setările sunt aplicate în următoarea ordine:

1. **Setări implicite de sistem** – Setări implicite încorporate, definite de Chloros
2. **Setări șablon** – Dacă încărcați un șablon la crearea unui proiect
3. **Setări proiect salvate** – Setări salvate împreună cu fișierul proiectului
4. **Ajustări manuale** – Orice modificări pe care le efectuați în timpul sesiunii curente

### Setări și procesarea imaginilor

Setările de procesare sunt citite la începerea unei sesiuni de procesare. Modificarea unei setări nu afectează retroactiv produsele care se află deja pe disc — rulați din nou procesarea pentru a aplica noile setări. Câteva setări nu afectează deloc procesarea:

* Rezoluția miniaturilor imaginilor (doar pentru afișare)
* Salvare șablon proiect
* Salvare folder proiect

***

## Referință chei de configurare

Pentru automatizare (CLI `--config`, SDK `configure` sau citirea directă a `project.json` direct), acestea sunt cheile exacte din cadrul `Project Settings`:

| Calea cheii | Tip | Implicit |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | număr între 0 și 10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | număr 0-100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | număr 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | număr -12..12 | `0` |
| `Processing → Apply PPK corrections` | boolean | `false` |
| `Processing → DAQ cap id` | ID profil de limită sau `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lista configurațiilor indexului | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Cheile `Array alignment` sunt scrise la prima afișare a secțiunii „Array Alignment” sau atunci când un apel de automatizare le setează. Atât timp cât acestea lipsesc, pipeline-ul utilizează aceleași valori prezentate mai sus (`true`, `true`, bilineare), astfel încât un proiect.json fără acestea se comportă identic cu unul care le conține.

### Chei stocate în `project.json` fără control în panoul de setări

Acestea se află în același arbore `Project Settings` și sunt citite în timpul procesării, dar nu veți găsi un widget pentru ele în bara laterală:

| Calea cheii | Tip | Implicit | Setat de |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Suprascrie modul în care sunt interpretate fișierele TIFF de intrare LATTICE; `auto` deduce din eticheta XMP a fiecărui fișier `Chloros:ProcessingLevel`, plus numărul de canale. Ignorat pentru capturile Survey3 `.raw`. Nu este în mod deliberat o setare din interfața grafică — opțiunea „auto” este corectă în toate cazurile normale. |
| `Processing → Target reflectance dir` | șir de caractere pentru cale | `""` | `chloros-cli process --target-reflectance-dir`, sau ținta proiectului API |
| `Processing → Target reflectance config` | dicționar indexat după numărul de serie al camerei | `{}` | Înregistrarea unei ținte în cadru (mod `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | șir de caractere reprezentând calea | `""` | SDK `process_folder(daq_log_path=…)`. Indică o înregistrare `.daq` sau un dosar care le conține |
| `Target Detection → Minimum calibration target squares` | număr | `4` | Valoare implicită veche; fără control și fără indicator CLI |
| `UI → Grid thumbnail size` | număr | `160` | Glisorul de zoom al miniaturilor din grila de imagini |

Două preferințe ale vizualizatorului sunt stocate **la nivel superior în `project.json`**, în afara `Project Settings`, deoarece reprezintă starea de afișare, nu setări de procesare:

| Cale cheie | Tip | Implicit | Setat de |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | număr întreg 1–256 | `1` | Controlul GSD (px) — a se vedea [Deschiderea unei imagini pe ecran complet](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Cele mai bune practici

1. **Începeți cu setările implicite**: Setările implicite funcționează bine pentru majoritatea sistemelor de camere MAPIR și pentru fluxurile de lucru tipice.
2. **Creați șabloane**: După ce ați optimizat setările pentru un anumit flux de lucru sau o anumită cameră, salvați-le ca șablon pentru a asigura consecvența între proiecte.
3. **Testați înainte de procesarea completă**: Când experimentați cu setări noi, testați-le pe un subset mic de imagini înainte de a procesa întregul set de date.
4. **Documentați-vă setările**: Folosiți nume descriptive pentru șabloane care să indice sistemul de camere, tipul de procesare și utilizarea prevăzută (de exemplu, „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Selectarea formatului de export**: Alegeți formatul de export în funcție de utilizarea finală:
   * Analiză științifică → TIFF (16 biți sau 32 biți)
   * Prelucrare GIS → TIFF (16 biți)
   * Vizualizare rapidă → PNG (8 biți)
   * Partajare web → JPG (8 biți)

***

Pentru mai multe informații despre indicii multispectrali din Chloros, consultați pagina [Formule pentru indici multispectrali](multispectral-index-formulas.md).
