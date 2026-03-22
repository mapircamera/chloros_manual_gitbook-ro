# Deschiderea unei imagini pe ecran complet

Vizualizatorul de imagini Chloros oferă o interfață dedicată pe ecran complet pentru vizualizarea, analizarea și manipularea imaginilor multispectrale. Indiferent dacă vizualizați imagini originale sau rezultate procesate, vizualizatorul de imagini oferă instrumente puternice pentru inspecție și analiză.

## Accesarea vizualizatorului de imagini

### Din browserul de fișiere

Cea mai comună modalitate de a deschide o imagine în vizualizatorul de imagini:

1. Asigurați-vă că vă aflați în fila **Browser de fișiere** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Faceți clic pe orice **miniatură a imaginii** din grila de imagini
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

Cea mai mare porțiune a ecranului afișează imaginea dvs.:

* **Rezoluție completă**: Imaginile sunt afișate la rezoluția nativă
* **Zoomabil**: Utilizați comenzile sau rotița mouse-ului pentru a mări
* **Panoramabil**: Faceți clic și trageți pentru a vă deplasa atunci când este mărită
* **Raport de aspect menținut**: Imaginile se scalează proporțional***

## Opțiuni de vizualizare

### Navigare de bază în imagini

#### Răsfoiți imaginile

Navigați prin setul de imagini utilizând comenzi rapide de la tastatură sau butoane:

* **Imaginea următoare**: Faceți clic pe butonul → sau apăsați tasta**→** (Săgeată dreapta)
* **Imaginea anterioară**: Faceți clic pe butonul ← sau apăsați tasta**←** (Săgeată stânga)
* **Săriți la o imagine specifică**: Reveniți la File Browser și faceți clic pe miniatura dorită

#### Comenzi de zoom

Reglați mărirea pentru a inspecta detaliile imaginii:

**Mărire:*** Faceți clic pe butonul **+** (Plus)
* Apăsați tasta **+**sau**=*** Rotiți rotița mouse-ului **în sus**

**Micșorare:*** Faceți clic pe butonul **−** (Minus)
* Apăsați tasta **−** (Minus)
* Rotiți rotița mouse-ului **în jos**

#### Panoramare la zoom

Când măriți imaginea peste dimensiunea ecranului:

1. Mutați cursorul mouse-ului peste imagine
2. Faceți clic și **țineți apăsat butonul stâng al mouse-ului**

3.**Trageți** pentru a deplasa imaginea
4. Eliberați pentru a opri panoramarea

**Alternativă**: Folosiți tastele săgeată pentru a panorama în pași mici***

## Inspectarea valorilor pixelilor

### Vizualizarea valorilor pixelilor la cursor

Pe măsură ce deplasați cursorul mouse-ului peste imagine, valorile pixelilor se afișează în timp real:**Locația afișării valorilor:*** **Număr flotant și linie roșie în legenda gradientului LUT din partea dreaptă*** **Când se mărește și mai mult, valoare flotantă lângă cursor și pixelul evidențiat*** Afișează valorile pentru pixelul **de sub cursor sau evidențiat*** Se actualizează pe măsură ce mișcați mouse-ul

***

## Tipuri de imagini pe care le puteți vizualiza

### JPG

**Imagini JPG de la cameră:**

* Afișează datele JPG așa cum sunt previzualizate
* Afișează valorile originale, necorectate
* Util pentru verificarea calității imaginii înainte de procesare

### RAW (Original)

### RAW (Reflectanță)

**După procesare:**

* Vigneta corectată
* Reflectanță calibrată
* Multi-bandă TIFF (Red, Green, NIR, etc.)
* Date științifice gata pentru analiză

### RAW (Index)

**NDVI, NDRE, GNDVI, etc. (fișiere \_NDVI.tif):**

* Imagini monocrome cu o singură bandă
* Valorile pixelilor reprezintă rezultatele calculului indicilor
* Intervalul este de obicei de la -1 la +1 pentru indicii normalizați
* Se pot aplica tabele de conversie (LUT) de culori pentru vizualizare

***

## Aplicarea indicilor și a tabelelor de conversie (LUT)

Aplicați indici multispectrali și tabele de conversie (LUT) de culori:

1. Localizați **Index/LUT Sandbox**în**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> bara laterală
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

* **+**sau**=**: Mărire
* **−**: Micșorare
* **Roata mouse-ului**: Mărire/micșorare***

### Verificarea calculelor indicilor

Verificați dacă indicii au fost calculați corect:

1. Deschideți NDVI sau o altă imagine de indice
2. Verificați zonele de vegetație:
   * **NDVI**: Ar trebui să indice 0,4-0,9 pentru plantele sănătoase
   * **NDRE**: Valori mai mari pentru creștere viguroasă
   * **GNDVI**: Similar cu NDVI, dar sensibil la clorofilă
3. Verificați zonele fără vegetație:
   * **Sol**: Aproape de 0 sau ușor negativ
   * **Apă**: Valori negative (-0,5 până la 0)***

## Rezolvarea problemelor de vizualizare

### Imaginea nu se deschide

**Cauze posibile:**

* Fișier corupt în timpul procesării
* Format de fișier neacceptat
* Memorie insuficientă pentru o imagine mare

**Soluții:**

1. Încercați să deschideți imaginea într-un vizualizator extern pentru a verifica integritatea fișierului
2. Verificați dacă formatul fișierului corespunde tipului așteptat
3. Închideți alte aplicații pentru a elibera memorie
4. Încercați o imagine mai mică/diferită

### Afișarea imaginii în alb sau negru

**Cauze posibile:**

* Intervalul valorilor depășește capacitatea de afișare
* Imagine de tip float pe 32 de biți cu valori neobișnuite
* Eroare de calcul al indexului

**Soluții:**

1. Verificați valorile pixelilor - dacă toate sunt foarte mici sau foarte mari, reglați intervalul de afișare
2. Încercați să deschideți fișierul în QGIS sau într-un program similar cu reglare automată a intervalului
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

* [**Straturi de imagine**](image-layers.md) - Aflați mai multe despre vizualizarea multibandă
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Aplicați indici personalizați și mapare de culori
* [**Formule de indici multispectrali**](../project-settings/multispectral-index-formulas.md) - Înțelegeți indicii disponibili

Pentru fluxul de lucru de procesare, consultați:

* [**Procesarea imaginilor (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Ghid complet de procesare
