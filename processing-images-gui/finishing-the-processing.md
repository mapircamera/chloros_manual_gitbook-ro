# Finalizarea procesării

Odată ce Chloros finalizează procesarea, este momentul să analizați rezultatele, să verificați calitatea ieșirii și să pregătiți imaginile procesate pentru a le utiliza în fluxul de lucru. Această pagină vă ghidează prin pașii finali și acțiunile următoare.

## Indicator de finalizare a procesării

Când procesarea se finalizează cu succes, veți vedea mai mulți indicatori:

* ✅ **Bara de progres**: Atinge 100% finalizare
* ✅ **Jurnal de depanare**: Afișează mesajul „Procesare finalizată”
* ✅ **Butonul Start**: Devine din nou activ (gata pentru următoarea rundă de procesare)
* ✅ **Fișiere de ieșire**: Toate imaginile procesate sunt salvate în subfolderul modelului de cameră***

## Localizarea imaginilor procesate

### Deschiderea folderului de ieșire

1. Faceți clic pe pictograma **Meniu principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (stânga sus)
2. Selectați **„Deschideți folderul proiectului”**

3. Explorerul de fișiere se deschide în directorul proiectului
4. Localizați proiectul după nume

***

## Vizualizarea imaginilor procesate

### Previzualizare rapidă în exploratorul de fișiere

**Previzualizare integrată Windows:**

1. Navigați la subfolderul modelului de cameră
2. Selectați un fișier imagine
3. Previzualizarea apare în panoul de previzualizare Windows Explorer
4. Utilizați tastele săgeată pentru a naviga printre imagini

### Previzualizare în vizualizatoare de imagini externe

**Vizualizatoare recomandate:*** **QGIS** - Software GIS gratuit (cel mai bun pentru analiza multispectrală georeferențiată)
* **IrfanView** - Vizualizator de imagini rapid și ușor (acceptă TIFF)
* **Adobe Photoshop** - Editare profesională (suport pentru TIFF)
* **GIMP** - Alternativă gratuită la Photoshop
* **Windows Photos** - Vizualizare de bază (este posibil să nu suporte TIFF pe 16 biți)

### Previzualizare în vizualizatorul de imagini Chloros

Utilizați vizualizatorul de imagini încorporat al Chloros pentru vizualizare avansată:

1. Faceți clic pe o miniatură a imaginii în browserul de fișiere
2. Imaginea se deschide în zona principală de previzualizare
3. Faceți clic pe fila **Vizualizator de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> din bara laterală din stânga
4. Utilizați [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) pentru analiză interactivă

Consultați [Vizualizator de imagini](../image-viewer-gui/opening-an-image-full-screen.md) pentru instrucțiuni detaliate.

***

## Verificarea jurnalului de depanare

### Verificați dacă există avertismente sau erori

1. Deschideți fila **Jurnal de depanare** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Derulați mesajele
3. Căutați avertismentele galbene sau erorile roșii
4. Verificați orice probleme semnalate
5. Contactați asistența MAPIR pentru ajutor

### Salvarea jurnalului

Pentru a păstra o înregistrare a procesării sau pentru a o trimite la asistența MAPIR:

1. Faceți clic pe butonul **„Copiere”**sau**„Descărcare”**

2. Salvați ca fișier text în folderul proiectului
3. Includeți-l în documentația proiectului
4. Trimiteți-l către asistența MAPIR dacă au apărut probleme

***

## Probleme comune de ieșire și soluții

### Problemă: Fișiere de ieșire lipsă

**Cauze posibile:**

* Fișierele nu au îndeplinit criteriile de procesare
* Imagini destinate exclusiv țintei (excluse din export)
* Spațiul pe disc s-a epuizat în timpul exportului
* Fișierele s-au corupt în timpul procesării

**Soluții:**

1. Verificați jurnalul de depanare pentru mesaje de omitere/eroare
2. Verificați dacă spațiul pe disc a fost suficient
3. Numărați fișierele: ar trebui să corespundă (numărul inițial - numărul țintă) × (indici + 1)
4. Reimportați și reprocesați orice fișiere lipsă

### Problemă: Margini întunecate sau luminoase (vignetarea este încă vizibilă)

**Cauze posibile:**

* Corecția vignetării este dezactivată
* Camera/obiectivul nu se află în baza de date a profilurilor Chloros
* Vignetare extremă, care depășește capacitatea de corecție

**Soluții:**

1. Verificați dacă corectarea vignetării a fost activată în Setările proiectului
2. Verificați dacă modelul camerei a fost detectat corect
3. Contactați asistența MAPIR dacă vignetarea persistă

### Problemă: Culori sau valori incorecte

**Cauze posibile:**

* Nu s-au detectat ținte de calibrare
* A fost selectat un model greșit de țintă de calibrare
* Calibrarea reflectanței este dezactivată
* Imagini țintă de calitate slabă

**Soluții:**

1. Verificați dacă calibrarea reflectanței a fost activată
2. Verificați mesajele „Țintă găsită” din Jurnalul de depanare
3. Verificați calitatea imaginii țintei
4. Reprocesați cu țintele corespunzătoare marcate

### Problemă: Valorile NDVI par incorecte

**Intervale NDVI așteptate:*** **Apă, roci, sol**: -0,1 până la 0,2
* **Vegetație rară/nesănătoasă**: 0,2 până la 0,4
* **Vegetație moderată**: 0,4 până la 0,6
* **Vegetație sănătoasă, densă**: 0,6 până la 0,9**Dacă valorile se află în afara acestor intervale:**

1. Verificați dacă a fost aplicată calibrarea reflectanței
2. Verificați dacă a fost inclus jurnalul senzorului de lumină
3. Verificați dacă au fost detectate țintele de calibrare
4. Asigurați-vă că a fost detectat modelul corect de cameră
5. Verificați momentul și condițiile de captare a imaginii țintă

***

## Utilizarea imaginilor procesate

### Pentru fotogrammetrie / crearea ortomosaicelor

**Flux de lucru recomandat:**

1.**Importați imaginile de reflectanță calibrate** în software-ul de fotogrammetrie:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Păstrați metadatele EXIF**: Asigurați-vă că datele GPS sunt păstrate pentru geotagging
3. **Fluxuri de lucru calibrate**: Utilizați imagini de reflectanță pentru precizie științifică
4. **Procesați mozaicuri index**: Creați ortomozaicuri NDVI din imagini index individuale
5. **Exportați GeoTIFF georeferențiat**: Pentru utilizare în aplicații GIS

### Pentru analiza GIS

**Flux de lucru recomandat:**

1.**Încărcați în QGIS, ArcGIS sau similar**

2.**Utilizați imagini de reflectanță TIFF** de 16 biți pentru analiza multibandă
3. **Utilizați imagini index** (NDVI, NDRE) ca straturi de vegetație gata de utilizare
4. **Calculator raster**: Combinați benzile pentru analize personalizate
5. **Export**: Creați hărți de clasificare, detectare a modificărilor, hărți ale stării de sănătate a vegetației

### Pentru analiză directă / raportare

**Flux de lucru recomandat:**

1.**Utilizați imagini index cu culori LUT** pentru rapoarte vizuale
2. **Extrageți statistici**: Media NDVI pe câmp/parcelă
3. **Seriile temporale**: comparați indicii între mai multe sesiuni
4. **Generați rapoarte**: includeți hărți, statistici și vizualizări***

## Arhivare și backup

### Strategia de backup recomandată

**Ce să salvați:*** ✅ **Imagini RAW/JPG originale** - Arhivați pe un disc separat/în cloud
* ✅ **Rezultate procesate** - Păstrați imaginile calibrate și indicii
* ✅ **Fișier de proiect** - Conține toate setările pentru reprocesare, dacă este necesar
* ✅ **Jurnal de depanare** - Documentează detaliile procesării
* ✅ **Imagini țintă de calibrare** - Pentru verificare și reprocesare**Recomandări de stocare:*** **Backup imediat**: hard disk extern
* **Arhivare pe termen lung**: stocare în cloud (Google Drive, Dropbox etc.)
* **Date critice**: păstrați 2-3 copii în locații diferite***

## Următoarele cicluri de procesare

### Reutilizarea setărilor proiectului

Dacă procesați seturi de date similare în viitor:

1. **Salvați șablonul de proiect** (dacă nu ați făcut-o deja)
2. **Creați un proiect nou** folosind șablonul salvat
3. **Importați imagini noi**

4.**Procesați**cu setări identice pentru consecvență

### Procesarea în lot a mai multor sesiuni

Pentru mai multe sesiuni/seturi de date:**Opțiunea 1: GUI - Proiecte multiple**

* Creați un proiect separat pentru fiecare sesiune
* Utilizați setări de șablon consecvente
* Procesați câte unul pe rând

**Opțiunea 2: Chloros CLI (numai Chloros+)**

* Automatizați procesarea în lot
* Procesați mai multe foldere cu scripturi
* Consultați [Documentația CLI](../CLI.md)

**Opțiunea 3: Python SDK (numai pentru Chloros+)**

* Control programatic
* Integrare cu fluxuri de analiză
* Consultați [Documentația API](../api-python-sdk.md)

***

## Depanarea post-procesării

### Reprocesarea cu setări diferite

Dacă rezultatele nu sunt satisfăcătoare:

1. Păstrați imaginile originale (nu le ștergeți niciodată)
2. Deschideți același proiect în Chloros
3. Reglați setările în panoul Setări proiect
4. Procesare din nou - rezultatele vor suprascrie rezultatele anterioare

### Procesarea unui subset de imagini

Pentru a reprocesa doar anumite imagini:

1. Creați un proiect nou
2. Importați doar imaginile care necesită reprocesare
3. Utilizați același șablon de setări
4. Procesare set de date mai mic

### Obținerea de ajutor

Dacă întâmpinați probleme:

* 📧 **E-mail**: info@mapir.camera (includeți jurnalul de depanare)
* 🌐 **Asistență**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Întrebări frecvente**: [Întrebări frecvente](../faq.md)
* 📖 **Documentație**: [Manualul Chloros](../)***

## Rezumat: Flux de lucru complet

Ați finalizat acum fluxul de lucru complet de procesare Chloros:

1. ✅ **Proiect creat** - Consultați [Proiecte](../projects.md)
2. ✅ **Fișiere adăugate** - Consultați [Adăugarea fișierelor](adding-files-to-a-project.md)
3. ✅ **Setări ajustate** - Consultați [Ajustarea setărilor proiectului](adjusting-project-settings.md)
4. ✅ **Ținte marcate** - Vezi [Alegerea imaginilor țintă](choosing-target-images.md)
5. ✅ **Prelucrare începută** - Vezi [Începerea prelucrării](starting-the-processing.md)
6. ✅ **Progres monitorizat** - Vezi [Monitorizarea prelucrării](monitoring-the-processing.md)
7. ✅ **Rezultate verificate** - Această pagină**Imaginile dvs. multispectrale calibrate și cu reflexia corectată sunt gata pentru analiză!**

***

## Resurse suplimentare

### Funcții avansate

* [**Vizualizator de imagini**](../image-viewer-gui/opening-an-image-full-screen.md) - Vizualizare și analiză interactivă
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Testarea indexurilor personalizate
* [**Formule de indice multispectral**](../project-settings/multispectral-index-formulas.md) - Referință completă privind indicii

### Automatizare și integrare

* [**Documentație CLI**](../CLI.md) - Procesare în serie din linia de comandă
* [**Python SDK**](../api-python-sdk.md) - Automatizare programatică
* [**Chloros+ Caracteristici**](../#chloros) - Capacități avansate de procesare

### Asistență și învățare

* [**Întrebări frecvente**](../faq.md) - Răspunsuri la întrebări frecvente
* [**Ținte de calibrare**](../calibration-targets.md) - Înțelegerea calibrării reflectanței
* [**Camere compatibile**](../supported-cameras.md) - Hardware compatibil
