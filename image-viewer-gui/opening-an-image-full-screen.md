# Deschiderea unei imagini pe ecran complet

Chloros Image Viewer oferă o interfață dedicată pe ecran complet pentru vizualizarea, analizarea și manipularea imaginilor multispectrale. Fie că vizualizați imagini originale sau rezultate procesate, Image Viewer oferă instrumente puternice pentru inspecție și analiză.

## Accesarea vizualizatorului de imagini

### Din browserul de fișiere

Cea mai comună modalitate de a deschide o imagine în vizualizatorul de imagini:

1. Asigurați-vă că vă aflați în fila **Browser de fișiere** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Faceți clic pe orice **miniatură de imagine** din grila de imagini
3. Imaginea se deschide în **zona principală de previzualizare** (centrul ecranului)
4. Imaginea este acum încărcată și gata pentru vizualizare pe ecran complet

### Deschiderea filei Image Viewer

Odată ce o imagine este încărcată în zona de previzualizare:

1. Faceți clic pe pictograma **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> din bara laterală din stânga
2. Se deschide fila Vizualizator de imagini, afișând imaginea selectată pe ecran complet
3. Instrumentele avansate de vizualizare și analiză devin disponibile în bara laterală din stânga

***

## Prezentare generală a interfeței Vizualizatorului de imagini

### Zona principală de afișare

Cea mai mare parte a ecranului afișează imaginea:

* **Rezoluție completă**: imagini afișate la rezoluția nativă
* **Zoomabil**: utilizați comenzile sau rotița mouse-ului pentru a mări
* **Panoramabil**: faceți clic și trageți pentru a vă deplasa atunci când măriți
* **Raportul de aspect menținut**: imaginile se scalează proporțional

***

## Opțiuni de vizualizare

### Navigare de bază în imagini

#### Răsfoiți imaginile

Navigați prin setul de imagini utilizând comenzi rapide de la tastatură sau butoane:

* **Imaginea următoare**: faceți clic pe butonul → sau apăsați tasta **→** (săgeată dreapta)
* **Imaginea anterioară**: faceți clic pe butonul ← sau apăsați tasta **←** (săgeată stânga)
* **Salt la o imagine specifică**: reveniți la browserul de fișiere și faceți clic pe miniatura dorită

#### Comenzi de zoom

Reglați mărirea pentru a inspecta detaliile imaginii:

**Mărire:**

* Faceți clic pe butonul **+** (Plus)
* Apăsați tasta **+** sau **=**
* Derulați rotița mouse-ului **în sus**

**Micșorare:**

* Faceți clic pe butonul **−** (Minus)
* Apăsați tasta **−** (Minus)
* Derulați rotița mouse-ului **în jos**

#### Panoramare la mărire

Când măriți peste dimensiunea ecranului:

1. Deplasați cursorul mouse-ului peste imagine
2. Faceți clic și **țineți apăsat butonul stâng al mouse-ului**
3. **Trageți** pentru a deplasa imaginea
4. Eliberați pentru a opri panoramarea

**Alternativă**: Utilizați tastele săgeată pentru a panorama în pași mici

***

## Inspectarea valorii pixelilor

### Vizualizarea valorilor pixelilor la cursor

Pe măsură ce mutați cursorul mouse-ului peste imagine, valorile pixelilor sunt afișate în timp real:

**Locația afișării valorii:**

* **Număr flotant și linie roșie în legenda gradientului LUT din indexul din partea dreaptă**
* **Când măriți și mai mult, valoarea flotantă lângă cursor și pixelul evidențiat**
* Afișează valorile pentru pixelul **de sub cursor sau evidențiat**
* Se actualizează pe măsură ce mișcați mouse-ul

***

## Tipuri de imagini pe care le puteți vizualiza

### JPG

**Imagini JPG de la cameră:**

* Afișează datele JPG așa cum au fost previzualizate
* Afișează valorile originale, necorectate
* Util pentru verificarea calității imaginii înainte de procesare

### RAW (Original)

### RAW (Reflectanță)

**După procesare:**

* Vigneta corectată
* Reflectanța calibrată
* Multi-bandă TIFF (Red, Green, NIR etc.)
* Date științifice gata pentru analiză

### RAW (Index)

**NDVI, NDRE, GNDVI, etc. (fișiere \_NDVI.tif):**

* Imagini monocromatice cu o singură bandă
* Valorile pixelilor reprezintă rezultatele calculului indicelui
* Intervalul este de obicei între -1 și +1 pentru indicii normalizați
* Se pot aplica LUT-uri de culoare pentru vizualizare

***

## Aplicarea indicelui și a LUT-ului

Aplicați indici multispectrali și tabele de căutare a culorilor:

1. Localizați **Index/LUT Sandbox** în **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> bara laterală
2. Selectați indicele de vegetație (NDVI, NDRE, etc.)
3. Selectați formula multispectrală sau creați-vă propria formulă personalizată (numai Chloros+)
4. Aplicați gradientul LUT de culoare pentru vizualizare
5. Reglați intervalele de valori și pragurile

Consultați [Index/LUT Sandbox](index-lut-sandbox.md) pentru instrucțiuni detaliate.

***

## Comenzi rapide de la tastatură

### Navigare

* **→** (Săgeată dreapta): Imaginea următoare
* **←** (Săgeată stânga): Imaginea anterioară
* **Home**: Prima imagine din listă
* **End**: Ultima imagine din listă

### Zoom

* **+** sau **=**: Mărire
* **−**: Micșorare
* **Roata mouse-ului**: Mărire/micșorare

***

### Verificarea calculelor indexului

Verificați dacă indicii au fost calculați corect:

1. Deschideți NDVI sau altă imagine index
2. Verificați zonele cu vegetație:
   * **NDVI**: Ar trebui să afișeze 0,4-0,9 pentru plantele sănătoase
   * **NDRE**: Valori mai mari pentru creștere viguroasă
   * **GNDVI**: Similar cu NDVI, dar sensibil la clorofilă
3. Verificați zonele fără vegetație:
   * **Sol**: Aproape 0 sau ușor negativ
   * **Apă**: Valori negative (-0,5 până la 0)

***

## Depanarea problemelor de vizualizare

### Imaginea nu se deschide

**Cauze posibile:**

* Fișier corupt în timpul procesării
* Format de fișier neacceptat
* Memorie insuficientă pentru imagini mari

**Soluții:**

1. Încercați să deschideți în vizualizatorul extern pentru a verifica integritatea fișierului
2. Verificați dacă formatul fișierului corespunde tipului așteptat
3. Închideți alte aplicații pentru a elibera memorie
4. Încercați o imagine mai mică/diferită

### Afișare imagine alb-negru

**Cauze posibile:**

* Intervalul de valori depășește capacitatea de afișare
* Imagine flotantă pe 32 de biți cu valori neobișnuite
* Eroare de calcul al indexului

**Soluții:**

1. Verificați valorile pixelilor - dacă toate sunt foarte mici sau foarte mari, reglați intervalul de afișare
2. Încercați să deschideți în QGIS sau similar, cu reglarea automată a intervalului
3. Verificați jurnalul de depanare din procesare pentru erori

### Valorile pixelilor par greșite

**Cauze posibile:**

* Vizualizarea unei imagini greșite (originală vs. procesată)
* Calibrarea nu s-a aplicat corect
* Datele senzorului de lumină nu au fost incluse în intrare
* Modul procentual a fost comutat incorect

**Soluții:**

1. Verificați dacă vizualizați rezultatul procesat (verificați sufixul numelui fișierului)
2. Verificați starea butonului modului procentual
3. Comparați cu imagini cunoscute ca fiind bune din același set de date

***

## Pași următori

Acum că puteți vizualiza imaginile pe ecran complet:

* [**Straturi de imagini**](image-layers.md) - Aflați mai multe despre vizualizarea multibandă
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Aplicați indici personalizați și mapare de culori
* [**Formule index multispectrale**](../project-settings/multispectral-index-formulas.md) - Înțelegeți indicii disponibili

Pentru fluxul de lucru de procesare, consultați:

* [**Procesarea imaginilor (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Ghid complet de procesare
