# Finalizarea procesării

Odată ce Chloros finalizează procesarea, este momentul să analizați rezultatele, să verificați calitatea ieșirii și să pregătiți imaginile procesate pentru a le utiliza în fluxul dvs. de lucru. Această pagină vă ghidează prin pașii finali și acțiunile următoare.

## Indicatori ai finalizării procesării

Când procesarea se finalizează cu succes, veți vedea mai mulți indicatori:

* ✅ **Bara de progres**: ajunge la 100% finalizare
* ✅ **Jurnal de depanare**: Afișează ultimele linii `[RUN-SUMMARY]` împreună cu numărul de elemente (imagini, grupuri de camere, ținte, imagini calibrate, fișiere salvate)
* ✅ **Butonul Start**: Devine din nou activ (gata pentru următoarea execuție de procesare)
* ✅ **Fișiere de ieșire**: Toate imaginile procesate sunt salvate în arborele de ieșire al proiectului (mai jos)

{% hint style="warning" %}
**O execuție care nu scrie nicio imagine este considerată eșuată.** Dacă ați solicitat produse imagistice și ciclul nu a scris niciuna, Chloros raportează acest lucru ca un eșec — `[RUN-SUMMARY]` sugerează în numele jurnalului cauza probabilă (nimic importat, nicio țintă detectată sau fiecare produs solicitat omis ca fiind inaplicabil). Echivalentul CLI se încheie cu un cod de ieșire diferit de zero. O execuție deliberată numai cu metadate (toate produsele de export dezactivate, fără indici) este totuși considerată reușită. Consultați [Referința CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localizarea imaginilor procesate

### Deschiderea folderului de ieșire

1. Faceți clic pe pictograma **Meniu principal** d<img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (stânga sus)
2. Selectați **„Deschideți folderul proiectului”**

3. Explorerul de fișiere se deschide în directorul proiectului
4. Localizați proiectul după nume

### Arborele de ieșire

Produsele sunt salvate **în folderul proiectului, grupate după cameră și apoi după formatul fișierului**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Dosarul camerei**: `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE (care corespunde cu EXIF-ul capturii `Model`), `<model>_<filter>` pentru Survey3 (de exemplu, `Survey3N_RGN`). Două camere care împart același senzor și filtru, dar au obiective diferite, păstrează arbori separați — vignetarea, câmpul vizual și distorsiunea diferă.
* **Dosarul format**: respectă setarea formatului de export — `tiff16`, `tiff8`, `png8`, `jpg8` sau `tiff32` pentru TIFF (32 de biți, procent). Radianța este întotdeauna de tip float32 și se încadrează întotdeauna sub `tiff32`.
* **Dosare de produse**:
  * `Reflectance_Calibrated_Images/` — reflectanță calibrată
  * `Debayered_Images/` — debayering liniar (LATTICE)
  * `Preview_Images/` — previzualizare pe ecran (LATTICE)
  * `Radiance_Images/` — radianță spectrală de tip float32, W/m²/sr/nm (LATTICE multispectral)
  * `Vignette_Corrected_Images/` **sau** `Sensor_Response_Images/` — opțiunea de rezervă necalibrată pentru cadre fără referință de reflectanță; există exact una dintre cele două opțiuni pe fiecare execuție, aleasă de setarea de corecție a vignetării
  * `<INDEX>_Index_Images/` — un folder pentru fiecare index selectat (de ex. `NDVI_Index_Images`)

{% hint style="info" %}
**Fiecare produs exportat păstrează numele fișierului SURSĂ.**Un export de radianță al fișierului `capture_..._raw.tif` se numește în continuare `capture_..._raw.tif` — doar că se află în `tiff32/Radiance_Images/`.**Folderul identifică produsul, nu numele fișierului**, așa că, dacă căutați `*radiance*.tif`, nu veți găsi nimic; căutați în schimb după director.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Câte fișiere ar trebui să existe?

Nu numărați după o formulă — numărul de fișiere de ieșire depinde de produsele care au fost activate și de cele aplicabile fiecărei camere (de exemplu, camerele RGB nu primesc date de radianță/reflectanță). Numărul oficial se găsește în jurnal: ultima linie `[RUN-SUMMARY]` indică exact câte fișiere au fost scrise, iar liniile explicative detaliază tot ce a fost omis.

***

## Vizualizarea imaginilor procesate

### Previzualizare rapidă în File Explorer

**Previzualizare integrată Windows:**

1. Navigați într-un dosar de produs (de ex. `tiff16/Reflectance_Calibrated_Images/`)
2. Selectați un fișier imagine
3. Previzualizarea apare în panoul de previzualizare al Explorer-ului Windows
4. Folosiți tastele săgeată pentru a naviga printre imagini

### Previzualizare în vizualizatoare de imagini externe

**Vizualizatoare recomandate:*** **QGIS** - Software GIS gratuit (cel mai potrivit pentru analiza multispectrală georeferențiată)
* **IrfanView** - Vizualizator de imagini rapid și ușor (suportă TIFF)
* **Adobe Photoshop** – Editare profesională (suportă TIFF)
* **GIMP** – Alternativă gratuită la Photoshop
* **Windows Photos** - Vizualizare de bază (este posibil să nu suporte formatul TIFF pe 16 biți)

### Previzualizare în vizualizatorul de imagini Chloros

Utilizați vizualizatorul de imagini încorporat în Chloros pentru vizualizare avansată:

1. Faceți clic pe o miniatură a imaginii în File Browser
2. Imaginea se deschide în zona principală de previzualizare
3. Faceți clic pe fila **Image Viewer** d<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> din bara laterală din stânga
4. Utilizați [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) pentru analiză interactivă

Consultați [Vizualizatorul de imagini](../image-viewer-gui/opening-an-image-full-screen.md) pentru instrucțiuni detaliate.

***

## Citirea valorilor pixelilor de reflectanță (GIS / Pix4D / Scripturi)

Reflectanța este stocată ca număr întreg DN, iar **valoarea DN care corespunde lui ρ = 1,0 depinde de camera sursă**:

| Sursă          | ρ = 1,0 este | Cum se poate determina                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (marjă de până la ρ 2,0) | Eticheta XMP `Chloros:PixelScale=32768` este înscrisă în fișier |
| Survey3         | **65535** (tăiat la ρ 1,0)     | Nu există etichete XMP de tip `Chloros:*` — această absență este semnalul |

**Citiți eticheta `Chloros:PixelScale` și împărțiți la valoarea acesteia**, în loc să presupuneți o valoare generală de 65535 — împărțirea reflectanței LATTICE la 65535 înjumătățește în mod tacit fiecare valoare. Un caz extrem nu conține nicio scală prin design: o captură cu sursă de 8 biți scrisă ca ieșire de 8 biți este decupată, nu redimensionată, și nu primește în mod deliberat nicio etichetă de scală — reexportați la 16 biți sau 32 de biți în loc să o împărțiți. Consultați [Formate de imagine de ieșire](../output-image-formats.md) pentru detalii complete.***

## Metadatele preluate în exporturi

Fiecare produs păstrează **blocul GPS**al capturii sursă și**sub-IFD-ul EXIF** al acesteia, astfel încât un
export conține `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` și
`CameraSerialNumber`, precum și georeferențierea.

{% hint style="warning" %}
**Dacă un ortomozaic are o scară absurdă, verificați mai întâi `FocalLength`.**
Pix4D calculează distanța dintre eșantioanele la sol pe baza distanței focale și a altitudinii. Fără această etichetă,
se revine la o scară extrem de eronată — într-un zbor măsurat cu 49 de capturi, o plantație de portocali de 411 m × 160 m
a fost reconstruită ca având dimensiunile de 47,8 km × 13 km, generând o ortografie de 455 de megapixeli alcătuită în mare parte din
spațiu gol. Asamblarea lentă a segmentelor și un fișier neașteptat de mare sunt simptome ale acestei probleme, nu probleme
separate.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nu sunt copiate *toate* etichetele. Etichetele structurale ale IFD0 sunt omise în mod deliberat (copierea
lora corupe ieșirea LATTICE), iar `ExifImageWidth` / `ExifImageHeight` sunt excluse
deoarece descriu captura originală — un export redimensionat ar revendica altfel
dimensiuni cu care propria sa rasterizare intră în contradicție.

***

## Analizarea jurnalului de depanare

### Verificați dacă există avertismente sau erori

1. Deschideți **Jurnalul de depanare** din fila<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Derulați mesajele
3. Căutați avertismentele galbene sau erorile roșii
4. Citiți liniile `[RUN-SUMMARY]` și eventualele indicii
5. Contactați asistența MAPIR pentru ajutor

### Salvarea jurnalului

Pentru a păstra o evidență a procesării sau pentru a-l trimite serviciului de asistență MAPIR:

1. Faceți clic pe butonul **„Copiere”**sau**„Descărcare”**

2. Salvați ca fișier text în folderul proiectului
3. Includeți-l în documentația proiectului
4. Trimiteți-l serviciului de asistență MAPIR dacă întâmpinați probleme

***

## Probleme comune legate de ieșire și soluții

### Problemă: Fișiere de ieșire lipsă

**Cauze posibile:**

* Produsul nu se aplică acelei camere (de exemplu, radianța/reflectanța pentru camerele RGB — așa se menționează în jurnal)
* Lipsea o referință necesară (de exemplu, reflectanță fără țintă și fără radiație descendentă `.daq`)
* Caseta de selectare pentru exportul produsului era dezactivată în Setările proiectului
* S-a epuizat spațiul pe disc în timpul exportului

**Soluții:**

1. Verificați indicațiile `[RUN-SUMMARY]` și liniile `[EXPORT-CHECK]` din jurnalul de depanare — acestea explică omisiunile specifice fiecărei camere
2. Verificați casetele de selectare ale produselor de export din [Setări proiect](adjusting-project-settings.md)
3. Verificați dacă spațiul pe disc a fost suficient
4. Repetați procesarea după remedierea cauzei

### Problemă: Margini întunecate sau luminoase (vignetarea este încă vizibilă)

**Cauze posibile:**

* Corecția vignetării este dezactivată
* Camera/obiectivul nu se află în baza de date a profilurilor Chloros
* Vignetare extremă, care depășește capacitatea de corecție

**Soluții:**

1. Verificați dacă corecția vignetării a fost activată în Setările proiectului
2. Verificați dacă modelul camerei a fost detectat corect
3. Contactați serviciul de asistență MAPIR dacă vignetarea persistă

### Problemă: Culori sau valori incorecte

**Cauze posibile:**

* Nu s-au detectat ținte de calibrare
* A fost selectat un model greșit de țintă de calibrare
* Calibrarea reflectanței este dezactivată
* Imagini ale țintelor de calitate slabă

**Soluții:**

1. Verificați dacă calibrarea reflectanței a fost activată
2. Verificați mesajele „Țintă găsită” din jurnalul de depanare
3. Verificați calitatea imaginii țintei
4. Reprocesați cu țintele corespunzătoare marcate

### Problemă: Valorile NDVI par incorecte

**Intervale așteptate pentru NDVI:*** **Apă, roci, sol**: de la -0,1 la 0,2
* **Vegetație rară/nesănătoasă**: de la 0,2 la 0,4
* **Vegetație moderată**: de la 0,4 la 0,6
* **Vegetație sănătoasă și densă**: de la 0,6 la 0,9**Dacă valorile se află în afara acestor intervale:**

1. Verificați dacă s-a aplicat calibrarea reflectanței
2. Verificați dacă a fost inclus jurnalul senzorului de lumină
3. Verificați dacă au fost detectate țintele de calibrare
4. Asigurați-vă că a fost detectat modelul corect de cameră
5. Verificați momentul și condițiile de captare a imaginii țintă
6. Dacă calculați singuri indicii pe baza fișierelor de reflectanță, confirmați că ați împărțit la valoarea `Chloros:PixelScale` a fișierului (vezi mai sus)

***

## Utilizarea imaginilor procesate

### Pentru fotogrammetrie / crearea ortomozaicelor

**Flux de lucru recomandat:**

1.**Importați imaginile de reflectanță calibrate** în software-ul de fotogrammetrie:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Păstrați metadatele EXIF**: Asigurați-vă că datele GPS sunt păstrate pentru geotagare
3. **Fluxuri de lucru calibrate**: Utilizați imagini de reflectanță pentru precizie științifică — reflectanța LATTICE conține etichetele de calibrare XMP citite de Pix4D
4. **Prelucrați mozaicurile index**: Creați ortomozaicuri NDVI din imagini indexate individuale
5. **Exportați imagini GeoTIFF georeferențiate**: Pentru utilizare în aplicații GIS

### Pentru analiza GIS

**Flux de lucru recomandat:**

1.**Încărcați în QGIS, ArcGIS sau un program similar**

2.**Utilizați imagini de reflectanță TIFF de 16 biți** pentru analiza multibandă (împărțiți la `Chloros:PixelScale` al fișierului)
3. **Utilizați imaginile index** (NDVI, NDRE) ca straturi de vegetație gata de utilizare
4. **Calculator raster**: Combinați benzile pentru analize personalizate
5. **Export**: Creați hărți de clasificare, detectarea schimbărilor, hărți ale stării de sănătate a vegetației

### Pentru analiză directă / raportare

**Flux de lucru recomandat:**

1.**Utilizați imagini index cu culori LUT** pentru rapoarte vizuale
2. **Extrageți statistici**: Media NDVI pe câmp/parcelă
3. **Seriile temporale**: Comparați indicii între mai multe sesiuni
4. **Generați rapoarte**: Includeți hărți, statistici și vizualizări***

## Arhivare și copii de rezervă

### Strategia recomandată de copiere de rezervă

**Ce trebuie salvat:*** ✅ **Imagini RAW/JPG originale sau capturi RAW LATTICE** – Arhivați pe o unitate separată/în cloud; fișierele RAW reprezintă sursa procesului de prelucrare, iar toate celelalte elemente pot fi regenerate pe baza acestora
* ✅ **Fișiere ale senzorului de lumină `.daq` / `.csv`** – Necesare pentru recalcularea reflectanței ulterior
* ✅ **Rezultate procesate** – Păstrați imaginile calibrate și indicii
* ✅ **Dosarul proiectului** (`project.json` și fișierele asociate) – Conține toate setările necesare pentru reprocesare, dacă este cazul
* ✅ **Jurnal de depanare** – Documentează detaliile procesării
* ✅ **Imagini țintă de calibrare** – Pentru verificare și reprocesare**Recomandări de stocare:*** **Copie de rezervă imediată**: Hard disk extern
* **Arhivare pe termen lung**: Spațiu de stocare în cloud (Google Drive, Dropbox etc.)
* **Date critice**: Păstrați 2-3 copii în locații diferite***

## Următoarele cicluri de procesare

### Reutilizarea setărilor proiectului

Dacă veți procesa seturi de date similare în viitor:

1. **Salvați șablonul proiectului** (dacă nu ați făcut-o deja)
2. **Creați un proiect nou** folosind șablonul salvat
3. **Importați imagini noi**

4.**Procesați**cu setări identice pentru a asigura consecvența

### Procesarea în lot a mai multor sesiuni

Pentru mai multe sesiuni/seturi de date:**Opțiunea 1: Interfață grafică (GUI) – Proiecte multiple**

* Creați un proiect separat pentru fiecare sesiune
* Utilizați setări consecvente ale șablonului
* Procesați câte unul pe rând

**Opțiunea 2: Chloros CLI (numai pentru Chloros+)**

* Automatizați procesarea în lot
* Procesați mai multe foldere cu ajutorul scripturilor
* Consultați [Documentația CLI](../CLI.md) și [Referința CLI](../reference/cli-reference.md)

**Opțiunea 3: Python SDK (numai pentru Chloros+)**

* Control programatic
* Integrare cu fluxurile de analiză
* Consultați [Documentația API](../api-python-sdk.md) și [Referința SDK](../reference/sdk-reference.md)

***

## Depanarea post-procesării

### Reprocesarea cu setări diferite

Dacă rezultatele nu sunt satisfăcătoare:

1. Păstrați imaginile originale (nu le ștergeți niciodată)
2. Deschideți același proiect în Chloros
3. Reglați setările în panoul Setări proiect
4. Procesati din nou — rezultatele sunt salvate în aceleași foldere de produse, astfel încât fișierele cu același nume din rularea anterioară vor fi înlocuite

### Procesarea unui subset de imagini

Pentru a reprocesa numai anumite imagini:

1. Creați un proiect nou
2. Importați numai imaginile care necesită reprocesare
3. Utilizați același șablon de setări
4. Procesați setul de date mai mic

### Obținerea de ajutor

Dacă întâmpinați probleme:

* 📧 **E-mail**: info@mapir.camera (includeți jurnalul de depanare)
* 🌐 **Asistență**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Întrebări frecvente**: [Întrebări frecvente](../faq.md)
* 📖 **Documentație**: [Manualul Chloros](../)***

## Rezumat: Fluxul de lucru complet

Ați finalizat acum întregul flux de lucru de procesare Chloros:

1. ✅ **Proiect creat** – Consultați [Proiecte](../projects.md)
2. ✅ **Fișiere adăugate** – Consultați [Adăugarea fișierelor](adding-files-to-a-project.md)
3. ✅ **Setări ajustate** - Vezi [Ajustarea setărilor proiectului](adjusting-project-settings.md)
4. ✅ **Ținte marcate** - Vezi [Alegerea imaginilor țintă](choosing-target-images.md)
5. ✅ **Prelucrare inițiată** - Vezi [Inițierea prelucrării](starting-the-processing.md)
6. ✅ **Progres monitorizat** - Consultați [Monitorizarea procesării](monitoring-the-processing.md)
7. ✅ **Rezultate verificate** - Această pagină**Imaginile dvs. multispectrale calibrate și cu reflexia corectată sunt gata pentru analiză!**

***

## Resurse suplimentare

### Funcții avansate

* [**Vizualizator de imagini**](../image-viewer-gui/opening-an-image-full-screen.md) - Vizualizare și analiză interactivă
* [**Sandbox pentru indici/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Testarea indicilor personalizați
* [**Formule de indici multispectrali**](../project-settings/multispectral-index-formulas.md) - Referință completă privind indicii

### Automatizare și integrare

* [**Documentație CLI**](../CLI.md) - Procesare în serie prin linia de comandă
* [**Python SDK**](../api-python-sdk.md) - Automatizare programată
* [**Caracteristici Chloros+**](../#chloros) - Capacități avansate de procesare

### Asistență și învățare

* [**Întrebări frecvente**](../faq.md) - Răspunsuri la întrebările frecvente
* [**Ținte de calibrare**](../calibration-targets.md) - Înțelegerea calibrării reflectanței
* [**Camere compatibile**](../supported-cameras.md) - Hardware compatibil
