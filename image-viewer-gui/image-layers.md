# Straturi de imagine

Meniul derulant Straturi de imagine din vizualizatorul de imagini Chloros vă permite să comutați rapid între diferite versiuni ale aceleiași imagini - de la capturile originale la ieșirile de reflectanță procesate și imaginile index calculate.

## Ce sunt straturile de imagine?

În Chloros, **straturile** se referă la diferitele ieșiri de imagine disponibile pentru o singură imagine sursă. Când procesați imagini, Chloros creează mai multe versiuni:

* **Imagini originale** (fișiere JPG și RAW de pe camera dvs.)
* Rezultate **calibrate în funcție de reflectanță** (dacă calibrarea reflectanței a fost activată)
* **Imagini țintă** (dacă imaginea conține ținte de calibrare)
* **Imagini index** (NDVI, NDRE, GNDVI etc., dacă au fost configurate indexuri)

**Meniul derulant Layer Selector** din partea dreaptă sus a Image Viewer vă permite să comutați instantaneu între aceste versiuni fără a părăsi vizualizatorul.

***

## Tipuri de straturi disponibile

### JPG

* Imaginea JPG originală de previzualizare din camera dvs. foto
* Disponibilă întotdeauna pentru toate imaginile
* Neprelucrată, așa cum a fost capturată de cameră
* Cea mai rapidă de încărcat și afișat

**Când să vizualizați:**

* Previzualizare rapidă a capturii originale
* Verificarea compoziției și încadrării imaginii
* Verificarea calității capturii înainte de procesare

### RAW (Original)

* Datele originale RAW ale senzorului din camera dvs.
* Debayered fără aplicarea post-procesării
* Adâncime de biți mai mare decât JPG (de obicei, date ale senzorului de 12 biți sau 14 biți)

**Când să vizualizați:**

* Inspectarea calității datelor originale ale senzorului
* Verificarea problemelor senzorului sau a artefactelor
* Compararea rezultatelor înainte/după procesare

### RAW (țintă)

* Apare numai pentru imaginile identificate ca conținând ținte de calibrare
* Afișează imaginea RAW originală cu ținta detectată
* Utilizat pentru a verifica dacă detectarea țintei a fost reușită

**Când se vizualizează:**

* Confirmarea detectării corecte a țintelor de calibrare
* Verificarea calității imaginii țintei
* Depanarea problemelor de calibrare

{% hint style=&quot;info&quot; %}
**Strat țintă**: Acest strat apare numai în meniul derulant pentru imaginile care conțin ținte de calibrare. Imaginile capturate în mod obișnuit nu vor avea această opțiune.
{% endhint %}

### RAW (Reflectanță)

* Imaginea de ieșire calibrată a reflectanței
* Vignette corectată (dacă este activată în procesare)
* Reflectanță calibrată folosind date țintă (dacă este activată)
* Multi-band TIFF cu toate canalele camerei
* Valorile pixelilor reprezintă procentul de reflectanță (când se folosește modul procentual)
* Gata de manipulare cu [Index/LUT Sandbox](index-lut-sandbox.md)

**Când se vizualizează:**

* Inspectarea rezultatelor calibrate
* Verificarea calității calibrării
* Verificarea valorilor pixelilor pentru acuratețe științifică
* Compararea cu originalul pentru a vedea efectele calibrării

{% hint style=&quot;success&quot; %}
**Recomandat**: Utilizați stratul RAW (Reflectanță) atunci când verificați valorile pixelilor pentru măsurători și analize științifice.
{% endhint %}

### RAW (NDVI Index)... și similare

* Imaginea indicelui de vegetație calculat (NDVI în acest exemplu)
* Numele indicelui se modifică în funcție de indicele configurat în timpul procesării.
* Exemple: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) etc.
* Imagine monocromă cu o singură bandă care arată rezultatele calculului indicelui
* Apare un strat pentru fiecare indice configurat în Setări proiect

**Nume posibile pentru indici:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* Și multe altele... (vezi [Formule index multispectrale](../project-settings/multispectral-index-formulas.md))

**Când să vizualizați:**

* Examinarea rezultatelor calculului indicelui
* Verificarea intervalelor valorilor indicelui
* Identificarea zonelor de interes
* Verificarea imaginilor indicelui înainte de utilizarea în GIS sau analiză

***

## Utilizarea selectorului de straturi

### Deschiderea meniului derulant

1. Deschideți o imagine în modul ecran complet (faceți clic pe orice miniatură din vizualizatorul de imagini)
2. Localizați **meniul derulant al straturilor** în colțul din dreapta sus al vizualizatorului
3. Meniul derulant afișează stratul selectat în prezent (de exemplu, „JPG”)
4. Faceți clic pe meniul derulant pentru a vedea toate straturile disponibile

### Comutarea straturilor

1. Faceți clic pe meniul derulant al straturilor pentru a deschide lista
2. Sunt afișate toate straturile disponibile pentru imaginea curentă
3. Faceți clic pe orice nume de strat pentru a comuta la versiunea respectivă
4. Imaginea se actualizează imediat pentru a afișa stratul selectat

**Comutare rapidă:**

* Meniul derulant reține ultima selecție
* Când navigați la imaginea următoare, Chloros încearcă să afișeze același tip de strat
* Dacă stratul respectiv nu există în imaginea următoare, se setează implicit JPG

### Disponibilitatea straturilor

Nu toate straturile sunt disponibile pentru fiecare imagine:

**Disponibile întotdeauna:**

* ✅ JPG (fiecare imagine are o previzualizare JPG)

**Disponibile condiționat:**

* ⚠️ RAW (Original) - Numai dacă imaginea a fost capturată în modul RAW sau RAW+JPG
* ⚠️ RAW (Țintă) - Numai dacă imaginea conține ținte de calibrare detectate
* ⚠️ RAW (Reflectanță) - Numai după procesarea cu calibrarea reflectanței activată
* ⚠️ RAW (\[Index] Index) - Numai după procesarea cu indici configurați

***

## Persistența stratului

### Navigarea între imagini

Când navigați la o altă imagine (folosind tastele săgeți sau făcând clic pe miniaturi):

**Preferința stratului este păstrată:**

* Dacă vizualizați „RAW (Reflectanță)”, imaginea următoare afișează „RAW (Reflectanță)” (dacă este disponibilă)
* Dacă vizualizați „RAW (NDVI Index)”, imaginea următoare afișează „RAW (NDVI Index)” (dacă este disponibilă)
* Dacă același strat nu există, se setează implicit JPG

**Exemplu de flux de lucru:**

1. Deschideți imaginea 1, comutați la RAW (NDVI Index)
2. Apăsați → pentru a vizualiza imaginea 2
3. Imaginea 2 afișează automat stratul RAW (NDVI Index)
4. Continuați navigarea - toate imaginile afișează stratul NDVI
5. Foarte eficient pentru revizuirea rezultatelor indexului pe mai multe imagini

***

## Fluxuri de lucru comune

### Flux de lucru 1: Comparație înainte/după

**Obiectiv**: Comparați imaginea originală cu cea calibrată

1. Deschideți imaginea procesată în Image Viewer
2. Selectați **RAW (Original)** din meniul derulant
3. Observați vignetarea și valorile necalibrate
4. Comutați la **RAW (Reflectanță)** din meniul derulant
5. Comparați - vignetarea a fost eliminată, valorile au fost calibrate

### Flux de lucru 2: Revizuirea indexului

**Obiectiv**: Revizuirea rapidă a rezultatelor NDVI din setul de date

1. Deschideți prima imagine procesată
2. Selectați **RAW (NDVI Index)** din meniul derulant
3. Utilizați tasta → săgeată pentru a naviga la imaginea următoare
4. Stratul NDVI persistă automat
5. Continuați cu toate imaginile, verificând modelele NDVI
6. Treceți la **RAW (NDRE Index)** pentru a compara

### Flux de lucru 3: Verificarea țintei

**Obiectiv**: Verificați dacă toate imaginile țintă au fost detectate corect

1. Navigați la o imagine țintă
2. Selectați **RAW (Target)** din meniul derulant
3. Verificați dacă țintele de calibrare sunt clar vizibile și detectate
4. Navigați la următoarea imagine țintă
5. Repetați verificarea pentru toate țintele

### Flux de lucru 4: Inspecția valorii pixelilor

**Obiectiv**: Verificați valorile de reflexie pentru acuratețe științifică

1. Deschideți imaginea procesată
2. Selectați stratul **RAW (Reflexie)**
3. Activați modul **Procent pixel** (butonul din bara de instrumente din dreapta sus)
4. Deplasați cursorul peste zonele de vegetație
5. Verificați dacă valorile pixelilor se încadrează în intervalele așteptate (30-70% pentru NIR, 5-15% pentru Red)
6. Verificați dacă zonele de sol și apă au valori adecvate

***

## Înțelegerea valorilor pixelilor pe straturi

Diferite straturi afișează intervale diferite de valori ale pixelilor:

### Strat JPG

* **Interval**: 0-255 (8 biți)
* **Semnificație**: Afișează valori, corectate gamma
* **Utilizare**: Numai inspecție vizuală, nu pentru măsurători științifice

### RAW (Original)

* **Interval**: 0-65535 (16 biți)
* **Semnificație**: Numere digitale brute ale senzorului
* **Utilizare**: Verificarea performanței senzorului, necalibrat

### RAW (Reflectanță)

* **Interval**: 0-65.535 (16 biți TIFF) sau 0,0-1,0 (32 biți procent)
* **Semnificație**: Reflectanță procentuală calibrată
* **Utilizare**: Măsurători și analize științifice

**Pentru 16 biți TIFF:** Împărțiți la 65.535 pentru a obține reflectanța procentuală **Pentru 32 biți Percent:** Valorile reprezintă direct procentul (0,5 = 50% reflectanță)

### RAW (imagini indexate)

* **Interval**: variază în funcție de index (de obicei între -1,0 și +1,0 pentru indicii normalizați)
* **Semnificație**: rezultatul calculului indexului
* **Exemple**:
  * NDVI: între -1 și +1 (vegetație de obicei între 0,4 și 0,9)
  * NDRE: de la -1 la +1 (detectarea stresului)
  * EVI: de la 0 la 1 (vegetație îmbunătățită)

***

## Sfaturi și bune practici

### Comutarea eficientă între straturi

* **Cunoașterea comenzilor rapide de la tastatură**: Deși nu există comenzi rapide de la tastatură pentru straturi, săgețile de navigare (←/→) funcționează pentru toate straturile
* **Fluxuri de lucru consecvente**: Alegeți un strat (de exemplu, NDVI) și examinați întregul set de date înainte de a trece la altul
* **Comparații rapide**: Comutați între Original și Reflectanță pentru a verifica calitatea procesării

### Considerații privind performanța

* **JPG se încarcă cel mai rapid**: utilizați pentru navigarea rapidă prin multe imagini.
* **Straturile RAW se încarcă mai lent**: rezoluție și adâncime de biți mai mari.
* **Straturi index**: viteză similară cu straturile Reflectance.
* **Prima încărcare este cea mai lentă**: vizualizările ulterioare ale aceluiași strat sunt stocate în cache și sunt mai rapide.

### Verificarea calității

* **Verificați întotdeauna RAW (Original)**: verificați calitatea datelor sursă înainte de a avea încredere în rezultatele procesate
* **Comparați straturile**: utilizați comutarea straturilor pentru a valida funcționarea corectă a procesării
* **Verificați intervalele indexului**: utilizați modul Pixel Percent cu straturile index pentru a verifica dacă valorile sunt rezonabile

***

## Depanare

### Stratul nu este disponibil

**Problemă**: Stratul așteptat nu apare în meniul derulant

**Cauze posibile:**

* Imaginea nu a fost procesată (sunt disponibile doar JPG și RAW (Original))
* Calibrarea reflectanței a fost dezactivată în timpul procesării
* Indexul specific nu a fost configurat în Setările proiectului
* Imaginea este o imagine numai pentru țintă (nu s-au generat indici pentru ținte)

**Soluții:**

1. Verificați dacă imaginea a fost procesată (verificați folderul de ieșire pentru fișierele procesate)
2. Verificați Setările proiectului pentru a confirma că indicii au fost configurați
3. Reprocesați cu indicii doriti activați

### Strat incorect afișat

**Problemă**: Imaginea se deschide într-un strat neașteptat

**Cauză**: Preferința de strat din imaginea anterioară a fost preluată, dar acel strat nu există în imaginea curentă

**Soluție**: Chloros revine automat la JPG când stratul preferat nu este disponibil - acesta este un comportament normal

### Nu se pot vedea țintele de calibrare

**Problemă**: Stratul RAW (țintă) nu afișează detectarea țintelor

**Cauze posibile:**

* Țintele nu au fost detectate în timpul procesării
* Imaginea nu conține de fapt ținte
* Setările de detectare a țintelor sunt prea stricte

**Soluții:**

1. Verificați jurnalul de depanare pentru mesajele „Țintă găsită”
2. Verificați dacă imaginea conține efectiv ținte de calibrare vizibile
3. Reglați setările de detectare a țintelor în Setări proiect
4. Consultați [Alegerea imaginilor țintă](../processing-images-gui/choosing-target-images.md)

***

## Funcții conexe

### Instrumente pentru vizualizarea imaginilor

Când vizualizați orice strat, puteți utiliza:

* **Comenzi de zoom**: măriți pentru a inspecta detaliile
* **Panoramare**: faceți clic și trageți pentru a vă deplasa în jurul imaginii mărite
* **Inspectarea valorii pixelilor**: vedeți valorile la locația cursorului
* **Săgeți de navigare**: deplasați-vă între imagini menținând stratul
* **Modul procentaj pixeli**: comutați între afișarea DN și procentaj

Consultați [Deschiderea unei imagini pe ecran complet](opening-an-image-full-screen.md) pentru documentația completă a vizualizatorului de imagini.

### Index/LUT Sandbox

Pentru testarea și vizualizarea interactivă a indexului:

* **Calcularea indexului în timp real**: Testați diferite formule de index
* **Mapare culori LUT**: Aplicați gradientele de culoare la indicii în tonuri de gri
* **Export vizualizări**: Salvați imagini indexate colorate

Consultați [Index/LUT Sandbox](index-lut-sandbox.md) pentru detalii.

***

## Pași următori

Acum că înțelegeți straturile de imagine:

* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) - Ghid complet pentru vizualizatorul de imagini
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Vizualizare interactivă a indexului
* [**Formule index multispectrale**](../project-settings/multispectral-index-formulas.md) - Referință indexuri disponibile
* [**Finalizarea procesării**](../processing-images-gui/finishing-the-processing.md) - Înțelegerea rezultatelor procesate
