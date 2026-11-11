# Finalizarea procesării

Odată ce Chloros finalizează procesarea, este momentul să analizați rezultatele, să verificați calitatea ieșirii și să pregătiți imaginile procesate pentru a le utiliza în fluxul dvs. de lucru. Această pagină vă ghidează prin pașii finali și acțiunile următoare.

## Indicatori ai finalizării procesării

Când procesarea se încheie cu succes, veți vedea mai mulți indicatori:

* ✅ **Bara de progres**: Atinge 100% finalizare
* ✅ **Jurnal de depanare**: Afișează ultimele linii `[RUN-SUMMARY]` cu numărul corespunzător (imagini, grupuri de camere, ținte, imagini calibrate, fișiere salvate)
* ✅ **Butonul „Start”**: Devine din nou activ (gata pentru următoarea rundă de procesare)
* ✅ **Fișiere de ieșire**: Toate imaginile procesate sunt salvate în arborele de ieșire al proiectului (mai jos)

{% hint style="warning" %}
**O execuție care nu scrie nicio imagine este considerată eșuată.** Dacă ați solicitat produse imagistice și ciclul nu a scris niciuna, Chloros raportează acest lucru ca un eșec — `[RUN-SUMMARY]` sugerează în numele jurnalului cauza probabilă (nu s-a importat nimic, nu s-a detectat nicio țintă sau toate produsele solicitate au fost omise ca fiind inaplicabile). Echivalentul CLI-ului se închide cu un cod de ieșire diferit de zero. O execuție deliberată numai cu metadate (toate produsele de export dezactivate, fără indici) este totuși considerată reușită. Consultați [Referința CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localizarea imaginilor procesate

### Deschiderea folderului de ieșire

1. Faceți clic pe pictograma **Meniul principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(stânga sus)
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

* **Folderul camerei**: `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE (care corespunde cu datele EXIF ale capturii `Model`), `<model>_<filter>` pentru Survey3 (de exemplu, `Survey3N_RGN`). Două camere care împart același senzor și filtru, dar au obiective diferite, păstrează arbori separați — vignetarea, câmpul vizual și distorsiunea diferă.
* **Format folder**: respectă setarea formatului de export — `tiff16`, `tiff8`, `png8`, `jpg8`, sau `tiff32` pentru TIFF (32 de biți, procent). Radianța este întotdeauna de tip float32 și se află întotdeauna sub `tiff32`.
* **Dosare de produse**:
  * `Reflectance_Calibrated_Images/` — reflectanță calibrată
  * `Debayered_Images/` — debayering liniar (LATTICE): offsetul de întuneric eliminat și demosaic, fără câmp plat sau demixare, în orientarea nativă a senzorului
  * `Preview_Images/` — previzualizare pe ecran (LATTICE)
  * `Radiance_Images/` — radianță spectrală float32, W/m²/sr/nm (multispectral LATTICE)
  * `Vignette_Corrected_Images/` **sau** `Sensor_Response_Images/` — varianta de rezervă necalibrată pentru cadre fără referință de reflectanță; există exact una dintre cele două pe fiecare serie, aleasă în funcție de setarea de corecție a vignetării
  * `<INDEX>_Index_Images/` — un dosar pentru fiecare index selectat (de exemplu, `NDVI_Index_Images`)
* **`clamp_masks/`** (nou în versiunea 1.3.0, numai pentru camerele multispectrale LATTICE M3C): apare într-un dosar de format, alături de dosarele de produse, atunci când procesul de separare a benzilor decupează valorile negative la zero într-un produs de radianță sau reflectanță. Acesta conține o mască per pixel pentru fiecare produs afectat, de exemplu `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Măștile nu se află niciodată în interiorul unui dosar de produs, astfel încât un dosar de produs poate fi transmis software-ului de fotogrammetrie așa cum este. A se vedea [Măști de limitare](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Fiecare produs exportat păstrează numele fișierului SURSĂ.**Un export de radianță al `capture_..._raw.tif` se numește în continuare `capture_..._raw.tif` — doar că se află în `tiff32/Radiance_Images/`.**Folderul identifică produsul, nu numele fișierului**, așa că o căutare după `*radiance*.tif` nu va găsi nimic; căutați în schimb după director.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Câte fișiere ar trebui să existe?

Nu numărați după o formulă — numărul de ieșiri depinde de produsele care au fost activate și de cele care se aplică fiecărei camere (de exemplu, camerele RGB nu primesc radianță/reflectanță). Numărul oficial se găsește în jurnal: ultima linie `[RUN-SUMMARY]` raportează exact câte fișiere au fost scrise, iar liniile de indicii explică tot ce a fost omis.

***

## Vizualizarea imaginilor procesate

### Previzualizare rapidă în File Explorer

**Previzualizare integrată în Windows:**

1. Navigați într-un dosar de produse (de exemplu, `tiff16/Reflectance_Calibrated_Images/`)
2. Selectați un fișier imagine
3. Previzualizarea apare în panoul de previzualizare al Windows Explorer
4. Folosiți tastele săgeată pentru a naviga printre imagini

### Previzualizare în vizualizatoare de imagini externe

**Vizualizatoare recomandate:*** **QGIS** - Software GIS gratuit (cel mai potrivit pentru analiza multispectrală georeferențiată)
* **IrfanView** – Vizualizator de imagini rapid și ușor (suportă formatul TIFF)
* **Adobe Photoshop** – Editare profesională (suportă formatul TIFF)
* **GIMP** – Alternativă gratuită la Photoshop
* **Windows Photos** – Vizualizare de bază (este posibil să nu suporte formatul TIFF pe 16 biți)

### Previzualizare în vizualizatorul de imagini Chloros


Utilizați vizualizatorul de imagini integrat în Chloros pentru vizualizare avansată:

1. Faceți clic pe o miniatură a imaginii în browserul de fișiere
2. Imaginea se deschide în zona principală de previzualizare
3. Faceți clic pe fila **Vizualizator de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
din bara laterală din stânga
4. Utilizați [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) pentru analiză interactivă

Consultați [Vizualizatorul de imagini](../image-viewer-gui/opening-an-image-full-screen.md) pentru instrucțiuni detaliate.

***

## Citirea valorilor pixelilor de reflectanță (GIS / Pix4D / Scripturi)

Reflectanța este stocată ca număr întreg DN, iar **valoarea DN care corespunde lui ρ = 1,0 depinde de camera sursă**:

| Sursă          | ρ = 1,0 este | Cum se determină                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (marjă de până la ρ 2,0) | Eticheta XMP `Chloros:PixelScale=32768` este înscrisă în fișier |
| Survey3          | **65535** (tăiat la ρ 1,0)     | Nu există eticheta `Chloros:PixelScale` — această absență este semnalul (produsele Survey3 conțin alte etichete de procesare de tip `Chloros:*`) |

**Citiți eticheta `Chloros:PixelScale` și împărțiți la valoarea acesteia**, în loc să presupuneți o valoare generală de 65535 — împărțirea reflectanței LATTICE la 65535 reduce în mod silențios la jumătate fiecare valoare. Un fișier de reflectanță dinainte de versiunea 1.3.0, scris pe 8 biți, a fost decupat, nu redimensionat, și nu are etichetă de scalare: reexportați-l pe 16 biți sau 32 de biți în loc să-l împărțiți. Consultați [Formate de imagine de ieșire](../output-image-formats.md) pentru detalii complete.***

## Metadate transferate în exporturi

Fiecare produs păstrează **blocul GPS**al capturii sursă și**sub-IFD-ul EXIF** al acesteia, astfel încât un
export conține `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` și
`CameraSerialNumber`, precum și georeferențierea.

{% hint style="warning" %}
**Dacă un ortomozaic are o scară absurdă, verificați mai întâi `FocalLength`.**
Pix4D calculează distanța dintre eșantioanele la sol pe baza distanței focale și a altitudinii. Fără această etichetă,
se ajunge la o scară extrem de eronată — într-un zbor măsurat cu 49 de capturi, o livadă de portocali de 411 m × 160 m
a fost reconstruită ca având dimensiunile de 47,8 km × 13 km, generând o ortografie de 455 de megapixeli alcătuită în mare parte din
spațiu gol. Asamblarea lentă a plăcilor și un fișier neașteptat de mare sunt simptome ale acestei probleme, nu
probleme separate.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nu sunt copiate *toate* etichetele. Etichetele structurale ale IFD0 sunt omise în mod deliberat (copierea
lora corupe rezultatul LATTICE), iar `ExifImageWidth` / `ExifImageHeight` sunt excluse
deoarece descriu captura originală — un export redimensionat ar pretinde altfel
dimensiuni cu care propria sa rasterizare intră în contradicție.

Chloros scrie, de asemenea, propriile sale etichete XMP de tip `Chloros:*` care descriu ce s-a făcut cu fiecare produs
(de exemplu, `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Începând cu versiunea 1.3.0, produsele Survey3 le includ în fiecare mod de procesare, produsele de indexare descriu indexul în sine (`Chloros:ProcessingLevel=index`,
formula și dacă a fost calculat pe baza reflectanței sau a radianței), în loc să repete
etichetele fișierului sursă brut, iar produsele din dosare cu nume mai lungi de 260 de caractere își păstrează și ele metadatele. Consultați [Formate de imagine de ieșire](../output-image-formats.md#index-products).

***

## Verificarea jurnalului de depanare

### Verificarea avertismentelor sau erorilor

1. Deschideți fila **Jurnal de depanare** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Derulați mesajele
3. Căutați avertismentele galbene sau erorile roșii
4. Citiți liniile `[RUN-SUMMARY]` și eventualele sugestii
5. Contactați asistența MAPIR pentru ajutor

### Salvarea jurnalului

Pentru a păstra o evidență a procesării sau pentru a-l trimite serviciului de asistență MAPIR:

1. Faceți clic pe butonul **„Copiere”**sau**„Descărcare”**

2. Salvați ca fișier text în folderul proiectului
3. Includeți-l în documentația proiectului
4. Trimiteți-l serviciului de asistență MAPIR dacă întâmpinați probleme

Pentru a raporta o problemă direct către MAPIR, utilizați pictograma de eroare (**Raportați o eroare**) din partea de sus a filei Jurnal de depanare — consultați [Raportarea unei erori din fila Jurnal](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).***

## Probleme frecvente legate de ieșire și soluții

### Problemă: Fișiere de ieșire lipsă

**Cauze posibile:**

* Produsul nu se aplică acelei camere (de exemplu, radianță/reflectanță pentru camerele RGB — jurnalul indică acest lucru)
* Lipsa unei referințe necesare (de exemplu, reflectanță fără țintă și fără `.daq` downwelling)
* Caseta de selectare pentru exportul produsului a fost dezactivată în Setările proiectului
* S-a epuizat spațiul pe disc în timpul exportului

**Soluții:**

1. Verificați indicațiile `[RUN-SUMMARY]` și rândurile `[EXPORT-CHECK]` din jurnalul de depanare — acestea explică omisiunile specifice fiecărei camere
2. Verificați casetele de selectare pentru exportul produsului din [Setările proiectului](adjusting-project-settings.md)
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
3. Verificați calitatea imaginilor țintă
4. Reprocesați cu țintele corespunzătoare marcate

### Problemă: Valorile NDVI par incorecte

**Intervale NDVI așteptate:*** **Apă, roci, sol**: de la -0,1 la 0,2
* **Vegetație rară/nesănătoasă**: de la 0,2 la 0,4
* **Vegetație moderată**: de la 0,4 la 0,6
* **Vegetație sănătoasă și densă**: 0,6 – 0,9**Dacă valorile se află în afara acestor intervale:**

1. Verificați dacă a fost aplicată calibrarea reflectanței
2. Verificați dacă a fost inclus jurnalul senzorului de lumină
3. Verificați dacă au fost detectate țintele de calibrare
4. Asigurați-vă că a fost detectat modelul corect de cameră
5. Verificați momentul și condițiile de captare a imaginilor țintă
6. Dacă calculați singuri indicii pe baza fișierelor de reflectanță, asigurați-vă că ați împărțit la valoarea `Chloros:PixelScale` a fișierului (a se vedea mai sus)

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
4. **Procesați mozaicuri indexate**: Creați ortomozaicuri NDVI din imagini indexate individuale
5. **Exportați fișiere GeoTIFF georeferențiate**: Pentru utilizare în aplicații GIS

### Pentru analiza GIS

**Flux de lucru recomandat:**

1.**Încărcați în QGIS, ArcGIS sau un program similar**

2.**Utilizați imagini de reflectanță TIFF pe 16 biți** pentru analiza multibandă (împărțiți la valoarea `Chloros:PixelScale` a fișierului)
3. **Utilizați imagini indexate** (NDVI, NDRE) ca straturi de vegetație gata de utilizare
4. **Calculator raster**: Combinați benzile pentru analize personalizate
5. **Export**: Creați hărți de clasificare, detectați schimbările, creați hărți ale stării de sănătate a vegetației

### Pentru analiză directă / raportare

**Flux de lucru recomandat:**

1.**Utilizați imagini indexate cu culori LUT** pentru rapoarte vizuale
2. **Extrageți statistici**: NDVI mediu pe câmp/parcelă
3. **Seriile temporale**: Comparați indicii între mai multe sesiuni
4. **Generați rapoarte**: Includeți hărți, statistici și vizualizări***

## Arhivare și copii de rezervă

### Strategia recomandată pentru copiile de rezervă

**Ce trebuie salvat:*** ✅ **Imagini RAW/JPG originale sau capturi RAW LATTICE** – Arhivați pe o unitate separată/în cloud; fișierele RAW reprezintă sursa procesului de prelucrare, iar toate celelalte elemente pot fi regenerate pe baza acestora
* ✅ **Fișiere ale senzorului de lumină `.daq` / `.csv`** – Necesare pentru a recalcula reflectanța ulterior
* ✅ **Rezultate procesate** – Păstrați imaginile calibrate și indicii
* ✅ **Dosarul proiectului** (`project.json` și fișierele asociate) – Conține toate setările necesare pentru reprocesare, dacă este nevoie
* ✅ **Jurnal de depanare** – Documentează detaliile procesării
* ✅ **Imagini țintă de calibrare** – Pentru verificare și reprocesare**Recomandări de stocare:*** **Copie de rezervă imediată**: hard disk extern
* **Arhivare pe termen lung**: stocare în cloud (Google Drive, Dropbox etc.)
* **Date critice**: păstrați 2-3 copii în locații diferite***

## Următoarele cicluri de procesare

### Reutilizarea setărilor proiectului

Dacă veți prelucra seturi de date similare în viitor:

1. **Salvați șablonul proiectului** (dacă nu ați făcut-o deja)
2. **Creați un proiect nou** folosind șablonul salvat
3. **Importați imagini noi**

4.**Prelucrați**cu setări identice pentru a asigura consecvența

### Procesarea în lot a mai multor sesiuni

Pentru mai multe sesiuni/seturi de date:**Opțiunea 1: Interfața grafică (GUI) – Proiecte multiple**

* Creați un proiect separat pentru fiecare sesiune
* Utilizați setări consistente ale șablonului
* Procesați câte unul pe rând

**Opțiunea 2: Chloros CLI (numai Chloros+)**

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
3. Reglați setările în panoul „Setări proiect”
4. Reprocesați — rezultatele sunt salvate în aceleași foldere de produse, astfel încât fișierele cu același nume din rularea anterioară sunt înlocuite

### Procesarea unui subset de imagini

Pentru a reprocesa doar anumite imagini:

1. Creați un proiect nou
2. Importați doar imaginile care necesită reprocesare
3. Utilizați același șablon de setări
4. Procesați setul de date mai mic

### Obținerea de ajutor

Dacă întâmpinați probleme:

* 📧 **E-mail**: info@mapir.camera (includeți jurnalul de depanare)
* 🌐 **Asistență**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Întrebări frecvente**: [Întrebări frecvente](../faq.md)
* 📖 **Documentație**: [Manualul Chloros](../)

***

## Rezumat: Flux de lucru complet

Ați finalizat acum întregul flux de lucru de procesare Chloros:

1. ✅ **Proiect creat** - Consultați [Proiecte](../projects.md)
2. ✅ **Ați adăugat fișiere** – Consultați [Adăugarea fișierelor](adding-files-to-a-project.md)
3. ✅ **Ați ajustat setările** – Consultați [Ajustarea setărilor proiectului](adjusting-project-settings.md)
4. ✅ **Ați marcat țintele** – Consultați [Alegerea imaginilor țintă](choosing-target-images.md)
5. ✅ **Prelucrarea a fost inițiată** - Consultați [Inițierea prelucrării](starting-the-processing.md)
6. ✅ **Progresul a fost monitorizat** - Consultați [Monitorizarea prelucrării](monitoring-the-processing.md)
7. ✅ **Rezultate verificate** - Această pagină**Imaginile dvs. multispectrale calibrate și cu reflexia corectată sunt gata pentru analiză!**

***

## Resurse suplimentare

### Funcții avansate

* [**Vizualizator de imagini**](../image-viewer-gui/opening-an-image-full-screen.md) - Vizualizare și analiză interactivă
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testarea indexurilor personalizate
* [**Formule de index multispectral**](../project-settings/multispectral-index-formulas.md) – Referință completă privind indexurile

### Automatizare și integrare

* [**Documentație CLI**](../CLI.md) - Prelucrare în lot prin linia de comandă
* [**Python SDK**](../api-python-sdk.md) - Automatizare programată
* [**Funcționalități Chloros+**](../#chloros) - Capacități avansate de procesare

### Asistență și învățare

* [**Întrebări frecvente**](../faq.md) - Răspunsuri la întrebările frecvente
* [**Ținte de calibrare**](../calibration-targets.md) - Înțelegerea calibrării reflectanței
* [**Camere compatibile**](../supported-cameras.md) - Hardware compatibil
