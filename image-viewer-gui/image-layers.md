# Straturi de imagine

Meniul derulant „Straturi de imagine” din vizualizatorul de imagini Chloros vă permite să comutați rapid între diferite versiuni ale aceleiași imagini – de la capturile originale la rezultatele de reflectanță procesate și imaginile cu indice calculat.

## Ce sunt straturile de imagine?

În Chloros, **straturile** se referă la diferitele rezultate ale imaginii disponibile pentru o singură imagine sursă. Când procesați imagini, Chloros creează mai multe versiuni:

* **Imagini originale** (fișiere JPG și RAW de pe camera dvs.)
* Rezultate **calibrate în funcție de reflectanță** (dacă a fost activată calibrarea reflectanței)
* **Imagini țintă** (dacă imaginea conține ținte de calibrare)
* **Imagini index** (NDVI, NDRE, GNDVI etc., dacă au fost configurați indici)**Meniul derulant Selector de straturi** din partea dreaptă sus a Vizualizatorului de imagini vă permite să comutați instantaneu între aceste versiuni fără a părăsi vizualizatorul.***

## Tipuri de straturi disponibile

### JPG

* Imaginea de previzualizare JPG originală de la aparatul foto
* Disponibilă întotdeauna pentru toate imaginile
* Neprelucrată, așa cum a fost capturată de aparatul foto
* Se încarcă și se afișează cel mai rapid

**Când să vizualizați:**

* Previzualizare rapidă a capturii originale
* Verificarea compoziției și a încadrării imaginii
* Verificarea calității capturii înainte de procesare

### RAW (Original)

* Datele originale RAW ale senzorului de la camera dvs.
* Debayered fără aplicarea vreunei post-procesări
* Adâncime de biți mai mare decât JPG (de obicei date ale senzorului de 12 biți sau 14 biți)

**Când să vizualizați:**

* Inspectarea calității datelor originale ale senzorului
* Verificarea problemelor senzorului sau a artefactelor
* Compararea rezultatelor înainte și după procesare

### RAW (Țintă)

* Apare doar pentru imaginile identificate ca conținând ținte de calibrare
* Afișează imaginea RAW originală cu ținta detectată
* Utilizat pentru a verifica dacă detectarea țintei a avut succes

**Când să vizualizați:**

* Confirmarea faptului că țintele de calibrare au fost detectate corect
* Verificarea calității imaginii țintei
* Depanarea problemelor de calibrare

{% hint style="info" %}
**Strat țintă**: Acest strat apare doar în meniul derulant pentru imaginile care conțin ținte de calibrare. Imaginile capturate obișnuite nu vor avea această opțiune.
{% endhint %}

### RAW (Reflectanță)

* Imaginea de ieșire cu reflectanță calibrată
* Vigneta corectată (dacă este activată în procesare)
* Reflectanță calibrată folosind datele țintelor (dacă este activată)
* Multi-bandă TIFF cu toate canalele camerei
* Valorile pixelilor reprezintă procentul de reflectanță (atunci când se utilizează modul procentual)
* Gata de manipulare cu [Index/LUT Sandbox](index-lut-sandbox.md)

**Când să vizualizați:**

* Inspectarea rezultatelor calibrate
* Verificarea calității calibrării
* Verificarea valorilor pixelilor pentru acuratețe științifică
* Compararea cu originalul pentru a vedea efectele calibrării

{% hint style="success" %}
**Recomandat**: Utilizați stratul RAW (Reflectanță) atunci când verificați valorile pixelilor pentru măsurători și analize științifice.
{% endhint %}

### RAW (NDVI Index)... și similare

* Imagine a indicelui de vegetație calculat (NDVI în acest exemplu)
* Numele indicelui se modifică în funcție de indicele configurat în timpul procesării
* Exemple: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) etc.
* Imagine monocromă cu o singură bandă care prezintă rezultatele calculului indicelui
* Apare un strat pentru fiecare indice configurat în Setările proiectului

**Nume posibile ale indicilor:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* Și multe altele... (vezi [Formule de indici multispectrali](../project-settings/multispectral-index-formulas.md))

**Când să vizualizați:**

* Examinarea rezultatelor calculului indicelui
* Verificarea intervalelor valorilor indicelui
* Identificarea zonelor de interes
* Verificarea imaginilor cu indici înainte de utilizarea lor în GIS sau analiză

***

## Utilizarea selectorului de straturi

### Deschiderea meniului derulant

1. Deschideți o imagine în modul ecran complet (faceți clic pe orice miniatură din Vizualizatorul de imagini)
2. Localizați **meniul derulant al straturilor** în colțul din dreapta sus al vizualizatorului
3. Meniul derulant afișează stratul selectat în prezent (de exemplu, „JPG”)
4. Faceți clic pe meniul derulant pentru a vedea toate straturile disponibile

### Comutarea între straturi

1. Faceți clic pe meniul derulant al straturilor pentru a deschide lista
2. Sunt afișate toate straturile disponibile pentru imaginea curentă
3. Faceți clic pe numele oricărui strat pentru a comuta la acea versiune
4. Imaginea se actualizează imediat pentru a afișa stratul selectat

**Comutare rapidă:**

* Meniul derulant reține ultima dvs. selecție
* Când navigați la imaginea următoare, Chloros încearcă să afișeze același tip de strat
* Dacă acel strat nu există pe imaginea următoare, se revine implicit la JPG

### Disponibilitatea straturilor

Nu toate straturile sunt disponibile pentru fiecare imagine:

**Întotdeauna disponibile:*** ✅ JPG (fiecare imagine are o previzualizare JPG)

**Disponibile condiționat:**

* ⚠️ RAW (Original) - Numai dacă imaginea a fost capturată în modul RAW sau RAW+JPG
* ⚠️ RAW (Țintă) - Numai dacă imaginea conține ținte de calibrare detectate
* ⚠️ RAW (Reflectanță) - Numai după procesare cu calibrarea reflectanței activată
* ⚠️ RAW (\[Index] Index) - Numai după procesare cu indicii configurați

***

## Persistența straturilor

### Navigarea între imagini

Când navigați la o altă imagine (folosind tastele săgeată sau făcând clic pe miniaturi):**Preferința stratului este păstrată:**

* Dacă vizualizați „RAW (Reflectanță)”, imaginea următoare afișează „RAW (Reflectanță)” (dacă este disponibilă)
* Dacă vizualizați „RAW (NDVI Index)”, imaginea următoare afișează „RAW (NDVI Index)” (dacă este disponibil)
* Dacă același strat nu există, se utilizează implicit JPG

**Exemplu de flux de lucru:**

1. Deschideți Imaginea 1, comutați la RAW (NDVI Index)
2. Apăsați → pentru a vizualiza Imaginea 2
3. Imaginea 2 afișează automat stratul RAW (NDVI Index)
4. Continuați navigarea - toate imaginile afișează stratul NDVI
5. Foarte eficient pentru revizuirea rezultatelor indexului pe mai multe imagini

***

## Fluxuri de lucru comune

### Flux de lucru 1: Comparație înainte/după

**Obiectiv**: Compararea imaginii originale cu cea calibrată

1. Deschideți imaginea procesată în Image Viewer
2. Selectați **RAW (Original)** din meniul derulant
3. Observați vignetarea și valorile necalibrate
4. Treceți la **RAW (Reflectance)** din meniul derulant
5. Comparați - vignetarea a fost eliminată, valorile au fost calibrate

### Flux de lucru 2: Revizuirea indexului

**Obiectiv**: Revizuirea rapidă a rezultatelor NDVI în cadrul setului de date

1. Deschideți prima imagine procesată
2. Selectați **RAW (NDVI Index)** din meniul derulant
3. Utilizați tasta săgeată → pentru a naviga la imaginea următoare
4. Stratul NDVI persistă automat
5. Continuați prin toate imaginile, verificând modelele NDVI
6. Comutați la **RAW (NDRE Index)** pentru a compara

### Flux de lucru 3: Verificarea țintelor

**Obiectiv**: Verificați dacă toate imaginile țintă au fost detectate corect

1. Navigați la o imagine țintă
2. Selectați **RAW (Țintă)** din meniul derulant
3. Verificați dacă țintele de calibrare sunt clar vizibile și detectate
4. Navigați la următoarea imagine țintă
5. Repetați verificarea pentru toate țintele

### Flux de lucru 4: Inspecția valorilor pixelilor

**Obiectiv**: Verificați valorile de reflectanță pentru acuratețe științifică

1. Deschideți imaginea procesată
2. Selectați stratul **RAW (Reflectanță)**

3. Activați modul**Procentaj pixeli** (butonul din bara de instrumente din dreapta sus)
4. Mutați cursorul peste zonele de vegetație
5. Verificați dacă valorile pixelilor se încadrează în intervalele așteptate (30-70% pentru NIR, 5-15% pentru Red)
6. Verificați zonele de sol și apă pentru a vă asigura că valorile sunt corespunzătoare

***

## Înțelegerea valorilor pixelilor pe straturi

Diferitele straturi prezintă intervale diferite de valori ale pixelilor:

### Stratul JPG

* **Interval**: 0-255 (8 biți)
* **Semnificație**: Valori de afișare, corectate gamma
* **Utilizare**: Numai inspecție vizuală, nu pentru măsurători științifice

### RAW (Original)

* **Interval**: 0-65535 (16 biți)
* **Semnificație**: Valori digitale brute ale senzorului
* **Utilizare**: Verificarea performanței senzorului, necalibrate

### RAW (Reflectanță)

* **Interval**: 0-65.535 (16 biți TIFF) sau 0,0-1,0 (32 biți procent)
* **Semnificație**: procent de reflectanță calibrat
* **Utilizare**: măsurători și analize științifice**Pentru TIFF pe 16 biți:**Împărțiți la 65.535 pentru a obține procentul de reflectanță**Pentru procent pe 32 de biți:** Valorile reprezintă direct procentul (0,5 = 50% reflectanță)

### RAW (Imagini index)

* **Interval**: Variază în funcție de index (de obicei de la -1,0 la +1,0 pentru indicii normalizați)
* **Semnificație**: Rezultatul calculului indexului
* **Exemple**:
  * NDVI: de la -1 la +1 (vegetație de obicei între 0,4 și 0,9)
  * NDRE: de la -1 la +1 (detectarea stresului)
  * EVI: de la 0 la 1 (vegetație îmbunătățită)

***

## Sfaturi și bune practici

### Comutarea eficientă între straturi

* **Cunoașterea comenzilor rapide de la tastatură**: Deși nu există comenzi rapide de la tastatură pentru straturi, săgețile de navigare (←/→) funcționează pentru toate straturile
* **Fluxuri de lucru consecvente**: Alegeți un strat (de ex., NDVI) și examinați întregul set de date înainte de a comuta la altul
* **Comparații rapide**: Comutați între Original și Reflectanță pentru a verifica calitatea procesării

### Considerații privind performanța

* **JPG se încarcă cel mai rapid**: Utilizați-l pentru navigarea rapidă prin multe imagini
* **Straturile RAW se încarcă mai lent**: Rezoluție și adâncime de biți mai mari
* **Straturi index**: Viteză similară cu straturile de reflectanță
* **Prima încărcare este cea mai lentă**: Vizualizările ulterioare ale aceluiași strat sunt stocate în cache și sunt mai rapide

### Verificarea calității

* **Verificați întotdeauna RAW (Original)**: Verificați calitatea datelor sursă înainte de a avea încredere în rezultatele procesate
* **Comparați straturile**: Utilizați comutarea între straturi pentru a valida dacă procesarea a funcționat corect
* **Verificați intervalele indexului**: Utilizați modul Procentaj pixeli cu straturile index pentru a verifica dacă valorile sunt rezonabile***

## Depanare

### Strat indisponibil

**Problemă**: Stratul așteptat nu apare în meniul derulant**Cauze posibile:**

* Imaginea nu a fost procesată (sunt disponibile doar formatele JPG și RAW (Original))
* Calibrarea reflectanței a fost dezactivată în timpul procesării
* Un indice specific nu a fost configurat în Setările proiectului
* Imaginea este o imagine doar pentru ținte (nu se generează indici pentru ținte)

**Soluții:**

1. Verificați dacă imaginea a fost procesată (verificați folderul de ieșire pentru fișierele procesate)
2. Verificați Setările proiectului pentru a confirma că indicii au fost configurați
3. Reprocesați cu indicii dorite activate

### Se afișează un strat greșit

**Problemă**: Imaginea se deschide într-un strat neașteptat**Cauză**: Preferința de strat din imaginea anterioară a fost preluată, dar acel strat nu există în imaginea curentă**Soluție**: Chloros revine automat la JPG când stratul preferat nu este disponibil – acesta este un comportament normal

### Nu se pot vedea țintele de calibrare

**Problemă**: Stratul RAW (Țintă) nu afișează detectarea țintelor**Cauze posibile:**

* Țintele nu au fost detectate în timpul procesării
* Imaginea nu conține de fapt ținte
* Setările de detectare a țintelor sunt prea stricte

**Soluții:**

1. Verificați jurnalul de depanare pentru mesaje de tip „Țintă găsită”
2. Verificați dacă imaginea conține efectiv ținte de calibrare vizibile
3. Reglați setările de detectare a țintelor în Setările proiectului
4. Consultați [Alegerea imaginilor țintă](../processing-images-gui/choosing-target-images.md)

***

## Funcții conexe

### Instrumente pentru vizualizarea imaginilor

Când vizualizați orice strat, puteți utiliza:

* **Comenzi de zoom**: Măriți pentru a inspecta detaliile
* **Panoramare**: Faceți clic și trageți pentru a vă deplasa în imaginea mărită
* **Inspecția valorii pixelilor**: Vedeți valorile la locația cursorului
* **Săgețile de navigare**: Deplasați-vă între imagini păstrând stratul
* **Modul Procentaj pixeli**: Comutați între afișarea DN și procentaj

Consultați [Deschiderea unei imagini pe ecran complet](opening-an-image-full-screen.md) pentru documentația completă a Vizualizatorului de imagini.

### Index/LUT Sandbox

Pentru testarea și vizualizarea interactivă a indexului:

* **Calculul indexului în timp real**: Testați diferite formule de index
* **Maparea culorilor LUT**: Aplicați gradientele de culoare la indexurile în tonuri de gri
* **Exportați vizualizările**: Salvați imaginile indexate colorate

Consultați [Index/LUT Sandbox](index-lut-sandbox.md) pentru detalii.

***

## Pași următori

Acum că ați înțeles straturile de imagine:

* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) - Ghid complet pentru Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Vizualizare interactivă a indexului
* [**Formule de indici multispectrali**](../project-settings/multispectral-index-formulas.md) - Referință pentru indicii disponibili
* [**Finalizarea procesării**](../processing-images-gui/finishing-the-processing.md) - Înțelegerea rezultatelor procesate
